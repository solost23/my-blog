---
title: golang基础之变量与常量
date: 2021-12-09 11:00
tags:
    - golang
---



变量与常量是编程中不可缺少的一部分。     

###   <center>表识符与关键字</center>

#### <font color=red>标识符</font>

在编程语言中标识符就是程序员定义的具有特殊意义的词，比如变量名、常量名、函数名等。golang中标识符由字母、数字和下划线组成，并且只能以字母和下划线开头。golang中推荐使用<font color=red>小驼峰体</font>命名，例如：studentName。

#### <font color=red>关键字与保留字</font>

关键字是指编程语言中预先定义好的具有特殊含义的标识符。<font color=red>关键字</font>和<font color=red>保留字</font>都不建议用作变量名。     

golang中有25个关键字：

```go
break	default	func	interface	select	
case	defer	go	map	struct
chan	else	goto	package	switch	
const	fallthrough	if	range	type
continue	for	import	return	var	
```

此外，golang还有37个保留字:

```go
Constants:	true	false	iota	nil

Types:	int	int8	int16	int32	int64
				uint	uint8	uint16	uint32	uint64
				float32	float64	complex128	complex64
				bool	byte	rune	string	error
Functions:	make	len	cap	new	append	copy	close	delete
						complex	real	imag
						panic	recover
```

### <center>变量</center>

#### <font color=red>变量的来历</font>

程序运行过程中的数据都是保存在内存中的，我们想要在代码中操作某个数据时就需要去内存中找到这个变量，但是我们直接在代码中通过内存地址去操作变量的话，代码的可读性会非常差而且容易出错，所以就利用变量将这个数据的内存地址保存起来，以后直接通过这个变量就能找到内存中对应的数据了。     

#### <font color=red>变量类型</font>

变量(Variable)的功能是存储数据，不同的变量保存的数据类型可能会不一样。经过半个多世纪的发展，编程语言已经基本形成了一套固定的类型，常见的变量类型有:整型、浮点型、布尔型等。     

#### <font color=red>变量声明</font>

golang是<font color=red>强类型语言</font>，golang中每个变量都有自己的类型，并且<font color=red>变量必须经过声明才能开始使用</font>，同一作用域内不支持重复声明。并且golang的<font color=red>非全局变量声明后必须使用</font>，否则无法编译。

##### <font color=red>标准声明</font>

golang的变量声明格式为:

```go
var variableName variableType
```

##### <font color=red>批量声明</font>

```go
var (
	variableName1 variableType1
  variableName2 variableType2
  variableName3 variableType3
  ...
)
```

#### <font color=red>变量的初始化</font>

golang在声明变量时会<font color=red>自动对变量对应的内存区域进行初始化操作</font>(引用类型不自动初始化)。每个变量会被初始化成其类型的零值。

##### <font color=red>类型推导</font>(声明+初始化)

有时候会将变量的类型省略，这个时候编译器会根据等号右边的值来推导变量的类型完成初始化。

```go
var name = "ty"
var age  = "20"
```

##### <font color=red>短声明变量</font>(声明+初始化 常用)

在函数内部，可以使用更简单的方式声明并初始化变量。

```
package main

import (
	"fmt"
)

// 全局变量
var m = 100

func main() {
	m := 10
	// 短声明
	n := 100
}
```

##### <font color=red>匿名变量</font>

在使用多重赋值时，如果想要<font color=red>忽略</font>某个值，可以使用匿名变量(anonymous variable)。匿名变量用一个下划线表示。匿名变量不占用命名空间，不分配内存，所以匿名变量之间不存在重复声明。

### <center>常量</center>

相对于便量，常量是恒定不变的值，多用于定义程序运行期间不会改变的值。常量的声明和变量声明非常相似，只是把var换为const，常量在定义的时候必须赋值。

```go
const pi = 3.1415
const e float32 = 2.7182
```

##### <font color=red>批量声明</font>

```go
const (
	pi float64 = 2.1415
	e = 2.7182
)
```

const同时声明多个变量时，如果省略了值则表示和上面一行的值相同，<font color=red>省略了类型表示和上面的类型一样</font>。

##### <font color=red>iota</font>

iota是golang的常量计数器，只能在常量表达式中使用。iota在const关键字出现时被重置为0。const中每新增一行常量声明将使iota计数一次。

Eg:

```go
const (
	n1 = iota  // 0
	n2				 // 1
	n3         // 2
	n4         // 3)
```

```go
const (
	n1 = iota  // 0
	n2 = 100   // 100
	n3 = iota  // 2
  n4 =       // 3
)
```

```go
const (
	n1 = iota  // 0
	n2         // 1
	_     
	n4         // 3
)
```

###### <font color=red>定义数量级</font>

Eg:

```go
const (
	_ = iota
	KB = 1 << (10 * iota)
	MB = 1 << (10 * iota)
	GB = 1 << (10 * iota)
	TB = 1 << (10 * iota)
	PB = 1 << (10 * iota)
)
```

```go
const (
	a, b = iota + 1, iota + 1  // 1, 2
	c, d                       // 2, 3
	e, f                       // 3, 4
)
```









