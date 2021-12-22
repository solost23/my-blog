---
title: 使用Air实现golang程序实时热重载
---

介绍一个神器 ——Air，Air 能够实时监听项目的代码文件，在代码发生变更之后自动重新编译并执行，大大提高 gin 框架项目的开发效率。

### <center>为什么需要实时加载</center>
使用`Python`编写`Web`项目的时候，常见的`Flask`和`Django`都支持实时加载，修改项目代码之后，程序能够自动重新加载并执行(live_reload), 这在日常开发阶段是十分方便的。在使用`Golang`的`Gin`框架做本地开发调试的时候经常需要在变更代码后频繁地按下`Ctrl + C`停止程序并重新编译，这样不是很方便。
### <center>Air介绍</center>
Air支持以下特性:
- 彩色日志输出
- 自定义构建成二进制命令
- 支持忽略子目录
- 启动后支持监听新目录
- 更好的构建过程
### <center>Air安装</center>
#### <font color=red>Golang</font>
```go
go get -u github.com/cosmtrek/air
```
#### <font color=red>MacOS</font>
```shell
curl -fLo air https://git.io/darwin_air
```
#### <font color=red>Linux</font>
```shell
curl -fLo air https://git.io/linux_air
```
#### <font color=red>Windows</font>
```go
curl -fLo air.exe https://git.io/windows_air
```
#### <font color=red>Docker</font>
```shell
docker run -it --rm \
    -w "<PROJECT>" \
    -e "air_wd=<PROJECT>" \
    -v $(pwd):<PROJECT> \
    -p <PORT>:<APP SERVER PORT> \
    cosmtrek/air
    -c <CONF>
```
然后按照下面的方式在`Docker`中运行你的项目
```shell
docker run -it --rm \
    -w "/go/src/github.com/cosmtrek/hub" \
    -v $(pwd):/go/src/github.com/cosmtrek/hub \
    -p 9090:9090 \
    cosmtrek/air
```
### <center>Air使用</center>
为了敲命令时更简单方便，你应该把 alias air='~/.air' 加到你的.bashrc 或.zshrc 中，Windows 不需要建立软链就可以在任何目录下执行。首先进入你的项目目录：
```shell
cd /path/to/your_project
```
最简单的用法就是直接执行下面的命令:
```shell
# 首先在当前目录下查找 `.air.conf`配置文件，如果找不到就使用默认的,windows下找不到文件不会运行程序，直接执行air就可以。
air -c .air.conf
```
推荐的使用方法是:
```shell
# 1. 在当前目录创建一个新的配置文件.air.conf
touch .air.conf  // windows使用鼠标创建
 
# 2. 复制 `air_conf.example` 中的内容到这个文件，然后根据你的需要去修改它
 
# 3. 使用你的配置运行 air, 如果文件名是 `.air.conf`，只需要执行 `air`。
air
```
#### <font color=red>air_example.conf示例</font>
完整的`air_example.conf`示例配置如下，可以根据自己的需要修改
```tex
# [Air](https://github.com/cosmtrek/air) TOML 格式的配置文件
 
# 工作目录
# 使用 . 或绝对路径，请注意 `tmp_dir` 目录必须在 `root` 目录下
root = "."
tmp_dir = "tmp"
 
[build]
# 只需要写你平常编译使用的shell命令。你也可以使用 `make`
# Windows平台示例: cmd = "go build -o tmp\main.exe ."
cmd = "go build -o ./tmp/main ."
# 由`cmd`命令得到的二进制文件名
# Windows平台示例：bin = "tmp\main.exe"
bin = "tmp/main"
# 自定义执行程序的命令，可以添加额外的编译标识例如添加 GIN_MODE=release
# Windows平台示例：full_bin = "tmp\main.exe"
full_bin = "APP_ENV=dev APP_USER=air ./tmp/main"
# 监听以下文件扩展名的文件.
include_ext = ["go", "tpl", "tmpl", "html"]
# 忽略这些文件扩展名或目录
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间
delay = 1000 # ms
# 发生构建错误时，停止运行旧的二进制文件。
stop_on_error = true
# air的日志文件名，该日志文件放置在你的`tmp_dir`中
log = "air_errors.log"
 
[log]
# 显示日志时间
time = true
 
[color]
# 自定义每个部分显示的颜色。如果找不到颜色，使用原始的应用程序日志。
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"
 
[misc]
# 退出时删除tmp目录
clean_on_exit = true
```
