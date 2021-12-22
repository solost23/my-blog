---
title: golang pprof性能调优 
---

在计算机性能调试领域里，profiling 是指对应用程序的画像，画像就是应用程序使用 CPU 和内存的情况。Go 语言是一个对性能特别看重的语言，因此语言中自带了 profiling 的库，这篇文章就要讲解怎么在 golang 中做 profiling。

### <center>Golang性能调优</center>

Go 语言项目中的性能优化主要有以下几个方面：

- CPU profile：报告程序的 CPU 使用情况，按照一定频率去采集应用程序在 CPU 和寄存器上面的数据。
- Memory Profile（Heap Profile）：报告程序的内存使用情况。
- Block Profiling：报告 goroutine 不在运行状态的情况，可以用来分析和查找死锁等性能瓶颈。
- Groutine Profiling：报告 goroutines 的使用情况，有哪些 goroutine，它们的调用关系是怎样的。

### <center>采集性能数据</center>

Go 语言内置了获取程序的运行数据的工具，包括以下两个标准库：

- runtime/pprof：采集工具型应用运行数据进行分析。
- net/http/pprof：采集服务型应用运行时数据进行分析。

pprof 开启后，每隔一段时间（10ms）就会收集下当前的堆栈信息，获取各个函数占用的 CPU 以及内存资源；最后通过对这些采样数据进行分析，形成一个性能分析报告。

<font color=red>注意</font>:我们只应该在性能测试的时候才在代码中引入`pprof`。

### <center>工具型应用</center>

如果你的应用程序是运行一段时间就结束退出类型。那么最好的办法是在应用退出的时候把 profiling 的报告保存到文件中，进行分析。对于这种情况，可以使用 runtime/pprof 库。首先在代码中导入 runtime/pprof 工具：

```go
import "runtime/pprof"
```

#### <font color=red>CPU性能分析</font>

开启 CPU 性能分析：

```go
pprof.StartCPUProfile(w io.Writer)
```

停止 CPU 性能分析：

```go
pprof.StopCPUProfile()
```

应用执行结束后，就会生成一个文件，保存了我们的 CPU profiling 数据。得到采样数据之后，使用 go tool pprof 工具进行 CPU 性能分析。

#### <font color=red>内存性能优化</font>

记录程序的堆栈信息

```go
pprof.WriteHeapProfile(w io.Writer)
```

得到采样数据之后，使用 go tool pprof 工具进行内存性能分析。go tool pprof 默认是使用 -inuse_space 进行统计，还可以使用 - inuse-objects 查看分配对象的数量。

### <center>服务型应用</center>

如果你的程序是一直运行的，比如 web 应用，那么可以使用 net/http/pprof 库，它能够在提供 HTTP 服务进行分析。如果使用了默认的 http.DefaultServeMux（通常是代码直接使用 http.ListenAndServer ("0.0.0.0:8080", nil)），只需要在你的 web server 端代码中按如下方式导入 net/http/pprof:

```go
import _ "net/http/pprof"
```

如果你使用自己定义的 Mux，则需要手动注册一些路由规则：

```go
r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```

如果你使用的是 gin 框架，那么推荐使用 [github.com/gin-contrib/pprof](https://github.com/gin-contrib/pprof)，在代码中通过以下命令注册 pprof 相关路由：

```go
pprof.Register(router)
```

不管哪种方式，你的 HTTP 服务都会多出 /debug/pprof endpoint，访问它会得到类似下面内容：

![](images/pprof2.png)

这个路径下还有几个子页面：

- /debug/pprof/profile：访问这个链接会自动进行 CPU profiling，持续 30s，并生成一个文件供下载
- /debug/pprof/heap： Memory Profiling 的路径，访问这个链接会得到一个内存 Profiling 结果的文件
- /debug/pprof/block：block Profiling 的路径
- /debug/pprof/goroutines：运行的 goroutines 列表，以及调用关系

### <center>go tool pprof命令</center>

不管是工具型应用还是服务型应用，我们使用相应的 pprof 库获取数据之后，下一步都要对这些数据进行分析，我们可以使用 go tool pprof 命令行工具。go tool pprof 最简单的使用方式为：

```go
go tool pprof [binary] [source]
```

其中：

- binary 是应用的二进制文件，用来解析各种符号；
- source 表示 profile 数据的来源，可以是本地的文件，也可以是 http 地址。

<font color=red>注意</font>:获取的 Profiling 数据是动态的，要想获得有效的数据，请保证应用处于较大的负载（比如正在生成中运行的服务，或者通过其他工具模拟访问压力）。否则如果应用处于空闲状态，得到的结果可能没有任何意义。

### <center>具体示例</center>

我们先来写一段有问题的代码：

```go
// runtime_pprof/main.go
package main
 
import (
	"flag"
	"fmt"
	"os"
	"runtime/pprof"
	"time"
)
 
// 一段有问题的代码
func logicCode() {
	var c chan int
	for {
		select {
		case v := <-c:
			fmt.Printf("recv from chan, value:%v\n", v)
		default:
 
		}
	}
}
 
func main() {
	var isCPUPprof bool
	var isMemPprof bool
 
	flag.BoolVar(&isCPUPprof, "cpu", false, "turn cpu pprof on")
	flag.BoolVar(&isMemPprof, "mem", false, "turn mem pprof on")
	flag.Parse()
 
	if isCPUPprof {
		file, err := os.Create("./cpu.pprof")
		if err != nil {
			fmt.Printf("create cpu pprof failed, err:%v\n", err)
			return
		}
		pprof.StartCPUProfile(file)
		defer pprof.StopCPUProfile()
	}
	for i := 0; i < 8; i++ {
		go logicCode()
	}
	time.Sleep(20 * time.Second)
	if isMemPprof {
		file, err := os.Create("./mem.pprof")
		if err != nil {
			fmt.Printf("create mem pprof failed, err:%v\n", err)
			return
		}
		pprof.WriteHeapProfile(file)
		file.Close()
	}
}
```

通过 flag 我们可以在命令行控制是否开启 CPU 和 Mem 的性能分析。将上面的代码保存并编译成 runtime_pprof 可执行文件，执行时加上 - cpu=true 命令行参数，如下：

```go
C:\Users\ASUS\go\src\calc\day09\runtime_pprof>runtime_pprof.exe -cpu=true
```

等待 30 秒会在当前目录下生成一个 cpu.pprof 文件。

#### <font color=red>命令行交互界面</font>

我们使用 go 工具链里的 pprof 来分析一下：

```go
go tool pprof cpu.pprof
```

执行上面的代码会进入交互界面：

```go
C:\Users\ASUS\go\src\calc\day09\runtime_pprof>go tool pprof cpu.pprof
Type: cpu
Time: Mar 20, 2021 at 9:47pm (CST)
Duration: 20.29s, Total samples = 2.54mins (749.71%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

我们可以在交互界面输入 top3 来查看程序中占用 CPU 前 3 位的函数：

```go
(pprof) top3
Showing nodes accounting for 151.15s, 99.36% of 152.13s total   
Dropped 60 nodes (cum <= 0.76s)
      flat  flat%   sum%        cum   cum%
    70.38s 46.26% 46.26%    122.68s 80.64%  runtime.selectnbrecv
    52.25s 34.35% 80.61%     52.27s 34.36%  runtime.chanrecv    
    28.52s 18.75% 99.36%    151.27s 99.43%  main.logicCode      
(pprof)
```

其中：

- flat：当前函数占用 CPU 的耗时
- flat：: 当前函数占用 CPU 的耗时百分比
- sun%：函数占用 CPU 的耗时累计百分比
- cum：当前函数加上调用当前函数的函数占用 CPU 的总耗时
- cum%：当前函数加上调用当前函数的函数占用 CPU 的总耗时百分比
- 最后一列：函数名称

在大多数的情况下，我们可以通过分析这五列得出一个应用程序的运行情况，并对程序进行优化。我们还可以使用 `list 函数名`命令查看具体的函数分析，例如执行 `list logicCode` 查看我们编写的函数的详细分析。

```go
(pprof) list logicCode
Total: 2.54mins
ROUTINE ======================== main.logicCode in C:\Users\ASUS\go\src\calc\day09\runtime_pprof\main.go
    28.52s   2.52mins (flat, cum) 99.43% of Total
         .          .     12:// 一段有问题的代码
         .          .     13:func logicCode() {
         .          .     14:   var c chan int
         .          .     15:   for {
         .          .     16:           select {
    28.52s   2.52mins     17:           case v := <-c:
         .          .     18:                   fmt.Printf("recv from chan, value:%v\n", v)
         .          .     19:           default:
         .          .     20:
         .          .     21:           }
         .          .     22:   }
```

通过分析发现大部分 CPU 资源被 17 行占用，我们分析出 select 语句中的 default 没有内容会导致上面的 case v := <-c: 一直执行。我们在 default 分支添加一行 time.Sleep (time.Second) 即可。

#### <font color=red>图形化</font>

或者可以直接输入 web，通过 svg 图的方式查看程序中详细的 CPU 占用情况。想要查看图形化的界面首先需要安装 [graphviz](https://graphviz.gitlab.io/) 图形化工具。Mac:

```shell
brew install graphviz
```

Windows：下载 graphviz，将 graphviz 安装目录下的 bin 文件夹添加到 Path 环境变量中。在终端输入 dot -version 查看是否安装成功。使用 web 命令用浏览器查看：

```shell
C:\Users\ASUS\go\src\calc\day09\runtime_pprof>go tool pprof cpu.pprof
Type: cpu
Time: Mar 20, 2021 at 9:47pm (CST)
Duration: 20.29s, Total samples = 2.54mins (749.71%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) web
```

![](images/屏幕截图-2021-03-21-112348-1024x711.png)

关于图形的说明： 每个框代表一个函数，理论上框的越大表示占用的 CPU 资源越多。 方框之间的线条代表函数之间的调用关系。 线条上的数字表示函数调用的次数。 方框中的第一行数字表示当前函数占用 CPU 的百分比，第二行数字表示当前函数累计占用 CPU 的百分比。除了分析 CPU 性能数据，pprof 也支持分析内存性能数据。比如，使用下面的命令分析 http 服务的 heap 性能数据，查看当前程序的内存占用以及热点内存对象使用的情况。

```shell
# 查看内存占用数据
go tool pprof -inuse_space http://127.0.0.1:8080/debug/pprof/heap
go tool pprof -inuse_objects http://127.0.0.1:8080/debug/pprof/heap
# 查看临时内存分配数据
go tool pprof -alloc_space http://127.0.0.1:8080/debug/pprof/heap
go tool pprof -alloc_objects http://127.0.0.1:8080/debug/pprof/heap
```

### <center>go-torch与火焰图</center>

待续...

