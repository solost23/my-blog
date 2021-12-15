---
title: golang基础之包
---

在工程化的`golang`开发项目中，`golang`语言的源码复用是建立在`package`的基础上的。

### <center>定义</center>

```go
package 包名
```

<font color=red>注意</font>:

- 一个文件夹下面直接包含的文件只能归属一个`package`，同样一个`package`不能在多个文件夹下。
- 包名可以不和文件夹的名字一样，包名不能包括`_`。
- 包名为`main`的包应为应用程序的入口包，这种包编译后会得到一个可执行文件，而编译不包含`main`包的源代码则不会得到可执行文件。

### <center>可见性</center>

如果想在一个包中引用另外一个包中的标识符，该标识符必须是对外可见的。在`golang`中只需<font color=red>将标识符的首字母大写就可以让标识符对外可见</font>了。eg:

```go
package pkg2
 
import "fmt"
 
// 包变量可见性
 
var a = 100 // 首字母小写，外部包不可见，只能在当前包内使用
 
// 首字母大写外部包可见，可在其他包中使用
const Mode = 1
 
type person struct {  // 首字母小写，外部包不可见，只能在当前包内使用
	name string
}
 
// 首字母大写，外部包可见，可在其他包中使用
func Add(x, y int) int {
	return x + y
}
 
func age() {  // 首字母小写，外部包不可见，只能在当前包内使用
	var Age = 18  // 函数局部变量，外部包不可见（因为这个函数就不可见，所以里面的变量也访问不到），只能在当前函数内使用
}
```

### <center>包的导入</center>

```go
import "包的路径"
```

<font color=red>注意</font>:

- `import`导入语句通常放在文件开头包声明语句的下面。
- 导入的包名需要使用双引号包裹。
- 包名是从`$GOPATH/src/`后开始计算的，使用`/`进行路径分割。
- `golang`禁止循环导入。

#### <font color=red>单行导入</font>

```go
import "包1"
import "包2"
```

#### <font color=red>多行导入</font>

```go
import (
	"包1"
	"包2"
)
```

### <center>自定义包名</center>

导入包时可以为导入的包设置别名。通常用于导入的包名太长或者导入的包的包名冲突。

```go
import 别名 "包的路径"
```

### <center>匿名导入包</center>

如果只希望导入包，而不使用包内部的数据，可以使用匿名导入包。匿名导入的包和其它方式导入的包一样都会被编译到可执行文件中。

```go
import _ "包的路径"
```

### <center>init初始化函数</center>

在`golang`中程序执行时导入包语句会自动触发包内部`init`函数的调用。需要注意的是`init`函数没有参数也没有返回值，`init`函数在程序运行时自动被调用执行，不能在代码中主动调用它，`init`函数一般用于做一些初始化操作。包初始化执行的顺序如下图:

![](/images/init1.png)

#### <font color=red>init函数执行顺序</font>

`golang`会从`main`包开始检查其导入的所有包，每个包又可能导入了其他包。`golang`编译器由此构建出一个树状的包引用关系，然后根据引用顺序决定编译顺序，依次编译这些包的代码。在运行时，最后被导入的包会最先初始化并调用其`init`函数，如下图所示:

![](/images/init02.png)





