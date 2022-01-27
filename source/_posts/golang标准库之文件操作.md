---
title: golang基础之文件操作
date: 2021-12-21 11:00
tags:
    - golang
---

### <center>打开和关闭文件</center>

`os.Open()`函数能打开一个文件，返回一个`*File`和一个`err`。对得到的文件实例调用`Close()`方法能关闭文件。

```go
package main
 
import (
	"fmt"
	"os"
)
 
func main() {
	// 只读方式打开当前目录下的main.go文件，路径可以填写相对或绝对路径
	file, err := os.Open("./main.go")
	if err != nil {
		fmt.Println("open file failed!, err:", err)
		return
	}
	// 关闭文件
	defer file.Close()
}
```

注意:`os.Open()`打开的文件只读。

### <center>读取文件</center>

### <center>`file.Read()`读取文件(自定义读多少)</center>

#### <font color=red>基本使用</font>

`Read`方法签名如下：

```go
func (f *File) Read(b []byte) (n int, err error)
```

它接收一个字节切片，返回读取的字节数和可能的具体错误，读到文件末尾时会返回 0 和 io.EOF。举个例子：

```go
func main() {
	// 只读方式打开当前目录下的main.go文件
	file, err := os.Open("./main.go")
	if err != nil {
		fmt.Println("open file failed!, err:", err)
		return
	}
	defer file.Close()
	// 使用Read方法读取数据
	var tmp = make([]byte, 128)
	n, err := file.Read(tmp)  // n为本次读取了多少字节
	if err == io.EOF {
		fmt.Println("文件读完了")
		return
	}
	if err != nil {
		fmt.Println("read file failed, err:", err)
		return
	}
	fmt.Printf("读取了%d字节数据\n", n)
	fmt.Println(string(tmp[:n]))
}
```

#### <font color=red>循环读取</font>

```go
func main() {
	// 只读方式打开当前目录下的main.go文件
	file, err := os.Open("./main.go")
	if err != nil {
		fmt.Println("open file failed!, err:", err)
		return
	}
	defer file.Close()
	// 循环读取文件
	var content []byte
	var tmp = make([]byte, 128)
	for {
		n, err := file.Read(tmp)
		if err == io.EOF {
			fmt.Println("文件读完了")
			break
		}
		if err != nil {
			fmt.Println("read file failed, err:", err)
			return
		}
		content = append(content, tmp[:n]...)
	}
	fmt.Println(string(content))
}
```

### <center>`bufio`读取文件(一行一行读)</center>

`bufio`是在`file`的基础上封装了一层`API`，支持更多功能:

```go
package main
 
import (
	"bufio"
	"fmt"
	"io"
	"os"
)
 
// bufio按行读取示例
func main() {
	file, err := os.Open("./xx.txt")
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()
	reader := bufio.NewReader(file)
	for {
		line, err := reader.ReadString('\n') //注意是字符
		if err == io.EOF {
			if len(line) != 0 {
				fmt.Println(line)
			}
			fmt.Println("文件读完了")
			break
		}
		if err != nil {
			fmt.Println("read file failed, err:", err)
			return
		}
		fmt.Print(line)
	}
}
```

### <center>`ioutil`读取整个文件</center>

`io/ioutil`包的`ReadFile`方法能够读取完整的文件，只需要将文件名作为参数传入:

```go
package main
 
import (
	"fmt"
	"io/ioutil"
)
 
// ioutil.ReadFile读取整个文件
func main() {
	content, err := ioutil.ReadFile("./main.go")
	if err != nil {
		fmt.Println("read file failed, err:", err)
		return
	}
	fmt.Println(string(content))
}
```

### <center>文件写入操作</center>

`os.OpenFile()`函数能够以指定模式打开文件，从而实现文件写入相关操作:

```go
func OpenFile(name string, flag int, perm FileMode) (*File, error) {
	...
}
```

其中:

- name: 要打开的文件名.

- flag: 打开文件的模式。模式有以下几种:

  |    模式     |   含义   |
  | :---------: | :------: |
  | os.O_WRONLY |   只写   |
  | os.O_CREATE | 创建文件 |
  | os.O_RDONLY |   只读   |
  |   os.RDWR   |   读写   |
  | os.O_TRUNC  |   清空   |
  | os.O_APPEND |   追加   |

   

- perm: 文件权限,一个八进制数。r(读) 04, w(写) 02, x(执行) 01， 在`windows`下这个参数没有作用。 

### <center>Write与WriteString</center>

```go
func main() {
        // |表示位运算，满足一个即可，详情参考文章末尾位运算补充
	file, err := os.OpenFile("xx.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()
	str := "hello ty\n"
	file.Write([]byte(str))       //写入字节切片数据
	file.WriteString("hello ty\n") //直接写入字符串数据
}
```

### <center>buffo.NewWriter</center>

```go
func main() {
	file, err := os.OpenFile("xx.txt", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0666)
	if err != nil {
		fmt.Println("open file failed, err:", err)
		return
	}
	defer file.Close()
	writer := bufio.NewWriter(file)
	for i := 0; i < 10; i++ {
		writer.WriteString("hello ty\n") //将数据先写入缓存
	}
	writer.Flush() //将缓存中的内容写入文件
}
```

### <center>ioutil.WriteFile</center>

```go
func main() {
	str := "hello 沙河"
	err := ioutil.WriteFile("./xx.txt", []byte(str), 0666)
	if err != nil {
		fmt.Println("write file failed, err:", err)
		return
	}
}
```

## 练习

### <center>借助`io.Copy()`实现一个拷贝文件函数:

```go
// CopyFile 拷贝文件函数
func CopyFile(dstName, srcName string) (written int64, err error) {
	// 以读方式打开源文件
	src, err := os.Open(srcName)
	if err != nil {
		fmt.Printf("open %s failed, err:%v.\n", srcName, err)
		return
	}
	defer src.Close()
	// 以写|创建的方式打开目标文件
	dst, err := os.OpenFile(dstName, os.O_WRONLY|os.O_CREATE, 0644)
	if err != nil {
		fmt.Printf("open %s failed, err:%v.\n", dstName, err)
		return
	}
	defer dst.Close()
	return io.Copy(dst, src) //调用io.Copy()拷贝内容
}
func main() {
	_, err := CopyFile("dst.txt", "src.txt")
	if err != nil {
		fmt.Println("copy file failed, err:", err)
		return
	}
	fmt.Println("copy done!")
}
```

### <center>实现一个cat命令</center>

```go
package main
 
import (
	"bufio"
	"flag"
	"fmt"
	"io"
	"os"
)
 
// cat命令实现
func cat(r *bufio.Reader) {
	for {
		buf, err := r.ReadBytes('\n') //注意是字符
		if err == io.EOF {
			// 退出之前将已读到的内容输出
			fmt.Fprintf(os.Stdout, "%s", buf)
			break
		}
		fmt.Fprintf(os.Stdout, "%s", buf)
	}
}
 
func main() {
	flag.Parse() // 解析命令行参数
	if flag.NArg() == 0 {
		// 如果没有参数默认从标准输入读取内容
		cat(bufio.NewReader(os.Stdin))
	}
	// 依次读取每个指定文件的内容并打印到终端
	for i := 0; i < flag.NArg(); i++ {
		f, err := os.Open(flag.Arg(i))
		if err != nil {
			fmt.Fprintf(os.Stdout, "reading from %s failed, err:%v\n", flag.Arg(i), err)
			continue
		}
		cat(bufio.NewReader(f))
	}
}
```

## 位运算补充

传入一个参数表示多种状态，就可以使用位运算进行穿参，eg:

```go
package main
 
import (
	"fmt"
)
 
const (
	eat   int = 4
	sleep int = 2
	movie int = 1
)
 
func f(x int) {
	fmt.Printf("%b \n", x)  // 101
}
 
/*
111 左边的1表示吃饭，中间的1表示睡觉，右边的1表示电影
吃饭 100
睡觉 010
电影 001
*/
 
func main() {
	// 用一个参数表示两种状态
	f(eat | movie)
}
```

