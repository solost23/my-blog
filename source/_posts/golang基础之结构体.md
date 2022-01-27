---
title: golang基础之结构体
date: 2021-12-11 11:00
tags:
    - golang
---

`golang`中没有类的概念，也不支持类的继承等面向对象的概念。`golang`中通过结构体的内嵌配合接口比面向对象具有更高的扩展性和灵活性。

### <center>自定义类型和类型别名</center>

#### <font color=red>自定义类型</font>

`golang`中有一些基本的数据类型，如string、整型、浮点型、布尔型等数据类型，`golang`中可以使用`type`来定义自定义类型，自定义类型是定义了一个全新的类型。我们可以基于内置的基本类型定义，也可以通过`struct`定义。eg:

```go
// 将MyInt定义为int类型
type MyInt int
```

#### <font color=red>类型别名</font>

类型别名规定:`TypeAlias`只是`Type`的别名，本质上`TypeAlias`与`Type`是同一个类型。eg:

```go
type TypeAlias = Type
```

之前见过的`rune`和`byte`就是类型别名，它们的定义如下:

```go
type byte = uint8
type rune = int32
```

### <center>类型定义和类型别名的区别</center>

```go
// 类型定义
type NewInt int
 
// 类型别名
type MyInt = int
 
func main() {
	var a NewInt
	var b MyInt
  // type of a:main.NewInt
	fmt.Printf("type of a:%T \n", a)
  // type of b:int
	fmt.Printf("type of b:%T \n", b)  
}
```

结果显示`a`的类型是main.NewInt，表示`main`包下定义的`NewInt`类型，代码编译完也保留。`b`的类型是`int`。`MyInt`类型只会在代码中存在，编译完成时并不会有`MyInt`类型。

### <center>结构体</center>

`golang`提供了一种自定义数据类型，可以封装多个基本数据类型，这种数据类型叫结构体，也就是说我们可以通过`struct`来定义自己的类型了，`golang`中通过`struct`来实现面向对象。

#### <font color=red>结构体定义</font>

```go
type typeName struct {
  fieldName fieldType
  fieldNmae fieldType
  ...
}
```

eg:

```go
type Person struct {
	name, city string
	age int8
}
```

这样我们就拥有了一个`person`的自定义类型，`golang`内置的基本数据类型用来描述一个值，而结构体用来描述一组值。

#### <font color=red>结构体实例化</font>

只有当结构体实例化时才会真正地分配内存，结构体本身也是一种类型，我们可以像声明内置类型一样使用`var`声明结构体类型，格式如下：

```go
var structName structType
```

##### <font color=red>基本实例化</font>

我们可通过`.`来访问结构体成员变量。

Eg:

```go
type Person struct {
  name, city string
  age int8
}

func main() {
  var p1 Person
  p1.name = "ty"
  p1.city = "北京"
  p1.age = 18
}
```

##### <font color=red>匿名结构体</font>

在定义一些临时数据结构等场景下还可以使用匿名结构体，例如函数中临时使用，eg:

```go
package main

import (
	"fmt"
)

func main() {
  var user struct {Name string; Age int}
  user.Name = "ty"
  user.Age = 18
}
```

#####  <font color=red>创建指针类型结构体</font>

我们可以通过`new`对结构体实例化，得到的是结构体的地址,eg:

```go
type Person struct {
  name, city string
  age int8
}

func main() {
  var p2 = new(person)
  
  // *main.Person
  fmt.Printf("%T \n", p2)
}
```

需要注意的是`golang`支持对结构体指针直接使用，来访问结构体的成员:

```go
type Person struct {
  name, city string
  age int8
}

func main() {
  var p2 = new(Person)
  p2.name = "ty"
  p2.age = 18
  p2.city = "北京"
}
```

##### <font color=red>取结构体地址实例化</font>

使用`&`对结构体进行取地址操作相当于对该结构体类型进行了一次`new`实例化操作:

```go
type Person struct {
  name, city string
  age int8
}

func main() {
  p3 := &Person{}
  
  // *main.Person
  fmt.Printf("%T \n", p3)
}
```

<font color=red>注意</font>:`p3.name="ty"`实际上在底层是`(*p3).name = "ty"`,这是`golang`帮我们实现的语法糖。

#### <font color=red>结构体初始化</font>

未初始化的结构体其成员变量都是对应其类型的零值。

##### <font color=red>访问成员变量的方式初始化</font>

```go
type Person struct {
  name, city string
  age int8
}

func main() {
  var p1 Person
  p1.name = "ty"
  p1.city = "北京"
  p1.age = 18
}
```

也可以对结构体指针初始化。

##### <font color=red>使用键值对初始化</font>

```GO
type Person struct {
  name, city string
  age int8
}

func main() {
  p5 := Person {
    name: "ty", 
    city: "北京", 
    age: 18, 
  }
}
```

也可以对结构体指针进行键值对初始化，eg:

```go
p6 := &Person {
  name: "ty", 
  city: "北京", 
  age: 18, 
}
```

##### <font color=red>使用值列表初始化</font>

```go
p7 := Person {
  "ty", 
  "北京", 
  18, 
}
```

也可以对结构体指针进行列表初始化。

使用列表初始化时<font color=red>注意</font>:

- 必须初始化结构体所有成员变量。
- 初始值的填充顺序必须与字段在结构体中的声明顺序一致。
- 该方式不能和键值对初始化混用。

### <center>函数参数是值结构体类型与函数参数是指针结构体类型的区别</center>

`golang`中函数传参永远是拷贝，所以传结构体修改也是修改副本，此时指针的作用就显现出来了，eg:

```go
type Person struct {
  name string
  gender string
}

func f2(x *Person) {
  // x.gender = "female"
  (*x).gender = "female"
}

func main() {
  p := Person {
    name: "ty", 
    gender: "male", 
  }
  
  fmt.Println(p.gender)
  f2(&p)
  fmt.Println(p.gender)
}
```

### <center>结构体内存布局</center>

结构体占用一块连续的内存空间:

```go
type test struct {
	a int8
	b int8
	c int8
	d int8
}
 
func main() {
	n := test {
		1, 2, 3, 4, 
	}
	fmt.Printf("n.a %p \n", &n.a)
	fmt.Printf("n.b %p \n", &n.b)
	fmt.Printf("n.c %p \n", &n.c)
	fmt.Printf("n.d %p \n", &n.d)
}
```

输出结果为:

```go
n.a 0xc0000120a0 
n.b 0xc0000120a1 
n.c 0xc0000120a2 
n.d 0xc0000120a3
```

####  <font color=red>空结构体</font>

空结构体不占用内存空间。

### <center>构造函数</center>

`golang`中结构体没有构造函数，但是我们可以自己实现。

```go
func newPerson(name, city string, age int) *Person {
	return &Person {
		name: name, 
		city: city, 
		age: age, 
	}
}
```

<font color=red>注意</font>:

- 构造函数只用于初始化对象。
- 构造函数约定俗成用new开头。
- 日常开发中一般都是返回结构体指针。

### <center>方法和接收者</center>

`golang`中方法是一种<font color=red>作用于特定类型变量的函数</font>。这种特定类型变量叫做<font color=red>接收者</font>。接收者的概念类似于其他语言中的`this`或`self`。`golang`规定方法的上面都要加注释。

```go
func (接收者变量 接收者类型) 方法名(参数列表) (返回参数) {
	函数体
}
```

其中:

- 接收者变量：接收者中的参数变量名在命名时官方建议使用接收者类型名称首字母小写，而不是`self`、`this`之类的命名。
- 接收者类型：接收者类型和参数类似，可以是指针类型和非指针类型。
- 方法名、参数列表、返回参数：具体格式与函数定义相同。

Eg:

```go
package main

import (
	"fmt"
)

type Person struct {
  name string
  age int8
}

func newPerson(name string, age int8) *Person {
  return &Person {
    name: name, 
    age: age, 
  }
}

func (p *Person) Dream() {
  fmt.Println(p.name, "正在做梦...")
}

func main() {
  p1 := newPerson("ty", 18)
  p1.Dream()
}
```

方法与函数的区别是：函数不属于任何类型，方法属于特定类型。

### <center>指针类型接收者与值类型接收者</center>

#### <font color=red>指针类型接受者(常用)</font>

指针类型的接收者由一个结构体的指针组成，由于指针的特性，调用方法时修改接收者指针的任意成员变量，在方法结束后，修改都是有效的。这种方式就十分接机其它语言中的`this`或`self`。

```go
func (p *Person) setAge (newAge int8) {
  p.age = newAge
}

func main() {
  p2 := newPerson("ty", 18)
  p2.setAge(12)
  
  // 12
  fmt.Println(p2.age)
}
```

#### <font color=red>值类型接受者</font>

当方法作用于值类型接收者时，`golang`会在代码运行时将接收者的值复制一份。在值类型接收者的方法中可以获取接收者的成员值，但修改操作只是针对脚本，无法修改接收者变量本身：

```go
func (p Person) setAge (newAge int8) {
	p.age = newAge
}
 
func main() {
	p2 := newPerson("ty", 21)
	p2.setAge(12)
	fmt.Println(p2.age)  // 21
}
```

### <center>任意类型添加方法</center>

`golang`中接收者的类型可以是任何类型，不仅仅是结构体，任何类型都可以拥有方法。eg:

```go
package main

import (
	"fmt"
)

type MyInt int

func (m *MyInt) sayHello() {
  fmt.Println("sayHello...")
}

func main() {
  var m1 MyInt
  m1.sayHello()
}
```

<font color=red>注意</font>:非本地类型不能定义方法，例如不能给`int`定义，也就是说我们不能给别的包的类型添加方法。

### <center>结构体的匿名字段</center>

结构体允许其成员字段在声明时没有字段名而只有类型，这种没有名字的字段称为匿名字段:

```go
type Person struct {
  string
  int
}

func main() {
  p1 := &Person {
    "ty", 
    18, 
  }
  
  // *main.Person{string:"ty",int:18}
  fmt.Printf("%#v \n")
  // ty 18
  fmt.Println(p1.string, p1.int)
}
```

<font color=red>注意</font>:这里的匿名字段并不代表没有字段名，而是默认采用类型名作为字段名，结构体要求字段名必须唯一，因此<font color=red>一个结构体中同种类型的匿名字段只能有一个。

### <center>嵌套结构体</center>

一个结构体可以嵌套包含另一个结构体或结构体指针，eg;

```go
type Address struct {
  Province string
  City string
}

type User struct {
  Name string
  Gender string
  Address Address
}

func main() {
  user1 := &User {
    Name: "ty", 
    Gender: "male", 
    Address: &Address {
      Province: "北京", 
      City: "北京", 
    }
  }
}
```

### <center>嵌套匿名字段</center>

```go
//Address 地址结构体
type Address struct {
	Province string
	City     string
}
 
//User 用户结构体
type User struct {
	Name    string
	Gender  string
	Address //匿名字段
}
 
func main() {
	var user2 User
	user2.Name = "ty"
	user2.Gender = "male"
	user2.Address.Province = "北京"    // 匿名字段默认使用类型名作为字段名
	user2.City = "北京"                // 匿名字段可以省略
	fmt.Printf("user2=%#v\n", user2) //user2=main.User{Name:"ty", Gender:"male", Address:main.Address{Province:"北京", City:"北京"}}
}
```

当访问结构体成员时会先在结构体中查找字段，找不到再去嵌套的匿名字段中查找。

### <center>结构体的“继承”</center>

`golang`中没有继承的概念，但使用结构体可以实现“继承”:

```go
//Animal 动物
type Animal struct {
	name string
}
 
func (a *Animal) move() {
	fmt.Printf("%s会动！\n", a.name)
}
 
//Dog 狗
type Dog struct {
	Feet    int8
	*Animal //通过嵌套匿名结构体实现继承
}
 
func (d *Dog) wang() {
	fmt.Printf("%s会汪汪汪~\n", d.name)
}
 
func main() {
	d1 := &Dog{
		Feet: 4,
		Animal: &Animal{ //注意嵌套的是结构体指针
			name: "乐乐",
		},
	}
	d1.wang() //乐乐会汪汪汪~
	d1.move() //乐乐会动！
}
```

### <center>结构体字段的可见性</center>

结构体中字段名大写开头表示可公开访问，小写表示私有(仅在定义当前结构体的包中可以访问)。

### <center>结构体与JSON序列化</center>

`json`是一种轻量级的数据交换格式。易于人阅读和编写。同时也易于机器解析和生成。`json`键值对是用来保存`js`对象的一种方式。

```go
//Student 学生
type Student struct {
	ID     int
	Gender string
	Name   string
}
 
//Class 班级
type Class struct {
	Title    string
	Students []*Student
}
 
func main() {
	c := &Class{
		Title:    "101",
		Students: make([]*Student, 0, 200),
	}
	for i := 0; i < 10; i++ {
		stu := &Student{
			Name:   fmt.Sprintf("stu%02d", i),
			Gender: "男",
			ID:     i,
		}
		c.Students = append(c.Students, stu)
	}
	//JSON序列化：结构体-->JSON格式的字符串
	data, err := json.Marshal(c)
	if err != nil {
		fmt.Println("json marshal failed")
		return
	}
	fmt.Printf("json:%s\n", data)
	//JSON反序列化：JSON格式的字符串-->结构体
	str := `{"Title":"101","Students":[{"ID":0,"Gender":"男","Name":"stu00"},{"ID":1,"Gender":"男","Name":"stu01"},{"ID":2,"Gender":"男","Name":"stu02"},{"ID":3,"Gender":"男","Name":"stu03"},{"ID":4,"Gender":"男","Name":"stu04"},{"ID":5,"Gender":"男","Name":"stu05"},{"ID":6,"Gender":"男","Name":"stu06"},{"ID":7,"Gender":"男","Name":"stu07"},{"ID":8,"Gender":"男","Name":"stu08"},{"ID":9,"Gender":"男","Name":"stu09"}]}`
	c1 := &Class{}
	err = json.Unmarshal([]byte(str), c1)
	if err != nil {
		fmt.Println("json unmarshal failed!")
		return
	}
	fmt.Printf("%#v\n", c1)
}
```

### <center>结构体标签(tag)</center>

`tag`是结构体的元信息，可以在运行时通过反射机制读取出来，`tag`在结构体字段后定义:

```go
`key1:"value1" key2:"value2"`
```

<font color=red>注意</font>:结构体编写`tag`时，必须严格遵循键值对规则。结构体标签的解析代码容错能力很差，一旦格式写错，编译和运行时都不会提示任何错误，通过反射也无法正确取值。

```go
//Student 学生
type Student struct {
	ID     int    `json:"id"` //通过指定tag实现json序列化该字段时的key
	Gender string //json序列化是默认使用字段名作为key
	name   string //私有不能被json包访问
}
 
func main() {
	s1 := Student{
		ID:     1,
		Gender: "male",
		name:   "ty",
	}
	data, err := json.Marshal(s1)
	if err != nil {
		fmt.Println("json marshal failed!")
		return
	}
	fmt.Printf("json str:%s\n", data) //json str:{"id":1,"Gender":"male"}
}
```







