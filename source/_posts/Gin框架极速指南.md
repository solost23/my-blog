---
title: Gin框架极速指南
---

Gin 是一个用 Go 语言编写的 web 微框架，封装比较优雅，API 友好，具有快速灵活，容错方便等特点。它是一个类似于 martini 但拥有更好性能的 API 框架，由于使用了 httpprouter，速度提高了近 40 倍。如果你是性能和高效的追求者，你会爱上 Gin。

### <center>Gin框架介绍</center>

Go 世界里最流行的 Web 框架，[Github](https://github.com/gin-gonic/gin) 上有 46k+star。基于 httprouter 开发的 Web 框架。[中文文档](https://gin-gonic.com/zh-cn/docs/)齐全，简单易用的轻量级框架。

### <center>Gin框架安装与使用</center>

#### <font color=red>安装</font>

```go
go get -u github.com/gin-gonic/gin
```

#### <font color=red>第一个Gin示例</font>

```go
package main
 
import (
	"github.com/gin-gonic/gin"
)
 
func main() {
	// 创建一个默认的路由引擎
	r := gin.Default()
	// GET：请求方式；/hello：请求的路径
	// 当客户端以GET方法请求/hello路径时，会执行后面的匿名函数
        // gin.Context封装了Request和Response
	r.GET("/hello", func(c *gin.Context) {
		// c.JSON：返回JSON格式的数据
		c.JSON(200, gin.H{
			"message": "Hello world!",
		})
	})
	// 启动HTTP服务，默认在0.0.0.0:8080启动服务
	r.Run()
}
```

将上面的代码保存并编译执行，然后使用浏览器打开 127.0.0.1:8080/hello 就能看到一串 JSON 字符串。

### <center>RESTful API</center>

REST 与技术无关，代表的是一种软件架构风格，REST 是 Representational State Transfer 的简称，中文翻译为 “表征状态转移” 或 “表现层状态转化”。

简单来说，REST 的含义就是客户端与 Web 服务器之间进行交互的时候，使用 HTTP 协议中的 4 个请求方法代表不同的动作：

- GET 用来获取资源
- POST 用来新建资源
- PUT 用来更新资源
- DELETE 用来删除资源

只要 API 程序遵循了 REST 风格，那就可以称其为 RESTful API。目前在前后端分离的架构中，前后端基本都是通过 RESTful API 来进行交互。例如，我们现在要编写一个管理书籍的系统，我们可以对一本书进行查询、创建、更新和删除等操作，我们在编写程序的时候就要设计客户端 / 浏览器与我们 Web 服务端交互的方式和路径。按照我们的经验我们通常会设计成如下模式：

| 请求方法 |     URL      |     含义     |
| :------: | :----------: | :----------: |
|   GET    |    /book     | 查询书籍信息 |
|   POST   | /create_book | 创建书籍信息 |
|   POST   | /update_book | 更新书籍信息 |
|   POST   | /delete_book | 删除书籍信息 |

同样的需求我们按照 RESTful API 设计如下：

| 请求方法 |  URL  |     含义     |
| :------: | :---: | :----------: |
|   GET    | /book | 查询书籍信息 |
|   POST   | /book | 创建书籍信息 |
|   PUT    | /book | 更新书籍信息 |
|  DELETE  | /book | 删除书籍信息 |

Gin 框架支持开发 RESTful API 的开发。

```go
func main() {
	r := gin.Default()
	r.GET("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "GET",
		})
	})
 
	r.POST("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "POST",
		})
	})
 
	r.PUT("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "PUT",
		})
	})
 
	r.DELETE("/book", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "DELETE",
		})
	})
    r.Run(":9000")
}
```

开发 RESTful API 的时候我们通常使用 [Postman](https://www.postman.com/) 来作为客户端的测试工具。

### <center>Gin渲染</center>

#### <font color=red>HTML渲染</font>

我们首先定义一个存放模板文件的 templates 文件夹，然后在其内部按照业务分别定义一个 posts 文件夹和一个 users 文件夹。posts/index.tmpl 文件的内容如下：

```go
{{define "posts/index.html"}}
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>posts/index</title>
</head>
<body>
    {{.title}}
</body>
</html>
{{end}}
```

users/index.tmpl 文件的内容如下：

```go
{{define "users/index.html"}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>users/index</title>
</head>
<body>
    {{.title}}
</body>
</html>
{{end}}
```

Gin 框架中使用 LoadHTMLGlob () 或者 LoadHTMLFiles () 方法进行 HTML 模板渲染：

```go
func main() {
	r := gin.Default()
	r.LoadHTMLGlob("templates/**/*")  // 解析模板
	//r.LoadHTMLFiles("templates/posts/index.html", "templates/users/index.html")  // 解析模板
	r.GET("/posts/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "posts/index.html", gin.H{
			"title": "posts/index",
		})
	})
 
	r.GET("users/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "users/index.html", gin.H{  // 渲染模板
			"title": "users/index",
		})
	})
 
	r.Run(":9090")
}
```

注意:

- 一个 * 表示通配文件，两个 * 表示通配文件夹，当需要渲染的文件很多的时候用 LoadHTMLGlob。
- 若不需要向模板文件传数据，gin.H 的位置写 nil。

#### <font color=red>自定义模板函数</font>

定义一个不转义相应内容的 safe 模板函数如下：

```go
func main() {
	router := gin.Default()
        // 解析模板之前设置模板函数
	router.SetFuncMap(template.FuncMap{
		"safe": func(str string) template.HTML{
			return template.HTML(str)
		},
	})
	router.LoadHTMLFiles("./index.tmpl")
 
	router.GET("/index", func(c *gin.Context) {
		c.HTML(http.StatusOK, "index.tmpl", "<a href='https://liwenzhou.com'>李文周的博客</a>")
	})
 
	router.Run(":9090")
}
```

在 index.html 中使用定义好的 safe 模板函数：

```go
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>修改模板引擎的标识符</title>
</head>
<body>
<div>{{ . | safe }}</div>
</body>
</html>
```

#### <font color=red>静态文件处理</font>

静态文件：html 页面上用到的 样式文件.css、js 文件、图片文件等。

当我们渲染的 HTML 文件中引用了静态文件时，我们只需要按照以下方式在渲染页面调用 gin.Static 方法即可：

```go
func main() {
	r := gin.Default()
        // 第一个参数为tmpl文件中引用的目录，第二个目录是存放静态文件的目录
	r.Static("/static", "./statics")
	r.LoadHTMLGlob("templates/**/*")
   // ...
	r.Run(":8080")
}
```

然后在 tmpl 文件中所有以 /static 开头的就会去./statics 目录里找，例如：

```go
<link rel="stylesheet" href="/static/index.css">
```

#### <font color=red>使用模板继承</font>

Gin 框架默认都是使用单模板，如果需要使用 block template 功能，可以通过 "github.com/gin-contrib/multitemplate" 库实现，具体示例如下：

首先，假设我们项目目录下的 templates 文件夹下有以下模板文件，其中 home.tmpl 和 inde.tmpl 继承了 base.tmpl：

```go
templates
├── includes
│   ├── home.tmpl
│   └── index.tmpl
├── layouts
│   └── base.tmpl
└── scripts.tmpl
```

然后我们定义一个 loadTemplates 函数如下：

```go
func loadTemplates(templatesDir string) multitemplate.Renderer {
	r := multitemplate.NewRenderer()
	layouts, err := filepath.Glob(templatesDir + "/layouts/*.tmpl")
	if err != nil {
		panic(err.Error())
	}
	includes, err := filepath.Glob(templatesDir + "/includes/*.tmpl")
	if err != nil {
		panic(err.Error())
	}
	// 为layouts/和includes/目录生成 templates map
	for _, include := range includes {
		layoutCopy := make([]string, len(layouts))
		copy(layoutCopy, layouts)
		files := append(layoutCopy, include)
		r.AddFromFiles(filepath.Base(include), files...)
	}
	return r
}
```

我们在 main 函数中:

```go
func indexFunc(c *gin.Context){
	c.HTML(http.StatusOK, "index.tmpl", nil)
}
 
func homeFunc(c *gin.Context){
	c.HTML(http.StatusOK, "home.tmpl", nil)
}
 
func main(){
	r := gin.Default()
	r.HTMLRender = loadTemplates("./templates")
	r.GET("/index", indexFunc)
	r.GET("/home", homeFunc)
	r.Run()
}
```

#### <font color=red>补充文件路径处理</font>

关于模板文件和静态文件的路径，我们需要根据公司 / 项目的要求进行设置。可以使用下面的函数获取当前执行程序的路径。

```go
func getCurrentPath() string {
	if ex, err := os.Executable(); err == nil {
		return filepath.Dir(ex)
	}
	return "./"
}
```

#### <font color=red>JSON渲染</font>

```go
func main() {
	r := gin.Default()
 
	// gin.H 是map[string]interface{}的缩写
	r.GET("/someJSON", func(c *gin.Context) {
		// 方式一：自己拼接JSON
		c.JSON(http.StatusOK, gin.H{"message": "Hello world!"})
	})
	r.GET("/moreJSON", func(c *gin.Context) {
		// 方法二：使用结构体
		var msg struct {
			Name    string `json:"user"`  // 当前端需要首字母小写的时候定义一个tag，前端展示的就是tag
			Message string
			Age     int
		}
		msg.Name = "ty"
		msg.Message = "Hello world!"
		msg.Age = 18
		c.JSON(http.StatusOK, msg)
	})
	r.Run(":8080")
}
```

注意：

- 临时返回数据结构时用 gin.H。
- 项目级别返回数据时一般用结构体。

#### <font color=red>XML 渲染</font>

注意需要使用具名的结构体类型。

```go
func main() {
	r := gin.Default()
	// gin.H 是map[string]interface{}的缩写
	r.GET("/someXML", func(c *gin.Context) {
		// 方式一：自己拼接JSON
		c.XML(http.StatusOK, gin.H{"message": "Hello world!"})
	})
	r.GET("/moreXML", func(c *gin.Context) {
		// 方法二：使用结构体
		type MessageRecord struct {
			Name    string
			Message string
			Age     int
		}
		var msg MessageRecord
		msg.Name = "ty"
		msg.Message = "Hello world!"
		msg.Age = 18
		c.XML(http.StatusOK, msg)
	})
	r.Run(":8080")
}
```

#### <font color=red>YMAL渲染</font>

```go
r.GET("/someYAML", func(c *gin.Context) {
	c.YAML(http.StatusOK, gin.H{"message": "ok", "status": http.StatusOK})
})
```

#### <font color=red>protobuf渲染</font>

```go
r.GET("/someProtoBuf", func(c *gin.Context) {
	reps := []int64{int64(1), int64(2)}
	label := "test"
	// protobuf 的具体定义写在 testdata/protoexample 文件中。
	data := &protoexample.Test{
		Label: &label,
		Reps:  reps,
	}
	// 请注意，数据在响应中变为二进制数据
	// 将输出被 protoexample.Test protobuf 序列化了的数据
	c.ProtoBuf(http.StatusOK, data)
})
```

### <center>获取参数</center>

#### <font color=red>获取query string(url)参数</font>

querystring 指的是 URL 中？后面携带的参数，例如：/user/search?username=ty&address=sjz。获取请求的 querystring 参数的方法如下：

```go
func main() {
	//Default返回一个默认的路由引擎
	r := gin.Default()
	r.GET("/user/search", func(c *gin.Context) {
		username := c.DefaultQuery("username", "取不到")  // 取不到就用指定的默认值
		//username := c.Query("username")  // 获取浏览器发请求携带的query string参数
        //username, ok := c.GetQuery("query")  // 取到，返回（value, true）取不到，返回（"", false）
		//if !ok {
		//	username = "取不到"
		//}
		address := c.Query("address")
		//输出json结果给调用方
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  address,
		})
	})
	r.Run()
}
```

#### <font color=red>获取FORM参数</font>

当前端请求的数据通过 form 表单提交时，例如向 /user/search 发送一个 POST 请求，获取请求数据的方式如下：

```go
func main() {
	//Default返回一个默认的路由引擎
	r := gin.Default()
	r.POST("/user/search", func(c *gin.Context) {
		// DefaultPostForm取不到值时会返回指定的默认值
		//username := c.DefaultPostForm("username", "取不到")
		username := c.PostForm("username")
		address := c.PostForm("address")
		//输出json结果给调用方
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  address,
		})
	})
	r.Run(":9000")
}
```

#### <font color=red>获取json参数</font>

当前端请求的数据通过 JSON 提交时，例如向 /json 发送一个 POST 请求，则获取请求参数的方式如下：

```go
r.POST("/json", func(c *gin.Context) {
	// 注意：下面为了举例子方便，暂时忽略了错误处理
	b, _ := c.GetRawData()  // 从c.Request.Body读取请求数据
	// 定义map或结构体
	var m map[string]interface{}
	// 反序列化
	_ = json.Unmarshal(b, &m)
 
	c.JSON(http.StatusOK, m)
})
```

更便利的获取请求参数的方式，参加下面的 **参数绑定** 小节。

#### <font color=red>获取path(api)参数</font>

请求的参数通过 URL 路径传递，例如：/user/search/ty/zjz。获取请求 URL 路径中的参数方式如下：

```go
func main() {
	//Default返回一个默认的路由引擎
	r := gin.Default()
        // :匹配字符串，*可以匹配任意
	r.GET("/user/search/:username/*address", func(c *gin.Context) {
		username := c.Param("username")
		address := c.Param("address")
		//输出json结果给调用方
		c.JSON(http.StatusOK, gin.H{
			"message":  "ok",
			"username": username,
			"address":  address,
		})
	})
 
	r.Run(":9000")
}
```

#### <font color=red>参数绑定</font>

为了能够更方便地获取请求相关参数，提高开发效率，我们可以基于请求的 Content-Type 识别请求数据类型并利用反射机制自动提取请求中 QueryString、form 表单、JSON、XML 等参数到结构体中。下面的示例代码演示了.ShouldBind () 强大的功能，它能够基于请求自动提取 JSON、form 表单和 QueryString 类型的数据，并把值绑定到指定的结构体对象。

```go
// Binding from JSON
type Login struct {
        // binding:"required"修饰的字段，若接收值为空，则报错，是必选字段
	User     string `form:"username" json:"user" uri:"user" xml:"user" binding:"required"`
	Password string `form:"password" json:"password" url:"password" xml: "password" binding:"required"`
}
 
func main() {
	router := gin.Default()
 
	// 绑定JSON的示例 ({"user": "ty", "password": "123456"})
	router.POST("/loginJSON", func(c *gin.Context) {
		var login Login
 
		if err := c.ShouldBind(&login); err == nil {
			fmt.Printf("login info:%#v\n", login)
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})
 
	// 绑定form表单示例 (user=ty&password=123456)
	router.POST("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})
 
	// 绑定QueryString示例 (/loginQuery?user=ty&password=123456)
	router.GET("/loginForm", func(c *gin.Context) {
		var login Login
		// ShouldBind()会根据请求的Content-Type自行选择绑定器
		if err := c.ShouldBind(&login); err == nil {
			c.JSON(http.StatusOK, gin.H{
				"user":     login.User,
				"password": login.Password,
			})
		} else {
			c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		}
	})
 
	// Listen and serve on 0.0.0.0:8080
	router.Run(":8080")
}
```

ShouldBind 会按照下面的顺序解析请求中的数据完成绑定：

- 如果是 GET 请求，只使用 Form 绑定引擎（query）。
- 如果是 POST 请求，首先检查 content-type 是否为 JSON 或 XML，然后再使用 Form（form-data）。

### <center>文件上传</center>

#### <font color=red>单个文件上传</font>

前端页面:

```go
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>上传文件示例</title>
</head>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    <input type="file" name="f1">
    <input type="submit" value="上传">
</form>
</body>
</html>
```

后端代码:

```go
func main() {
	router := gin.Default()
	// 处理multipart forms提交文件时默认的内存限制是32 MiB
	// 可以通过下面的方式修改
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// 单个文件
		file, err := c.FormFile("f1")
		if err != nil {
			c.JSON(http.StatusInternalServerError, gin.H{
				"message": err.Error(),
			})
			return
		}
 
		log.Println(file.Filename)
		dst := fmt.Sprintf("C:/tmp/%s", file.Filename)
		// 上传文件到指定的目录
		c.SaveUploadedFile(file, dst)
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("'%s' uploaded!", file.Filename),
		})
	})
	router.Run()
}
```

#### <font color=red>多个文件上传</font>

```go
func main() {
	router := gin.Default()
	// 处理multipart forms提交文件时默认的内存限制是32 MiB
	// 可以通过下面的方式修改
	// router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// Multipart form
		form, _ := c.MultipartForm()
		files := form.File["file"]
 
		for index, file := range files {
			log.Println(file.Filename)
			dst := fmt.Sprintf("C:/tmp/%s_%d", file.Filename, index)
			// 上传文件到指定的目录
			c.SaveUploadedFile(file, dst)
		}
		c.JSON(http.StatusOK, gin.H{
			"message": fmt.Sprintf("%d files uploaded!", len(files)),
		})
	})
	router.Run()
}
```

### <center>重定向</center>

#### <font color=red>HTTP重定向</font>

HTTP 重定向很容易。内部、外部重定向均支持。

```go
r.GET("/test", func(c *gin.Context) {
	c.Redirect(http.StatusMovedPermanently, "http://www.sogo.com/")
})
```

#### <font color=red>路由重定向(定向到别的路由)</font>

```go
r.GET("/test", func(c *gin.Context) {
    // 指定重定向的URL
    c.Request.URL.Path = "/test2"
    r.HandleContext(c)
})
r.GET("/test2", func(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{"hello": "world"})
})
```

### <center>Gin路由</center>

#### <font color=red>普通路由</font>

```go
r.GET("/index", func(c *gin.Context) {...})
r.GET("/login", func(c *gin.Context) {...})
r.POST("/login", func(c *gin.Context) {...})
r.DELETE("/login", func(c *gin.Context) {...})
```

此外，还有一个可以匹配所有请求方法的 Any 方法如下：

```go
router.Any("/user", func(c *gin.Context) {
 
        // 拿到请求的方法，然后处理
	switch c.Request.Method {
	case http.MethodGet:
		c.JSON(http.StatusOK, gin.H {"method": "GET"})
	case http.MethodPost:
		c.JSON(http.StatusOK, gin.H {"method": "POST"})
	default:
		c.JSON(http.StatusOK, gin.H {"method": "其他"})
	}
})
```

为没有配置处理函数的路由添加处理程序，默认情况下它返回 404 代码，下面的代码为没有匹配到路由的请求都返回 views/404.html 页面。

```go
r.NoRoute(func(c *gin.Context) {
	c.HTML(http.StatusNotFound, "views/404.html", nil)
})
```

#### <font color=red>路由组</font>

我们可以将拥有共同 URL 前缀的路由划分为一个路由组。习惯性以一对 {} 包裹同组的路由，这只是为了看着清晰，用不用 {} 包裹功能上没什么区别。

```go
func main() {
	r := gin.Default()
	userGroup := r.Group("/user")
	{
		userGroup.GET("/index", func(c *gin.Context) {...})
		userGroup.GET("/login", func(c *gin.Context) {...})
		userGroup.POST("/login", func(c *gin.Context) {...})
 
	}
	shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
	}
	r.Run()
}
```

路由组也是支持嵌套的，例如：

```go
shopGroup := r.Group("/shop")
	{
		shopGroup.GET("/index", func(c *gin.Context) {...})
		shopGroup.GET("/cart", func(c *gin.Context) {...})
		shopGroup.POST("/checkout", func(c *gin.Context) {...})
		// 嵌套路由组
		xx := shopGroup.Group("xx")
		xx.GET("/oo", func(c *gin.Context) {...})
	}
```

通常我们将路由分组用在划分业务逻辑或划分 API 版本时。

#### <font color=red>路由原理</font>

Gin 框架中的路由使用的是 [httprouter](https://github.com/julienschmidt/httprouter) 这个库。

其基本原理就是构造一个路由地址的前缀树。

### <center>Gin中间件</center>

Gin 框架允许开发者在处理请求的过程中，加入用户自己的钩子（Hook）函数。这个钩子函数就叫中间件，中间件适合处理一些公共的业务逻辑，比如登陆认证、权限校验、数据分页、记录日志、耗时统计等。

#### <font color=red>定义中间件</font>

Gin 中的中间件必须是一个 gin.HandlerFunc 类型。例如我们像下面的代码一样定义一个统计请求耗时的中间件。

```go
// StatCost 是一个统计耗时请求耗时的中间件
func StatCost() gin.HandlerFunc {
        // 之所以把中间件写成闭包的形式，是因为这里可以做一些连接数据库等操作。
	return func(c *gin.Context) {
		start := time.Now()
		c.Set("name", "ty") // 可以通过c.Set在请求上下文中设置值，后续的处理函数能够取到该值
		// 调用该请求的剩余处理程序
		c.Next()
		// 不调用该请求的剩余处理程序
		// c.Abort()
		// 计算耗时
		cost := time.Since(start)
		log.Println(cost)
	}
}
```

**注意**：如果在一个中间件中没有调用 c.Abort () 函数，那么中间件函数结束后会执行它后面的函数，用了 return 也不管用（return 结束的是中间件函数本身）。

#### <font color=red>注册中间件</font>

在 gin 框架中，我们可以为每个路由添加任意数量的中间件。

##### <font color=red>为全局路由注册</font>

```go
func main() {
	// 新建一个没有任何默认中间件的路由
	r := gin.New()
	// 注册一个全局中间件
	r.Use(StatCost())  // 可传多个
	
	r.GET("/test", func(c *gin.Context) {
		name, ok := c.Get("name") // 从上下文取值
        if !ok {
            name = "匿名用户"
        }
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "Hello world!",
		})
	})
	r.Run()
}
```

##### <font color=red>为某个路由单独创建</font>

```go
// 给/test2路由单独注册中间件（可注册多个）
	r.GET("/test2", StatCost(), func(c *gin.Context) {
		name, ok := c.Get("name") // 从上下文取值
        if !ok {
            name = "匿名用户"
        }
		log.Println(name)
		c.JSON(http.StatusOK, gin.H{
			"message": "Hello world!",
		})
	})
```

##### <font color=red>为路由组注册中间件</font>

###### <font color=red>写法1:</font>

```go
shopGroup := r.Group("/shop", StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

###### <font color=red>写法2:</font>

```go
shopGroup := r.Group("/shop")
shopGroup.Use(StatCost())
{
    shopGroup.GET("/index", func(c *gin.Context) {...})
    ...
}
```

还可以为中间件手动设置一个开关，举个例子，我们做一个用户登陆认证的中间件：

```go
package main
 
import (
	"fmt"
	"net/http"
 
	"github.com/gin-gonic/gin"
)
 
// 用户登陆中间件
func authMiddleware(swit bool) gin.HandlerFunc {
	fmt.Println("连接数据库...")
	return func(c *gin.Context) {
		if swit {
			username := c.Query("username")
			password := c.Query("password")
			if username == "ty" && password == "123" {
				c.Next()
			} else {
				c.JSON(http.StatusNotFound, gin.H{
					"msg": "用户名或密码错误!",
				})
				c.Abort()
			}
		} else {
			c.Next()
		}
	}
}
 
func main() {
	r := gin.Default()
 
	r.Use(authMiddleware(true))
 
	r.GET("/index", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"msg": "index"})
	})
 
	r.GET("/video", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"msg": "video"})
	})
 
	r.Run() // 默认开8080端口
}
```

#### <font color=red>中间件注意事项</font>

##### <font color=red>gin默认中间件</font>

gin.Default () 默认使用了 Logger 和 Recovery 中间件，其中：

- Logger 中间件将日志写入 gin.DefaultWriter，即使配置了 GIN_MODE=release。
- Recovery 中间件会 recover 任何 panic。如果有 panic 的话，会写入 500 响应码。

如果不想使用上面两个默认的中间件，可以使用 gin.New () 新建一个没有任何默认中间件的路由。

##### <font color=red>gin中间件中使用goroutine</font>

当在中间件或 handler 中启动新的 goroutine 时，**不能使用**原始的上下文（c *gin.Context），必须使用其只读副本（c.Copy ()）例如 go Write (c.Copy ())。

### <center>运行多个服务</center>

我们可以在多个端口启动服务，例如：

```go
package main
 
import (
	"log"
	"net/http"
	"time"
 
	"github.com/gin-gonic/gin"
	"golang.org/x/sync/errgroup"
)
 
var (
	g errgroup.Group
)
 
func router01() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 01",
			},
		)
	})
 
	return e
}
 
func router02() http.Handler {
	e := gin.New()
	e.Use(gin.Recovery())
	e.GET("/", func(c *gin.Context) {
		c.JSON(
			http.StatusOK,
			gin.H{
				"code":  http.StatusOK,
				"error": "Welcome server 02",
			},
		)
	})
 
	return e
}
 
func main() {
	server01 := &http.Server{
		Addr:         ":8080",
		Handler:      router01(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
 
	server02 := &http.Server{
		Addr:         ":8081",
		Handler:      router02(),
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 10 * time.Second,
	}
   // 借助errgroup.Group或者自行开启两个goroutine分别启动两个服务
	g.Go(func() error {
		return server01.ListenAndServe()
	})
 
	g.Go(func() error {
		return server02.ListenAndServe()
	})
 
	if err := g.Wait(); err != nil {
		log.Fatal(err)
	}
}
```

