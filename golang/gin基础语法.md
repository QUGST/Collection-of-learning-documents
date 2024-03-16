gin特性

1. 快速 基于 Radix 树的路由，小内存占用。没有反射。可预测的 API 性能。
    
2. 支持中间件 传入的 HTTP 请求可以由一系列中间件和最终操作来处理。 例如：Logger，Authorization，GZIP，最终操作 DB。
    
3. Crash 处理 Gin 可以 catch 一个发生在 HTTP 请求中的 panic 并 recover 它。这样，你的服务器将始终可用。例如，你可以向 Sentry 报告这个 panic！
    
4. JSON 验证 Gin 可以解析并验证请求的 JSON，例如检查所需值的存在。
    
5. 路由组 更好地组织路由。是否需要授权，不同的 API 版本…… 此外，这些组可以无限制地嵌套而不会降低性能。
    
6. 错误管理 Gin 提供了一种方便的方法来收集 HTTP 请求期间发生的所有错误。最终，中间件可以将它们写入日志文件，数据库并通过网络发送。
    
7. 内置渲染 Gin 为 JSON，XML 和 HTML 渲染提供了易于使用的 API。
    
8. 可扩展性 新建一个中间件非常简单
    

# 路由

路由（Routing）是由一个 URI（或者叫路径）和一个特定的 HTTP 方法（GET、POST 等） 组成的，涉及到应用如何响应客户端对某个网站节点的访问。

在 RESTful 架构中，每个网址代表一种资源，不同的请求方式表示执行不同的操作：

![截图](attachment:6a4c337c3c5bed3679d94eba201194de)

## 路由方式

在 Gin 框架中，路由是用来将 HTTP 请求映射到相应的处理函数的机制。Gin 提供了两种路由方式：基于 HTTP 方法的路由和基于路径的路由。以下是基于 HTTP 方法的路由和基于路径的路由的示例代码：

- 基于 HTTP 方法的路由

```golang
// GET 请求路由
router.GET("/users", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "get users",
    })
})
// POST 请求路由
router.POST("/users", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "create user",
    })
})
// PUT 请求路由
router.PUT("/users/:id", func(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{
        "message": fmt.Sprintf("update user %!s(MISSING)", id),
    })
})
// DELETE 请求路由
router.DELETE("/users/:id", func(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{
        "message": fmt.Sprintf("delete user %!s(MISSING)", id),
    })
})
```

- 基于路径的路由

```golang
action := c.Param("action")// 精确匹配路径路由
// /users/1  /users/2
router.GET("/users/:id", func(c *gin.Context) {
    id := c.Param("id")
    c.JSON(http.StatusOK, gin.H{
        "message": fmt.Sprintf("get user %!s(MISSING)", id),
    })
})
// /users/1/2  /users/2/4
router.GET("/users/:id/:name", func(c *gin.Context) {
    id := c.Param("id")
    name := c.Param("name")
    c.JSON(http.StatusOK, gin.H{
        "message": fmt.Sprintf("get user %!s(MISSING)", id),
    })
})
// 匹配任意路径路由
// /users/  /users/1 /users/1/2
router.GET("/users/*action", func(c *gin.Context) {
    action := c.Param("action")
    c.JSON(http.StatusOK, gin.H{
        "message": fmt.Sprintf("get user %!s(MISSING)", action),
    })
})
```

c.Param("action")获取对应参数的值

## c.String()、c.JSON()、c.JSONP()、c.XML() 和 c.HTML()

- c.String()：返回一个字符串类型的响应。可以使用该方法返回 HTML、CSS、JavaScript 或纯文本等内容。

```golang
c.String(http.StatusOK, "Hello, Gin")
```

- c.JSON()：返回一个 JSON 格式的响应。可以使用该方法返回 JSON 格式的数据。

```golang
data := gin.H{
    "name":  "Tom",
    "email": "tom@example.com",
}
c.JSON(http.StatusOK, data)
```

- c.JSONP()：返回一个 JSONP 格式的响应。可以使用该方法返回 JSONP 格式的数据。

```golang
data := gin.H{
    "name":  "Tom",
    "email": "tom@example.com",
}
c.JSONP(http.StatusOK, data)
```

- c.XML()：返回一个 XML 格式的响应。可以使用该方法返回 XML 格式的数据。

```golang
type Person struct {
    Name  string `xml:"name"`
    Email string `xml:"email"`
}
data := &Person{Name: "Tom", Email: "tom@example.com"}
c.XML(http.StatusOK, data)
```

- c.HTML()：返回一个 HTML 格式的响应。可以使用该方法返回网页内容。

```golang
c.HTML(http.StatusOK, "<h1>Hello, Gin</h1>")
```

## 路由组

在 Gin 中，可以通过路由分组来对不同的 URL 进行分类管理。这样做有助于提高代码可读性、可维护性和可扩展性。

可以使用 Group() 方法创建一个路由分组，并将不同的路由注册到该分组中。

```golang
func main() {
    router := gin.Default()

    // 简单的路由组: v1
    v1 := router.Group("/v1")
    {
      v1.GET("/users", getUsers)
    v1.GET("/users/:id", getUserByID)
    }

    // 简单的路由组: v2
    v2 := router.Group("/v2")
    {
        v2.POST("/login", loginEndpoint)
        v2.POST("/submit", submitEndpoint)
        v2.POST("/read", readEndpoint)
    }

    router.Run(":8080")
}
```

# HTML 模板渲染

在 Gin 中，可以使用 HTML 模板引擎来进行页面渲染。Gin 默认内置了 Go 的 html/template 包作为模板引擎。

## 全部模板放在一个目录里面的配置方法

具体步骤如下：

- 首先，需要创建一个 templates 目录，用于存放 HTML 模板文件。
- 编写 HTML 模板文件，例如 index.html，并将其保存到 templates 目录下。模板文件中可以使用模板语法，例如 {{ .title }} 表示输出 title 变量的值。
- 设置 HTML 模板目录 r.LoadHTMLGlob("templates/*")
- 在路由处理函数中，使用 HTML 模板引擎渲染页面，并将结果返回给客户端。
    
    ```golang
    func main() {
        r := gin.Default()
    
        // 设置 HTML 模板目录
        r.LoadHTMLGlob("templates/*")
    
        // 定义路由
        r.GET("/", func(c *gin.Context) {
            c.HTML(http.StatusOK, "index.html", gin.H{
                "title": "Gin HTML Template",
            })
        })
    
        // 启动服务器
        r.Run()
    }
    ```
    

## 文件不是.html

需要注意的是，在使用 LoadHTMLGlob 方法加载 HTML 模板文件时，文件名必须以 .html 结尾。如果需要使用其他扩展名（例如 .tmpl），可以使用 LoadHTMLFiles 方法进行加载。

在 Gin 中，可以使用 LoadHTMLFiles 方法逐个加载 HTML 模板文件。该方法需要指定每个 HTML 模板文件的路径，并将其注册到模板引擎中。

```golang
func main() {
    r := gin.Default()

    // 设置 HTML 模板目录
    r.LoadHTMLFiles("templates/index.html", "partials/header.html", "partials/footer.html")

    // 定义路由
    r.GET("/", func(c *gin.Context) {
        c.HTML(http.StatusOK, "index.html", gin.H{
            "title": "Gin HTML Template",
        })
    })

    // 启动服务器
    r.Run()
}
```

**LoadHTMLFiles 方法会按照参数列表中的顺序依次加载各个 HTML 模板文件，因此需要保证它们的加载顺序正确。如果多个 HTML 模板文件存在相同的名称或路径，那么只有最后一个被加载的文件会生效。**

## 模板放在不同目录里面的配置方法

```golang
router.LoadHTMLGlob("templates/**/*") 
router.GET("/", func(c *gin.Context) { 
     c.HTML(http.StatusOK, "default/index.html", gin.H{ "title": "前台首页", }) 
})
router.GET("/admin", func(c *gin.Context) { 
     c.HTML(http.StatusOK, "admin/index.html", gin.H{ "title": "后台首页", }) 
})
```

**需要注意的是，在加载 HTML 模板文件时，需要指定相对于程序运行目录的路径**

## 嵌套模版

# 静态文件服务

gin 是一个 Go 语言的 Web 框架，可以用来快速构建 Web 应用程序。它提供了静态文件服务的功能，可以将本地磁盘上的静态文件（如图片、CSS 文件、JavaScript 文件等）映射到 Web 服务器的 URL 上，使这些静态文件能够通过 Web 访问。要在 gin 中设置静态文件服务，可以使用 gin.Static() 方法，指定静态文件所在的目录和 URL 前缀即可。

```golang
router := gin.Default()
router.Static("/static", "./static")
```