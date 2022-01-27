---
title: golang基础之并发编程
date: 2021-12-13 11:00
tags:
    - golang
---

并发是编程里面一个非常重要的概念，`golang`在语言层面天生支持并发，这也是`golang`流行的一个非常重要的原因。

### <center>并发与并行</center>

- 并发：同一时间段执行多个任务，但多个任务同一时刻只有一个在执行。
- 并行：同一时刻执行多个任务。

`golang`中并发通过`goroutine`实现。`groutine`类似于线程，属于用户态的线程，成为<font color=red>协程</font>，我们可以根据需要创建成千上万个`goroutine`并发工作。`goroutine`是由`golang`的`runtime`调度完成，而线程是由操作系统调度完成。

`golang`还提供`channel`在多个`goroutine`间进行通信。`goroutine`和`channel`是`golang`秉承的`CSP`并发模式的重要实现基础。

### <center>goroutine</center>

在`java/c++`中要实现并发编程的时候，我们通常需要维护一个线程池，并且需要自己去包装一个又一个任务，同时需要自己去调度线程执行任务并维护上下文切换，这一切通常会损耗程序员大量的心智。那么能不能又一种机制，程序员只需要定义很多个任务，让系统去帮助我们把这些任务分配到`cpu`上实现并发执行呢？

`golang`中的`goroutine`就是这样一种机制，`goroutine`的概念类似于线程，但`goroutine`是由`golang`的`runtime`调度和管理的。`golang`程序会智能地将`goroutine`中的任务合理分配到每个`CPU`。<font color=red>`golang`之所以被称为现代化的编程语言，就是因为它在语言层面上已经内置了调度和上下文切换的机制</font>。

在`golang`中不需要自己写进程、线程、协程，你只有一种方法开启并发(`goroutine`)，当你需要让某个任务并发执行的时候，你只需要把这个任务包装成一个函数，开启一个`goroutine`去执行这个函数就可以了。

#### <font color=red>goroutine的使用</font>

在调用函数前加上`go`关键字。

##### <font color=red>单个goroutine启动</font>

```go
func f1() {
  fmt.Println("hello Goroutine")
}

func main() {
  // 启动另一个goroutine去执行hello函数
  go f1()
  fmt.Println("main goroutine done")
}
```

此时执行结果只打印了`main grouting done`这是因为程序在启动时，`go`程序会为`main`函数创建一个默认`goroutine`。当`main`函数返回的时候`goroutine`就结束了，所有在`main`函数中的启动的`goroutine`会一同结束。所以我们要想办法让`main`函数等一等`f1`，最简单的办法就是`time.Sleep`。

```go
func main() {
  go f1()
  fmt.Println("main goroutine done")
  time.Sleep(time.Second)
}
```

#### <font color=red>多个goroutine启动</font>

`goroutine`何时结束?

- `goroutine`对应的函数结束，`goroutine`就会结束。
- `main`函数执行完，由`main`创建的所有`goroutine`就都结束。

```go
import (
"sync"
"fmt"
)
var wg sync.WaitGroup
 
func f1(i int) {
	defer wg.Done() // goroutine结束就登记-1
	fmt.Println("Hello Goroutine!", i)
}

func main() {
	for i := 0; i < 10; i++ {
		wg.Add(1) // 启动一个goroutine就登记+1
		go f1(i)
	}
	wg.Wait() // 等待所有登记的goroutine都结束
}
```

多次执行上面的代码会发现每次打印的数字顺序都不一致。这是因为`10`个`goroutine`是并发执行的，而`goroutine`调度是随机的。

### <center>`goroutine`与`os`线程</center>

#### <font color=red>可增长的栈</font>

`os`线程一般都有固定的栈内存(通常2MB)，一个`goroutine`的栈在其生命周期开始只有很小的栈(典型情况2KB)，`goroutine`的栈不是固定的，他可以按需增大和缩小，`goroutine`栈大小限制可以达到`1GB`虽然极少会用到这么大。所以在`golang`中一次创建十万左右的`goroutine`也是可以的。

#### <font color=red>goroutine调度模型(面试)</font>

`GPM`是`golang`运行时层面的实现，是`golang`自己的一套调度系统。区别于操作系统调度`os线程`。

- `G` 很好理解，就是个 goroutine，里面除了存放本 goroutine 信息外 还有与所在 P 的绑定等信息。
- `P` 管理着一组 goroutine 队列，P 里面会存储当前 goroutine 运行的上下文环境（函数指针，堆栈地址及地址边界），P 会对自己管理的 goroutine 队列做一些调度（比如把占用 CPU 时间较长的 goroutine 暂停、运行后续的 goroutine 等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他 P 的队列里抢任务。
- `M（machine）`是 Go 运行时（runtime）对操作系统内核线程的虚拟， M 与内核线程一般是一一映射的关系， 一个 groutine 最终是要放到 M 上执行的。

`P`与`M`一般也是一一对应的。他们关系是： P 管理着一组 G 挂载在 M 上运行。当一个 G 长久阻塞在一个 M 上时，runtime 会新建一个 M，阻塞 G 所在的 P 会把其他的 G 挂载在新建的 M 上。当旧的 G 阻塞完成或者认为其已经死掉时 回收旧的 M。

P 的个数是通过 `runtime.GOMAXPROCS` 设定（最大 256），Go1.5 版本之后默认为物理线程数。 在并发量大的时候会增加一些 P 和 M，但不会太多，切换太频繁的话得不偿失。

单从线程调度讲，Go 语言相比起其他语言的优势在于其它语言的 OS 线程是由 OS 内核来调度的，`goroutine` 则是由 Go 运行时（runtime）自己的调度器调度的，这个调度器使用一个称为 m:n 调度的技术（复用 / 调度 m 个 goroutine 到 n 个 OS 线程）。 其一大特点是 goroutine 的调度是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的 malloc 函数（除非内存池需要改变），成本比调度 OS 线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干 goroutine 均分在物理线程上， 再加上本身 goroutine 的超轻量，以上种种保证了 go 调度方面的性能。

[点我了解更多](https://www.cnblogs.com/sunsky303/p/9705727.html)

#### <font color=red>GOMAXPROCS</font>

`golang`运行时的调度器使用`GOMAXPROCS`参数来确定需要使用多少个`os线程`来同时执行`golang`代码。<font color=red>默认值是机器上的CPU核心数</font>。调度器会把`golang`代码同时调度到`8`个`os线程`上(GOMAXPROCS是m:n调度中的n)。`golang`可以通过`runtime.GOMAXPROCS()`函数设置当前程序并发时占用的`cpu`逻辑核心。eg:

```go
func a() {
	for i := 1; i < 10; i++ {
		fmt.Println("A:", i)
	}
}
 
func b() {
	for i := 1; i < 10; i++ {
		fmt.Println("B:", i)
	}
}
 
func main() {
	runtime.GOMAXPROCS(1)
	go a()
	go b()
	time.Sleep(time.Second)
}
```

`golang`中`os线程`和`goroutine`的关系：

- 一个`os线程`对应用户态多个`goroutine`。
- `golang`可以同时使用多个`os线程`。
- `goroutine`和`OS线程`是多对多的关系，即`m:n`。

### <center>channel</center>

单纯地将程序并发没有意义。函数与函数间需要交换数据才能体现并发执行函数的意义。虽然可以使用共享内存进行数据交换，但是共享内存在不同的`goroutine`中容易发生竞态问题。位了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必会造成性能问题。

`golang`中实现了`共享内存实现通信`和`通道实现通信`，`golang`的并发模型是`CSP`，提倡<font color=red>通过通信共享内存而不是通过共享内存实现通信</font>。如果说`goroutine`是`golang`代码并发的执行体，`channel`就是他们之间的连接。`channel`是可以让一个`goroutine`发送特定的值到另一个`goroutine`的通信机制。

`golang`中的`channel`是一种特殊的类型，通道像一个队列，总是遵循先入先出的规则，保证收发数据的顺序，每个通道都是一个具体类型的导管，也就是声明`channel`的时候需要为其指定元素类型。

#### <font color=red>channel类型</font>

```go
var chanName chan elementType
```

#### <font color=red>创建channel</font>

`channel`是引用类型的，通道类型的空值是`nil`,声明通道后需要使用`make`函数初始化才能使用。

```go
var ch chan int
// <nil>
fmt.Println(ch)
ch = make(chan elementType, [cacheSize])
```

#### <font color=red>channel操作</font>

`channel`有`Send`、`Receive`和`Close`三种操作。

##### <font color=red>发送</font>

```go
ch := make(chan int, 10)
// 将10发送到通道中
ch <- 10
```

##### <font color=red>接收</font>

```go
// 从ch中接收值并赋值给x
x := <- ch
// 从ch中接收值，忽略结果
<-ch
```

##### <font color=red>关闭</font>

```go
close(ch)
```

<font color=red>注意</font>:只有在通知接收方`goroutine`所有的数据都发送完的时候才需要关闭通道。通道可以被垃圾回收机制回收，它和关闭文件不一样，在结束操作后文件必须要关闭，但关闭通道不是必须的。关闭后的通道有以下特点:

- 对一个关闭的通道再发送数据就会导致`panic`。
- 对一个关闭的通道进行接收会一直获取值直到通道为空。
- 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。
- 关闭一个已经关闭的通道会导致`panic`。

#### <font color=red>无缓冲通道</font>

无缓冲通道又称阻塞通道。

```go
func main() {
  ch := make(chan int)
  ch <- 10
  fmt.Println("send successful")
}
```

执行代码，出现了`deadlock`错误，之所以发生错误的原因是无缓冲通道只有在有人接收值的时候才能发送值。解决方式：

```go
func recv(c chan int) {
  ret := <-c
  fmt.Println("receive successful")
}

func main() {
  ch := make(chan int)
  go recv(ch)
  ch <- 10
  fmt.Println("send successful")
}
```

无缓冲通道上的发送操作会阻塞，直到另一个 goroutine 在该通道上执行接收操作，这时值才能发送成功，两个 goroutine 将继续执行。相反，如果接收操作先执行，接收方的 goroutine 将阻塞，直到另一个 goroutine 在该通道上发送一个值。使用无缓冲通道进行通信将导致发送和接收的`goroutine`同步化。因此，无缓冲通道也被称为<font color=red>同步通道</font>。

#### <font color=red>有缓冲通道</font>

解决上面问题的方法还有一种就是使用有缓冲通道，我们可以使用`make`函数初始化通道的时候为其指定通道的容量，eg:

```go
func main() {
  ch := make(chan int, 1)
  ch <- 10
  fmt.Println("send successful")
}
```

可以使用内置`len`函数来获取通道内元素的数量，使用`cap`函数获取通道的容量，虽然我们很少会这么做。

#### <font color=red>for range从通道循环取值</font>

当向通道中发送完数据时我们可以通过`close`函数来关闭通道。

```go
// channel 练习
func main() {
        var wg sync.WaitGroup
	ch1 := make(chan int, 50)
	ch2 := make(chan int, 100)
	// 开启goroutine将0~100的数发送到ch1中
        wg.Add(2)
	go func() {
                defer wg.Done()
		for i := 0; i < 100; i++ {
			ch1 <- i
		}
		close(ch1)
	}()
	// 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
	go func() {
                defer wg.Done()
		for {
			i, ok := <-ch1 // 通道关闭后再取值ok=false
			if !ok {
				break
			}
			ch2 <- i * i
		}
		close(ch2)
	}()
        wg.Wait()
	// 在主goroutine中从ch2中接收值打印
	for i := range ch2 { // 通道关闭后会退出for range循环
		fmt.Println(i)
	}
}
```

通常使用`for range`。

#### <font color=red>单向通道</font>

有时我们会将通道作为参数在多个任务函数间传递，很多时候我们在不同的任务函数中通道都会对其进行限制，比如只能发送或接收。

```go
func counter(out chan<- int) {
	for i := 0; i < 100; i++ {
		out <- i
	}
	close(out)
}
 
func squarer(out chan<- int, in <-chan int) {
	for i := range in {
		out <- i * i
	}
	close(out)
}
func printer(in <-chan int) {
	for i := range in {
		fmt.Println(i)
	}
}
 
func main() {
	ch1 := make(chan int)
	ch2 := make(chan int)
	go counter(ch1)
	go squarer(ch2, ch1)
	printer(ch2)
}
```

在函数传参及任何赋值操作中可以将双向通道转换为单向通道，但反过来可以。

#### <font color=red>通道总结</font>

`channel`常见的异常:

| channel |  Nil  |             非空             |        空的        |             满了             |             没满             |
| :-----: | :---: | :--------------------------: | :----------------: | :--------------------------: | :--------------------------: |
|  接收   | 阻塞  |            接收值            |        阻塞        |            接收值            |            接收值            |
|  发送   | 阻塞  |            发送值            |       发送值       |             阻塞             |            发送值            |
|  关闭   | panic | 关闭成功，读完数据后返回零值 | 关闭成功，返回零值 | 关闭成功，读完数据后返回零值 | 关闭成功，读完数据后返回零值 |

关闭已经关闭的`channel`也会引发`panic`。

### <center>worker pool(goroutine池)</center>

在工作中通常会使用可以指定启动的`goroutine`数量-worker pool模式，控制`goroutine`，防止`goroutine`泄露和暴涨。

```go
func worker(id int, jobs <-chan int, results chan<- int) {
	for j := range jobs {
		fmt.Printf("worker:%d start job:%d\n", id, j)
		time.Sleep(time.Second)
		fmt.Printf("worker:%d end job:%d\n", id, j)
		results <- j * 2
	}
}
 
 
func main() {
	jobs := make(chan int, 100)
	results := make(chan int, 100)
	// 开启3个goroutine
	for w := 1; w <= 3; w++ {
		go worker(w, jobs, results)
	}
	// 5个任务
	for j := 1; j <= 5; j++ {
		jobs <- j
	}
	close(jobs)
	// 输出结果
	for a := 1; a <= 5; a++ {
		<-results
	}
}
```

### <center>select多路复用</center>

某些场景下我们需要同时从多个通道接收数据。通道在接收数据时，如果没有数据接收将会发生阻塞。可以用以下方式解决:

```go
for{
    // 尝试从ch1接收值
    data, ok := <-ch1
    // 尝试从ch2接收值
    data, ok := <-ch2
    …
}
```

这种方式虽然可以从多个通道接收值，但是运行性能会差很多。为了应对这种场景，`golang`内置了`select`关键字，可以同时响应多个通道的操作。eg:

```go
func main() {
	ch := make(chan int, 1)
	for i := 0; i < 10; i++ {
		select {
		case x := <-ch:
			fmt.Println(x)
		case ch <- i:
		}
	}
}
```

使用`select`可以提高代码的可读性：

- 可处理一个或多个`channel`的发送/接收操作。
- 如果多个`case`同时满足，`select`会随机选择一个。
- 对于没有`case`的`select`会一直等待，可用于阻塞`main`函数。

### <center>并发和安全锁</center>

有时候在`golang`代码中可能会存在多个`goroutine`同时操作一个临界资源，这种情况会发生竞态问题。eg:

```go
var x int64
var wg sync.WaitGroup
 
func add() {
	for i := 0; i < 50000; i++ {
		x = x + 1
	}
	wg.Done()
}
func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

代码中开启了两个`goroutine`去累加变量`x`, 这两个`goroutine`在访问和修改`x`的时候就会存在数据竞争，导致最后结果与期待不符。

#### <font color=red>互斥锁</font>

互斥锁是一种常用的控制共享资源访问的方法，它能够保证同时只有一个`goroutine`可以访问资源。eg:

```go
var x int64
var wg sync.WaitGroup
var lock sync.Mutex
 
func add() {
	for i := 0; i < 50000; i++ {
		lock.Lock() // 加锁
		x = x + 1
		lock.Unlock() // 解锁
	}
	wg.Done()
}
func main() {
	wg.Add(2)
	go add()
	go add()
	wg.Wait()
	fmt.Println(x)
}
```

使用互斥锁能够保证同一时间有且只有一个`goroutine`进入临界区，其他的`goroutine`则在等待锁；多个`goroutine`同时等待一个锁时，唤醒的策略是随机的，但凡是<font color=red>涉及到访问公共资源一般都要加锁</font>。

#### <font color=red>读写互斥锁</font>

互斥锁是完全排斥的，但是有很多场景是读多写少，当我们并发去读取一个资源不涉及资源修改的时候没必要加锁，这种场景下使用读写锁是更好的一种选择。

读写锁分为两种，读锁和写锁。当一个`goroutine`获取读锁后，其他的`goroutine`如果是获取读锁会继续获得锁，如果是获取写锁就会等待；当一个`goroutine`获得写锁后，其他的`goroutine`无论是获取读锁还是写锁都会等待。

```go
var (
	x      int64
	wg     sync.WaitGroup
	lock   sync.Mutex
	rwlock sync.RWMutex
)
 
func write() {
	// lock.Lock()   // 加互斥锁
	rwlock.Lock() // 加写锁
	x = x + 1
	time.Sleep(10 * time.Millisecond) // 假设读操作耗时10毫秒
	rwlock.Unlock()                   // 解写锁
	// lock.Unlock()                     // 解互斥锁
	wg.Done()
}
 
func read() {
	// lock.Lock()                  // 加互斥锁
	rwlock.RLock()               // 加读锁
	time.Sleep(time.Millisecond) // 假设读操作耗时1毫秒
	rwlock.RUnlock()             // 解读锁
	// lock.Unlock()                // 解互斥锁
	wg.Done()
}
 
func main() {
	start := time.Now()
	for i := 0; i < 10; i++ {
		wg.Add(1)
		go write()
	}
 
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go read()
	}
 
	wg.Wait()
	end := time.Now()
	fmt.Println(end.Sub(start))
}
```

读写锁非常适合读多写少的场景，如果读和写操作差别不大，读写锁的优势就发挥不出来。

#### <font color=red>sync.WaitGroup</font>

在代码中生硬地使用`time.Sleep`不合适，`golang`中可以使用`sync.WaitGroup`来实现并发任务的同步。

|             方法名              |         功能         |
| :-----------------------------: | :------------------: |
| (wg * WaitGroup) Add(delta int) |    计数器 + delta    |
|     (wg *WaitGroup) Done()      |      计数器 - 1      |
|     (wg *WaitGroup) Wait()      | 阻塞直到计数器变为 0 |

sync.WaitGroup 内部维护着一个计数器，计数器的值可以增加和减少。例如当我们启动了 N 个并发任务时，就将计数器值增加 N。每个任务完成时通过调用 Done () 方法将计数器 - 1。通过调用 Wait () 来等待并发任务执行完，当计数器值为 0 时，表示所有并发任务已经完成。eg:

```go
var wg sync.WaitGroup
 
func hello() {
	defer wg.Done()
	fmt.Println("Hello Goroutine!")
}
func main() {
	wg.Add(1)
	go hello() // 启动另外一个goroutine去执行hello函数
	fmt.Println("main goroutine done!")
	wg.Wait()
}
```

<font color=red>注意</font>:`sync.WaitGroup`是一个结构体，传递的时候要传递指针。

#### <font color=red>sync.Once</font>

在编程的很多场景下我们需要确保某些操作在高并发的场景下只执行一次，例如只加载一次配置、只关闭一次通道等。`golang`中`sync.Once`提供了一个解决方案，其函数签名如下:

```go
func (o *Once) Do(f func()) {}
```

<font color=red>注意</font>：如果要执行的函数`f`需要传递参数就需要搭配闭包来使用。

##### <font color=red>加载配置文件示例</font>

延迟一个开销很大的初始化操作到真正用到它的时候再执行是一个很好的实践。因为预先初始化一个变量（比如在 init 函数中完成初始化）会增加程序的启动耗时，而且有可能实际执行过程中这个变量没有用上，那么这个初始化操作就不是必须要做的。我们来看一个例子：

```go
var icons map[string]image.Image
 
func loadIcons() {
	icons = map[string]image.Image{
		"left":  loadIcon("left.png"),
		"up":    loadIcon("up.png"),
		"right": loadIcon("right.png"),
		"down":  loadIcon("down.png"),
	}
}
 
// Icon 被多个goroutine调用时不是并发安全的
func Icon(name string) image.Image {
	if icons == nil {
		loadIcons()
	}
	return icons[name]
}
```

多个 goroutine 并发调用 Icon 函数时不是并发安全的，现代的编译器和 CPU 可能会在保证每个 goroutine 都满足串行一致的基础上自由地重排访问内存的顺序。loadIcon 函数可能会被重排为以下结果：

```go
func loadIcons() {
	icons = make(map[string]image.Image)
	icons["left"] = loadIcon("left.png")
	icons["up"] = loadIcon("up.png")
	icons["right"] = loadIcon("right.png")
	icons["down"] = loadIcon("down.png")
}
```

在这种情况下就会出现即使判断了 icons 不是 nil 也不意味着变量初始化完成了。考虑到这些情况，我们能想到的办法就是添加互斥锁，保证初始化 icons 的时候不会被其他的 goroutine 操作，但是这样做又会引发性能的问题。使用`sync.Once`改造的代码如下:

```go
var icons map[string]image.Image
 
var loadIconsOnce sync.Once
 
func loadIcons() {
	icons = map[string]image.Image{
		"left":  loadIcon("left.png"),
		"up":    loadIcon("up.png"),
		"right": loadIcon("right.png"),
		"down":  loadIcon("down.png"),
	}
}
 
// Icon 是并发安全的
func Icon(name string) image.Image {
	loadIconsOnce.Do(loadIcons)
	return icons[name]
}
```

##### <font color=red>并发安全的单例模式</font>

```go
package singleton
 
import (
    "sync"
)
 
type singleton struct {}
 
var instance *singleton
var once sync.Once
 
func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

sync.Once 其实内部包含一个互斥锁和一个布尔值，互斥锁保证布尔值和数据的安全，而布尔值用来记录初始化是否完成。这样设计就能保证初始化操作的时候是并发安全的并且初始化操作也不会被执行多次。

#### <font color=red>sync.Map</font>

`golang`中内置的`map`不是并发安全的，eg：

```go
var m = make(map[string]int)
 
func get(key string) int {
	return m[key]
}
 
func set(key string, value int) {
	m[key] = value
}
 
func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			set(key, n)
			fmt.Printf("k=:%v,v:=%v\n", key, get(key))
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

上面的代码开启少量几个 goroutine 的时候可能没什么问题，当并发多了之后执行上面的代码就会报 `fatal error: concurrent map writes` 错误，是因为并发的对 map 进行写入了。

像这种场景下就需要为 map 加锁来保证并发的安全性了，第一种方法是在写操作之前和之后加锁（效率低），第二种方法是 Go 语言的 sync 包中提供了一个开箱即用的并发安全版 map-sync.Map。开箱即用表示不用像内置的 map 一样使用 make 函数初始化就能直接使用。同时 sync.Map 内置了诸如 Store、Load、LoadOrStore、Delete、Range 等操作方法。

```go
var m = sync.Map{}
 
func main() {
	wg := sync.WaitGroup{}
	for i := 0; i < 20; i++ {
		wg.Add(1)
		go func(n int) {
			key := strconv.Itoa(n)
			m.Store(key, n)  // 存值
			value, _ := m.Load(key)  // 取值
			fmt.Printf("k=:%v,v:=%v\n", key, value)
			wg.Done()
		}(i)
	}
	wg.Wait()
}
```

### <center>原子操作</center>

代码中的加锁操作因为涉及内核态的上下文切换会比较耗时、代价比较高。针对基本数据类型我们还可以使用原子操作来保证并发安全，因为原子操作是 Go 语言提供的方法，它在用户态就可以完成，因此性能比加锁操作更好。Go 语言中原子操作由内置的标准库 sync/atomic 提供。

#### <font color=red>atomic包</font>

| 方法                                                         | 解释           |
| ------------------------------------------------------------ | -------------- |
| func LoadInt32(addr *int32) (val int32)<br/>func LoadInt64(addr *int64) (val int64)<br/>func LoadUint32(addr *uint32) (val uint32)<br/>func LoadUint64(addr *uint64) (val uint64)<br/>func LoadUintptr(addr *uintptr) (val uintptr)<br/>func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer) | 读取操作       |
| func StoreInt32(addr *int32, val int32)<br/>func StoreInt64(addr *int64, val int64)<br/>func StoreUint32(addr *uint32, val uint32)<br/>func StoreUint64(addr *uint64, val uint64)<br/>func StoreUintptr(addr *uintptr, val uintptr)<br/>func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer) | 写入操作       |
| func AddInt32(addr *int32, delta int32) (new int32)<br/>func AddInt64(addr *int64, delta int64) (new int64)<br/>func AddUint32(addr *uint32, delta uint32) (new uint32)<br/>func AddUint64(addr *uint64, delta uint64) (new uint64)<br/>func AddUintptr(addr *uintptr, delta uintptr) (new uintptr) | 修改操作       |
| func SwapInt32(addr *int32, new int32) (old int32)<br/>func SwapInt64(addr *int64, new int64) (old int64)<br/>func SwapUint32(addr *uint32, new uint32) (old uint32)<br/>func SwapUint64(addr *uint64, new uint64) (old uint64)<br/>func SwapUintptr(addr *uintptr, new uintptr) (old uintptr)<br/>func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer) | 交换操作       |
| func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool)<br/>func CompareAndSwapInt64(addr *int64, old, new int64) (swapped bool)<br/>func CompareAndSwapUint32(addr *uint32, old, new uint32) (swapped bool)<br/>func CompareAndSwapUint64(addr *uint64, old, new uint64) (swapped bool)<br/>func CompareAndSwapUintptr(addr *uintptr, old, new uintptr) (swapped bool)<br/>func CompareAndSwapPointer(addr *unsafe.Pointer, old, new unsafe.Pointer) (swapped bool) | 比较并交换操作 |

Eg:比较互斥锁与原子操作的性能：

```go
package main
 
import (
	"fmt"
	"sync"
	"sync/atomic"
	"time"
)
 
type Counter interface {
	Inc()
	Load() int64
}
 
// 普通版
type CommonCounter struct {
	counter int64
}
 
func (c CommonCounter) Inc() {
	c.counter++
}
 
func (c CommonCounter) Load() int64 {
	return c.counter
}
 
// 互斥锁版
type MutexCounter struct {
	counter int64
	lock    sync.Mutex
}
 
func (m *MutexCounter) Inc() {
	m.lock.Lock()
	defer m.lock.Unlock()
	m.counter++
}
 
func (m *MutexCounter) Load() int64 {
	m.lock.Lock()
	defer m.lock.Unlock()
	return m.counter
}
 
// 原子操作版
type AtomicCounter struct {
	counter int64
}
 
func (a *AtomicCounter) Inc() {
	atomic.AddInt64(&a.counter, 1)
}
 
func (a *AtomicCounter) Load() int64 {
	return atomic.LoadInt64(&a.counter)
}
 
func test(c Counter) {
	var wg sync.WaitGroup
	start := time.Now()
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			c.Inc()
			wg.Done()
		}()
	}
	wg.Wait()
	end := time.Now()
	fmt.Println(c.Load(), end.Sub(start))
}
 
func main() {
	c1 := CommonCounter{} // 非并发安全
	test(c1)
	c2 := MutexCounter{} // 使用互斥锁实现并发安全
	test(&c2)
	c3 := AtomicCounter{} // 并发安全且比互斥锁效率更高
	test(&c3)
}
```

atomic 包提供了底层的原子级内存操作，对于同步算法的实现很有用。这些函数必须谨慎地保证正确使用。除了某些特殊的底层应用，使用通道或者 sync 包的函数 / 类型实现同步更好。

