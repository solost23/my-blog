---
title: golang基础之网络编程
date: 2021-12-15 11:00
tags:
    - golang
---

### <center>互联网协议介绍</center>

互联网的核心是一系列协议，总称为"互联网协议"，正是这些协议规定了电脑如何连接和组网。我们理解了这些协议就理解了互联网的原理，由于这些协议太庞大和复杂，没办法一概而全，所以只介绍日常开发中接触较多的几个协议。

### <center>互联网分层模型</center>

互联网的逻辑实现被分为好几层，每一层都有自己的功能。用户接触到的只是最上面的一层，根本不会感觉到下面的几层。

![](images/osi.png)

如上图所示，互联网按照不同的模型划分会有不同的分层，但是无论按照什么模型去划分，越往上的层越靠近用户，越往下的层越靠近硬件。在软件开发中我们使用最多的是上图中将互联网划分为五层的模型。

#### <font color=red>物理层</font>

我们的电脑要与外界互联网通信，需要先把电脑连接网络，我们可以用双绞线、光纤、无线电波等方式。这就叫做” 实物理层”，它就是把电脑连接起来的物理手段。它主要规定了网络的一些电气特性，作用是负责传送 0 和 1 的电信号。

#### <font color=red>数据链路层</font>

单纯的 0 和 1 没有任何意义，所以我们使用者会为其赋予一些特定的含义，规定解读电信号的方式：例如：多少个电信号算一组？每个信号位有何意义？这就是” 数据链接层” 的功能，它在” 物理层” 的上方，确定了物理层传输的 0 和 1 的分组方式及代表的意义。早期的时候，每家公司都有自己的电信号分组方式。逐渐地，一种叫做” 以太网”（Ethernet）的协议，占据了主导地位。

以太网规定，一组电信号构成一个数据包，叫做” 帧”（Frame）。每一帧分成两个部分：标头（Head）和数据（Data）。其中” 标头” 包含数据包的一些说明项，比如发送者、接受者、数据类型等等；” 数据” 则是数据包的具体内容。” 标头” 的长度，固定为 18 字节。” 数据” 的长度，最短为 46 字节，最长为 1500 字节。因此，整个” 帧” 最短为 64 字节，最长为 1518 字节。如果数据很长，就必须分割成多个帧进行发送。

那么，发送者和接受者是如何标识呢？以太网规定，连入网络的所有设备都必须具有” 网卡” 接口。数据包必须是从一块网卡，传送到另一块网卡。网卡的地址，就是数据包的发送地址和接收地址，这叫做 MAC 地址。每块网卡出厂的时候，都有一个全世界独一无二的 MAC 地址，长度是 48 个二进制位，通常用 12 个十六进制数表示。前 6 个十六进制数是厂商编号，后 6 个是该厂商的网卡流水号。有了 MAC 地址，就可以定位网卡和数据包的路径了。

我们会通过 ARP 协议来获取接受方的 MAC 地址，有了 MAC 地址之后，如何把数据准确的发送给接收方呢？其实这里以太网采用了一种很” 原始” 的方式，它不是把数据包准确送到接收方，而是向本网络内所有计算机都发送，让每台计算机读取这个包的” 标头”，找到接收方的 MAC 地址，然后与自身的 MAC 地址相比较，如果两者相同，就接受这个包，做进一步处理，否则就丢弃这个包。这种发送方式就叫做” 广播”（broadcasting）。

#### <font color=red>网络层</font>

按照以太网协议的规则我们可以依靠 MAC 地址来向外发送数据。理论上依靠 MAC 地址，你电脑的网卡就可以找到身在世界另一个角落的某台电脑的网卡了，但是这种做法有一个重大缺陷就是以太网采用广播方式发送数据包，所有成员人手一” 包”，不仅效率低，而且发送的数据只能局限在发送者所在的子网络。也就是说如果两台计算机不在同一个子网络，广播是传不过去的。这种设计是合理且必要的，因为如果互联网上每一台计算机都会收到互联网上收发的所有数据包，那是不现实的。

因此，必须找到一种方法区分哪些 MAC 地址属于同一个子网络，哪些不是。如果是同一个子网络，就采用广播方式发送，否则就采用” 路由” 方式发送。这就导致了” 网络层” 的诞生。它的作用是引进一套新的地址，使得我们能够区分不同的计算机是否属于同一个子网络。这套地址就叫做” 网络地址”，简称” 网址”。

“网络层” 出现以后，每台计算机有了两种地址，一种是 MAC 地址，另一种是网络地址。两种地址之间没有任何联系，MAC 地址是绑定在网卡上的，网络地址则是网络管理员分配的。网络地址帮助我们确定计算机所在的子网络，MAC 地址则将数据包送到该子网络中的目标网卡。因此，从逻辑上可以推断，必定是先处理网络地址，然后再处理 MAC 地址。

规定网络地址的协议，叫做 IP 协议。它所定义的地址，就被称为 IP 地址。目前，广泛采用的是 IP 协议第四版，简称 IPv4。IPv4 这个版本规定，网络地址由 32 个二进制位组成，我们通常习惯用分成四段的十进制数表示 IP 地址，从 0.0.0.0 一直到 255.255.255.255。

根据 IP 协议发送的数据，就叫做 IP 数据包。IP 数据包也分为” 标头” 和” 数据” 两个部分：” 标头” 部分主要包括版本、长度、IP 地址等信息，” 数据” 部分则是 IP 数据包的具体内容。IP 数据包的” 标头” 部分的长度为 20 到 60 字节，整个数据包的总长度最大为 65535 字节。

#### <font color=red>传输层</font>

有了 MAC 地址和 IP 地址，我们已经可以在互联网上任意两台主机上建立通信。但问题是同一台主机上会有许多程序都需要用网络收发数据，比如 QQ 和浏览器这两个程序都需要连接互联网并收发数据，我们如何区分某个数据包到底是归哪个程序的呢？也就是说，我们还需要一个参数，表示这个数据包到底供哪个程序（进程）使用。这个参数就叫做” 端口”（port），它其实是每一个使用网卡的程序的编号。每个数据包都发到主机的特定端口，所以不同的程序就能取到自己所需要的数据。

“端口” 是 0 到 65535 之间的一个整数，正好 16 个二进制位。0 到 1023 的端口被系统占用，用户只能选用大于 1023 的端口。有了 IP 和端口我们就能实现唯一确定互联网上一个程序，进而实现网络间的程序通信。

我们必须在数据包中加入端口信息，这就需要新的协议。最简单的实现叫做 UDP 协议，它的格式几乎就是在数据前面，加上端口号。UDP 数据包，也是由” 标头” 和” 数据” 两部分组成：” 标头” 部分主要定义了发出端口和接收端口，” 数据” 部分就是具体的内容。UDP 数据包非常简单，” 标头” 部分一共只有 8 个字节，总长度不超过 65,535 字节，正好放进一个 IP 数据包。

UDP 协议的优点是比较简单，容易实现，但是缺点是可靠性较差，一旦数据包发出，无法知道对方是否收到。为了解决这个问题，提高网络可靠性，TCP 协议就诞生了。TCP 协议能够确保数据不会遗失。它的缺点是过程复杂、实现困难、消耗较多的资源。TCP 数据包没有长度限制，理论上可以无限长，但是为了保证网络的效率，通常 TCP 数据包的长度不会超过 IP 数据包的长度，以确保单个 TCP 数据包不必再分割。

#### <font color=red>应用层</font>

应用程序收到” 传输层” 的数据，接下来就要对数据进行解包。由于互联网是开放架构，数据来源五花八门，必须事先规定好通信的数据格式，否则接收方根本无法获得真正发送的数据内容。” 应用层” 的作用就是规定应用程序使用的数据格式，例如我们 TCP 协议之上常见的 Email、HTTP、FTP 等协议，这些协议就组成了互联网协议的应用层。

如下图所示，发送方的 HTTP 数据经过互联网的传输过程中会依次添加各层协议的标头信息，接收方收到数据包之后再依次根据协议解包得到数据。

![](images/httptcpip.png)

### ### <center>Socker编程</center>

Socket 是 BSD UNIX 的进程通信机制，通常也称作 “套接字”，用于描述 IP 地址和端口，是一个通信链的句柄。Socket 可以理解为 TCP/IP 网络的 API，它定义了许多函数或例程，程序员可以用它们来开发 TCP/IP 网络上的应用程序。电脑上运行的应用程序通常通过 “套接字” 向网络发出请求或者应答网络请求。

### <center>Socker图解</center>

Socket 是应用层与 TCP/IP 协议族通信的中间软件抽象层。在设计模式中，Socket 其实就是一个门面模式，它把复杂的 TCP/IP 协议族隐藏在 Socket 后面，对用户来说只需要调用 Socket 规定的相关函数，让 Socket 去组织符合指定的协议数据然后进行通信。

![](images/socket.png)

### <center>golang实现TCP通信</center>

#### <font color=red>TCP协议</font>

TCP/IP（Transmission Control Protocol/Internet Protocol）即传输控制协议 / 网间协议，是一种面向连接（连接导向）的、可靠的、基于字节流的传输层（Transport layer）通信协议，因为是面向连接的协议，数据像流水一样传输，会存在粘包问题。

#### <font color=red>TCP服务端</font>

一个 TCP 服务端可以同时连接很多个客户端，例如世界各地的用户使用自己的电脑上的浏览器访问京东网。因为 Go 语言中创建多个 goroutine 实现并发非常方便和高效，所以我们可以每建立一次链接就创建一个 goroutine 去处理。

TCP 服务端程序的处理流程：

1. 监听端口。
2. 接收客户端请求建立链接。
3. 创建 goroutine 处理链接。

我们使用 Go 语言的 net 包实现的 TCP 服务端代码如下：

```go
// tcp/server/main.go
 
// TCP server端
 
// 处理函数
func process(conn net.Conn) {
	defer conn.Close() // 关闭连接
	for {
		reader := bufio.NewReader(conn)
		var buf [128]byte
		n, err := reader.Read(buf[:]) // 读取数据
		if err != nil {
			fmt.Println("read from client failed, err:", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client端发来的数据：", recvStr)
		conn.Write([]byte(recvStr)) // 发送数据
	}
}
 
func main() {
        // 1.监听端口
	listen, err := net.Listen("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	for {
                // 2.接收客户端请求建立链接
		conn, err := listen.Accept() // 建立连接
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
                // 3.创建goroutine处理链接
		go process(conn) // 启动一个goroutine处理连接
	}
}
```

#### <font color=red>TCP客户端</font>

一个 TCP 客户端进行 TCP 通信的流程如下：

1. 建立与服务端的链接
2. 进行数据收发
3. 关闭连接

使用 Go 语言的 net 包实现的 TCP 客户端代码如下：

```go
// tcp/client/main.go
 
// 客户端
func main() {
        // 1.建立与服务端的链接
	conn, err := net.Dial("tcp", "127.0.0.1:20000")
	if err != nil {
		fmt.Println("err :", err)
		return
	}
	defer conn.Close() // 关闭连接
        // 2.进行数据收发
	inputReader := bufio.NewReader(os.Stdin)
	for {
		input, _ := inputReader.ReadString('\n') // 读取用户输入
		inputInfo := strings.Trim(input, "\r\n")
		if strings.ToUpper(inputInfo) == "Q" { // 如果输入q就退出
			return
		}
		_, err = conn.Write([]byte(inputInfo)) // 发送数据
		if err != nil {
			return
		}
		buf := [512]byte{}
		n, err := conn.Read(buf[:])
		if err != nil {
			fmt.Println("recv failed, err:", err)
			return
		}
		fmt.Println(string(buf[:n]))
	}
}
```

### <center>TCP粘包</center>

#### <font color=red>粘包示例</font>

服务端代码如下：

```go
// socket_stick/server/main.go
 
func process(conn net.Conn) {
	defer conn.Close()
	reader := bufio.NewReader(conn)
	var buf [1024]byte
	for {
		n, err := reader.Read(buf[:])
		if err == io.EOF {
			break
		}
		if err != nil {
			fmt.Println("read from client failed, err:", err)
			break
		}
		recvStr := string(buf[:n])
		fmt.Println("收到client发来的数据：", recvStr)
	}
}
 
func main() {
 
	listen, err := net.Listen("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn)
	}
}
```

客户端代码如下:

```go
// socket_stick/client/main.go
 
func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("dial failed, err", err)
		return
	}
	defer conn.Close()
	for i := 0; i < 20; i++ {
		msg := `Hello, Hello. How are you?`
		conn.Write([]byte(msg))
	}
}
```

将上面的代码保存后，分别编译，先启动服务端再启动客户端，可以看到服务端输出结果如下：

```go
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?Hello, Hello. How are you?
收到client发来的数据： Hello, Hello. How are you?Hello, Hello. How are you?
```

客户端分 10 次发送的数据，在服务端并没有成功的输出 10 次，而是多条数据 “粘” 到了一起。

#### <font color=red>为什么会产生粘包</font>

主要原因就是 tcp 数据传递模式是流模式，在保持长连接的时候可以进行多次的收和发。

“粘包” 可发生在发送端也可发生在接收端：

1. 由 Nagle 算法造成的发送端的粘包：Nagle 算法是一种改善网络传输效率的算法。简单来说就是当我们提交一段数据给 TCP 发送时，TCP 并不立刻发送此段数据，而是等待一小段时间看看在等待期间是否还有要发送的数据，若有则会一次把这两段数据发送出去。

2. 接收端接收不及时造成的接收端粘包：TCP 会把接收到的数据存在自己的缓冲区中，然后通知应用层取数据。当应用层由于某些原因不能及时的把 TCP 的数据取出来，就会造成 TCP 缓冲区中存放了几段数据。

   

#### <font color=red>解决方法</font>

出现 “粘包” 的关键在于接收方不确定将要传输的数据包的大小，因此我们可以对数据包进行封包和拆包的操作。

封包：封包就是给一段数据加上包头，这样一来数据包就分为包头和包体两部分内容了（过滤非法包时封包会加入 “包尾” 内容）。包头部分的长度是固定的，并且它存储了包体的长度，根据包头长度固定以及包头中含有包体长度的变量就能正确的拆分出一个完整的数据包。

我们可以自己定义一个协议，比如数据包的前 4 个字节为包头，里面存储的是发送的数据的长度。

```go
// socket_stick/proto/proto.go
package proto
 
import (
	"bufio"
	"bytes"
	"encoding/binary"
)
 
// Encode 将消息编码
func Encode(message string) ([]byte, error) {
	// 读取消息的长度，转换成int32类型（占4个字节）
	var length = int32(len(message))
	var pkg = new(bytes.Buffer)
	// 写入消息头
	err := binary.Write(pkg, binary.LittleEndian, length)
	if err != nil {
		return nil, err
	}
	// 写入消息实体
	err = binary.Write(pkg, binary.LittleEndian, []byte(message))
	if err != nil {
		return nil, err
	}
	return pkg.Bytes(), nil
}
 
// Decode 解码消息
func Decode(reader *bufio.Reader) (string, error) {
	// 读取消息的长度
	lengthByte, _ := reader.Peek(4) // 读取前4个字节的数据
	lengthBuff := bytes.NewBuffer(lengthByte)
	var length int32
	err := binary.Read(lengthBuff, binary.LittleEndian, &length)
	if err != nil {
		return "", err
	}
	// Buffered返回缓冲中现有的可读取的字节数。
	if int32(reader.Buffered()) < length+4 {
		return "", err
	}
 
	// 读取真正的消息数据
	pack := make([]byte, int(4+length))
	_, err = reader.Read(pack)
	if err != nil {
		return "", err
	}
	return string(pack[4:]), nil
}
```

接下来在服务端和客户端分别使用上面定义的 proto 包的 Decode 和 Encode 函数处理数据。

服务端代码如下：

```go
// socket_stick/server2/main.go
 
func process(conn net.Conn) {
	defer conn.Close()
	reader := bufio.NewReader(conn)
	for {
		msg, err := proto.Decode(reader)
		if err == io.EOF {
			return
		}
		if err != nil {
			fmt.Println("decode msg failed, err:", err)
			return
		}
		fmt.Println("收到client发来的数据：", msg)
	}
}
 
func main() {
 
	listen, err := net.Listen("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		conn, err := listen.Accept()
		if err != nil {
			fmt.Println("accept failed, err:", err)
			continue
		}
		go process(conn)
	}
}
```

客户端代码如下：

```go
// socket_stick/client2/main.go
 
func main() {
	conn, err := net.Dial("tcp", "127.0.0.1:30000")
	if err != nil {
		fmt.Println("dial failed, err", err)
		return
	}
	defer conn.Close()
	for i := 0; i < 20; i++ {
		msg := `Hello, Hello. How are you?`
		data, err := proto.Encode(msg)
		if err != nil {
			fmt.Println("encode msg failed, err:", err)
			return
		}
		conn.Write(data)
	}
}
```

### <center>golang实现UDP通信</center>

UDP 协议（User Datagram Protocol）中文名称是用户数据报协议，是 OSI（Open System Interconnection，开放式系统互连）参考模型中一种<font color=red>无连接</font>的传输层协议，不需要建立连接就能直接进行数据发送和接收，属于不可靠的、没有时序的通信，但是 UDP 协议的实时性比较好，通常用于视频直播相关领域。

#### <font color=red>UDP服务端</font>

使用 Go 语言的 net 包实现的 UDP 服务端代码如下：

```go
// UDP/server/main.go
 
// UDP server端
func main() {
	listen, err := net.ListenUDP("udp", &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("listen failed, err:", err)
		return
	}
	defer listen.Close()
	for {
		var data [1024]byte
		n, addr, err := listen.ReadFromUDP(data[:]) // 接收数据
		if err != nil {
			fmt.Println("read udp failed, err:", err)
			continue
		}
		fmt.Printf("data:%v addr:%v count:%v\n", string(data[:n]), addr, n)
		_, err = listen.WriteToUDP(data[:n], addr) // 发送数据
		if err != nil {
			fmt.Println("write to udp failed, err:", err)
			continue
		}
	}
}
```

#### <font color=red>UDP客户端</font>

使用 Go 语言的 net 包实现的 UDP 客户端代码如下：

```go
// UDP 客户端
func main() {
	socket, err := net.DialUDP("udp", nil, &net.UDPAddr{
		IP:   net.IPv4(0, 0, 0, 0),
		Port: 30000,
	})
	if err != nil {
		fmt.Println("连接服务端失败，err:", err)
		return
	}
	defer socket.Close()
	sendData := []byte("Hello server")
	_, err = socket.Write(sendData) // 发送数据
	if err != nil {
		fmt.Println("发送数据失败，err:", err)
		return
	}
	data := make([]byte, 4096)
	n, remoteAddr, err := socket.ReadFromUDP(data) // 接收数据
	if err != nil {
		fmt.Println("接收数据失败，err:", err)
		return
	}
	fmt.Printf("recv:%v addr:%v count:%v\n", string(data[:n]), remoteAddr, n)
}
```

