---
title: golang基础之流程控制
---

流程控制是每种编程语言控制逻辑走向和执行次序的重要部分，流程控制可以说是一门语言的"经脉"。`golang`中最常用的流程控制有 `if`和`for`，而`switch`和`goto`主要是位了简化代码、降低重复代码而生的结构，属于扩展类的流程控制。

### <center>if else选择结构<center>

#### <font color=red>if条件判断基本写法</font>

`golang`中`if`条件判断的格式如下:

```go
if expression1 {
  branch1
} else if expression2 {
  branch2
} else {
  branch3
}
```

`if`判断中的`if`和`else`都是可选的，可以根据实际需要进行选择。

#### <font color=red>if条件判断特殊写法</font>

`if`条件判断还有一种特殊写法，可以在`if`表达式之前添加一个执行语句，再根据变量的值进行判断，eg:

```go
func ifDemo2() {
  if score := 65; score >= 90 {
    fmt.Println("A")
  } else if score > 75 {
    fmt.Println("B")
  } else {
    fmt.Println("C")
  }
}
```

<font color=red>注意:</font>特殊写法中`score`作用域仅为`if`。

### <center>for循环结构</center>

golang中所有的循环类型均可以用`for`关键字完成，golang中只有`for`循环。`for`循环的基本格式如下:

```go
for 初始语句; 条件表达式; 结束语句 {
  循环体语句
}
```

Eg:

```go
func forDemo() {
  for i := 0; i < 10; i ++ {
    fmt.Println(i)
  }
}
```

此时`i`的作用域仅为`for`循环内部，`for`循环的初始语句可以被忽略，但是初始语句后的分号必须要写。eg:

```go
func forDemo2() {
  i := 0
  for ; i < 10; i ++ {
    fmt.Println(i)
  }
}
```

此时`i`的作用域为函数内部。`for`循环的初始语句和结束语句都可以省略。

#### <font color=red>无限循环</font>

```go
for {
  循环语句
}
```

`for`循环可以通过break、goto、return、panic语句强制退出循环。

##### <font color=red>跳出多层循环技巧</font>

设置一个变量`flag := false`，跳出内层循环的同时设置`flag = true`，外层循环判断`flag == true`再决定是否跳出外部循环。

### <center>for range 键值循环</center>

`golang`可以使用`for range`遍历数组、切片、字符串、map以及通道。通过`for range`遍历的返回值有以下规律:

- 数组、切片、字符串返回索引和值。
- `map`返回键和值。
- 通道只返回通道内的值。

eg:

```go
s := "hello world"
for i, v := range s {
  fmt.Println(i, string(v))
}
```

### <center>switch case</center>

使用switch语句可以方便地对大量的值进行条件判断。

```go
func switchDemo1() {
  finger := 3
  switch finger {
    case 1:
    	fmt.Println("大拇指")
  	case 2:
    fmt.Println("食指")
  	case 3:
    	fmt.Println("中指")
  	case 4:
    	fmt.Println("无名指")
    case 5:
    	fmt.Println("小拇指")
    default:
    	fmt.Println("无效输入")
  }
}
```

`golang`规定每个`switch`只能有一个`default`。一个分支可以有多个值，多个`case`值之间表示或的关系，使用英文逗号分隔。分支还可以使用表达式，这时`switch`后面不需要跟判断变量，eg:

```go
func switchDemo2() {
	age := 30
	switch {
		case age < 25:
			fmt.Println("好好学习")
		case age > 25:
			fmt.Println("好好工作")
		case age > 60:
			fmt.Println("好好享受")
		default:
			fmt.Println("活着真好")
	}
}
```

### <center>goto 跳转到指定标签</center>

`golang`中不推荐使用`goto`。

### <center>break 跳出循环</center>

`break`语句可以在语句后面添加标签，表示退出某个标签对应的代码块，标签要求必须定义在对应的`for`、`switch`和`select`的代码块上，eg:

```go
func breakDemo() {
  BREAKDEMO:
  for i := 0; i < 10; i ++ {
    for j ;= 0; j < 10; j ++ {
      if j == 2 {
        break BREAKDEMO
      }
      fmt.Printf("%v-%v \n", i, j)
    }
  }
}
```

### <center>continue 继续下次循环</center>

`continue`语句可以结束当前循环并开启下一次循环，仅限在`for`循环内使用。在`continue`语句后添加标签时，表示开始标签对应的循环，eg:

```go
func continueDemo() {
  forloop1:
  for i ;= 0; i < 5; i ++ {
    // forloop2
    for j := 0; j < 5; j ++ {
      if i == 2 && j == 2 {
        continue forloop1
      }
      fmt.Printf("%v-%v \n", i, j)
    }
  }
}
```

