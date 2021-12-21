---
title: golang标准库之fmt与格式化字符
---

`fmt`包实现了类似`c`语言`printf`与·`scanf`的格式化`I/O`。主要分为向外输出内容和获取输入内容两大部分。

### <center>向外输出</center>

#### <font color=red>Print</font>

Print 系列函数会将内容输出到系统的标准输出，区别在于 Print 函数直接输出内容，Printf 函数支持格式化输出字符串，Println 函数会在输出内容的结尾添加一个换行符。

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

#### <font color=red>Fprint</font>

Fprint 系列函数会将内容输出到一个 io.Writer 接口类型的变量 w 中，我们通常用这个函数往文件中写入内容：

```go
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

eg:

```go
// 向标准输出写入内容
fmt.Fprintln(os.Stdout, "向标准输出写入内容")
fileObj, err := os.OpenFile("./xx.txt", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
	fmt.Println("打开文件出错，err:", err)
	return
}
name := "ty"
// 向打开的文件句柄中写入内容
fmt.Fprintf(fileObj, "往文件中写如信息：%s", name)
```

注意:只要满足`io.Writer`接口类型都支持写入。

#### <font color=red>Sprint</font>

`Sprint`系列函数会把传入的数据生成并返回一个字符串。

```go
func Sprint(a ...interface{}) string
func Sprintf(format string, a ...interface{}) string
func Sprintln(a ...interface{}) string
```

#### <font color=red>Errorf</font>

`Errorf`函数根据`format`参数生成格式化字符串并返回一个包含该字符串的错误。

```go
func Errorf(format string, a ...interface{}) error
```

通常使用这种方式来自定义错误类型，eg:

```go
err := fmt.Errorf("这是一个错误")
```

Go1.13 版本为 fmt.Errorf 函数新加了一个 % w 占位符用来生成一个可以包裹 Error 的 Wrapping Error。

```go
e := errors.New("原始错误e")
w := fmt.Errorf("Wrap了一个错误%w", e)
```

### <center>格式化占位符</center>

`*printf`系列函数都支持`format`	格式化参数，在这里我们按照占位符将被替换的变量类型划分，方便查询和记忆。

#### <font color=red>通用占位符</font>

| 占位符 |                 说明                 |
| :----: | :----------------------------------: |
|   %v   |           值的默认格式标识           |
|  %+v   | 类似 % v，但输出结构体时会添加字段名 |
|  %#v   |           值的 Go 语法表示           |
|   %T   |             打印值的类型             |
|   %%   |                百分号                |

eg:

```go
fmt.Printf("%v\n", 100)
fmt.Printf("%v\n", false)
o := struct{ name string }{"ty"}
fmt.Printf("%v\n", o)
fmt.Printf("%#v\n", o)
fmt.Printf("%T\n", o)
fmt.Printf("100%%\n")
```

result:

```go
100
false
{ty}
struct { name string }{name:"ty"}
struct { name string }
100%
```

#### <font color=red>布尔型</font>

| 占位符 |     说明      |
| :----: | :-----------: |
|   %t   | true 或 false |

#### <font color=red>整型</font>

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %b   |                         表示为二进制                         |
|   %c   |                   该值对应的 unicode 码值                    |
|   %d   |                         表示为十进制                         |
|   %o   |                         表示为八进制                         |
|   %x   |                   表示为十六进制，使用 a~f                   |
|   %X   |                   表示为十六进制，使用 A~F                   |
|   %U   |         表示为 Unicode 格式：U+1234，等价于 "U+%04X"         |
|   %q   | 该值对应的单引号括起来的 go 语法字符字面值，必要时会采用安全的转义表示 |

eg:

```go
n := 65
fmt.Printf("%b\n", n)
fmt.Printf("%c\n", n)
fmt.Printf("%d\n", n)
fmt.Printf("%o\n", n)
fmt.Printf("%x\n", n)
fmt.Printf("%X\n", n)
```

Result:

```go
1000001
A
65
101
41
41
```

#### <font color=red>浮点数与复数</font>

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %b   |     无小数部分、二进制指数的科学计数法，如 - 123456p-78      |
|   %e   |                 科学计数法，如 - 1234.456+78                 |
|   %E   |                 科学计数法，如 - 123456E+78                  |
|   %f   |              有小数部分但无指数部分，如 123.456              |
|   %F   |                          等价于 % f                          |
|   %g   | 根据实际情况采用 % e 或 % f 格式（以获得更简洁、准确的输出） |
|   %G   | 根据实际情况采用 % E 或 % F 格式（以获得更简洁、准确的输出） |

eg:

```go
f := 12.34
fmt.Printf("%b\n", f)
fmt.Printf("%e\n", f)
fmt.Printf("%E\n", f)
fmt.Printf("%f\n", f)
fmt.Printf("%g\n", f)
fmt.Printf("%G\n", f)
```

result:

```go
6946802425218990p-49
1.234000e+01
1.234000E+01
12.340000
12.34
12.34
```

#### <font color=red>字符串和[]byte</font>

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|   %s   |                  直接输出字符串或者 [] byte                  |
|   %q   | 该值对应的双引号括起来的 go 语法字符串字面值，必要时会采用安全的转义表示 |
|   %x   |          每个字节用两字符十六进制数表示（使用 a~f）          |
|   %X   |          每个字节用两字符十六进制数表示（使用 A~F）          |

eg:

```go
s := "小王子"
fmt.Printf("%s\n", s)
fmt.Printf("%q\n", s)
fmt.Printf("%x\n", s)
fmt.Printf("%X\n", s)
```

Result:

```go
小王子
"小王子"
e5b08fe78e8be5ad90
E5B08FE78E8BE5AD90
```

#### <font color=red>指针</font>

| 占位符 |              说明               |
| :----: | :-----------------------------: |
|   %p   | 表示为十六进制，并加上前导的 0x |

eg:

```go
a := 10
fmt.Printf("%p\n", &a)
fmt.Printf("%#p\n", &a)
```

result:

```go
0xc000094000
c000094000
```

#### <font color=red>宽度标识符</font>

宽度通过一个紧跟在百分号后面的十进制数指定，如果未指定宽度，则表示值时除必需之外不作填充。精度通过（可选的）宽度后跟点号后跟的十进制数指定。如果未指定精度，会使用默认精度；如果点号后没有跟数字，表示精度为 0。举例如下：

| 占位符 |        说明        |
| :----: | :----------------: |
|   %f   | 默认宽度，默认精度 |
|  %9f   | 宽度为 9，默认精度 |
|  %.2f  | 默认宽度，精度为 2 |
| %9.2f  | 宽度为 9，精度为 2 |
|  %9.f  | 宽度为 9，精度为 0 |

eg:

```go
n := 12.34
fmt.Printf("%f\n", n)
fmt.Printf("%9f\n", n)
fmt.Printf("%.2f\n", n)
fmt.Printf("%9.2f\n", n)
fmt.Printf("%9.f\n", n)
```

result:

```go
12.340000
12.340000
12.34
    12.34
       12
```

#### <font color=red>其他flag</font>

| 占位符 |                             说明                             |
| :----: | :----------------------------------------------------------: |
|  '+'   | 总是输出数值的正负号；对 % q (%+q) 会生成全部是 ASCII 字符的输出（通过转义）； |
|   ''   | 对数值，正数前加空格而负数前加负号；对字符串采用 % x 或 % X 时（% x 或 % X）会给各打印的字节之间加空格 |
|  '_'   | 在输出右边填充空白而不是默认的左边（即从默认的右对齐切换为左对齐） |
|  '#'   | 八进制数前加 0（%#o），十六进制数前加 0x（%#x）或 0X（%#X），指针去掉前面的 0x（%#p）对 % q（%#q），对 % U（%#U）会输出空格和单引号括起来的 go 字面值。 |
|  '0'   | 使用 0 而不是空格填充，对于数值类型会把填充的 0 放在正负号后面； |

eg:

```go
s := "ty"
fmt.Printf("%s\n", s)
fmt.Printf("%5s\n", s)
fmt.Printf("%-5s\n", s)
fmt.Printf("%5.7s\n", s)
fmt.Printf("%-5.7s\n", s)
fmt.Printf("%5.2s\n", s)
fmt.Printf("%05s\n", s)
```

result:

```go
ty
   ty
ty   
   ty
ty   
   ty
000ty
```

### <center>获取输入</center>

Go 语言 fmt 包下有 fmt.Scan、fmt.Scanf、fmt.Scanln 三个函数，可以在程序运行过程中从标准输入获取用户的输入。

#### <font color=red>fmt.Scan</font>

函数签名如下：

```go
func Scan(a ...interface{}) (n int, err error)
```

- Scan 从标准输入扫描文本，读取由空白符分隔的值保存到传递给本函数的参数，换行符视为空白符。
- 本函数返回成功扫描的数据个数和遇到的任何错误。如果读取的数据个数比提供的参数少，会返回一个错误报告原因。

eg:

```go
func main() {
	var (
		name    string
		age     int
		married bool
	)
	fmt.Scan(&name, &age, &married)
	fmt.Printf("扫描结果 name:%s age:%d married:%t \n", name, age, married)
}
```

将上面的代码编译后在终端执行，在终端依次输入 ty、21 和 false 使用空格分隔。

```go
$ ./scan_demo 
ty 21 false
扫描结果 name:ty age:21 married:false
```

fmt.Scan 从标准输入中扫描用户输入的数据，将以空白符分隔的数据分别存入指定的参数。

#### <font color=red>fmt.Scanf</font>

函数签名如下：

```go
func Scanf(format string, a ...interface{}) (n int, err error)
```

- Scanf 从标准输入扫描文本，根据 format 参数指定的格式去读取由空白符分隔的值保存到传递给本函数的参数中。
- 本函数返回成功扫描的数据个数和遇到的任何错误。

eg:

```go
func main() {
	var (
		name    string
		age     int
		married bool
	)
	fmt.Scanf("1:%s 2:%d 3:%t", &name, &age, &married)
	fmt.Printf("扫描结果 name:%s age:%d married:%t \n", name, age, married)
}
```

将上面的代码编译完后在终端执行，在终端按照指定的格式依次输入 ty、21 和 false。

```go
$ ./scan_demo 
1:ty 2:21 3:false
扫描结果 name:ty age:21 married:false
```

fmt.Scanf 不同于 fmt.Scan 简单的以空格作为输入数据的分隔符，fmt.Scanf 为输入数据指定了具体的输入内容格式，只有按照格式输入数据才会被扫描并存入对应变量。

例如，我们还是按照上个示例中以空格的方式输入，fmt.Scanf 就不能正确扫描到输入的数据。

```go
$ ./scan_demo 
ty 21 false
扫描结果 name: age:0 married:false
```

#### <font color=red>fmt.Scanln(常用)</font>

函数签名如下:

```go
func Scanln(a ...interface{}) (n int, err error)
```

- Scanln 类似 Scan，它在遇到换行时才停止扫描。最后一个数据后面必须有换行或者到达结束位置。
- 本函数返回成功扫描的数据个数和遇到的任何错误。

eg:

```go
func main() {
	var (
		name    string
		age     int
		married bool
	)
	fmt.Scanln(&name, &age, &married)
	fmt.Printf("扫描结果 name:%s age:%d married:%t \n", name, age, married)
}
```

将上面的代码编译后在终端执行，在终端依次输入 ty、21 和 false 使用空格分隔。

```go
$ ./scan_demo 
ty 21 false
扫描结果 name:ty age:21 married:false 
```

fmt.Scanln 遇到回车就结束扫描了，这个比较常用。

#### <font color=red>buffo.NewReader</font>

有时候我们想完整获取输入的内容，而输入的内容可能包含空格，这种情况下可以使用 bufio 包来实现。示例代码如下：

```go
func bufioDemo() {
	reader := bufio.NewReader(os.Stdin) // 从标准输入生成读对象
	fmt.Print("请输入内容：")
	text, _ := reader.ReadString('\n') // 读到换行
	text = strings.TrimSpace(text)
	fmt.Printf("%#v\n", text)
}
```

#### <font color=red>Fscan系列</font>

这几个函数功能分别类似 fmt.Scan、fmt.Scanf、fmt.Scanln 三个函数，只不过它们不是从标准输入中读取数据而是从 io.Reader 中读取数据。

```go
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

#### <font color=red>Sscan系列</font>

这几个函数功能分别类似于 fmt.Scan、fmt.Scanf、fmt.Scanln 三个函数，只不过他们不是从标准输入中读取数据而是从指定字符串中读取数据。

```go
func Sscan(str string, a ...interface{}) (n int, err error)
func Sscanln(str string, a ...interface{}) (n int, err error)
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

