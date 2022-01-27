---
title: 使用Docker部署Go Web程序
date: 2021-12-22 11:00
tags:
    - golang
    - docker
---

### <center>Docker部署示例</center>

#### <font color=red>`Docker`安装</font>

[MacOS Docker安装](https://www.runoob.com/docker/macos-docker-install.html)

#### <font color=red>准备代码</font>

```go
package main
 
import (
	"fmt"
	"net/http"
)
 
func Hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World!"))
}
 
func main() {
	http.HandleFunc("/", Hello)
	fmt.Println("server start...")
	if err := http.ListenAndServe(":8080", nil); err != nil {
		fmt.Printf("server start failed, err:%v \n", err)
	}
}
```

<font color=red>注意</font>:项目要完整，下面需要有`go.mod`文件等，只写一个`main.go`容器中无法编译。

#### <font color=red>创建`Docker`镜像</font>

##### <font color=red>编写`Dockerfile`</font>

要创建`Docker`镜像必须在配置文件中指定步骤，这个文件我们通常称之为`Dockerfile`，虽然这个文件名可以随意命名，但最好还是使用默认的`Dockerfile`。

```tex
FROM golang:alpine
 
# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
 
# 移动到工作目录：/build
WORKDIR /build
 
# 将代码复制到容器中
COPY . .
 
# 将我们的代码编译成二进制可执行文件app
RUN go build -o app .
 
# 移动到用于存放生成的二进制文件的 /dist 目录
WORKDIR /dist
 
# 将二进制文件从 /build 目录复制到这里
RUN cp /build/app .
 
# 声明服务端口
EXPOSE 8080
 
# 启动容器时运行的命令
CMD ["/dist/app"]
```

#### <font color=red>构建镜像</font>

在项目目录下打开`shell`，执行下面的命令创建镜像，并制定镜像名称为`goWebApp`

```shell
docker build . -t goWebApp
```

执行下面的命令来运行镜像:

```shell
docker run -p 8080:8080 goweb_app
```

现在就可以测试一下我们的`Web`是否正常工作，打开浏览器输入`http://localhost:8080`就可以能看到我们事先定义的相应内容。

### <center>分阶段构建示例</center>

我们的`Web`程序编译后会得到一个可执行文件，其实在最终的镜像中是不需要`go`编译器的，也就是说我们只需要一个运行最终二进制文件的容器即可。`Docker`的最佳实践之一是通过仅保留二进制文件来减小镜像大小，为此我们将使用一种称为多阶段构建的技术，这意味着我们将通过多个步骤构建镜像:

```tex
FROM golang:alpine AS builder
# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
 
# 移动到工作目录：/build
WORKDIR /build
 
# 将代码复制到容器中
COPY . .
 
# 将我们的代码编译成二进制可执行文件 app
RUN go build -o app .
 
###################
# 接下来创建一个小镜像
###################
FROM scratch
 
# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/app /
 
# 需要运行的命令
ENTRYPOINT ["/app"]
```

使用这种技术，我们剥离了使用`golang:alpine`作为编译镜像来编译得到二进制可执行文件的过程，并基于`scratch`镜像的更多信息，请查看:[https://hub.docker.com/_/scratch](https://hub.docker.com/_/scratch)。

### <center>附带其它文件的部署示例</center>

这里以`go Web`中的小清单项目为例，项目的`github`仓库地址为:[https://github.com/Solost23/bubble](https://github.com/Solost23/bubble), 如果项目中带有静态文件或配置文件，需要将其拷贝到最终的镜像中，`bubble`项目用到了静态文件和配置文件，具体目录结构如下:

```tex
bubble
├── README.md
├── controller
│   └── controller.go
├── dao
│   └── mysql.go
├── go.mod
├── go.sum
├── main.go
├── models
│   └── todo.go
├── routers
│   └── routers.go
├── static
│   ├── css
│   │   ├── app.8eeeaf31.css
│   │   └── chunk-vendors.57db8905.css
│   ├── fonts
│   │   ├── element-icons.535877f5.woff
│   │   └── element-icons.732389de.ttf
│   └── js
│       ├── app.007f9690.js
│       └── chunk-vendors.ddcb6f91.js
└── templates
    ├── favicon.ico
    └── index.html
```

我们需要将`templates`、`static`两个文件夹中的内容拷贝到最终的镜像中。更新后的`Dockerfile`文件内容如下:

```tex
FROM golang:alpine AS builder
 
# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
 
 
# 移动到工作目录：/build
WORKDIR /build
 
# 复制项目中的 go.mod 和 go.sum文件并下载依赖信息
 
# 设置代理，防止后面因为墙的原因下载go.mod依赖信息超时
RUN go env -w GOPROXY=https://goproxy.io
COPY go.mod .
COPY go.sum .
RUN go mod download
 
# 将代码复制到容器中
COPY . .
 
# 将我们的代码编译成二进制可执行文件 bubble
RUN go build -o bubble .
 
###################
# 接下来创建一个小镜像
###################
FROM scratch
 
COPY ./templates /templates
COPY ./static /static
 
# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/bubble /
 
# 需要运行的命令
ENTRYPOINT ["/bubble"]
```

<font color=red>注意</font>:这里把`COPY静态文件`放到了上层，把`COPY二进制可执行文件`放在下层，争取多使用缓存。

#### <font color=red>关联其它容器</font>

因为项目中使用了`MySQL`, 我们可以使用如下命令启动一个`MySQL`容器，它的别名为`oneMysql`; `root`用户的密码为`1234`; 内部服务端口为3306, 映射到外部的`13306`端口。

```shell
docker run --name oneMysql -p 13306:3306 -e MYSQL_ROOT_PASSWORD=1234 -d mysql:5.7
```

#### <font color=red>创建`bubble`数据库的两种方式</font>

当本机有`MySQL`客户端的时候，进入`shell`, 然后连接`MySQL`创建`bubble`。

```shell
# CMD
mysql -P 13306 -uroot -p1234
 
# 创建数据库（不指定utf8中文无法存入）
CREATE DATABASE bubble CHARACTER SET utf8;
```

本地没有`MySQL`客户端，进入`shell`，然后进入容器`oneMysql`，进入`mysql`，创建`bubble`:
```shell
# 进入 oneMysql容器
docker exec -it oneMysql bash
 
# oneMysql终端，因为端口为3306所以可以省略
mysql -uroot -p1234
 
# 创建数据库bubble
CREATE DATABASE bubble CHARACTER SET utf8;
```

这里需要修改一下在`dao/mysql.go`的`dsn`变量中配置数据库，使它们在内部通过别名（此处为 oneMysql）联通。

```go
dsn := "root:1234@(oneMysql:3306)/bubble"
```

修改后记得重新构建`bubble_app`镜像：
```go
docker build . -t bubble_app
```

这里运行`bubble_app`容器的时候需要使用`--link`的方式与上面的`oneMysql`容器关联起来，并将容器中`go Web`程序使用的端口映射到宿主机端口上，具体命令如下：

```shell
docker run --link=oneMysql:oneMysql -p 8888:8080 bubble_app
```

### <center>`DockerCompose`模式</center>

除了像上面一样使用`--link`的方式来关联两个容器之外，还可以使用`Docker Compose`来定义和运行多个容器。

`Docker Compose`是用于定义和运行多容器`Docker`应用程序的工具。通过`Compose`，你可以使用 `YML`文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从`YML`文件配置中创建并启动所有服务。

使用`Docker Compose`基本上有三步:
- 使用`Dockerfile`定义你的应用环境以便可以在任何地方复制。
- 定义组成应用程序的服务, `docker-compose.yml`以便它们可以在隔离的环境中一起运行。
- 执行`docker-compose up`命令来启动并运行整个程序。

我们的项目需要两个容器分别运行`mysql`和`bubble_app`, 我们编写的`docker-compose.yml`文件内容如下:

```tex
# yaml 配置
version: "3.7"
services:
  mysql8019:
    image: "mysql:8.0.19"
    ports:
      - "33061:3306"
    command: "--default-authentication-plugin=mysql_native_password --init-file /data/application/init.sql"
    environment:
      MYSQL_ROOT_PASSWORD: "root1234"
      MYSQL_DATABASE: "bubble"
      MYSQL_PASSWORD: "root1234"
    volumes:
      - ./init.sql:/data/application/init.sql
  bubble_app:
    build: .
    command: sh -c "./wait-for.sh mysql8019:3306 -- ./bubble ./conf/config.ini"
    depends_on:
      - mysql8019
    ports:
      - "8888:8888"
```

这个`Docker Compose`文件定义了两个服务: `bubble_app`和`mysql8019`, 其中:

#### <font color=red>bubble_app</font>

使用当前目录下的`Dockerfile`文件构建镜像，并通过`depends_on`指定依赖`mysql8019` 服务，声明服务端口`8888`并绑定对外`8888`端口。

#### <font color=red>mysql8019</font>

mysql8019 服务使用 Docker Hub 的公共 mysql:8.0.19 镜像，内部端口 3306，外部端口 33061。这里需要注意一个问题，我们的 bubble_app 容器需要等待 mysql8019 容器正常启动之后再尝试启动，因为我们的 web 程序在启动的时候会初始化 MySQL 连接，这里共有两个地方要更改，第一个就是我们 Dockerfile 中要把最后一句注释掉：

```tex
# Dockerfile
...
# 需要运行的命令（注释掉这一句，因为需要等MySQL启动之后再启动我们的Web程序）
# ENTRYPOINT ["/bubble", "conf/config.ini"]
```

第二个地方是在`bubble_app`下面添加如下命令，使用提前编写的`wait-for.sh`脚本检测`mysql8019:3306`正常后再执行后续启动`Web`应用程序的命令:

```shell
command: sh -c "./wait-for.sh mysql8019:3306 -- ./bubble ./conf/config.ini"
```

当然，因为我们现在要在 bubble_app 镜像中执行 sh 命令，所以不能使用 scrath 镜像构建了，这里改为使用 debian:stretch-slim，同时还要安装 wait-for.sh 脚本用到的 netcat，最后不要忘了把 wait-for.sh 脚本文件 COPY 到最终的镜像中，并赋予可执行权限。更新后的 Dockerfile 内容如下：

```tex
FROM golang:alpine AS builder
 
# 为我们的镜像设置必要的环境变量
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux \
    GOARCH=amd64
 
# 移动到工作目录：/build
WORKDIR /build
 
# 复制项目中的 go.mod 和 go.sum文件并下载依赖信息
COPY go.mod .
COPY go.sum .
RUN go mod download
 
# 将代码复制到容器中
COPY . .
 
# 将我们的代码编译成二进制可执行文件 bubble
RUN go build -o bubble .
 
###################
# 接下来创建一个小镜像
###################
FROM debian:stretch-slim
 
COPY ./wait-for.sh /
COPY ./templates /templates
COPY ./static /static
COPY ./conf /conf
 
 
# 从builder镜像中把/dist/app 拷贝到当前目录
COPY --from=builder /build/bubble /
 
RUN set -eux; \
	apt-get update; \
	apt-get install -y \
		--no-install-recommends \
		netcat; \
        chmod 755 wait-for.sh
 
# 需要运行的命令
# ENTRYPOINT ["/bubble", "conf/config.ini"]
```

所有的条件都准备就绪后，就可以执行下面的命令跑起来了：

```shell
docker-compose up
```

### <center>总结</center>

使用 Docker 容器能够极大简化我们在配置依赖环境方面的操作，但同时也对我们的技术储备提了更高的要求。对于 Docker 不管你熟悉或者不熟悉，技术发展的车轮都将滚滚向前。