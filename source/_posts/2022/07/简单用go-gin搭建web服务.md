---
title: 简单用go+gin搭建web服务
date: 2022-07-29 11:08:06
id: 0702
tags: ['Go', 'Gin']
categories: Go
---

> 前言

学了挺久的 go 语言，一直也没有怎么去记录自己的学习过程，今天简单的记录下 go 的 web 服务器搭建过程（包含用 `mysql` 连接数据库）。

### 项目环境搭建

- 首先来看下整个项目的目录结构（作为初学者，这里我搭建的比较简单）：

![Snipaste_2022-07-29_09-59-28.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1d6174afcc04a689ceb0a13693595a0~tplv-k3u1fbpfcp-watermark.image?)

- 关于本地 mysql 数据库

  - 数据库就两张表：（表名当时起的比较随意）

    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/96f238b13bbe4b86b3dc13fbed9227a0~tplv-k3u1fbpfcp-watermark.image?)

  - articles 表的数据及其结构：

    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/730f7ca2fbc54c61b0d79b98fc056680~tplv-k3u1fbpfcp-watermark.image?)

  - user 表的数据及其结构：

    ![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8affb054f8e049ebbbfec61577fbb374~tplv-k3u1fbpfcp-watermark.image?)

- 初始化项目

  这里进入的项目的目录，打开终端，输入：
  (注：这里用 server 作为目录，当然也可以用自己喜欢的)

  ```go
  go mod init server
  ```

  初始化完成后就是这样了，会在根目录下创建 `go.mod` 文件以及其中的两行代码：

![Snipaste_2022-07-29_10-05-20.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8069c75feff8478a8b3ac8e187b050d3~tplv-k3u1fbpfcp-watermark.image?)

- 依赖安装，我们这里用的是 `gin` 框架，因为相对简单及轻便，对初学者也比较友好

  这里使用 `go get` 下载依赖：

  ```go
  go get -u github.com/gin-gonic/gin
  ```

  安装完就可以看到 `go.mod` 文件中多了如下的依赖，以及多出的 `go.sum` 文件。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9387a7cb1ae5456b9f6c4ee556865402~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fc4d1a021a194edba8531731dde9b5e7~tplv-k3u1fbpfcp-watermark.image?)

- 后面要用到 `mysql` ，这里也需要安装下。

```go
go get github.com/go-sql-driver/mysql
```

### 路由创建

- 在 `routers/router.go` 中开始创建路由

```go
package routers // routers包

import (
    "database/sql"
    "fmt"
    "server/utils"  // 工具包里做了数据库的连接，下面会详细说

    "github.com/gin-gonic/gin"
)

var (
    Db *sql.DB
)

// 路由中间件
func MiddleWare() gin.HandlerFunc {
    return func(c *gin.Context) {
        fmt.Println("中间件执行完毕")
        c.Next()
    }
}

func SetupRouter() *gin.Engine {
    Db = utils.DbConnect()
    r := gin.Default()
    r.Use(MiddleWare())
    r.POST("/login", LoginHandler) // 子路由单独抽离出来在login.go中
    user := r.Group("/user")  // 二级路由设置
    {
        user.GET("/list", UserHandler)
    }
    return r
}
```

- login 和 user 的大致如下所示：（这里也贴下 `login.go` 和 `user.go` 的代码）

```go
// login.go

package routers

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

// 登录数据的结构体（确保和数据库中的结构一致）
type Login struct {
    Id       int    `json:"id" form:"id"`
    Username string `json:"username" form:"username"`
    Password string `json:"password" form:"password"`
}

// 登录接口
func LoginHandler(c *gin.Context) {
    var loginForm, user Login
    if err := c.ShouldBindJSON(&loginForm); err != nil {
        // 返回错误信息
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    // 确保数据库中有这张user的表
    rows, err := Db.Query("select * from user where username=?", loginForm.Username)
    defer rows.Close()
    if err != nil {
        panic(err)
    }
    for rows.Next() {
        rows.Scan(&user.Id, &user.Username, &user.Password)
    }
    // 判断用户名密码是否正确
    if loginForm.Username != user.Username || loginForm.Password != user.Password {
        c.JSON(http.StatusBadRequest, gin.H{
            "code": -1,
            "msg":  "用户名或密码不正确",
        })
        return
    }
    c.JSON(http.StatusOK, gin.H{
        "code":  0,
        "msg":   "登录成功",
        "token": "123jkasdqwe1231a12r13",
    })
}

```

```go
// user.go
package routers

import (
    "log"
    "net/http"

    "github.com/gin-gonic/gin"
)

// user的结构
type User struct {
    Id       string `json:"id" form:"id"`
    Username string `json:"username" form:"username"`
    Age      string `json:"age" form:"age"`
    Sex      string `json:"sex" form:"sex"`
}

func UserHandler(c *gin.Context) {
    var user User
    // 定义一个切片数据，存放user数据
    userList := make([]User, 0)
    rows, err := Db.Query("select * from articles where username=?", "张三")
    defer rows.Close() // 即时关闭
    if err != nil {
        log.Fatal(err)
    }
    i := 0
    for rows.Next() { //循环显示所有的数据
        rows.Scan(&user.Id, &user.Username, &user.Age, &user.Sex)
        userList = append(userList, user)
        i++
    }
    // 接口返回给前端的数据
    c.JSON(http.StatusOK, gin.H{
        "code": 0,
        "msg":  "请求成功",
        "data": &userList,
    })
}

```

### 连接 mysql 数据库

- 在 `utils/db.go` 中编写连接数据库的代码

```go
package utils

// import的时候不需要用到mysql这个包，但是要引入一下
import (
    "database/sql"
    "fmt"

    _ "github.com/go-sql-driver/mysql"
)

func DbConnect() *sql.DB {
    // root:password@(localhost:3306)/blog
    // 用户名:密码@(数据库地址及端口号)/数据库名
    db, err := sql.Open("mysql", "root:password@(localhost:3306)/blog")
    db.SetMaxOpenConns(10) // 最大连接数
    db.SetMaxIdleConns(5)
    if err != nil {
        panic(err)
    }
    if err := db.Ping(); err != nil {
        fmt.Println("连接失败")
        panic(err.Error())
    }
    fmt.Println("连接成功")
    return db // 连接成功后返回db以便在路由中调用
}

```

### 项目启动

- 这里我们来看下 `main.go`

```go
package main

import (
    "server/routers"
)

func main() {
    // 在routers/router.go文件中已经创建了路由以及Db.DbConnect，这里执行一下即可
    r := routers.SetupRouter()
    // 在3001端口启动我们的项目（端口号随意）
    r.Run(":3001")
}

```

- 启动项目

```go
go run main.go
```

可以看到项目已经启动成功：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd112cc387884ea8bd45431d405cc036~tplv-k3u1fbpfcp-watermark.image?)

接下来测试下 `user/list` 的接口，打开浏览器，输入 `http://localhost:3001/user/list`:
(我用了浏览器的 json 插件，显示的效果会有点点不一样)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/229bb6fb75fb497b8ba1a5b40664f369~tplv-k3u1fbpfcp-watermark.image?)

> 以上就是关于 go 的简单 web 服务器的创建了，我们也就可以开始写一些简单的接口了！
