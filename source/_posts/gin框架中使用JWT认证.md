---
title: gin框架中使用JWT认证
date: 2021-12-21 11:00
tags:
    - gin
---

JWT 全称 JSON Web Token，是一种跨域认证解决方案，属于一个开放的标准，它规定了一种 Token 实现方式，目前多用于前后端分离项目和 OAuth2.0 业务场景下。

### <center>为什么需要JWT?</center>

在之前的一些 Web 项目中，我们通常使用的是 Cookies-Session 模式实现用户认证。相关流程大致如下：

- 用户在浏览器填写用户名和密码，并发送给服务端。
- 服务端对用户名和密码校验通过后会生成一份保存当前用户相关信息的 session 数据和一个与之对应的标识（通常称为 session_id）。
- 服务端返回响应时将上一步的 session_id 写入用户浏览器的 Cookie。
- 后续用户来自该浏览器的每次请求都会自动携带包含 session_id 的 Cookie。
- 服务端通过请求中的 session_id 就能找到之前保存的该用户那份 session 数据，从而获取该用户的相关信息。

这种方案依赖于客户端（浏览器）保存 Cookie，并且需要在服务端存储用户的 session 数据。在移动互联网时代，我们的用户可能使用浏览器也可能使用 APP 来访问我们的服务，我们的 Web 应用可能是前后端分开部署在不同的端口，有时候我们还需要支持第三方登录，这下 Cookie-Session 的模式就有些力不从心了。

JWT 就是一种基于 Token 的轻量级认证模式，服务端认证通过后，会生成一个 JSON 对象，经过签名后得到一个 Token（令牌）再发回给用户，用户后续请求只需要带上这个 Token，服务端解密之后就能获取该用户的相关信息了。

### <center>生成JWT与解析JWT</center>

直接使用`jwt-go`实现我们生成`JWT`与解析`JWT`的功能。

#### <font color=red>定义需求</font>

我们需要定制自己的需求来决定 JWT 中保存哪些数据，比如我们规定在 JWT 中要存储 username 信息，那么我们就定义一个 MyClaims 结构体如下：

```go
// MyClaims 自定义声明结构体并内嵌jwt.StandardClaims
// jwt包自带的jwt.StandardClaims只包含了官方字段
// 我们这里需要额外记录一个username字段，所以要自定义结构体
// 如果想要保存更多信息，都可以添加到这个结构体中
type MyClaims struct {
	Username string `json:"username"`
	jwt.StandardClaims
}
```

然后我们定义 JWT 的过期时间，这里以 2 小时为例：

```go
const TokenExpireDuration = time.Hour * 2
```

接下来还需要定义 Secret：

```go
var MySecret = []byte("用来加密数据")
```

#### <font color=red>生成JWT</font>

```go
// GenToken 生成JWT
func GenToken(username string) (string, error) {
	// 创建一个我们自己的声明
	c := MyClaims{
		"username", // 自定义字段
		jwt.StandardClaims{
			ExpiresAt: time.Now().Add(TokenExpireDuration).Unix(), // 过期时间
			Issuer:    "my-project",                               // 签发人
		},
	}
	// 使用指定的签名方法创建签名对象
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, c)
	// 使用指定的secret签名并获得完整的编码后的字符串token
	return token.SignedString(MySecret)
}
```

#### <font color=red>解析JWT</font>

```go
// ParseToken 解析JWT
func ParseToken(tokenString string) (*MyClaims, error) {
	// 解析token
	token, err := jwt.ParseWithClaims(tokenString, &MyClaims{}, func(token *jwt.Token) (i interface{}, err error) {
		return MySecret, nil
	})
	if err != nil {
		return nil, err
	}
	if claims, ok := token.Claims.(*MyClaims); ok && token.Valid { // 校验token
		return claims, nil
	}
	return nil, errors.New("invalid token")
}
```

### <center>Gin框架中使用JWT</center>

首先我们注册一条路由 /auth，对外提供获取 Token 的渠道：

```go
r.POST("/auth", authHandler)
```

authHandler 定义如下：

```go
func authHandler(c *gin.Context) {
	// 用户发送用户名和密码过来
	var user UserInfo
	err := c.ShouldBind(&user)
	if err != nil {
		c.JSON(http.StatusOK, gin.H{
			"code": 2001,
			"msg":  "无效的参数",
		})
		return
	}
	// 校验用户名和密码是否正确
	if user.Username == "Solost23" && user.Password == "123" {
		// 生成Token
		tokenString, _ := GenToken(user.Username)
		c.JSON(http.StatusOK, gin.H{
			"code": 2000,
			"msg":  "success",
			"data": gin.H{"token": tokenString},
		})
		return
	}
	c.JSON(http.StatusOK, gin.H{
		"code": 2002,
		"msg":  "鉴权失败",
	})
	return
}
```

用户通过上面的接口获取 Token 之后，后续就会携带这 Token 再来请求我们的其他接口，这个时候就需要对这些请求的 Token 进行校验操作了，很显然我们应该实现一个检验 Token 的中间件，具体实现如下：

```go
// JWTAuthMiddleware 基于JWT的认证中间件
func JWTAuthMiddleware() func(c *gin.Context) {
	return func(c *gin.Context) {
		// 客户端携带Token有三种方式 1.放在请求头 2.放在请求体 3.放在URI
		// 这里假设Token放在Header的Authorization中，并使用Bearer开头
		// 这里的具体实现方式要依据你的实际业务情况决定
		authHeader := c.Request.Header.Get("Authorization")
		if authHeader == "" {
			c.JSON(http.StatusOK, gin.H{
				"code": 2003,
				"msg":  "请求头中auth为空",
			})
			c.Abort()
			return
		}
		// 按空格分割
		parts := strings.SplitN(authHeader, " ", 2)
		if !(len(parts) == 2 && parts[0] == "Bearer") {
			c.JSON(http.StatusOK, gin.H{
				"code": 2004,
				"msg":  "请求头中auth格式有误",
			})
			c.Abort()
			return
		}
		// parts[1]是获取到的tokenString，我们使用之前定义好的解析JWT的函数来解析它
		mc, err := ParseToken(parts[1])
		if err != nil {
			c.JSON(http.StatusOK, gin.H{
				"code": 2005,
				"msg":  "无效的Token",
			})
			c.Abort()
			return
		}
		// 将当前请求的username信息保存到请求的上下文c上
		c.Set("username", mc.Username)
		c.Next() // 后续的处理函数可以用过c.Get("username")来获取当前请求的用户信息
	}
}
```

注册一个 /index 路由，发个请求验证一下吧！

```go
r.GET("/index", JWTAuthMiddleware(), indexHandler)
 
func indexHandler(c *gin.Context) {
	username := c.MustGet("username").(string)
	c.JSON(http.StatusOK, gin.H{
		"code": 2000,
		"msg":  "success",
		"data": gin.H{"username": username},
	})
}
```

如果不想自己实现上述功能，也可以使用 Github 上别人封装好的包，比如：https://github.com/appleboy/gin-jwt。