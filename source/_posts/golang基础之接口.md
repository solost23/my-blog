---
title: golang基础之接口
date: 2021-12-12
tags:
    - golang
---

接口定义了一个对象的行为规范，只定义规范不实现，由具体的对象来实现规范的细节。接口是一种特殊的类型，它规定了变量应该有哪些方法，即<font color=red>不限制类型，只限制方法</font>。

### <center>接口类型</center>

`golang`中接口是一种抽象类型。`interface`是一组`method`的集合，是`duck-type programming`的一种实现。接口做的事情就是定义一个规范，只要一台机器有洗衣服和甩干的功能，我们就称它为洗衣机。即不关心属性(数据)，只关心行为(方法)。

#### <font color=red>什么是`duck-type programming?</font>

[什么是鸭子类型?](https://www.jianshu.com/p/cd06818935a8)

为了保护你的`golang`职业生涯，请牢记:`interface`是一种类型。

### <center>为什么要使用接口?</center>

```go
type Cat struct{}
 
func (c Cat) Say() string { return "喵喵喵" }
 
type Dog struct{}
 
func (d Dog) Say() string { return "汪汪汪" }
 
func main() {
	c := Cat{}
	fmt.Println("猫:", c.Say())
	d := Dog{}
	fmt.Println("狗:", d.Say())
}
```

上面的代码中定义了猫和狗，然后他们都会叫，你会发现 main 函数中明显有重复的代码，如果我们后续再加上猪、青蛙等动物的话，我们的代码还会一直重复下去。那我们能不能把它们都当成 “能叫的动物” 来处理呢？

像类似的例子在我们编程过程中经常会遇到：

比如一个网上商城可能使用支付宝、微信、银联等多种方式去在线支付，我们能不能把它们当成 “支付方式” 来处理呢？

比如三角形，四边形，圆形都能计算周长和面积，我们能不能把他们当成 “图形” 来处理呢？

比如销售、行政、程序员都能计算月薪，我们能不能把他们当成 “员工” 来处理呢？

`golang`中为了解决类似上面的问题，就设计了接口这个概念。接口区别于我们之前所有的具体类型，接口是一种抽象的类型。当你看到一个接口类型的值时，你不知道它是什么，唯一知道的是通过它的方法能做什么。

### <center>接口的定义</center>

`golang`提倡面向接口编程。每个接口由数个方法组成:

```go
type 接口类型名 interface{
    方法名1( 参数列表1 ) 返回值列表1
    方法名2( 参数列表2 ) 返回值列表2
    …
}
```

- 接口类型名：使用`type`将接口定义为自定义的类型名。`golang`的接口在命名时，一般会在单词后面添加`er`，例如有写操作的接口叫`Writer`接口名最好要能突出该接口的类型含义。
- 方法名：当方法首字母是大写且这个接口类型名首字母也是大写时，这个方法可以被接口所在的包之外的代码访问。
- 参数列表、返回值列表：参数列表和返回值列表中的参数变量名可以省略。

Eg:

```go
type Writer interface {
  Write([]byte) error
}
```

### <center>实现接口的条件</center>

一个变量若实现了接口中的所有方法，那么这个变量就实现了这个接口，可以称为这个接口类型的变量。

```go
type Sayer interface {
  say()
}

type dog struct {}
type cat struct {}

func (d *dog) say() {
  fmt.Println("汪汪汪")
}

func (c *cat) say() {
  fmt.Println("喵喵喵")
}
```

此时`dog`和`cat`变量就实现了`Sayer`这个接口。

### <center>接口类型变量</center>

那实现了接口有什么用呢？接口类型变量能够存储所有实现了该接口的实例。例如上面的示例中，`Sayer`类型的变量能够存储`dog`和`cat`类型的变量。

```GO
func main() {
	var x Sayer // 声明一个Sayer类型的变量x
	a := cat{}  // 实例化一个cat
	b := dog{}  // 实例化一个dog
	x = a       // 可以把cat实例直接赋值给x
	x.say()     // 喵喵喵
        fmt.Println("%T \n", x)  // main.cat
	x = b       // 可以把dog实例直接赋值给x
	x.say()     // 汪汪汪
        fmt.Println("%T \n", x)  // main.dog
}
```

### <center>值接收者与指针接收者实现接口的区别</center>

```go
type Mover interface {
	move()
}
 
type dog struct {}
```

#### <font color=red>值接收者实现接口</font>

```go
func (d dog) move() {  // 使用值接收者实现接口的move方法
	fmt.Println("狗会动")
}
```

此时实现接口的是`dog`类型：

```go
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x可以接收dog类型
	var fugui = &dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
	x.move()
}
```

从上面的代码可以发现，使用值接收者实现接口后，不管是`dog`结构体还是结构体指针`*dog`类型的变量都可以赋值给该接口变量。因为<font color=red>`golang`有对指针类型变量求值的语法糖，`dog`指针`fugui`内部会自动求值`*fugui`</font>。

#### <font color=red>指针接收者实现接口</font>

```go
func (d *dog) move() {  // 使用指针接收者实现接口的move方法
	fmt.Println("狗会动")
}
func main() {
	var x Mover
	var wangcai = dog{} // 旺财是dog类型
	x = wangcai         // x不可以接收dog类型
	var fugui = &dog{}  // 富贵是*dog类型
	x = fugui           // x可以接收*dog类型
}
```

此时实现`Mover`接口的是`*dog`类型，所以不能给`x`传入`dog`类型的`wangcai`，此时`x`只能存储`*dog`类型的值。

<font color=red>小结</font>

使用值接收者实现接口与使用指针接收者实现接口的区别：

- 使用值接收者实现接口，结构体类型和结构体指针类型的变量都可以存到接口中。
- 使用指针接收者实现接口，只能存结构体指针类型的变量到接口中。
- <font color=red>实际开发中一般都用指针接收者实现接口</font>。

### <center>类型与接口的关系</center>

#### <font color=red>一个类型实现多个接口</font>

一个类型可以同时实现多个接口，而接口之间彼此独立，不知道对方的实现。例如，狗可以叫，也可以动。我们就可以分别定义`Sayer`与`Mover`接口，eg;

```go
// Sayer 接口
type Sayer interface {
	say()
}
 
// Mover 接口
type Mover interface {
	move()
}
```

`dog`既可以实现`Say`接口，也可以实现`mover`接口。

```go
type dog struct {
	name string
}
 
// 实现Sayer接口
func (d dog) say() {
	fmt.Printf("%s会叫汪汪汪\n", d.name)
}
 
// 实现Mover接口
func (d dog) move() {
	fmt.Printf("%s会动\n", d.name)
}
 
func main() {
	var x Sayer
	var y Mover
 
	var a = dog{name: "旺财"}
	x = a
	y = a
	x.say()
	y.move()
}
```

#### <font color=red>多个类型实现一个接口</font>

`golang`中不同的类型可以实现同一接口，首先先定义一个`Mover`接口:

```go
// Mover 接口
type Mover interface {
	move()
}
```

假设狗可以动，汽车可以动，可以使用如下代码实现这个关系：

```go
type dog struct {
	name string
}
 
type car struct {
	brand string
}
 
// dog类型实现Mover接口
func (d dog) move() {
	fmt.Printf("%s会跑\n", d.name)
}
 
// car类型实现Mover接口
func (c car) move() {
	fmt.Printf("%s速度70迈\n", c.brand)
}
```

这个时候我们在代码中就可以把狗和汽车当成一个会动的动物来处理，不需要关注它们具体是什么，只需要调用它们的`move`方法就可以了。

```go
func main() {
	var x Mover
	var a = dog{name: "旺财"}
	var b = car{brand: "保时捷"}
	x = a
	x.move()
	x = b
	x.move()
}
```

并且一个接口的方法，不一定需要由一个类型完全实现，接口的方法可以通过在类型中嵌入其它类型或者结构体来实现:

```go
// WashingMachine 洗衣机
type WashingMachine interface {
	wash()
	dry()
}
 
// 甩干器
type dryer struct{}
 
// 实现WashingMachine接口的dry()方法
func (d dryer) dry() {
	fmt.Println("甩一甩")
}
 
// 海尔洗衣机
type haier struct {
	dryer //嵌入甩干器
}
 
// 实现WashingMachine接口的wash()方法
func (h haier) wash() {
	fmt.Println("洗刷刷")
}
```

### <center>接口嵌套</center>

接口与接口间可以通过嵌套创造出新的接口:

```go
// Sayer 接口
type Sayer interface {
	say()
}
 
// Mover 接口
type Mover interface {
	move()
}
 
// 接口嵌套
type animal interface {
	Sayer
	Mover
}
```

嵌套得到的接口的使用与普通接口一样，我们让`cat`实现`animal`接口:

```go
type cat struct {
	name string
}
 
func (c *cat) say() {
	fmt.Println("喵喵喵")
}
 
func (c *cat) move() {
	fmt.Println("猫会动")
}
 
func main() {
	var x animal
	x = &cat{name: "花花"}
	x.move()
	x.say()
}
```

### <center>空接口</center>

#### <font color=red>定义</font>

空接口是指没有定义任何方法的接口，因此任何类型都实现了空接口。

特点:

- 没必要给空接口起名字。
- 所有类型都实现了，也就是任意类型的变量都能保存到空接口中。

```GO
func main() {
	// 定义一个空接口x
	var x interface{}
	s := "Hello ty"
	x = s
	fmt.Printf("type:%T value:%v\n", x, x)
	i := 100
	x = i
	fmt.Printf("type:%T value:%v\n", x, x)
	b := true
	x = b
	fmt.Printf("type:%T value:%v\n", x, x)
}
```

#### <font color=red>空接口的应用</font>

##### <font color=red>空接口作为函数的参数(常用)</font>

使用空接口实现可以接收任意类型的函数参数:

```go
// 空接口作为函数参数
func show(a interface{}) {
	fmt.Printf("type:%T value:%v\n", a, a)
}
```

##### <font color=red>空接口作为`map`值</font>

使用空接口实现可以保存任意值的字典:

```go
// 空接口作为map值
func main() {
	var m1 map[string]interface{}
	m1 = make(map[string]interface{}, 16)
	m1["name"] = "ty"
	m1["age"] = 20
	m1["married"] = false
	fmt.Println(m1)  // map[age:20 married:false name:ty]
}
```

### <center>类型断言</center>

空接口可以存储任意类型的值，那我们如何获取其存储的具体数据，然后根据不同的类型去做不同的事呢？

#### <font color=red>接口值</font>

一个接口的值是由一个具体类型和具体类型的值两部分组成的。这两部分分别称为接口的动态类型与动态值。eg:

```go
var w io.Writer
w = os.Stdout
w = new(bytes.Buffer)
w = nil
```

![](/images/interface-1024x825.png)

想要判断空接口中的值可以使用类型断言，其语法格式如下:

```go
x.(T)
```

Eg:

```go
func main() {
  var x interface{}
  x = "hello ty"
  v, ok := x.(string)
  if !ok {
    fmt.Println("类型断言失败")
  } else {
    fmt.Println(v)
  }
}
```

上面的实例中如果要断言多次就需要写多个if判断，这个时候我们可以使用`switch`语句来实现:

```go
func justifyType(x interface{}) {
	switch x.(type) {  // 拿到的是类型，且x.(type)只能用于switch内
	case string:
		fmt.Printf("x is a string，value is %v\n", x.(string))
	case int:
		fmt.Printf("x is a int is %v\n", x.(int))
	case bool:
		fmt.Printf("x is a bool is %v\n", x.(bool))
	default:
		fmt.Println("unsupport type！")
	}
}
```

<font color=red>注意</font>:

- 只有当两个或两个以上的具体类型必须以相同的方式进行处理时才需要定义接口。
- 不要为了写接口而写接口，那样只会增加不必要的抽象，导致不必要的运行时损耗。

