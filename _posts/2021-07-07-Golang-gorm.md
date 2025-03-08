---
title:  Golang Gorm - learning notes
categories:
  - Golang
tags:
  - Gorm
  - Learning notes
---
Content

{% include toc %}

## Introduction

ORM: Object Relational Mapping

The fantastic ORM library for Golang aims to be developer friendly.

* Advantages of ORM

Improve development efficiency

* Disadvantages of ORM

Sacrifice execution performance;

Sacrifice flexibility;

Weaken the ability to write SQL by yourself;

<font color = seagreen>Very comprehensive Chinese document address: https://gorm.io/zh_CN/</font>

### Install dependency packages

```shell

# You can install this dependency package
go get -u github.com/jinzhu/gorm

# Official document installation below
go get -u gorm.io/gorm
go get -u gorm.io/driver/sqlite
```

### Create a database

```mysql
CREATE DATABASE db1;
```

### Connect to the database

Different databases need to import different dependency packages

```go
import _ "github.com/jinzhu/gorm/dialects/mysql"
// import _ "github.com/jinzhu/gorm/dialects/postgres"
// import _ "github.com/jinzhu/gorm/dialects/sqlite"
// import _ "github.com/jinzhu/gorm/dialects/mssql"
```

> Connect to MySQL

```go
import (
"github.com/jinzhu/gorm"
_ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
// TCP connection to remote MySQL
dsn := "<user>:<password>@tcp(<ip>:<port, usually 3306>)/<dbname>?charset=utf8&parseTime=true&loc=Local"
db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
 //db, err := gorm.Open("mysql", "<user>:<password>@(<localhost>)/d<dbname>?charset=utf8mb4&parseTime=True&loc=Local")
 defer db.Close()
}
```

> Connect to Sqlite3

```go
import (
 "github.com/jinzhu/gorm"
 _ "github.com/jinzhu/gorm/dialects/sqlite"
)

func main() {
 db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
 defer db.Close()
}
```

> Connect to SQL Server

```go
import (
 "github.com/jinzhu/gorm"
 _ "github.com/jinzhu/gorm/dialects/mssql"
)

func main() {
 db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
 defer db.Close()
}
```

### Quick Start Example

```go
package main

import (
 "fmt"
 "github.com/jinzhu/gorm"
 _ "github.com/jinzhu/gorm/dialects/mysql"
)

// UserInfo user information
type UserInfo struct {
 ID uint
 Name string
 Gender string
 Hobby string
}


func main() {
 db, err := gorm.Open("mysql", "root:root1234@(127.0.0.1:13306)/db1?charset=utf8mb4&parseTime=True&loc=Local")
 if err!= nil{
 panic(err) }
defer db.Close()

// Automatic migration (automatically create tables)
db.AutoMigrate(&UserInfo{})

u1 := UserInfo{1, "timi", "female", "basketball"}
u2 := UserInfo{2, "jay", "male", "football"}
// Create records
db.Create(&u1)
db.Create(&u2)
// Query
var u = new(UserInfo)
db.First(u)
fmt.Printf("%#v\n", u)

var uu UserInfo
db.Find(&uu, "hobby=?", "football")
fmt.Printf("%#v\n", uu)

// Update
db.Model(&u).Update("hobby", "Double Color Ball")
// Delete
db.Delete(&u)
}
```

## Model definition

> Example

```go
type User struct {
gorm.Model
Name string
Age sql.NullInt64
Birthday *time.Time
Email string `gorm:"type:varchar(100);unique_index"`
Role string `gorm:"size:255"` // Set the field size to 255
MemberNumber *string `gorm:"unique;not null"` // Set the member number to be unique and not null
Num int `gorm:"AUTO_INCREMENT"` // Set num to auto-increment type
Address string `gorm:"index:addr"` // Create an index named addr for the address field
IgnoreMe int `gorm:"-"` // Ignore this field
}
```

### Create/Update Time

GORM uses `CreatedAt` and `UpdatedAt` to track the creation/update time. If you define such a field, GORM will automatically fill it with the [current time](https://gorm.io/zh_CN/docs/gorm_config.html#now_func) when creating or updating.

To use fields with different names, you can configure the `autoCreateTime`, `autoUpdateTime` tags.

If you want to store UNIX (milli/nano) seconds instead of time, you can simply change `time.Time` to `int`.

```go
type User struct {
CreatedAt time.Time // When creating, if the field value is zero, fill it with the current time
UpdatedAt int // When creating, if the field value is zero or when updating, fill it with the current timestamp seconds
Updated int64 `gorm:"autoUpdateTime:nano"` // Fill the update time with nanoseconds of the timestamp
Updated int64 `gorm:"autoUpdateTime:milli"` // Fill the update time with milliseconds of the timestamp
Created int64 `gorm:"autoCreateTime"` // Fill creation time with timestamp seconds
}
```

* If the model has a `DeletedAt` field, calling `Delete` to delete the record will set the `DeletedAt` field to the current time instead of directly deleting the record from the database.

### Field tags

When declaring a model, tags are optional. GORM supports the following tags: Tag names are case insensitive, but it is recommended to use the `camelCase` style

| Tag name | Description |
| :--------------------- | :----------------------------------------------------------- |
| column | Specify the db column name |
| type | Column data type. It is recommended to use a common type with good compatibility, for example: all databases support bool, int, uint, float, string, time, bytes and can be used with other tags, such as: `not null`, `size`, `autoIncrement`... Specifying database data types like `varbinary(8)` is also supported. When using a specified database data type, it needs to be a complete database data type, such as: `MEDIUMINT UNSIGNED not NULL AUTO_INCREMENT` |
| size | Specify the column size, for example: `size:256` |
| primaryKey | Specify the column as the primary key |
| unique | Specify the column as unique |
| default | Specify the default value of the column |
| precision | Specify the precision of the column |
| scale | Specify the size of the column |
| not null | Specify the column as NOT NULL |
| autoIncrement | Specify the column as auto-increment |
| autoIncrementIncrement | Automatic step, control the interval between consecutive records |
| embedded | Nested fields |
| embeddedPrefix | Column name prefix for embedded fields |
| autoCreateTime | Tracks the current time when creating. For `int` fields, it tracks the timestamp in seconds. You can use `nano`/`milli` to track nanoseconds and milliseconds, for example: `autoCreateTime:nano` |
| autoUpdateTime | Tracks the current time when creating/updating. For `int` fields, it tracks the timestamp in seconds. You can use `nano`/`milli` to track nanoseconds and milliseconds, for example: `autoUpdateTime:milli` |
| index | Creates an index based on parameters. Multiple fields with the same name will create a composite index. See [Index](https://gorm.io/zh_CN/docs/indexes.html) for details |
| uniqueIndex | Same as `index`, but creates a unique index |
| check | Creates a check constraint, for example, `check:age > 13`. See [Constraints](https://gorm.io/zh_CN/docs/constraints.html) for details |
| <- | Set the permission to write to the field, `<-:create` only creates, `<-:update` only updates, `<-:false` no write permission, `<-` creates and updates permissions |
| -> | Set the permission to read the field, `->:false` no read permission |
| - | Ignore the field, `-` no read or write permission |
| comment | Add comments to the field during migration |

### Default rules for primary key, table name, and column name

> ID is used as the primary key by default

GORM uses the field named ID as the primary key of the table by default.

```go
type User struct {
ID string // Field named `ID` will be used as the primary key of the table by default
Name string
}
// Use `AnimalID` as the primary key
type Animal struct {
AnimalID int64 `gorm:"primary_key"`
Name string
Age int64
}
```

> The table name is the plural form of the structure by default

The table name is the plural form of the structure name by default, for example:

```go
type User struct {} // The default table name is `users`

// Set the User table name to `profiles`
func (User) TableName() string {
return "profiles"
}

func (u User) TableName() string {
if u.Role == "admin" {
return "admin_users"
} else {
return "users"
}
}

// Disable the default table name plural form. If set to true, `User` The default table name is `user`
db.SingularTable(true)
```

You can also specify the table name through `Table()`:

```go
// Create a table named `deleted_users` using the User structure
db.Table("deleted_users").CreateTable(&User{})

var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
//// DELETE FROM deleted_users WHERE name = 'jinzhu';
```

GORM also supports changing the default table name rule:

```go
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string {
return "prefix_" + defaultTableName;
}
```

> Column Name

The column name is generated by separating the field name by underscores

```go
type User struct {
ID uint // column name is `id`
Name string // column name is `name`
Birthday time.Time // column name is `birthday`
CreatedAt time.Time // column name is `created_at`
}
```

You can use the structure tag to specify the column name:

```go
type Animal struct {
AnimalId int64 `gorm:"column:beast_id"` // set column name to `beast_id`
Birthday time.Time `gorm:"column:day_of_the_beast"` // set column name to `day_of_the_beast`
Age int64 `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast`
}
```

## Add, delete, modify and query

### Create a record

```go
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}

result := db.Create(&user) // Create through data pointer

user.ID // Returns the primary key of the inserted data
result.Error // Returns error
result.RowsAffected // Returns the number of inserted records
```

> Use pointer

```go
// Use pointer zero value to store
type User struct {
ID int64
Name *string `gorm:"default:'Little Prince'"`
Age int64
}
user := User{Name: new(string), Age: 18))}
db.Create(&user) // At this time, the value of the name field of the record in the database is ''
```

`PostgreSQL` database can use the following method to implement merge insertion, Update if there is, insert if there is none.

```go
// Add extended SQL options for INSERT statement
db.Set("gorm:insert_option", "ON CONFLICT").Create(&product)
// INSERT INTO products (name, code) VALUES ("name", "code") ON CONFLICT;
```

### Query

GORM provides `First`, `Take`, `Last` methods to retrieve a single object from the database. It adds `LIMIT 1` condition when querying the database, and when no record is found, it returns `ErrRecordNotFound` error

```go
// Get the first record (primary key ascending)
db.First(&user)
// SELECT * FROM users ORDER BY id LIMIT 1;

// Get a record, no sort field specified
db.Take(&user)
// SELECT * FROM users LIMIT 1;

// Get the last record (primary key descending)
db.Last(&user)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

result := db.First(&user)
result.RowsAffected // Returns the number of records found
result.Error // returns error

// Check ErrRecordNotFound error
errors.Is(result.Error, gorm.ErrRecordNotFound)
```

> Where condition

- Ordinary SQL queries

```go
// Get first matched record
db.Where("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name = 'jinzhu' limit 1;

// Get all matched records
db.Where("name = ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu';

// <>
db.Where("name <> ?", "jinzhu").Find(&users)
//// SELECT * FROM users WHERE name <> 'jinzhu';

// IN
db.Where("name IN (?)", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name in ('jinzhu','jinzhu 2');

// LIKE
db.Where("name LIKE ?", "%jin%").Find(&users)
//// SELECT * FROM users WHERE name LIKE '%jin%';

// AND
db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' AND age >= 22;

// Time
db.Where("updated_at > ?", lastWeek).Find(&users)
//// SELECT * FROM users WHERE updated_at > '2000-01-01 00:00:00';

//BETWEEN
db.Where("created_at BETWEEN ? AND ?", lastWeek, today).Find(&users)
//// SELECT * FROM users WHERE created_at BETWEEN '2000-01-01 00:00:00' AND '2000-01-08 00:00:00';
```

* Struct & Map query

```go
// Struct
db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;

// Slice of primary key
db.Where([]int64{20, 21, 22}).Find(&users)
//// SELECT * FROM users WHERE id IN (20, 21, 22);
```

**Tip:** When querying by struct, GORM will only query by non-zero value fields, which means that if your field value is `0`, `''`, `false` or other `zero value`, it will not be used to build query conditions, for example:

```go
db.Where(&User{Name: "jinzhu", Age: 0}).Find(&users)
//// SELECT * FROM users WHERE name = "jinzhu";
```

You can use pointers or implement Scanner/Valuer interfaces to avoid this problem.

```go
// Use pointers
type User struct {
gorm.Model
Name string
Age *int
}

// Use Scanner/Valuer
type User struct {
gorm.Model
Name string
Age sql.NullInt64 // sql.NullInt64 implements Scanner/Valuer interface
}
```

> Not condition

The following is a similar situation to Where:

```go
db.Not("name", "jinzhu").First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu" LIMIT 1;

// Not In
db.Not("name", []string{"jinzhu", "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name NOT IN ("jinzhu", "jinzhu 2");

// Not In slice of primary keys
db.Not([]int64{1,2,3}).First(&user)
//// SELECT * FROM users WHERE id NOT IN (1,2,3);

db.Not([]int64{}).First(&user)
//// SELECT * FROM users;

// Plain SQL
db.Not("name = ?", "jinzhu").First(&user)
//// SELECT * FROM users WHERE NOT(name = "jinzhu");

// Struct
db.Not(User{Name: "jinzhu"}).First(&user)
//// SELECT * FROM users WHERE name <> "jinzhu";
```

> Or condition

```go
db.Where("role = ?", "admin").Or("role = ?", "super_admin").Find(&users)
//// SELECT * FROM users WHERE role = 'admin' OR role = 'super_admin';

// Struct
db.Where("name = 'jinzhu'").Or(User{Name: "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';

//Map
db.Where("name = 'jinzhu'").Or(map[string]interface{}{"name": "jinzhu 2"}).Find(&users)
//// SELECT * FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2';
```

> Inline conditions

Similar to the `Where` query, when inline conditions are used with multiple [immediate execution methods](https://www.liwenzhou.com/posts/Go/gorm_crud/#autoid-1-3-1), the inline conditions will not be passed to the subsequent immediate execution methods.

```go
// Get records by primary key (only for integer primary key)
db.First(&user, 23)
//// SELECT * FROM users WHERE id = 23 LIMIT 1;
// Get records by primary key, if it is a non-integer primary key
db.First(&user, "id = ?", "string_primary_key")
//// SELECT * FROM users WHERE id = 'string_primary_key' LIMIT 1;

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
//// SELECT * FROM users WHERE name = "jinzhu";

db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
//// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
//// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
//// SELECT * FROM users WHERE age = 20;
```

> Additional query options

```go
// Add additional SQL operations to query SQL
db.Set("gorm:query_option", "FOR UPDATE").First(&user, 10)
//// SELECT * FROM users WHERE id = 10 FOR UPDATE;
```

> FirstOrInit

Get the first matching record, otherwise initialize a new object according to the given conditions (only supports struct and map conditions)

```go
// Not found
db.FirstOrInit(&user, User{Name: "non_existing"})
//// user -> User{Name: "non_existing"}

// Found
db.Where(User{Name: "Jinzhu"}).FirstOrInit(&user)
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
db.FirstOrInit(&user, map[string]interface{}{"name": "jinzhu"})
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
```

* Attrs

If the record is not found, the struct will be initialized with the parameters.

```go
// Not found
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

db.Where(User{Name: "non_existing"}).Attrs("age", 20).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = 'non_existing';
//// user -> User{Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "Jinzhu"}).Attrs(User{Age: 30}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 20}
```

* Assign

Regardless of whether the record is found, assign the parameter to struct.

```go
// Not found
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrInit(&user)
//// user -> User{Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "Jinzhu"}).Assign(User{Age: 30}).FirstOrInit(&user)
//// SELECT * FROM USERS WHERE name = jinzhu';
//// user -> User{Id: 111, Name: "Jinzhu", Age: 30}
```

> FirstOrCreate

Get the first matching record, otherwise create a new record according to the given conditions (only supports struct and map conditions)

```go
// Not found
db.FirstOrCreate(&user, User{Name: "non_existing"})
//// INSERT INTO "users" (name) VALUES ("non_existing");
//// user -> User{Id: 112, Name: "non_existing"}

// Found
db.Where(User{Name: "Jinzhu"}).FirstOrCreate(&user)
//// user -> User{Id: 111, Name: "Jinzhu"}
```

* Attrs

If the record is not found, the struct and record will be created with the parameters.

```go
// Not found
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// turn up
db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 30}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// user -> User{Id: 111, Name: "jinzhu", Age: 20}
```

*Assign

Regardless of whether the record is found or not, assign the parameter to the struct and save it to the database.

```go
// Not found
db.Where(User{Name: "non_existing"}).Assign(User{Age: 20}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'non_existing';
//// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
//// user -> User{Id: 112, Name: "non_existing", Age: 20}

// Found
db.Where(User{Name: "jinzhu"}).Assign(User{Age: 30}).FirstOrCreate(&user)
//// SELECT * FROM users WHERE name = 'jinzhu';
//// UPDATE users SET age=30 WHERE id = 111;
//// user -> User{Id: 111, Name: "jinzhu", Age: 30}
```

> Advanced query

* Subquery

Subquery based on `*gorm.expr`

```go
db.Where("amount > ?", db.Table("orders").Select("AVG(amount)").Where("state = ?", "paid").SubQuery()).Find(&orders)
// SELECT * FROM "orders" WHERE "orders"."deleted_at" IS NULL AND (amount > (SELECT AVG(amount) FROM "orders" WHERE (state = 'paid')));
```

* Select fields

Select, specify the fields you want to retrieve from the database, all fields will be selected by default.

```go
db.Select("name, age").Find(&users)
//// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
//// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
//// SELECT COALESCE(age,'42') FROM users;
```

* Sorting

Order, specifies the order in which records are retrieved from the database. Setting the second parameter reorder to `true` can override the previously defined sorting conditions.

```go
db.Order("age desc, name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// Multi-field sorting
db.Order("age desc").Order("name").Find(&users)
//// SELECT * FROM users ORDER BY age desc, name;

// Override sorting
db.Order("age desc").Find(&users1).Order("age", true).Find(&users2)
//// SELECT * FROM users ORDER BY age desc; (users1)
//// SELECT * FROM users ORDER BY age; (users2)
```

* Quantity

Limit, specifies the maximum number of records retrieved from the database.

```go
db.Limit(3).Find(&users)
//// SELECT * FROM users LIMIT 3;

// -1 cancels the Limit condition
db.Limit(10).Find(&users1).Limit(-1).Find(&users2)
//// SELECT * FROM users LIMIT 10; (users1)
//// SELECT * FROM users; (users2)
```

* Offset

Offset specifies the number of records to skip before starting to return records.

```go
db.Offset(3).Find(&users)
//// SELECT * FROM users OFFSET 3;

// -1 cancels the Offset condition
db.Offset(10).Find(&users1).Offset(-1).Find(&users2)
//// SELECT * FROM users OFFSET 10; (users1)
//// SELECT * FROM users; (users2)
```

* Count

Count, the total number of records that the model can obtain.

```go
db.Where("name = ?", "jinzhu").Or("name = ?", "jinzhu 2").Find(&users).Count(&count)
//// SELECT * from USERS WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (users)
//// SELECT count(*) FROM users WHERE name = 'jinzhu' OR name = 'jinzhu 2'; (count)

db.Model(&User{}).Where("name = ?", "jinzhu").Count(&count)
//// SELECT count(*) FROM users WHERE name = 'jinzhu'; (count)

db.Table("deleted_users").Count(&count)
//// SELECT count(*) FROM deleted_users;

db.Table("deleted_users").Select("count(distinct(name))").Count(&count)
//// SELECT count( distinct(name) ) FROM deleted_users; (count)
```

**Note** `Count` must be the last operation of the chain query, because it will overwrite the previous `SELECT`, but it will not overwrite if `count` is used inside

* Group & Having

```go
rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
for rows.Next() {
...
}

// Use Scan to scan multiple results into a pre-prepared structure slice
type Result struct {
Date time.Time
Total int
}
var rets []Result
db.Table("users").Select("date(created_at) as date, sum(age) as total").Group("date(created_at)").Scan(&rets)

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
for rows.Next() {
 ...
}

type Result struct {
 Date time.Time
 Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

* connect

Joins, specify join conditions

```go
rows, err := db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Rows()
for rows.Next() {
 ...
}

db.Table("users").Select("users.name, emails.email").Joins("left join emails on emails.user_id = users.id").Scan(&results)

//Multiple connections and parameters
db.Joins("JOIN emails ON emails.user_id = users.id AND emails.email = ?", "jinzhu@example.org").Joins("JOIN credit_cards ON credit_cards.user_id = users.id").Where("credit_cards.number = ?", "4111111111111").Find(&user)
```

> Pluck

Pluck, query a column in the model as a slice, if you want to query multiple columns, you should use [`Scan`](https://www.liwenzhou.com/posts/Go/gorm_crud/#Scan)

```go
var ages []int64
db.Find(&users).Pluck("age", &ages)

var names []string
db.Model(&User{}).Pluck("name", &names)

db.Table("deleted_users").Pluck("name", &names)

// Want to query multiple fields? Do this:
db.Select("name, age").Find(&users)
```

> Scan

Scan, scan the results into a struct.

```go
type Result struct {
Name string
Age int
}

var result Result
db.Table("users").Select("name, age").Where("name = ?", "Antonio").Scan(&result)

var results []Result
db.Table("users").Select("name, age").Where("id > ?", 0).Scan(&results)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```

### Update

> Update all fields

`Save` All fields will be saved, even if the field is zero value

```go
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
// UPDATE users SET name='jinzhu 2', age=100, birthday = '2016-01-01', updated_at = '2013-11-17 21:34:10' WHERE id = 111;
```

> Update a single column

When using `Update` to update a single column, you need to specify a condition, otherwise it will return an `ErrMissingWhereClause` error. See [Block Global Updates](https://gorm.io/zh_CN/docs/update.html#block_global_updates) for details. When the `Model` method is used and the object's primary key has a value, the value will be used to build the condition, for example:

```go
// Conditional update
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User's ID is `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// Update based on the condition and model value
db.Model(&user).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;
```

> Update multiple columns

The `Updates` method supports `struct` and `map[string]interface{}` parameters. When updating with `struct`, by default, GORM will only update fields with non-zero values

```go
// Update properties based on `struct`, only update fields with non-zero values
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// Update properties based on `map`
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

**NOTE** When updating via struct, GORM will only update non-zero fields. If you want to ensure that specific fields are updated, you should use `Select` to update the selected fields, or use `map` to complete the update operation

> Update modified fields

If you only want to update specific fields, you can use `Update` or `Updates`

```go
// Update a single attribute if it has changed
db.Model(&user).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// Update a single attribute based on a given condition
db.Model(&user).Where("active = ?", true).Update("name", "hello")
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111 AND active=true;

// Use map Update multiple attributes, only update the attributes that have changed
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// Use struct to update multiple attributes, only update the fields that have changed and are non-zero values
db.Model(&user).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// Warning: When using struct to update, GORM will only update those fields with non-zero values
// For the following operation, no update will occur, "", 0, false are all zero values ​​of their types
db.Model(&user).Updates(User{Name: "", Age: 0, Active: false})
```

> Update selected fields

If you want to update or ignore certain fields, you can use `Select`, `Omit`

```go
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
//// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```

> Update without Hooks

The above update operation will automatically run the model's `BeforeUpdate`, `AfterUpdate` methods, update the `UpdatedAt` timestamp, and save its `Associations` when updating. If you don't want to call these methods, you can use `UpdateColumn`, `UpdateColumns`

```go
// Update a single attribute, similar to `Update`
db.Model(&user).UpdateColumn("name", "hello")
//// UPDATE users SET name='hello' WHERE id = 111;

// Update multiple attributes, similar to `Updates`
db.Model(&user).UpdateColumns(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18 WHERE id = 111;
```

> Batch update

Hooks will not run during batch update.

```go
db.Table("users").Where("id IN (?)", []int{10, 11}).Updates(map[string]interface{}{"name": "hello", "age": 18})
//// UPDATE users SET name='hello', age=18 WHERE id IN (10, 11);

// When using struct to update, only non-zero value fields will be updated. If you want to update all fields, please use map[string]interface{}
db.Model(User{}).Updates(User{Name: "hello", Age: 18})
//// UPDATE users SET name='hello', age=18;

// Use `RowsAffected` to get the total number of updated records
db.Model(User{}).Updates(User{Name: "hello", Age: 18}).RowsAffected
```

> Use SQL expression to update

First query the first data in the table and save it to the user variable.

```go
var userUser
db.First(&user)
db.Model(&user).Update("age", gorm.Expr("age * ? + ?", 2, 100))
//// UPDATE `users` SET `age` = age * 2 + 100, `updated_at` = '2020-02-16 13:10:20' WHERE `users`.`id` = 1;

db.Model(&user).Updates(map[string]interface{}{"age": gorm.Expr("age * ? + ?", 2, 100)})
//// UPDATE "users" SET "age" = age * '2' + '100', "updated_at" = '2020-02-16 13:05:51' WHERE `users`.`id` = 1;

db.Model(&user).UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1';

db.Model(&user).Where("age > 10").UpdateColumn("age", gorm.Expr("age - ?", 1))
//// UPDATE "users" SET "age" = age - 1 WHERE "id" = '1' AND quantity > 10;
```

> Modify values ​​in Hooks

If you want to modify the updated values ​​in Hooks such as `BeforeUpdate`, `BeforeSave`, you can use `scope.SetColumn`, for example:

```go
func (user *User) BeforeSave(scope *gorm.Scope) (err error) {
if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
scope.SetColumn("EncryptedPassword", pw)
}
}
```

> Other update options

```go
// Add other SQL to update SQL
db.Model(&user).Set("gorm:update_option", "OPTION (OPTIMIZE FOR UNKNOWN)").Update("name", "hello")
//// UPDATE users SET name='hello', updated_at = '2013-11-17 21:34:10' WHERE id=111 OPTION (OPTIMIZE FOR UNKNOWN);
```

### Delete

>Delete a record

When deleting a record, you need to specify the primary key of the object to be deleted, otherwise [batch Delete](https://gorm.io/zh_CN/docs/delete.html#batch_delete) will be triggered, for example:

```go
// Email's ID is `10`
db.Delete(&email)
// DELETE from emails where id = 10;

// Delete with additional conditions
db.Where("name = ?", "jinzhu").Delete(&email)
// DELETE from emails where id = 10 AND name = "jinzhu";
```

> Delete by primary key

GORM allows to delete objects using primary key(s) with inline condition, it works with numbers, check out [Query Inline Conditions](https://gorm.io/zh_CN/docs/query.html#inline_conditions) for details

```go
db.Delete(&User{}, 10)
// DELETE FROM users WHERE id = 10;

db.Delete(&User{}, "10")
// DELETE FROM users WHERE id = 10;

db.Delete(&users, []int{1,2,3})
// DELETE FROM users WHERE id IN (1,2,3);
```

> Delete Hook

For delete operations, GORM supports `BeforeDelete` and `AfterDelete` Hooks, which are called when deleting records. See [Hook](https://gorm.io/zh_CN/docs/hooks.html) for details

```go
func (u *User) BeforeDelete(tx *gorm.DB) (err error) {
if u.Role == "admin" {
return errors.New("admin user not allowed to delete")
}
return
}
```

> Bulk delete

If the specified value does not include the primary attribute, GORM will perform a bulk delete, which will delete all matching records

```go
db.Where("email LIKE ?", "%jinzhu%").Delete(Email{})
// DELETE from emails where email LIKE "%jinzhu%";

db.Delete(Email{}, "email LIKE ?", "%jinzhu%")
// DELETE from emails where email LIKE "%jinzhu%";
```

> Prevent global delete

If you perform a bulk delete without any conditions, GORM will not perform the operation and return an `ErrMissingWhereClause` error

For this, you must add some conditions, or use raw SQL, or enable the `AllowGlobalUpdate` mode, for example:

```go
db.Delete(&User{}).Error // gorm.ErrMissingWhereClause

db.Where("1 = 1").Delete(&User{})
// DELETE FROM `users` WHERE 1=1

db.Exec("DELETE FROM users")
// DELETE FROM users

db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})
// DELETE FROM users
```

> Soft Delete

If your model contains a `gorm.DeletedAt` field (`gorm.Model` already contains this field), it will automatically gain the ability to soft delete!

When calling `Delete` on a model with soft delete capability, the record will not be actually deleted from the database. However, GORM will set `DeletedAt` to the current time, and you can no longer find the record through normal query methods.

```go
// user's ID is `111`
db.Delete(&user)
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE id = 111;

// Batch delete
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// Soft deleted records will be ignored when querying
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```

If you don't want to introduce `gorm.Model`, you can also enable the soft delete feature like this:

```go
type User struct {
ID int
Deleted gorm.DeletedAt
Name string
}
```

* Find soft deleted records

You can use `Unscoped` to find soft deleted records

```go
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```

> Permanently delete

You can also use `Unscoped` to permanently delete matching records

```go
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```

> Delete Flag

Use unix timestamp as delete flag

```go
import "gorm.io/plugin/soft_delete"

type User struct {
ID uint
Name string
DeletedAt soft_delete.DeletedAt
}

// Query
SELECT * FROM users WHERE deleted_at = 0;

// Delete
UPDATE users SET deleted_at = /* current unix second */ WHERE ID = 1;
```

**INFO** When using soft delete with unique fields, you need to create a composite index using the `DeletedAt` field based on the unix timestamp, for example: use `1` / `0` as the delete flag

```go
import "gorm.io/plugin/soft_delete"

type User struct {
ID uint
Name string
IsDel soft_delete.DeletedAt `gorm:"softDelete:flag"`
}

// Query
SELECT * FROM users WHERE is_del = 0;

// Delete
UPDATE users SET is_del = 1 WHERE ID = 1;
```



