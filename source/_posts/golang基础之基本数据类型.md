---
title: golang基础之基本数据类型
date: 2021-12-10 23:00
tags:
    - golang
---

golang中有丰富的数据类型，除了基本的整型、浮点型、布尔型、字符串以外，还有数组、切片、结构体、函数、map、通道等。golang的基本数据类型和其他语言大同小异。

### <center>整型</center>

整型分为一下两个大类：按长度分为：int8、int16、int32、int64，对应的无符号整型：uint8、uint16、uint32、uint64。其中uint8就是我们熟知的byte类型，int16对应C语言中的short型，int64对应C语言中的lang型。

|  类型  |                             描述                             |
| :----: | :----------------------------------------------------------: |
| uint8  |                     无符号8位整型(0~255)                     |
| uint16 |                   无符号16位整型(0~65535)                    |
| uint32 |                 无符号32位整型(0~4294967295)                 |
| uint64 |          无符号 64 位整型（0~18446744073709551615）          |
|  int8  |                 有符号 8 位整型（-128~127）                  |
| int16  |               有符号 16 位整型（-32768~32767）               |
| int32  |          有符号 32 位整数（-2147483648~2147483647）          |
| int64  | 有符号 64 位整型（-9223372036854775808~9223372036854775807） |

#### <font color=red>特殊整型</font>

|  类型   |                          描述                          |
| :-----: | :----------------------------------------------------: |
|  uint   | 32 位操作系统上就是 uint32，64 位操作系统上就是 uint64 |
|   int   |  32 位操作系统上就是 int32，64 位操作系统上就是 int64  |
| uintptr |              无符号整型，用于存放一个指针              |

<font color=red>注意</font>：在使用int和uint类型时，不能假定他是一个32位或64位的整型，而是考虑int和uint可能在不同平台上的差异，声明时明确指定类型，否则<font color=red>默认为int型</font>。

获取对象的长度的内建len()函数返回的长度可以根据不同平台的字节长度进行变化。实际使用中，切片或map的元素数量等都可以用int表示。在涉及到二进制传输、读写文件的结构扫描时，为了保持文件的结构不会受到不同编译目标平台字节长度的影响，不要使用int和uint。

### <center>数字字面量语法(Number literals syntax)</center>

go1.13版本之后引入了数字字面量语法，这样便于开发者以二进制、八进制或十六进制浮点数的格式定义数字。

eg:

```go
package main

import (
	"fmt"
)

func main() {
  // 十进制
  var a int = 10
  fmt.Printf("%d \n", a)  // 10
  fmt.Printf("%b \n", a)  // 1010
  
  // 二进制
  var b int = 0b00101101
  fmt.Printf("%d \n", b)  // 45
  
  // 八进制 0或0o开头
  var c int = 0o77
  fmt.Printf("%o \n", c)  // 077
  
  // 十六进制 以0x开头
  var d int = 0xff
  fmt.Printf("%x \n", c)  // ff
  fmt.Printf("%X \n", c)  // FF
}
```

### <center>浮点型</center>

golang支持两种浮点型数：float32和float64。这两种浮点型数据格式遵循IEEE 754标准，声明时<font color=red>未定义类型默认float64</font>。float32浮点数的最大范围约为3.3e38，可以使用常量定义：math.MaxFloat32。float64浮点数的最大范围约为1.8e308，可以使用常量定义：math.MaxFloat64。

打印浮点数时，可以使用fmt包配合占位符%f:

```go
package main

import (
	"fmt"
  "math"
)

func main() {
  fmt.Printf("%f \n", math.Pi)
  fmt.Printf("%.2f \n", math.Pi)
}
```

### <center>复数</center>

complex和complex128:

```go
var c1 complex64
c1 = 1 + 2i
var c2 complex128
c2 = 2 + 3i
fmt.Println(c1)  // (1+2i)
fmt.Println(c2)  // (2+3i)
```

### <center>布尔值</center>

golang中以bool类型进行声明布尔型数据，布尔型数据只有true和false两个值。

注意：

- 布尔型变量的<font color=red>默认值为false</font>。
- golang中不允许将整型强制转换为布尔型。
- 布尔型无法参与数值运算，也无法与其它类型进行转换。

### <center>字符串</center>

golang中的字符串以原生数据类型出现，使用字符串就像使用其他原生数据类型(int、bool、float32、float64。。。)一样。golang里的字符串内部实现使用了UTF-8编码。字符串的值为<font color=red>双引号</font>中的内容，可以在golang的源码中直接添加非ASCII码，eg:

```go
s1 := "hello"
s2 := "你好"
```

#### <font color=red>字符串转义符</font>

golang的字符串常见转义符包括回车、换行、单双引号、制表符等，如下表：

| 转义符 |                  含义                  |
| :----: | :------------------------------------: |
|   \r   |            回车符(返回行首)            |
|   \n   |    换行符(直接跳到下一行的同列位置)    |
|   \t   |                 制表符                 |
|  \\'   | 单引号(将单引号从定界符转变为普通字符) |
|  \\"   |                 双引号                 |
|  \\\   |                 反斜杠                 |

#### <font color=red>多行字符串</font>

golang中要定义一个多行字符串时，就必须使用反引号字符:

```go
s1 := `
	第一行
	第二行
	第三行
`

fmt.Println(s1)
```

反引号间换行将被作为字符串中的换行，但是所有的转义字符均无效，文本将按照原样输出。

#### <font color=red>字符串的常用操作</font>

|                 方法                 |             介绍              |
| :----------------------------------: | :---------------------------: |
|               len(str)               |            求长度             |
|           \+ 或 fmt.Spritf           |          拼接字符串           |
|            strings.Split             | 分割，返回 strings 类型的切片 |
|           strings.Contains           |         判断是否包含          |
| strings.HasPrefix，strings.HasSuffix |        前缀 / 后缀判断        |
| strings.Index()，strings.Lastindex() |        字串出现的位置         |
| strings.Join(a[]string, sep string)  |           join 操作           |

### <center>byte和rune类型</center>

组成每个字符串的元素叫做“字符”，可以通过遍历或者单个获取字符串元素获得字符。字符用单引号包裹起来，eg:

```go
var a = '中'
var b = 'x'
```

golang的字符有以下两种：

- byte类型，或者叫uint8类型，代表了ASCII码的一个字符。
- rune类型，或者叫int32类型，代表一个UTF-8字符，声明时<font color=red>不指定类型默认是int32</font>。

当需要处理中文、日文或者其他复合字符时，需要用到rune类型。golang使用了特殊的rune类型来处理Unicode，让基于Unicode的文本处理更为方便，也可以使用byte类型进行默认字符串处理，<font color=red>性能</font>和<font color=red>扩展性</font>都有照顾。

```go
// 遍历字符串
func traversalString() {
  s := "hello北京"
  // byte, len(字符串)获得字节长度
  for i := 0; i < len(s); i ++ {
    fmt.Printf("%v(%c) \n", s[i], s[i])
  }
  fmt.Println()
  // rune
  for _, value := range s {
    fmt.Printf("%v(%c) \n", value, value)
  }
}
```

由于UTF-8编码下一个中文汉字由3~4个字节组成，所以我们不能简单地按照字节去遍历一个包含中文的字符串，否则就会输出乱码。<font color=red>字符串底层就是一个byte数组</font>，所以可以和[]byte类型相互转换。<font color=red>字符串是不能修改的，字符串是由byte字节组成，所以字符串的长度是byte字节的长度</font>。rune类型用来表示UTF-8字符，一个rune字符由一个或多个byte组成。

### <center>修改字符串</center>

修改字符串需要先将其转为[]byte或[]rune，完成后再转换为String。无论哪种转换，都会重新分配内存，并复制字节数组，eg:

```go
func changString() {
  s1 := "big"
  // 强制类型转换
  byteS1 := []byte(s1)
  byteS1[0] = 'p'
  // pig
  fmt.Println(string(byteS1))
  
  s2 := "白萝卜"
  runeS2 := []rune(s2)
  runeS2[0] = '红'
  // 红萝卜
  fmt.Println(string(runeS2))
}
```

### <center>类型转换</center>

golang中只有强制类型转换，没有隐式类型转换。该语法只能在两个互相支持转换的类型间使用，eg:

```go
T(表达式)
```

表达式包括变量、复杂算式和函数返回值等。



