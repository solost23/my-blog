---
title: golang基础之函数
date: 2021-12-12 11:00
tags:
    - golang
---

- 函数是组织好的、可重复使用的、用于执行<font color=red>最基本任务</font>的代码块。
- `golang`中支持函数、匿名函数和闭包，并且函数在`golang`中属于一等公民。
- `golang`中函数穿参永远是拷贝，即生成一个副本。
- `golang`中函数不支持嵌套定义带名字的函数，支持嵌套调用。

### <center>函数定义</center>

`golang`中定义函数使用`func`关键字：

```go
func 函数名(参数) (返回值) {
  函数体
}
```

Eg:

```go
func intSum(x int, y int) int {
  return x + y
}
```

### <center>函数调用</center>

```go
ret := intSum(10, 20)
```

<font color=red>注意</font>:调用有返回值的函数时，可以不接收其返回值。

### <center>参数</center>

#### <font color=red>类型简写</font>

函数的参数中如果相邻变量的类型相同，则可以省略类型,eg:

```go
func intSum(x, y int) int {
  return x + y
}
```

#### <font color=red>可变参数</font>

可变参数是指函数的参数数量不固定，`golang`中可变参数通过在参数名后加...来标识,<font color=red>注意</font>:可变参数通常作为函数的最后一个参数，<font color=red>可变参数类型为切片</font>。eg:

```go
func intSum2(x ... int) int {
  // [10 20 30]
  fmt.Println(x)
  sum := 0
  for _, v := range x {
    sum = sum + v
  }
  return sum
}

func main() {
  ret := intSum2(10, 20, 30)
}
```

### <center>返回值</center>

#### <font color=red>多返回值</font>

`golang`中支持多返回值，函数如果有多个返回值必须用`()`将所有返回值包裹起来。eg:

```go
func calc(x, y int) (int, int) {
  sum := x + y
  sub := x - y
  return sum, sub
}

func main() {
  m, n := calc(3, 5)
}
```

#### <font color=red>返回值命名</font>

函数定义时可以给返回值命名，此时相当于在函数中声明了变量，在函数体中可以直接使用这些变量，最后通过`return`返回。eg：

```go
func calc(x, y int) (sum, sub int) {
  sum = x + y
  sub = x - y
  return
}
```

### <center>函数进阶</center>

#### <font color=red>变量作用域</font>

##### <font color=red>全局变量</font>

全局变量是定义在函数外部的变量，它在程序整个运行周期内都有效。在函数中可以访问到全局变量。<font color=red>注意</font>:无法在函数内部声明全局变量。

```go
package main

import (
	"fmt"
)

var num int64 = 10

func testGlobalVar() {
  
  // num=10
  fmt.Println("num=%d \n", num)
}

func main() {
  testGlobalVar()
}
```

##### <font color=red>局部变量</font>

局部变量分为两种：函数内部定义的变量和语句块定义的变量，函数内部定义的变量无法在函数外使用，语句块定义的变量无法在语句块外使用。

<font color=red>注意</font>:函数中变量的查找顺序:

- 在函数中查找。
- 找不到则去函数外层查找(全局or外层函数)。
- 一直找到全局，全局都没有则报错。

### <center>函数类型与变量</center>

#### <font color=red>定义函数类型</font>

```go
type calculation func(int, int) int
```

上面定义了一个`calculation`类型，它是一种函数类型，这种函数接收两个`int`并返回一个`int`。凡是满足这个条件的函数都是`calculation`类型的函数，eg:

```go
func add(x, y int) int {
  return x + y
}
```

`add`可以赋值给`calculation`类型的变量:

```go
var c calculation
c = add
```

#### <font color=red>函数类型变量</font>

我们可以声明函数类型的变量并且为该变量赋值：

```go
package main
 
import "fmt"
 
func add(x, y int) int {
	return x + y
}
 
func sub(x, y int) int {
	return x - y
}
 
type calculation func(int, int) int
 
func main() {
	var c calculation  // 声明一个calculation类型的变量c
	c = add  // 把add赋值给c
	fmt.Printf("type of c:%T \n", c)  // type of c:main.calculation
	fmt.Println(c(1, 2))  // 3 像调用add一样调用c
 
	f := add  // 将函数add赋值给变量f
	fmt.Printf("type of f:%T \n", f)  // type of f:func(int, int) int
	fmt.Println(f(10, 20))  // 30 像调用add一样调用f
}
```

### <center>高阶函数</center>

高阶函数分为函数作为参数和函数作为返回值两部分。

#### <font color=red>函数作为参数</font>

```go
package main
 
import "fmt"
 
func f1() {
	fmt.Println("hello ty")
}
 
func f2() int {
	return 2
}
 
func f3(x func()) {
	x()  // 调用x
}
 
func f4(x, y int) int {
	return x + y
}
 
func main() {
	// typef1:func(), typef2:func() int f1, f2函数类型不同
	fmt.Printf("typef1:%T, typef2:%T \n", f1, f2)
 
	// typef3:func(func()), typef4:func(int, int) int
	fmt.Printf("typef3:%T, typef4:%T \n", f3, f4)
	f3(f1)
}
```

#### <font color=red>函数作为返回值</font>

```go
package main
 
import "fmt"
 
func f2() int {
	return 2
}
 
func f5(a, b int) int {
	return a + b
}
 
func ff(x func() int) func(int, int) int {
	return f5
}
 
func main() {
	f7 := ff(f2)  // f7为f5的内存地址
	fmt.Println(f7)
}
```

### <center>匿名函数和闭包</center>

#### <font color=red>匿名函数</font>

`golang`中函数内部不能再像其他语言那样定义函数，函数内部无法声明带名字的函数，只能定义匿名函数。

```go
func(参数)(返回值) {
  函数体
}
```

匿名函数因为没有函数名，所以没办法像普通函数那样调用，匿名函数需要保存到某个变量或作为立即执行函数:

```go
func main() {
	// 将匿名函数保存到变量
	add := func (x, y int) int {
		return x + y
	}
	add(10, 20)  // 通过变量调用匿名函数
 
	// 自执行函数：匿名函数定义完加（）直接执行
	func(x, y int) {
		fmt.Println( x + y )
	}(10, 20)
}
```

匿名函数多用于实现回掉函数和闭包。

#### <font color=red>闭包</font>

闭包指一个函数与其相关的引用环境组合而成的实体，简单来说，闭包=函数+引用环境。

闭包的意义:返回的函数，不仅仅是一个函数，在该函数外还包裹了一层作用域，这使得该函数无论在何处调用会优先使用自己外层包裹的作用域。eg:

```go
package main
 
import "fmt"
 
func adder() func(int) int {
	var x int
	return func(y int) int {
		x += y
		return x
	}
}
 
func main() {
	var f = adder()
	fmt.Println(f(10))  // 10
	fmt.Println(f(20))  // 30
	fmt.Println(f(30))  // 60
 
	f1 := adder()
	fmt.Println(f1(40))  // 40
	fmt.Println(f1(50))  // 90
 
}
```

#### ###  <center>defer语句</center>

`golang`中的`defer`语句会将其后面跟随的语句进行延迟处理。在`defer`归属的函数即将返回时将延迟处理的语句按`defer`定义的逆序执行，也就是说先被`defer`的语句最后被执行。eg:

```go
package main
 
import (
	"fmt"
)
 
func main() {
	fmt.Println("start")
	defer fmt.Println(1)
	defer fmt.Println(2)
	defer fmt.Println(3)
	fmt.Println("end")
}
```

输出结果:

```go
start
end
3
2
1
```

由于`defer`语句延迟调用的特性，所以`defer`语句能非常方便地处理资源释放问题。比如:资源清理、文件关闭、解锁记录时间等。

#### <font color=red>defer执行时机</font>

`golang`中函数的`return`语句在底层并不是原子操作，它分为给返回值赋值和`ret`指令两步。而`defer`语句执行的时机就在返回值赋值操作后，`ret`指令执行前，具体如下图所示:

![](/images/defer.png)

#### <font color=red>defer经典案例</font>

```go
package main
 
import (
	"fmt"
)
 
func f1() int {
	x := 5
	defer func() {
		x ++
	}()
	return x  // 5
}
 
func f2() (x int) {
	defer func(){
		x ++
	}()
	return 5  // 6
}
 
func f3() (y int) {
	x := 5
	defer func() {
		x ++
	}()
	return x  // 5
}
 
func f4() (x int) {
	defer func(x int) {
		x ++
	}(x)
	return 5  // 5
}
 
func main() {
	fmt.Println(f1())
	fmt.Println(f2())
	fmt.Println(f3())
	fmt.Println(f4())
}
```

### <center>内置函数</center>

|    内置函数    |                             介绍                             |
| :------------: | :----------------------------------------------------------: |
|     close      |                     主要用来关闭channel                      |
|      len       |      用来求长度，比如string、array、slice、map、channel      |
|      new       | 用来分配内存，主要用来分配值类型，比如int、struct。返回值是指针 |
|      make      | 用来分配内存，主要用来分配引用类型，比如channel、map、slice  |
|     append     |                 用来追加元素到数组、slice中                  |
|     delete     |                    用来删除map中的键值对                     |
| panic和recover |                        用来做错误处理                        |

### <center>panic/recover</center>

`golang`中目前没有异常机制，但是可以使用`panic/recover`模式来处理错误。`panic`可以在任何地方引发，但`recover`只有在`defer`调用的函数中有效。eg：

```go
package main
 
import "fmt"
 
func funcA() {
	fmt.Println("func A")
}
 
func funcB() {
	panic("panic in B")
}
 
func funcC() {
	fmt.Println("func C")
}
 
func main() {
	funcA()
	funcB()
	funcC()
}
```

结果为:

```go
func A
panic: panic in B
 
goroutine 1 [running]:
main.funcB(...)
        E:/桌面文件/github.com/test/test_panic/main.go:10      
main.main()
        E:/桌面文件/github.com/test/test_panic/main.go:19 +0xa5
exit status 2
```

程序运行期间函数`funcB`引发了`panic`导致程序崩溃，这个时候我们就可以通过`recover`将程序恢复，继续向后执行。

```go
package main
 
import "fmt"
 
func funcA() {
	fmt.Println("func A")
}
 
func funcB() {
	defer func() {
		err := recover()
		// 如果程序出现了panic错误，可以通过recover恢复过来
		if err != nil {
			fmt.Println("recover in B")
			fmt.Println("func B")
		}
	}()
	panic("panic in B")
}
 
func funcC() {
	fmt.Println("func C")
}
 
func main() {
	funcA()
	funcB()
	funcC()
}
```

结果为:

```go
func A
recover in B
func B      
func C
```

<font color=red>注意</font>:

- Recover()必须搭配`defer`使用。
- `defer`一定要在可能引发`panic`的语句之前定义。
- 一般不推荐修复，该`panic`就`panic`。

### <center>函数传参永远是拷贝补充</center>

```go
package main
 
import "fmt"
 
func f1_int(x int) {
	fmt.Printf("%p \n", &x)
}
 
func f2_slice(y []int) {
	fmt.Printf("%p \n", &y)
}
 
func main() {
	m := 30
	fmt.Printf("%p \n", &m)  // 0xc0000120a0
	f1_int(m)                // 0xc0000120a8
	
        // 可以发现引用类型作为函数参数也是拷贝
	n := []int{1, 2, 3}      // 0xc00000a420
	fmt.Printf("%p \n", &n)   // 0xc00000a423
	f2_slice(n)
}
```









