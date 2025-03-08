---
title:  golang gorm custom types
categories:
  - Golang
tags:
  - Gorm
  - Problem solving
---
Content

{% include toc %}

# Preface

Following the previous article [Solution to the problem that golang-gorm library does not support [] string type](https://blog.csdn.net/weixin_43811340/article/details/119455488?spm=1001.2014.3001.5501), the problem of correctly identifying complex structure types in a gorm library was solved. Here, two common custom types of data are simply implemented according to the official documentation.

Here is the address of the [gorm official documentation](https://gorm.io/zh_CN/docs/data_types.html). If you encounter related problems, the best solution is always to check the standard solution in the official documentation.

# []string type

Define the structure `RateLimitPolicy` that needs to be passed to the database, two of which have data types, which are defined below:

```go
type RateLimitPolicy struct {
AllowedIpRange AllowedIpRange `json:"allowedIpRange,omitempty"`
AllowedIps AllowedIps `json:"allowedIps,omitempty"`
}
```

Define `AllowedIps` as `[]string`:

```go
type AllowedIps []string
```

This type cannot be directly recognized in gorm, and a custom data type is required.

```go
func (a *AllowedIps) Scan(value interface{}) error {
 bytesValue, ok := value.([]byte)
 if !ok {
 return errors.New(fmt.Sprint("Failed to unmarshal GlobalSslVisitedHosts value:", value))
 }
 return json.Unmarshal(bytesValue, a)
}

// Value return json value, implement driver.Valuer interface
func (a AllowedIps) Value() (driver.Value, error) {
 return json.Marshal(a)
}
```

Customization is completed through the two functions `Scan()` and `Value()`.



# struct{} type

Define the structure `RateLimitPolicy` that needs to be passed to the database. Two data types are defined below:

```go
type RateLimitPolicy struct {
AllowedIpRange AllowedIpRange `json:"allowedIpRange,omitempty"`
AllowedIps AllowedIps `json:"allowedIps,omitempty"`
}
```

Define `AllowedIpRange` as `struct{}`:

```go
type AllowedIpRange struct {
Begin gwtype.String `json:"begin,omitempty"`
End gwtype.String `json:"end,omitempty"`
}
```

This type cannot be directly recognized in gorm and requires a custom data type.

```go
func (ar *AllowedIpRange) Scan(value interface{}) error {
 bytesValue, ok := value.([]byte)
 if !ok {
 return errors.New(fmt.Sprint("Failed to unmarshal GlobalSslVisitedHosts value:", value))
 }
 return json.Unmarshal(bytesValue, ar)
}

// Value return json value, implement driver.Valuer interface
func (ar AllowedIpRange) Value() (driver.Value, error) {
 return json.Marshal(ar)
}

```

Customization is completed through the two functions `Scan()` and `Value()`.



# Appendix: Custom data types in official documents

GORM provides a small number of interfaces that allow users to define supported data types for GORM. Here we take [json](https://github.com/go-gorm/datatypes/blob/master/json.go) as an example

**Implement custom data types**

`Scanner / Valuer`

Custom data types must implement the [Scanner](https://pkg.go.dev/database/sql#Scanner) and [Valuer](https://pkg.go.dev/database/sql/driver#Valuer) interfaces so that GORM knows how to receive and save the type to the database

For example:

```go
type JSON json.RawMessage

// Implement the sql.Scanner interface, Scan scans value to Jsonb
func (j *JSON) Scan(value interface{}) error {
bytes, ok := value.([]byte)
if !ok {
return errors.New(fmt.Sprint("Failed to unmarshal JSONB value:", value))
 }

 result := json.RawMessage{}
 err := json.Unmarshal(bytes, &result)
 *j = JSON(result)
 return err
}

// Implement driver.Valuer interface, Value returns json value
func (j JSON) Value() (driver.Value, error) {
 if len(j) == 0 {
 return nil, nil
 }
 return json.RawMessage(j).MarshalJSON()
}
```

# `gorm` custom json format

## Encountering problems

The structure of the database defined using the `gorm` package is as follows:

```go
type SystemParameter struct {
GlobalSslEnable bool `json:"globalSslEnable,omitempty"`
GlobalSslVisitedHosts []string `json:"globalSslVisitedHosts,omitempty"`
UserInfoInterfaceURL string `json:"userInfoInterfaceURL,omitempty"`
}
```

Error when creating mysql data for the database: <font color = red>(sql: converting argument $1 type: unsupported type []string, a slice of string)</font>

## Analyze the cause

MySQL does not support `[]string` type data, and the `[]string` type cannot be defined directly in the structure.

The data structure types supported in MySQL are as follows:

* Numeric types: TINYINT, SMALLINT, MEDIUMINT, INT or INTEGER, BIGINT, FLOAT, DOUBLE, DECIMAL;

* Date and time types: DATE, TIME, YEAR, DATETIME, TIMESTAMP;

* String types: CHAR, VARCHAR, TINYBLOB, TINYTEXT, BLOB, TEXT, MEDIUMBLOB, MEDIUMTEXT, LONGBLOB, LONGTEXT;

Therefore, MySQL does not support `[]string` type data, and cannot be directly defined in the structure. In order to store `[]string` type data in the database, it is necessary to encapsulate this type of data in the structure and convert it into a data type supported by MySQL.

## Solution

Based on the characteristics of the `gorm` library, data can be passed in through a custom `JSON` type, so the solution is divided into two steps:

* Step 1: Encode the `[]string` type to `JSON` type data;

* Step 2: Customize the `JSON` type data so that `gorm` can support it. You can refer to [GORM-Custom Data Type](https://gorm.io/zh_CN/docs/data_types.html);

### step 1

First, change the data type definition in the structure to a new custom type, `NullGlobalSslVisitedHosts` is used to determine whether `GlobalSslVisitedHosts` is empty.

```go
//Modify types.go
type SystemParameter struct {
 GlobalSslEnable bool `json:"globalSslEnable,omitempty"`
 GlobalSslVisitedHosts NullGlobalSslVisitedHosts `json:"globalSslVisitedHosts,omitempty"`
 UserInfoInterfaceURL string `json:"userInfoInterfaceURL,omitempty"`
}

type NullGlobalSslVisitedHosts struct {
 GlobalSslVisitedHosts GlobalSslVisitedHosts
 Valid bool
}

type GlobalSslVisitedHosts []string

// GormDataType return the gorm data type
func (GlobalSslVisitedHosts) GormDataType() string {
 return "json"
}
```

In the dependency library of `gorm` (address: `gorm.io/gorm/schema/field.go`), modify and add the function just defined above

```go
// Modify gorm.io/gorm/schema/field.go

if dataTyper, ok := fieldValue.Interface().(GormDataTypeInterface); ok {
field.DataType = DataType(dataTyper.GormDataType())
}
```

In the dependency library of `gorm` (address: `gorm.io/gorm/schema/interfaces.go`), add the defined interface

```go
// Modify gorm.io/gorm/schema/interfaces.go

type GormDataTypeInterface interface {
GormDataType() string
}
```

### step 2

Customize a `JSON` type, please refer to [GORM-Custom Data Type](https://gorm.io/zh_CN/docs/data_types.html)

```go
// Modify types.go

// Scan implements the Scanner interface.
func (n *NullGlobalSslVisitedHosts) Scan(value interface{}) error {
if value == nil {

n.GlobalSslVisitedHosts, n.Valid = GlobalSslVisitedHosts{}, false

return nil
}
n.Valid = true

var t GlobalSslVisitedHosts
err := t.Scan(value)
if err != nil {
return err
}
n.GlobalSslVisitedHosts, n.Valid = t, true

return nil
}

// Value implements the driver Valuer interface.
func (n NullGlobalSslVisitedHosts) Value() (driver.Value, error) {
 if !n.Valid {
 return nil, nil
 }

 return n.GlobalSslVisitedHosts.Value()

}

func (i *GlobalSslVisitedHosts) Scan(value interface{}) error {
 bytesValue, ok := value.([]byte)
 if !ok {
 return errors.New(fmt.Sprint("Failed to unmarshal GlobalSslVisitedHosts value:", value))
 }
 return json.Unmarshal(bytesValue, i)
}

// Value return json value, implement driver.Valuer interface
func (i GlobalSslVisitedHosts) Value() (driver.Value, error) {
 return json.Marshal(i)
}

func (v NullGlobalSslVisitedHosts) MarshalJSON() ([]byte, error) {
 if v.Valid {
 return json.Marshal(v.GlobalSslVisitedHosts)
 } else {
 return json.Marshal(nil)
 }
}

func (v *NullGlobalSslVisitedHosts) UnmarshalJSON(data []byte) error {
 // Unmarshalling into a pointer will let us detect null
 var x *GlobalSslVisitedHosts
 if err := json.Unmarshal(data, &x); err != nil {
 return err
 }
 if x != nil {
 v.Valid = true
 v.GlobalSslVisitedHosts = *x
 } else {
 v.Valid = false
 }
 return nil
}
```