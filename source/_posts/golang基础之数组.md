---
title: golang基础之数组
---

本文主要介绍`golang`中`Array`以及它的基本使用。

### <center>定义</center>

数组是一种数据类型元素的集合。在`golang`中数组从声明时就确定，使用时可以修改数组成员，但是数组大小不可变化。

```go
var arrayName [arrayCount]T
```

Eg:

```go
var a [3]int
```

数组长度必须是常量，并且长度是数组类型的一部分。一旦定义，长度不可变。`[5]int`和`[10]int`是不同类型。eg:

```go
var a [3]int
var b [4]int
// 不可以，因为ab类型不同
a = b 
```

### <center>数组初始化</center>

#### <font color=red>方式一</font>

初始化数组时可以使用初始化列表来设置数组元素的值，eg:

```go
func arrayDemo1() {
  // 数组会初始化为int类型的零值
  var testArray [3]int
  // 使用指定的初始值完成初始化
  var numArray = [3]int{1, 2}
  // 使用指定的初始值完成初始化
  var cityArray [3]string = [3]string{"北京", "上海", "深圳"}
  
  // [0 0 0]
  fmt.Println(testArray)
  // [1 2 0]
  fmt.Println(numArray)
  // [北京 上海 深圳]
  fmt.Println(cityarray)
}
```

### <font color=red>方式二</font>

按照上面的方式每次都要确保提供的初始值和数组长度一致，一般情况下我们可以让编译器根据初始值的个数自行推断数组长度，eg:

```go
func arrayDemo2() {
  var testArray [3]int
  var numArray = [...]int{1, 2}
  var cityArray = [...]int{"北京"}
  
  // [0 0 0]
  fmt.Println(testArray)
  // [1 2]
  fmt.Println(numArray)
  // type of numArray:[2]int
  fmt.Printf("type of numArray:%T \n", numArray)
  // [北京]
  fmt.Println(cityArray)
  // type of cityArray:[1]string
  fmt.Printf("type of cityArray:%T \n", cityArray)
}
```

#### <font color=red>方法三</font>

使用索引值来初始化数组，eg:

```go
func arrayDemo3() {
  a := [...]int{1: 1, 3: 5}
  
  // [0 1 0 5]
  fmt.Println(a)
  // type of a:[4]int
  fmt.Printf("type of a:%T \n", a)
}
```

### <center>数组遍历</center>

```go
func arrayDemo4() {
  var a = [...]string{"北京", "上海"，"深圳"}
  // 方法1:for循环遍历
  for i := 0; i < len(a); i ++ {
    fmt.Println(a[i])
  }
  
  // 方法2:for range遍历
  for index, value := range a {
    fmt.Println(index, value)
  }
}
```

### <center>多维数组</center>

golang支持多维数组，这里以二维数组举例。

#### <font color=red>二维数组定义</font>

```go
func arrayDemo5() {
  a := [3][2]string{
    {"北京", "上海"}, 
    {"广州", "深圳"}, 
    {"成都", "重庆"}, 
  }
  
  // [[北京 上海] [广州 深圳] [成都 重庆]]
  fmt.Println(a)
}
```

<font color=red>注意</font>:多维数组只有第一层可以使用`...`来让编译器推导数组长度，eg:

```go
func arrayDemo6() {
  // 支持
  a := [...][2]int {
    {1, 2, }, 
    {3, 4, }, 
    {5, 6, }, 
  }
  
  // 不支持
  b := [3][...]int{
    {1, 2, }, 
    {3, 4, }, 
    {5, 6, }, 
  }
}
```

#### <font color=red>二维数组遍历</font>

```go
func arrayDemo7() {
  // 支持
  a := [...][2]int {
    {1, 2, }, 
    {3, 4, }, 
    {5, 6, }, 
  }
  
  // 遍历二维数组
  for _, v1 := range a {
    for _, v2 := range v1 {
      fmt.Printf("%s \t", v2)
    }
    fmt.Println()
  }
}
```

<font color=red>注意</font>:数组是值类型，赋值和传参会复制整个数组，因此改变副本的值，不会改变本身的值。eg:

```go
func modifyArray1(x [3]int) {
  x[0] = 100
}

func modifyArray2(x [3][2]int) {
  x[2][0] = 100
}

func main() {
  a := [3]int{10, 20, 30, }
  // 在modifyArray1中修改的是a的副本
  modifyArray(a)
  // [10 20 30]
  fmt.Println(a)
  b := [3][2]int {
    {1, 1, }, 
    {1, 1, }, 
    {1, 1, }, 
  }
  modifyArray2(b)
  // [[1 1] [1 1] [1 1]]
  fmt.Println(b)
}
```

- 数组支持`==`, `!=`操作符，因为内存总是被初始化过的。
- [n]*T表示指针数组, *[n]T表示数组指针。