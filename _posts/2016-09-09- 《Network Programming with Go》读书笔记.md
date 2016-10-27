---
layout: post
title:  "《Network Programming with Go》学习笔记"
date:   2016-09-09 20:20:00
description: '《Network Programming with Go》学习笔记'
---

# 《Network Programming with Go》学习笔记

~需要修改。。。。~


## 第一章： Architecture(体系结构)
### Protocol Layers（协议层）

**ISO OSI Protocol**
![Alt text](./1473431812089.png)

每层的功能：
- `网络层`提供交换及路由技术
- `传输层`提供了终端系统之间的数据透明传输，并且负责端到端的错误恢复及流程控制
- `会话层`用来建立、管理、以及终止应用程序之间的连接
- `表现层`提供数据表现差异的独立性（例如加密）
- `应用层`支持应用程序和用户程序

**TCP/IP Protocol**

![Alt text](./1473431977093.png)

### Gateways（网关）

网关是一个统称，它用于连接起一个或多个网络。
- 其中的`中继器`在物理层面上进行操作，它将信息从一个子网复制到另一个子网上。
- `桥接`在数据连接层面上进行操作，它在网络之间复制帧。
- `路由器`在网络层面上进行操作，它不仅在网络之间复制信息，还决定了信息的传输路线。


### Packet encapsulation（数据包封装）

- 在OIS或TCP/IP协议栈层与层之间的通信，是通过将数据包从一个层发送到下一个层，最终穿过整个网络的。
-每一层都有必须保持其自身层的管理信息。
-从上层接收到的数据包在向下传递时，会添加头信息。
- 在接收端，这些头信息会在向上传递时移除。

TFTP（普通文件传输协议）将文件从一台计算机移动到另一台上。它使用IP协议上的UDP协议，该协议可通过以太网发送。看起来就像这样：
![Alt text](./1473434814430.png)

### Connection Models(连接模型)

- Connection oriented 面向连接模型
- Connectionless 无连接模型
- 面向连接模型即为会话建立单个连接，沿着连接进行双向通信，例如 TCP
- 在无连接系统中，消息的发送彼此独立。这类似于普通的邮件。无连接模型的消息可能不按顺序抵达。例子就是IP协议
- 面向连接的传输可通过无连接模型——基于IP的TCP协议建立。
- 无连接传输可通过面向连接模型——基于IP的HTTP协议建立。

### Communications Models(通信模型)

**Communications Models(消息传递)**
- 并发语言大多使用消息传递的机制，比如 Unix的管道
- Parlog 能在并发的进程之间，将任意的逻辑数据结构当做消息来发送
- 消息传递是分布式系统最基本的机制

![Alt text](./1473674193873.png)


### Distributed Computing Models(分布式计算模型)


![Alt text](./1473674216217.png)

考虑分布式系统的组件是否等价,三种模型：
- 点对点（peer-to-peer）: 若两个组件等价，且均可发起并响应信息
- 过滤器（filter）:有一个组件将信息传至另一个组件，它在修改该信息后会传至第三个组件。
例如：中间组件通过SQL从数据库中获取信息，并将其转换为HTML表单提供给第三个组件（它可能是个浏览器）。
- 客户端-服务器(客户端-服务器): 最常见的就是不对等的情况：客户端向服务器发送请求，然后服务端响应

### Client/Server System

![Alt text](./1473674889739.png)

### Client/Server Application
![Alt text](./1473674904487.png)


### Server Distribution（服务器分布）

单一客户端，单个服务器：
![Alt text](./1473675214266.png)

多个客户端，单一服务器：
![Alt text](./1473675235161.png)
主站只需接收请求并处理一次，而无需将它们传递给其它服务器来处理。当客户端可能并发时，这就是个通用的模型

单一客户端，多个服务器，例如当业务逻辑服务器从数据库服务器获取信息时
![Alt text](./1473675269067.png)


### Component Distribution

分解一些应用的一个简单有效的方式就是把它们看做三部分：

Presentation component 表现组件
Application logic 应用逻辑
Data access 数据访问

**表现组件**负责与用户进行交互，即显示数据和采集输入，可以是 GUI 界面，也可以是命令行界面
**应用逻辑组件**负责解释用户的响应，根据应用业务规则，准备查询并管理来自其组件的响应
**数据访问组件**负责存储并检索数据。这一般是通过数据库进行，不过也不一定

![Alt text](./1473676063281.png)

Example: Distributed Database：
Gartner第一种分类
![Alt text](./1473676645585.png)

例如 google map 会下载附近的地图为浏览器中的小型数据库，当用户移动了地图时，可以快速响应


Example: Network File Service 网络文件服务
![Alt text](./1473676805222.png)

Gartner第二种分类允许远程客户端访问已共享的文件系统
这类系统的例子：NFS、Microsoft共享和DCE等等。

Example: Web:
![Alt text](./1473676919558.png)
Gartner第三种分类的一个例子就是Web上的小型Java应用


Example: Terminal Emulation
![Alt text](./1473677181590.png)

Gartner第四种分类就是终端仿真。这允许远程系统在本地系统上作为普通的终端：
Telnet就是最常见的例子。


**Three Tier Models**:
可以有三层、四层甚至多层。下图展示了一些可能的三层模型:
![Alt text](./1473677365117.png)

### Middleware model 中间件模型

![Alt text](./1473677961430.png)


中间件示例

- 像终端模拟器、文件传输或电子邮件这样的基础服务
- 像RPC这样的基础服务
- 像DCE、网络O/S这样的一体化服务
- 像CORBA、OLE/ActiveX这样的分布式对象服务
- 像RMI、Jini这样的移动对象服务
- 万维网

中间件的功能包括：

- 在不同计算机上初始化过程
- 进行会话管理
- 允许客户端定位服务器的目录服务
- 进行远程数据访问
- 允许服务器处理多个客户端的并发控制
- 保证安全性和完整性
- 监控
- 终止本地处理和远程处理

### Continuum of Processing
Gartner模型基于将一个应用分解为表现组件、应用逻辑和数据处理。一个更细粒度的分解方式为:
![Alt text](./1473678080293.png)


### Points of Failure

分布式应用一般运行在复杂的环境中。这使得它比单一计算机上的独立应用更易发生故障。故障点包括：

- The client side of the application could crash
- The client system may have h/w problems
- The client's network card could fail
- Network contention could cause timeouts
- There may be network address conflicts
- Network elements such as routers could fail
- Transmission errors may lose messages
- The client and server versions may be incompatable
- The server's network card could fail
- The server system may have h/w problems
- The server s/w may crash
- The server's database may become corrupted

### Acceptance Factors

- Reliability
- Performance
- Responsiveness
- Scalability
- Capacity
- Security

### Transparency
分布式系统的“圣杯”就是提供以下几点：

- access transparency
- location transparency
- migration transparency
- replication transparency
- concurrency transparency
- scalability transparency
- performance transparency
- failure transparency

### Eight fallacies of distributed computing:分布式计算的八个误区

- The network is reliable.
- Latency is zero.
- Bandwidth is infinite.
- The network is secure.
- Topology doesn't change.
- There is one administrator.
- Transport cost is zero.
- The network is homogeneous.


## 第3章： Socket-level Programming(套接字层编程)

### The TCP/IP stack

The TCP/IP stack is shorter than the OSI one:
![Alt text](./1473773483879.png)

- TCP is a connection-oriented protocol,
- UDP (User Datagram Protocol) is a connectionless protocol.

#### IP datagrams

- IP 是无连接协议
- IP datagrams 之间的关联必须由高层协议来提供支持
- IP层包头支持数据校验，在包头包括源地址和目的地址
- IP层包头支持数据校验，在包头包括源地址和目的地址

#### UDP&TCP

- UDP是无连接的，不可靠的。它包括IP数据报的内容和端口号的校验
- TCP是构建于IP之上的面向链接的协议。它提供了一个虚电路使得两个应用进程可以通过它来通信。它通过端口号来识别主机上的服务

### Internet addresses

- IPV4 由四位数字组成，各自的范围在 0~255,每一组数字可以用 8 位二进制数字来表示，合计共需 32 位二进制
- IPv6使用128位地址，即使表达同样的地址，字节数变得很麻烦，由':'分隔的4位16进制组成。一个典型的例子如：2002:c0e8:82e7:0:0:0:c0e8:82e7。

### IP address type

``` go
type IPAddr {
    IP IP
}
```

IPAddr 最主要的用法是 DNS 查询
``` go
func ResolveIPAddr(net, addr string) (*IPAddr, os.Error)
```

``` go
  addr, err := net.ResolveIPAddr("ip","www.google.com")
  if err != nil {
    fmt.Println("Resolution error", err.Error())
    os.Exit(1)
  }
  fmt.Println("Resolved address is ", addr.String())
  // 输出
  // Resolved address is  243.185.187.39
```

**Host lookup**

`ResolveIPAddr` 执行一个 DNS 查找，返回单个的 IP 地址

`LookupHost` 执行 DNS 查找，返回字符串切片，ipv4 和 ipv6 的 ip 地址

`LookupCNAME`返回公认的主机名称


### 3.4 Services

端口号：This is an unsigned integer between 1 and 65,535

"standard" ports：

- Telnet usually uses port 23 with the TCP protocol. 
- DNS uses port 53, either with TCP or with UDP. 
- FTP uses ports 21 and 20
- HTTP usually uses port 80, but it often uses ports 8000, 8080 and 8088, all with TCP
- The X Window System often takes ports 6000-6007, both on TCP and UDP.

unix 系统中常用的端口列在 `/etc/services`
`LookupPort` 方法查询整个端口
``` go
func LookupPort(network, service string) (port int, err os.Error)
```

**The type TCPAddr**:

`TCPAddr` 是一个包含 IP 和 Port 的结构
``` go
type TCPAddr struct {
    IP   IP
    Port int
}
```

`ResolveTCPAddr` 创建一个 `TCPAddr`
``` go
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```
`net` 可选： `tcp`, `tcp4` or `tcp6`
`addr`:  主机名或 IP 地址，中间是 `:`,后面跟端口号，本机的话，可以简写 `:80`


### 3.5 TCP Sockets

` net.TCPConn` 支持在客户端和服务端，全双工可读可写的通信

``` go
func (c *TCPConn) Write(b []byte) (n int, err os.Error)
func (c *TCPConn) Read(b []byte) (n int, err os.Error)
```

**TCP client**：

``` go
func DialTCP(net string, laddr, raddr *TCPAddr) (c *TCPConn, err os.Error)
```

- `DialTCP` 函数可以建立 TCP 连接
- 客户端和服务器使用 `TCPConn`  交换信息，请求或者响应，直到关闭连接
- `laddr` 是本机地址， `raddr` 是远程服务地址
- `net` 是可选的  `tcp4`, `tcp6` or `tcp`

``` go
func ListenTCP(net string, laddr *TCPAddr) (l *TCPListener, err os.Error)
func (l *TCPListener) Accept() (c Conn, err os.Error)
```

`ListenTCP` 函数，侦听本地的地址在指定端口，`Accept` 阻塞，然后等待客户端连接


### 3.6 Controlling TCP connections

``` go
func (c *TCPConn) SetTimeout(nsec int64) os.Error
```

客户端和服务器设置超时用 `SetTimeout`，函数在 "net" 包


**Staying alive:**

``` go
func (c *TCPConn) SetKeepAlive(keepalive bool) os.Error
```
`SetKeepAlive` 可以设置客户端保持连接，函数在 "net" 包


### 3.7 UDP Datagrams

UDP 的函数：

``` go
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)
```


### 3.8 Server listening on multiple sockets

- go 的底层利用的是系统调用 `select(2)`
- `select(2)` 可以检测同时等待的多个 I/O,告诉哪个可以读写


``` gcode
/* c 函数*/
int select(int maxfd, fd_set *readfds, fd_set *writefds, fe_set *exceptfds, const struct timeval *timeout);
```
- select 的第一个参数是文件描述符集中要被检测的比特数，这个值必须至少比待检测的最大文件描述符大1
- 参数 readfds 指定了被读监控的文件描述符集
- 参数 writefds 指定了被写监控的文件描述符集
- 参数exceptfds指定了被例外条件监控的文件描述符集。
- 参数timeout起了定时器的作用：到了指定的时间，无论是否有设备准备好，都返回调用


未完。。。。
