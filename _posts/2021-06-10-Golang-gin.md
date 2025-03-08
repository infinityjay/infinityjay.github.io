---
title:  Golang Gin - learning notes
categories:
  - Golang
tags:
  - Gin
  - Web framework
  - Learning notes
---

Content

{% include toc %}

## Introduction

(Official) Gin is a web framework written in Go (Golang). It features a martini-like API with performance that is up to 40 times faster thanks to [httprouter](https://github.com/julienschmidt/httprouter). If you need performance and good productivity, you will love Gin.

Official address: https://github.com/gin-gonic/gin

Chinese document: https://www.kancloud.cn/shuangdeyu/gin_book/949433

<font color = seagreen>Note: Using gin requires Go version 1.6 or higher</font>

## Installation

Install the required dependencies for `gin`

```shell
go get -u github.com/gin-gonic/gin
```

Import in code

```go
import "github.com/gin-gonic/gin"
```

## Usage examples

### Quick Start

```go
package main

import (
"github.com/gin-gonic/gin"
)

func sayHello(c *gin.Context) {
c.JSON(200, gin.H{
"message": "Hello Golang!",
})
}

func main() {
// Create a default routing engine
r := gin.Default()
// Specify that when a user uses a GET request to access /hello, the sayHello function will be executed
r.GET("/hello", sayHello)
// Start the service, the default port is 8080, which can be specified as 9090
r.Run(":9090")
}
```

### Different request methods - RESTful API

Four different request methods in HTTP protocol

- `GET` is used to obtain resources
- `POST` is used to create resources
- `PUT` is used to update resources
- `DELETE` is used to delete resources

![image-20210702152345765](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210702152345765.png)

```go
func main() {
 r := gin.Default()
 r.GET("/book", func(c *gin.Context) {
 c.JSON(http.StatusOK, gin.H{
 "message": "GET",
 })
 })

 r.POST("/book", func(c *gin.Context) {
 c.JSON(200, gin.H{
 "message": "POST",
 })
 })

 r.PUT("/book", func(c *gin.Context) {
 c.JSON(200, gin.H{ "message": "PUT",
})
})

r.DELETE("/book", func(c *gin.Context) {
c.JSON(200, gin.H{
"message": "DELETE",
})
})
}
```

### Return JSON, XML and other formats

> XML, JSON, YAML and ProtoBuf rendering

```go
func main() {
r := gin.Default()

// gin.H is a map type map[string]interface{}
r.GET("/someJSON", func(c *gin.Context) {
c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
})

r.GET("/moreJSON", func(c *gin.Context) {
// You can also use a structure, but the first letter of the field in the structure needs to be uppercase, lowercase cannot be exported
// Add a tag to the structure as follows, so that the output content is user
var msg struct {
Name string `json:"user"`
Message string
Number int
}
msg.Name = "Lena"
msg.Message = "hey"
msg.Number = 123
// Note that msg.Name becomes "user" in the JSON
// Will output : {"user": "Lena", "Message": "hey", "Number": 123}
c.JSON(http.StatusOK, msg)
})

r.GET("/someXML", func(c *gin.Context) {
c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
})

r.GET("/someYAML", func(c *gin.Context) {
 c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
 })

 r.GET("/someProtoBuf", func(c *gin.Context) {
 reps := []int64{int64(1), int64(2)}
 label := "test"
 // The specific definition of protobuf is written in the testdata/protoexample file.
 data := &protoexample.Test{
 Label: &label,
 Reps: reps,
 }
 // Note that data becomes binary data in the response
 // Will output protoexample.Test protobuf serialized data
 c.ProtoBuf(http.StatusOK, data)
 })

 // Listen and serve on 0.0.0.0:8080 r.Run(":8080")
}
```

### Get querystring request parameters

> What is querystring?

**query string** is the part of the URL passed to this program. By using it, we can send data from the HTTP client to the application that generates the web page.

GET request, in the URL? The following is the query string parameter, in key-value format

For example:

https://cn.bing.com/search?q=querystring

> Example

```go
package main

import (
"net/http"

"github.com/gin-gonic/gin"

)

func main() {
r := gin.Default()

// Get the query string parameter carried by the browser request
r.GET("/web", func(c *gin.Context) {
// Three ways to get the query string parameter carried in the request through query
// name := c.Query("query")
// name := c.DefaultQuery("query","somebody")
name, ok := c.GetQuery("query")
if !ok {
// No value is obtained
name = "somebody"
}
c.JSON(http.StatusOK, gin.H{
"Name": name,
}) 
})
 r.Run(":9090")
}

//Request http://192.168.166.89:9090/web?query=1233445
//Return {"Name":"1233445"}
```



### Get form form parameters

> login.html file

```html
<!DOCTYPE html>
<html lang="en">
<head>
 <meta charset="UTF-8">
 <title>login</title>
</head>
<body>
<form action="/login" method="POST" novalidate autocomplete="off">
 <label for="username">username</label>
 <input type="text" name="username" id="username">

 <label for="password">password:</label>
 <input type="password" name="password" id="password">

 <input type="submit" value="Login">
</form>

</body>
</html>
```

> Complete main function

```go
package main

import (
"net/http"

"github.com/gin-gonic/gin"
)

func main() {
r := gin.Default()
r.LoadHTMLFiles("./login.html", "./index.html")

// get login.html page
r.GET("/login", func(c *gin.Context) {
c.HTML(http.StatusOK, "login.html", nil)
})

// post login operation
r.POST("/login", func(c *gin.Context) {
// Get the data submitted by the form
username := c.PostForm("username")
password := c.PostForm("password")

// After clicking the login button, display the index.html file
c.HTML(http.StatusOK, "index.html", gin.H{
"Name": username,
"Password": password,
})
})
r.Run(":9090")
}
```

### Get path (URI) parameters

The parameters of the request are passed URL path (or URI), such as `/login/data`

> main function

```go
package main

import (
"net/http"

"github.com/gin-gonic/gin"

)

func main() {
r := gin.Default()

r.GET("/:name/:age", func(c *gin.Context) {
// get the path parameter
name := c.Param("name")
age := c.Param("age")
// JSON reveive the return
c.JSON(http.StatusOK, gin.H{
"name": name,
"age": age,
})
})
r.Run(":9090")
}

```

> Result

Webpage test: http://localhost IP:9090/chen/18

Result return: {"age":"18","name":"chen"}

### Parameter Binding

To bind the request body to a structure, use model binding. Currently, JSON, XML, YAML, and standard form values ​​(foo=bar&boo=baz) are supported.

Gin uses [go-playground/validator.v8](https://github.com/go-playground/validator) to validate parameters. [See the full documentation](https://godoc.org/gopkg.in/go-playground/validator.v8#hdr-Baked_In_Validators_and_Tags).

You need to set the tag on the bound field. For example, if the binding format is json, you need to set `json:"fieldname"` like this.

In addition, Gin provides two sets of binding methods:

- Must bind
- - Methods - `Bind`, `BindJSON`, `BindXML`, `BindQuery`, `BindYAML`
- - Behavior - These methods use `MustBindWith` at the bottom. If there is a binding error, the request will be aborted by the following instruction `c.AbortWithError(400, err).SetType(ErrorTypeBind)`, the response status code will be set to 400, and the request header `Content-Type` will be set to `text/plain; charset=utf-8`. Note that if you try to set the response code after this, a warning `[GIN-debug] [WARNING] Headers were already written. Wanted to override status code 400 with 422` will be issued. If you want more control over the behavior, use the `ShouldBind` related methods
- Should bind
- - Methods - `ShouldBind`, `ShouldBindJSON`, `ShouldBindXML`, `ShouldBindQuery`, `ShouldBindYAML`
- - Behavior - These methods use `ShouldBindWith` under the hood. If there is a binding error, an error is returned, and the developer can handle the request and error correctly.

When we use the binding method, Gin will infer which binder to use based on the Content-Type. If you are sure what you are binding, you can use `MustBindWith` or `BindingWith`.

You can also specify rule-specific modifiers for fields. If a field is modified with `binding:"required"` and the value of the field is empty when binding, an error will be returned.

> Bind the request parameters to the backend structure

```go
package main

import (
"fmt"
"net/http"

"github.com/gin-gonic/gin"
)

// Use tag to specify the type to correspond to the field after binding
type UserInfo struct {
Username string `form:"username"`
Password string `form:"password"`
}

func main() {
r := gin.Default()
r.GET("/user", func(c *gin.Context) {
// username := c.Query("username")
// password := c.Query("password")
// u := UserInfo{
// username: username,
// password: password,
// }

var u UserInfo
// Reference passing required
err := c.ShouldBind(&u) // ？
if err != nil {
c.JSON(http.StatusBadRequest, gin.H{
"error": err.Error(),
})
} else {
fmt.Printf("%#v\n", u)
c.JSON(http.StatusOK, gin.H{
"status": "ok",
})
}
})
r.Run(":8080")
}
```

> Result

```shell
# Request link
http://localhost ip:8080/user?username=timi&password=123445

# Return parameter
main.UserInfo{Username:"timi", Password:"123445"}
[GIN] 2021/07/05 - 17:57:19 | 200 | 310.5µs | xxx | GET "/user?username=timi&password=123445"

```

> Request parameters are bound to JSON and HTML files

```go
type Login struct {
 User string `form:"user" json:"user" xml:"user" binding:"required"`
 Password string `form:"password" json:"password" xml:"password" binding:"required"`
}

func main() {
 router := gin.Default()

 // Example for binding JSON ({"user": "manu", "password": "123"})
 router.POST("/loginJSON", func(c *gin.Context) {
 var json Login
 if err := c.ShouldBindJSON(&json); err != nil {
 c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()}) return
 }

 if json.User != "manu" || json.Password != "123" {
 c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
 return
 }

 c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
 })

 // Example for binding XML (
 // <?xml version="1.0" encoding="UTF-8"?>
 // <root>
 // <user>user</user>
 // <password>123</password>
 // </root>)
 router.POST("/loginXML", func(c *gin.Context) {
 var xml Login
 if err := c.ShouldBindXML(&xml); err != nil {
 c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
 return
 }

 if xml.User != "manu" || xml.Password != "123" {
 c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
 return
 }

 c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
 })
 // Listen and serve on 0.0.0.0:8080
 router.Run(":8080")
```

### File upload

> main function

```go
package main

import (
 "net/http"
 "path"

 "github.com/gin-gonic/gin"
)

func main() {
 r := gin.Default()
 r.LoadHTMLFiles("./index.html")
 r.GET("/index", func(c *gin.Context) {
c.HTML(http.StatusOK, "index.html", nil)
})

// Process the operation after clicking the upload button
r.POST("/upload", func(c *gin.Context) {
// Read the file from the request
f, err := c.FormFile("f1")
if err != nil {
c.JSON(http.StatusBadRequest, gin.H{
"error": err.Error(),
})
} else {
// Save the read file locally (local on the server)
// dst := fmt.Sprintf("./%s", f.Filename)
// The path package is stored in the current directory
dst := path.Join("./", f.Filename)
c.SaveUploadedFile(f, dst)
c.JSON(http.StatusOK, gin.H{
"status": "OK",
})
} 

})
 r.Run(":8080")
}
```

> Index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
 <meta charset="UTF-8">
 <meta http-equiv="X-UA-Compatible" content="IE=edge">
 <meta name="viewport" content="width=device-width, initial-scale=1.0">
 <title>index</title>
</head>
<body>
 <form action="/upload" method="post" enctype="multipart/form-data">
 <input type="file" name="f1">
 <input type="submit" value="Upload">
 </form>
</body>
</html>
```

> Result

* Select the file by index path

![image-20210705182648884](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210705182648884.png)

* Click upload after selecting the file

![image-20210705182801832](https://raw.githubusercontent.com/infinityjay/myImageHost/main/typora/image-20210705182801832.png)

* Automatically switch to the /upload path, the file is successfully uploaded and saved to the current directory

> Multiple file uploads--main function

```go
func main() {
router := gin.Default()
// Process multipart The default memory limit for submitting files in forms is 32 MiB
// You can modify it in the following way
// router.MaxMultipartMemory = 8 << 20 // 8 MiB
router.POST("/upload", func(c *gin.Context) {
// Multipart form
form, _ := c.MultipartForm()
files := form.File["file"]

for index, file := range files {
log.Println(file.Filename)
dst := fmt.Sprintf("C:/tmp/%s_%d", file.Filename, index)
// Upload files to the specified directory
c.SaveUploadedFile(file, dst)
}
c.JSON(http.StatusOK, gin.H{
"message": fmt.Sprintf("%d files uploaded!", len(files)),
})
})
router.Run()
}
```

### Redirect

It's easy to issue HTTP redirects, supporting internal and external links

```go
r.GET("/test", func(c *gin.Context) {
c.Redirect(http.StatusMovedPermanently, "http://www.google.com/")
})
```

Gin route redirection, use the following `HandleContext`

```go
r.GET("/test", func(c *gin.Context) {
c.Request.URL.Path = "/test2"
r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
c.JSON(200, gin.H{"hello": "world"})
})
```

### Routing Groups

> main function

```go
func main() {
router := gin.Default()

// Simple group: v1
v1 := router.Group("/v1")
{
v1.POST("/login", loginEndpoint)
v1.POST("/submit", submitEndpoint)
v1.POST("/read", readEndpoint)
}

// Simple group: v2
v2 := router.Group("/v2")
{
v2.POST("/login", loginEndpoint)
v2.POST("/submit", submitEndpoint)
v2.POST("/read", readEndpoint)
}

router.Run(":8080")
}
```

### Using middleware

The Gin framework allows developers to add their own Hook function in the process of processing requests. This Hook function is middleware. Middleware is suitable for processing some common business logic, such as login authentication, permission verification, data paging, logging, time statistics, etc.

> Define middleware

Gin's middleware is of type `gin.HandlerFunc`. The following defines a middleware that counts request time and prints it in the log

```go
func Logger() gin.HandlerFunc {
return func(c *gin.Context) {
t := time.Now()

// Set example variable
c.Set("example", "12345")

// before request

c.Next()

// after request
latency := time.Since(t)
log.Print(latency)

// access the status we are sending
status := c.Writer.Status()
log.Println(status)
}
}
```

> example 1

```go
package main

import (
"fmt"
"net/http"
"time"

"github.com/gin-gonic/gin"
)

// This function is of type HandlerFunc
func indexHandler(c *gin.Context) {
fmt.Println("index")
c.JSON(http.StatusOK, gin.H{
"msg": "index",
})
}
// Define a middleware m1: Count the time consumption of the request processing function
func m1(c *gin.Context) {
fmt.Println("m1 in ...")
// Calculate the request time consumption
start := time.Now()
// Call the subsequent processing function, that is, indexHandler
c.Next()
// // Prevent the subsequent function from being called
// c.Abort()
cost := time.Since(start)
fmt.Printf("cost of time: %v\n", cost)
fmt.Println("m1 out...")
}

func main() {
r := gin.Default()

// func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
r.GET("/index", m1, indexHandler)

r.Run(":8080")
}
```

> Result

```shell
# Visit
http://192.168.166.89:8080/index
# Output
m1 in ...
index
cost of time: 106.292µs
m1 out...
[GIN] 2021/07/05 - 19:59:27 | 200 | 142.041µs | 192.168.166.89 | GET "/index"
```

> Global registration of middleware

```go
func main() {
r := gin.Default()

// Global registration of middleware m1
r.Use(m1)

// func (group *RouterGroup) GET(relativePath string, handlers ...HandlerFunc) IRoutes {
 r.GET("/index", indexHandler)
 r.GET("/shop", func(c *gin.Context) {
 c.JSON(http.StatusOK, gin.H{
 "msg": "shop",
 })
 })
 r.GET("/user", func(c *gin.Context) {
 c.JSON(http.StatusOK, gin.H{
 "msg": "user",
 })
 })

 r.Run(":8080")
}


// result
Visit: http://192.168.166.89:8080/user
Output:
m1 in...
cost of time: 51.75µs
m1 out...
[GIN] 2021/07/05 - 20:06:53 | 200 | 84.541µs | 192.168.166.89 | GET "/user"

```

> Example 2

```go
// Commonly used writing when registering the middleware
func authMiddleware(doCheck bool) gin.HandlerFunc {
// Connect to the database or other work. . .
return func(c *gin.Context) {
// Login judgment
// if it is a logged-in user
// c.Next()
// else
// c.Abort()
}
}
```

> Other notes

* Use middleware

`gin.Default()` uses Logger and Recovery middleware by default

* Do not use middleware

`gin.New()` Create a router without middleware

```go
func main() {
// Create a router without middleware
r := gin.New()

// Global middleware
// Use Logger middleware
r.Use(gin.Logger())

// Use Recovery middleware
r.Use(gin.Recovery())

// Add middleware to the route, you can add as many as you want
r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

// Add middleware to the route group
// authorized := r.Group("/", AuthRequired())
// exactly the same as:
authorized := r.Group("/")
// per group middleware! in this case we use the custom created
// AuthRequired() middleware just in the "authorized" group.
authorized.Use(AuthRequired())
{
authorized.POST("/login", loginEndpoint)
authorized.POST("/submit", submitEndpoint)
authorized.POST("/read", readEndpoint)

// nested group
testing := authorized.Group("testing")
testing.GET("/analytics", analyticsEndpoint)
}

// Listen and serve on 0.0.0.0:8080
r.Run(":8080")
}
```

* Using goroutines in middleware

When starting new goroutines in middleware or handlers, you should not use the original context inside them, you must use a read-only copy (`c.Copy()`)

```go
func main() {
 r := gin.Default()

 r.GET("/long_async",func(c *gin.Context) {
// Create a copy to be used in goroutine
cCp := c.Copy()
go func() {
// simulate a long task with time.Sleep(). 5 seconds
time.Sleep(5 * time.Second)
// Use the copy you created here
log.Println("Done! in path " + cCp.Request.URL.Path)
}()
})

r.GET("/long_sync", func(c *gin.Context) {
// simulate a long task with time.Sleep(). 5 seconds
time.Sleep(5 * time.Second)

// No goroutine is used here, so no copy is used
log.Println("Done! in path " + c.Request.URL.Path)
})

// Listen and serve on 0.0.0.0:8080
r.Run(":8080")
}
```

### Custom HTTP configuration

* Use the default configuration directly

```go
func main() {
router := gin.Default()
http.ListenAndServe(":8080", router)
}
```

* Custom configuration is available

```go
func main() {
router := gin.Default()

s := &http.Server{
Addr: ":8080",
Handler: router,
ReadTimeout: 10 * time.Second,
WriteTimeout: 10 * time.Second,
MaxHeaderBytes: 1 << 20,
}
s.ListenAndServe()
}
```

### Create a log

> Write a log file

```go
func main() {
// Disable console color
gin.DisableConsoleColor()

// Create a file for logging
f, _ := os.Create("gin.log")
gin.DefaultWriter = io.MultiWriter(f)

// If you need to write logs to both the file and the console, use the following code
// gin.DefaultWriter = io.MultiWriter(f, os.Stdout)

router := gin.Default()
router.GET("/ping", func(c *gin.Context) {
c.String(200, "pong")
})

router.Run(":8080")
}
```

> Custom log format

```go
func main() {
router := gin.New()

// LoggerWithFormatter middleware will write logs to gin.DefaultWriter
// By default gin.DefaultWriter = os.Stdout
router.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string {

// Your custom format
return fmt.Sprintf("%s - [%s] \"%s %s %s %d %s \"%s\" %s\"\n",
param.ClientIP,
param.TimeStamp.Format(time.RFC1123),
param.Method,
param.Path,
param.Request.Proto,
param.StatusCode,
param.Latency,
param.Request.UserAgent(),
param.ErrorMessage,
)
}))
router.Use(gin.Recovery())

router.GET("/ping", func(c *gin.Context) {
c.String(200, "pong")
})

router.Run(":8080")
}
```

### Set and get cookies

```go
import ( 
"fmt"

 "github.com/gin-gonic/gin"
)

func main() {

 router := gin.Default()

 router.GET("/cookie", func(c *gin.Context) {

 cookie, err := c.Cookie("gin_cookie")

 if err != nil {
 cookie = "NotSet"
 c.SetCookie("gin_cookie", "test", 3600, "/", "localhost", false, true)
 }

 fmt.Printf("Cookie value: %s \n", cookie)
 })

 router.Run()
}
```


