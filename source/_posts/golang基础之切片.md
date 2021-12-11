---
title: golang基础之切片
---

本文主要介绍`golang`中切片以及它的基本使用。

由于数组的长度固定，所以有很多局限性，eg:

```go
func arraySum(x [3]int) int {
  sum := 0
  for _, value := range x {
    sum += value
  }
  return sum
}
```

这个求和函数只能接收`[3]int`类型，其他的都不支持，再比如:

```go
a := [3]int{1, 2, 3, }
```

这个数组中已经有三个元素，不能继续再继续向数组中添加元素。

### <center>定义</center>

```go
var sliceName []T
```

特性:

- 切片是一个<font color=red>拥有相同类型元素的可变长度序列</font>。它是基于数组类型做的一层封装。它非常灵活，支持自动扩容。
- 切片是一个<font color=red>引用类型</font>，它的内部结构包含地址、长度和容量。
- 切片一般作用于快速操作一块数据集合。
- 切片必须初始化，否则为未申请内存，切片为`nil`，无法使用。

Eg:

```go
func main() {
  // 声明一个字符串切片
  var a []string
  // 声明一个整型切片并初始化
  var b = []int{}
  // 声明一个布尔型切片并初始化
  var c = []bool{false, true}
  
  // []
  fmt.Println(a)
  // []
  fmt.Println(b)
  // [false true]
  fmt.Println(c)
  // true
  fmt.Println(a == nil)
  // false
  fmt.Println(b == nil)
  // false
  fmt.Println(c == nil)
}
```

注意:切片是引用类型，不支持直接比较，只能和`nil`比较。

### <center>长度和容量</center>

切片拥有自己的长度和容量，我们可以通过使用内置的`len()`函数求长度，使用内置的`cap()`函数求切片容量。切片的长度为`high - low + 1`，即当前切片中的元素个数。切片的容量是指底层数组从切片的第一个元素到底层数组最后一个元素的数量。

### <center>切片表达式</center>

切片表达式从字符串、数组、指向数组或切片的指针构造<font color=red>子字符串或切片</font>。它有两种变体:一种指定`low`和`high`两个索引界限值的简单的形式，另一种是除了`low`和`high`索引界限值外还指定容量的完整形式。

#### <font color=red>简单的切片表达式</font>

```go
func main() {
  a := [5]int{1, 2, 3, 4, 5, }
  s := a[1:3]
  
  // s:[2 3] len(s):2 cap(s):4
  fmt.Printf("%s:%v len(s):%v cap(s):%v \n", s, len(s), cap(s))
  
  // 等同于 a[2:len(a)]
  a[2:]
  // 等同于 a[0:3]
  a[:3]
  // 等同于 a[0:len(a)]
  a[:]
}
```

<font color=red>注意</font>:对于数组或字符串，如果`0<=low<=high<=len(a)`，则索引合法，否则就会索引越界(out of range)。对切片再执行切片表达式时`high`的上限边界是切片容量`cap(a)`，而不是长度。常量索引必须是非负的，并且可以用`int`型表示;对于数组或常量字符串，常量索引也必须在有效范围内。如果`low`与`high`都是常数，它们必须满足`low<=high`。如果索引在运行时超出范围，就会发生运行时`panic`。

```go
func main() {
  a := [5]int{1, 2, 3, 4, 5, }
  s := a[1:3]
  
  // s:[2 3] len(s):2 cap(s):4
  fmt.Printf("s:%v len(s):%v cap(s):%v \n", s, len(s), cap(s))
  // s2[5] len(s2):1 cap(s2):1
  fmt.Printf("s2:%v len(s2):%v cap(s2):%v \n", s2, len(s2), cap(s2))
}
```

#### <font color=red>完整切片表达式</font>

对于数组，指向数组的指针和切片支持完整切片表达式。

```go
a[low: high: max]
```

上面的代码会构造与简单切片表达式`a[low:high]`相同类型、相同长度和元素的切片。另外，它将会得到的结果切片的容量设置为`max - low`。在完整切片表达式中只有`low`可以省略；它默认为0，eg:

```
func main() {
	a := [5]int{1, 2, 3, 4, 5, }
	s := a[1:3:4]
	
	// s:[2 3] len(s):2 cap(s):3
	fmt.Printf("s:%v len(s):%v cap(s):%v \n")
}
```

完整切片表达式需要满足`0<=low<=high<=max<=cap(a)`，其他条件和简单切片表达式相同。

### <center>使用make函数构造切片(常用)</center>

```go
// 容量未指定时默认与长度一致
make([]T, size, [cap])
```

Eg:

```go
func main() {
  s := make([]int, 2, 10)
  
  // s:[0 0] len(s):2 cap(s):10
  fmt.Printf("s:%v len(s):%v cap(s):%v \n", s, len(s), cap(s))
}
```

### <center>切片的本质</center>

切片本质就是一个框，框住了一块连续存储的内存。切片是对底层数组的封装，它包含了三个信息：底层数组的指针、切片的长度和切片的容量。

### <center>判断切片是否为空</center>

始终使用`len(s) == 0`来判断，不应该是用`s == nil`来判断，因为切片不为`nil`时也可能为空，`nil`仅表示未分配内存空间。

### <center> 切片的赋值拷贝</center>

Eg:

```go
func main() {
  // [0 0 0]
  s1 := make([]int, 3)
  // 将s1直接赋值给s2,s1和s2共用一个底层数组
  s2 := s1
  s2[0] = 100
  
  // [100 0 0]
  fmt.Println(s1)
  // [100 0 0]
  fmt.Println(s2)
}
```

### <center>切片遍历</center>

```go
func main() {
  s := []int{1, 3, 5, }
  for i := 0; i < len(s); i ++ {
    fmt.Println(i, s[i])
  }
  
  for key, value := range s {
    fmt.Println(key, value)
  }
}
```

### <center>切片进阶操作</center>

#### <font color=red>append方法为切片添加元素</font>

`golang`的内建函数`append`可以为切片动态添加元素。可以依次添加一个元素也可以添加多个元素，还可以添加另一个切片中的元素。eg;

```go
func main() {
  var s []int
  // [1]
  s = append(s, 1)
  // 1 1
  fmt.Println(len(s), cap(s))
  // [1 2 3 4]
  s = append(s, 2, 3, 4)
  
  s2 := []int{5, 6, 7, }
  // [1 2 3 4 5 6 7]
  s = append(s, s2...)
}
```

<font color=red>注意:</font>通过`var`声明的切片可以在`append()`函数直接使用，无需初始化。

```go
var s []int
s = append(s, 1, 2, 3, )
```

每个切片都会指向一个底层数组，这个数组的容量够用就添加新增元素，当底层数组不能容纳新增元素时，切片就会自动按照一定的策略进行扩容，此时该切片指向的底层数组就会更换。扩容操作往往发生在`append`函数调用时，所以我们通常都需要用原变量接收`append`函数的返回值。

### <center>切片的扩容策略</center>

可以通过查看`$GOROOT/src/runtime/slice.go`源码，其中扩容相关代码如下:

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
	newcap = cap
} else {
	if old.len < 1024 {
		newcap = doublecap
	} else {
		// Check 0 < newcap to detect overflow
		// and prevent an infinite loop.
		for 0 < newcap && newcap < cap {
			newcap += newcap / 4
		}
		// Set newcap to the requested cap when
		// the newcap calculation overflowed.
		if newcap <= 0 {
			newcap = cap
		}
	}
}
```

从以上代码可以看出以下内容：

- 首先判断，如果新申请`cap`大于`2`倍的旧`cap`，最终`newcap`就是新申请的`cap`。
- 否则，如果旧切片的`len < 1024`，则最终`newcap`就是旧`cap`的`2`倍。
- 否则，如果旧切片`len >= 1024`，则最终`newcap`从旧`oldcap`开始循环增加原来的`1/4`。直到最终`newcap >= cap`。
- 如果最终`cap`计算值溢出，则`newcap`就是新申请`cap`。

需要注意的是切片扩容还会根据切片中元素的类型不同做出不同的处理，比如`int`和`string`类型的处理方式就不一样。

### <center>使用copy函数复制切片</center>

`golang`内建的`copy`函数可以迅速地将一个切片的数据复制到另外一个切片空间中，`copy`函数的函数签名如下：

```go
func copy(dst, src []Type) int
```

### <center>从切片中删除元素</center>

`golang`中没有删除切片元素的专用方法，我们可以使用切片本身的特性来删除元素。eg:

```go
func main() {
  // 从切片中删除元素
  a := []int{30, 31, 32, 33, 34, 35, 36, 37, }
  // 删除索引为2的元素
  a = append(a[:2], a[3:]...)
  
  // [30 31 33 34 35 36 37]
}
```

### <center>多维切片初始化</center>

Eg:

```go
func main() {
  flag := make([][]bool, 3)
  for i := 0; i < len(flag); i ++ {
    // 初始化flag中的每个一维切片
    flag[i] = make([]bool, 3)
  }
  fmt.Println(flag)
}
```











