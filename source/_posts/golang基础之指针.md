---
title: golang基础之指针
---

区别于`C/C++`中的指针，`golang`中的指针不支持偏移和计算，是安全指针，搞明白`golang`中的指针需要先知道三个概念:指针地址、指针类型和指针取值。

### <center>golang中的指针</center>

任何程序数据载入内存后，在内存中都有他们的地址，这就是指针。而为了保存一个数据在内存中的地址，我们就需要指针变量。

比如 “憧憬是距离理解最遥远的感情”，我想把这句话写入程序，程序一旦启动这句话是要加载到内存（假设内存地址 0x123456），我在程序中把这段话赋值给变量 A，把内存地址赋值给变量 B。这时候 B 就是一个指针变量。通过变量 A 和变量 B 都能找到这句话。

Go 语言中指针不能进行偏移和运算。因此 Go 语言中的指针操作非常简单，我们只需要记住两个符号：&（取地址）和 *（根据地址取值）。

### <center>指针地址和指针类型</center>

每个变量在运行时都有一个地址，这个地址代表变量在内存中的位置。Go 语言中使用 & 字符放在变量前面对变量进行 “取地址” 操作。Go 语言中的值（int、float、bool、string、array、struct）都有对应的指针类型，如：*int、*float64、*string 等。

取变量指针的语法如下：

```go
ptr := &v  // v的类型为T
```

Eg:

```go
func main() {
	a := 10
	b := &a
	fmt.Printf("a:%d ptr:%p \n", a, &b)
	fmt.Printf("b:%p type:%T \n", b, b)
	fmt.Println(&b)  // 指针地址
	fmt.Printf("%T \n", b) // 指针类型
}
```

result:

```go
a:10 ptr:0xc000006028 
b:0xc0000120a0 type:*int 
0xc000006028
*int
```

看一下`b := &a`的图示:

![](images/ptr-1024x452.png)

### <center>指针取值</center>

在对普通变量使用 & 操作符取地址后会获得这个变量的指针，然后可以对指针使用 * 操作，也就是指针取值，代码如下：

```go
func main() {
	//指针取值
	a := 10
	b := &a // 取变量a的地址，将指针保存到b中
	fmt.Printf("type of b:%T\n", b)
	c := *b // 指针取值（根据指针去内存取值）
	fmt.Printf("type of c:%T\n", c)
	fmt.Printf("value of c:%v\n", c)
}
```

result:

```go
type of b:*int
type of c:int
value of c:10
```

总结：取地址操作符 & 和取值操作符 * 是一对互补操作符，& 取出地址，* 根据地址取出地址指向的值。

变量、指针地址、指针变量、取地址、取值的相互关系和特性如下：

- 对变量进行取地址（&）操作，可以获得这个变量的指针变量。
- 指针变量的值是指针地址。
- 对指针变量进行取值（*）操作，可以获得指针变量指向的原变量的值。

### <center>指针传值示例:</center>

```go
func modify1(x int) {
	x = 100
}
 
func modify2(x *int) {
	*x = 100
}
 
func main() {
	a := 10
	modify1(a)
	fmt.Println(a) // 10
	modify2(&a)
	fmt.Println(a) // 100
}
```

### <center>new与make</center>

eg:

```go
func main() {
	var a *int
	*a = 100
	fmt.Println(*a)
 
	var b map[string]int
	b["张三"] = 100
	fmt.Println(b)
}
```

执行上面的代码会引发`panic`，为什么呢？<font color=red>在`golang`中对于引用类型的变量，我们在使用的时候不仅要声明它，还要为它分配内存空间，否则我们的值就无法存储。而对于值类型的声明不需要分配内存空间，是因为他们在声明的时候就已经默认分配好了内存空间</font>。要分配内存，就引出了今天的`new`与`make`。`golang`中`new`和`make`是内建的两个函数，主要用来分配内存。

#### <font color=red>new(一般不用)</font>

```go
func new(Type) *Type
```

其中：

- Type 表示类型，new 函数只接收一个参数，这个参数是一个类型。
- *Type 表示类型指针，new 函数返回一个指向该类型内存地址的指针。

new 函数不太常用，使用 new 函数得到的是一个类型的指针，并且该指针对应的值为该类型的零值。

eg:

```go
func main() {
	a := new(int)
	b := new(bool)
	fmt.Printf("%T \n", a)  // *int
	fmt.Printf("%T \n", b)  // *bool
	fmt.Println(*a)  // 0
	fmt.Println(*b)  // false
}
```

本节开始的示例代码中 var a *int 只是声明了一个指针变量 a 但是没有初始化，指针作为引用类型需要初始化后才会拥有内存空间，才可以给它赋值。应该按照如下方式使用内置的 new 函数对 a 进行初始化后就可以正常对其赋值了：

```go
func main() {
	var a *int
	a = new(int)
	*a = 10
	fmt.Println(*a)
}
```

#### <font color=red>make(常用)</font>

make 也是用于分配内存的，区别于 new，它只用于 slice、map 以及 chan 的内存创建，而且它返回的类型就是这三个类型本身，而不是它们的指针类型，因为这三种类型就是引用类型，所以没必要返回他们的指针了。make 函数的函数签名如下：

```go
func make(t Type, size ...IntegerType) Type
```

make 函数是无可替代的，我们在使用 slice、map 以及 channel 的时候，都需要使用 make 进行初始化，然后才可以对它们进行操作。这个我们在上一章有说明，关于 channel 会在后续章节详细说明。

本节开始的实例中 var b map [string] int 只是声明变量 b 是一个 map 类型的变量，需要像下面的示例代码一样使用 make 函数进行初始化操作后，才能对其进行键值对赋值：

```go
func main() {
	var b map[string]int
	b = make(map[string]int, 10)
	b["张三"] = 100
	fmt.Println(b)
}
```

### <center>new与make的区别</center>

- `make`与`new`都是用来做内存分配的。
- `new`很少用，一般用来给基本的数据类型申请内存，string/int...返回对应类型的指针。
- `make`用来给`slice`/`map`/`chan`申请内存，返回的是对应的三个类型本身。