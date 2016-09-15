---
title:  "《TCP Sockets 编程》读书笔记"
date:   2016-03-29 09:06:00
description: '《TCP Sockets 编程》读书笔记'
---

# 《TCP Sockets 编程》读书笔记

## 第一章：建立套接字

### 1.1 ruby 的套接字库
socket 库是 ruby 标准库的组成，包含各种用于 TCP 套接字、UDP 套接字的类

### 1.2 创建首个套接字

``` ruby
require 'socket'
socket = Socket.new(Socket::AF_INET, Socket::SOCK_STREAM)
```
- INET 是 internet 的缩写，特别用于指代 IPv4 版本的套接字。
- STREAM 表示用数据流通信，由 TCP 提供。
- DGRAM(datagram 的缩写，数据报),则表示 UDP 套结字

### 1.3 什么是端点

- 套接字使用 IP 地址将消息指向特定的主机。
- 主机由唯一的 IP 地址来标识

### 1.4 环回地址

- 环回接口(loopback interface)。与硬件无关、完全虚拟的接口。发送到环回接口的数据立即会在同一个接口上被接收。
- 环回接口对应的主机名是 localhost, 对应的 IP 地址通常是 127.0.0.1，定义在 hosts 文件中

### 1.5 IPv6
- IPv4 由 4 组数字组成，各自的范围在 0 ~ 255, 每组数字可以用 8 位二进制数字来表示，合计共需 32 位进制，意味着有 2的32次方或 43亿个地址。
- IPv6 用另一个不同的格式，可以拥有天文数字级别的独立 IP 地址。

### 1.6 端口
套接字的 IP 地址和端口号的组合必须是唯一，端口号就是套接字端点的 "分机号"。

### 1.7 创建第二个套接字

**IPv6 域中的套接字**:

```  ruby
# ./code/snippets/create_socket_memoized.rb

require 'socket'
socket = Socket.new(:INET6, :STREAM)
```

### 1.8 系统调用

```Socket.new -> socket(2)```

---

## 第二章：建立连接

- TCP 在两个端点之间建立连接。端点可以处在同一台主机或不同主机
- 套接字必须担任以下角色之一：
  1. 发起者(initiator)
  2. 侦听者(listener)
- 网络编程中，从事侦听的套接字称为 “服务器”，发起连接的套接字称为 “客户端”


---

### 第三章：服务器套接字生命周期
用于侦听连接而非发起连接，其典型的生命周期如下：
1. 创建
2. 绑定
3. 侦听
4. 接受
5. 关闭

### 3.1 服务器绑定
```  ruby
require 'socket'
# 首先创建一个新的 TCP 套接字
socket = Socket.new(:INET, :STREAM)

# 创建一个 C 结构体来保存用于侦听的地址。
addr = Socket.pack_sockaddr_in(4481, '0.0.0.0')

#执行绑定
socket.bingd(addr)
```

这个套接字已绑定到本机的 4481 端口，其它套接字不可用此端口，否则会产生异常 `Errno::EADDRINUSE`

### 3.1.1 该绑定到哪个端口

- 规则1: 不要使用  0~1024 之间的端口。这些端口是保留给系统使用的。 
  http: 80，SMTP: 25，RSYNC: 873，绑定这些端口需要 root 权限。
- 规则2：不要使用 49 000 ~ 65 535 之间的端口。
  这些是临时(tphemeral)端口。通常是用于临时之需的服务使用。
- 除此之外,1025~48 999 的端口使用时一视同仁的。可以参考 [IANA 的注册端口列表](http://www.iana.org/assignments/port-numbers)，确保不与其他流行的服务器冲突。

### 3.1.2 该绑定到哪个地址

- 绑定环回地址(127.0.0.1) ，仅限于本地连接使用。无法用于外部连接，只有 localhost 或 127.0.0.1 的连接才会被服务器套接字接受
- 绑定 `192.168.0.5` ，套接字只侦听此接口，无法用于本地连接
- 使用用 0.0.0.0，会侦听所有可用接口、环回地址等


### 3.2 服务器侦听

创建套接字绑定到端口之后，需要进行侦听.

```  ruby
require 'socket'

# 创建套接字并绑定到端口 4481
socket = Socket.new(:INET, :STREAM)
addr = Socket.pack_sockadd_in(4481, '0.0.0.0')
socket.bind(addr)

#告诉套接字侦听接入的连接
socket.listen(5)
```

### 3.2.1 侦听队列

- listen 方法传递了一个整数类型的参数，表示服务器套接字能容纳的待处理的最大连接数
- 待处理的连接列表被称为侦听队列。
- 如果客户端连接到达且侦听队列已满，客户端会产生 Errno::ECONNREFUSED

### 3.2.2 侦听队列的长度

- `Socket::SOMAXCONN` 可以获知当前锁允许的最大的侦听队列长度，Mac 上是 128
- 需要 root 权限来增加系统级别限制。
- 可使用 server.listen(Socket::SOMAXCONN) 将侦听队列长度设置为允许的最大值。

### 3.3 接受连接

``` ruby
require 'socket'
# 创建套接字并绑定到端口 4481
socket = Socket.new(:INET, :STREAM)
addr = Socket.pack_sockadd_in(4481, '0.0.0.0')
socket.bind(addr)

#告诉套接字侦听接入的连接
socket.listen(5)

#接受连接
connection, - = server.accept
```
用 netcat 发起一个连接, 运行后 nc 和 ruby 程序都会顺利退出

``` bash
$ echo ohai | nc localhost 4481
```

#### accept

- accept 调用时阻塞式的，没有接收到新的连接，它会一直阻塞当前线程。
- accept 调用返回一个数组，第一个是建立好的连接，第二个元素是 Addrinfo 对象，该对象描述客户端连接的远程地址

#### Addrinfo

- Addrinfo 类描述了一台主机机器端口号
- 有用的方法包括 #ip_address 和 #ip_port 
- 构建例子：Addrinfo.tcp('localhost', 4481)

### 连接详解

``` ruby
require 'socket'
# 创建套接字并绑定到端口 4481
socket = Socket.new(:INET, :STREAM)
addr = Socket.pack_sockadd_in(4481, '0.0.0.0')
socket.bind(addr)

#告诉套接字侦听接入的连接
socket.listen(128)

#接受连接
connection, - = server.accept

print 'Connection class:'
p connection.class

print 'Server fileno:'
p server.fileno

print 'Connection fileno:'
p connection.fileno

print 'Local address:'
p connection.local_address

print 'Remote address:'
p connection.remote_address # accept 第二个返回值相同
```

使用 netcat 命令发起连接后会输出：
```  ruby
Connection class: Socket
Server fileno: 7
Connection fileno: 8
Local address: #<Addrinfo: 127.0.0.1:4481 TCP>
Remote address: #<Addrinfo: 127.0.0.1:50488 TCP>
Addrinfo"#<Addrinfo: 127.0.0.1:50488 TCP>"
```
#### 3.3.3 连接类
- 连接类是 Socket 表示一个连接就是一个 Socket 的实例

####  3.3.4 文件描述符

- fileno(文件描述符编号)是内核用于跟踪当前进程所打开文件的一种方法。
- 在 Unix 世界中，所有的一些都被视为文件。包括文件系统中的文件以及管道、套接字和打印机，等等。
- accept 返回了一个不同于服务器套接字的全新 Socket，每个连接都是一个全新的 Socket 对象描述

#### 3.3.5 连接地址

- 本地地址：本地主机、本地端口
- 远程地址：远程主机、远程端口
- 以上 4 个属性的组合必须是唯一的

#### 3.3.6 accept 循环
accept 返回一个连接

``` ruby
require 'socket'
# 创建套接字并绑定到端口 4481
socket = Socket.new(:INET, :STREAM)
addr = Socket.pack_sockadd_in(4481, '0.0.0.0')
socket.bind(addr)

#告诉套接字侦听接入的连接
socket.listen(128)

#进入无限循环，接受并处理连接
loop do
  #接受连接
  connection, - = server.accept # 返回连接，第二个 - 是远程地址
  # 处理连接
  connection.close
end
```

### 3.4 关闭服务器
服务器接受某个连接并处理完毕，那么最后需要关闭该连接。这样才算完成一个连接的
"创建-处理-关闭" 的生命周期。

#### 3.4.1 退出时关闭
关闭连接的理由：

- 资源使用
- 打开文件的数量限制
 

> `Process.getrlimit(:NOFILE)`，获知当前进程所需要打开文件的数量。
> 返回值是数组，包含软限制(用户配置的设置)和硬限制(系统限制)。
> `Process.setrlimit(Process.getrlimit(:NOFILE)[1])` 将进程的打开文件限制改为最大值

#### 3.4.2 不同的关闭方式
套接字允许双向通信(读/写)，实际上可以只关闭其中一个通道。

- `Socket#close_write`  关闭写操作流 (wite stream) 会发送一个EOF到套接字的另一端。
- `Socket#close_read` 关闭读操作流
- `Socket#close_write`  `Socket#close_read` 方法在底层都利用 `shutdown(2)` 
- `Socket#close` 关闭套接字实例并回收所用资源，但不会关闭副本，副本所占用资源也不会被回收。
- `Socket#shutdown`  会关闭当前套接字及其副本上的通信，但不会回收所用资源。每个套接字都需要使用 close 结束它的生命周期。
- `Socket#dup` 创建文件描述符副本
- `Process.fork` 可以获得一个文件描述符副本，该方法创建了一个全新的进程(仅在 Unix 环境中)，这个进程和当前进程一模一样，除了拥有当前进程在内存中的所有内容外，新进程还通过 `dup` 获得了所有已打开的文件描述符的副本

### 3.5 Ruby 包装器

#### 3.5.1 服务器创建

``` ruby
require 'socket'
server = TCPServer.new(4481)
```
简便的服务器创建方式

- `TCPServer#accept` 值返回连接
- `TCPServer` 默认的侦听队列长度是 5, 可以用 `TCPSrver#listen` 修改侦听队列长度

``` ruby
require 'socket'
servers = Socket.tcp_server_sockets(4481)
```
以上代码同时返回两个套接字，IPv4 和 IPv6

#### 3.5.2 连接处理

`accept_loop` 无限循环，并且还可以接受多个侦听套接字

``` ruby
require 'socket'

# 创建侦听套接字
server = TCPServer.new(4481)

# 进入无线循环接手并处理连接
Socket.accept_loop(server) do |connection|
  # 处理连接
  connection.close
end
```

多个套接字处理
```
# 创建侦听套接字
servers = Socket.tcp_server_sockets(4481) # 创建两个套接字 IPv4 和 IPv6 

# 进入无线循环接手并处理连接
Socket.accept_loop(servers) do |connection|
  # 处理连接
  connection.close
end
```


#### 3.5.3 合而为一

```ruby
require 'socket'

Socket.tcp_server_loop(4481) do |connection|
  # 处理连接
  connection.close
end
```

### 3.6 系统调用

```  bash
Socket#bind -> bind(2)
Socket#listen -> listen(2)
Socket#accept -> accept(2)
Socket#local_address -> getsockname(2)
Socket#remote_address -> getpeername(2)
Socket#close -> close(2)
Socket#close_write -> shutdown(2)
Socket#shutdown -> shutdown(2)
```

---


## 第四章：客户端生命周期

**网络连接两部分：**

- 服务器负责侦听及处理接入的连接
- 客户端负责向服务器发起连接

**客户端的生命周期**

- (1) 创建
- (2)  绑定
- (3)  连接
- (4)  关闭

### 4.1 客户端绑定

- 建议：不要给客户端绑定端口
- 客户端不需要调用 bind，他会从临时端口范围内获得一个随机端口号

### 4.2 客户端连接

- connect 调用默认有一段较长时间的超时
- 出现超时，会产生一个 `Errno::ETIMEOUT` 异常

``` ruby
require 'socket'
socket = Socket.new(:INET, :STREAM)

# 发起到 baidu.com 端口 80 的连接
remote_addr = Socket.pack_sockaddr_in(80, 'baidu.com')
socket.connect(remote_addr)
```

### 4.3 ruby 包装器

客户端创建简写版本：

``` ruby
require 'socket'
socket = TCPSocket.new('baidu.com', 80)
```

代码块：

``` ruby
require 'socket'

Socket.tcp('baidu.com', 80) do |connection|
  connection.write "GET / HTTP/1.1\r\n"
  connection.close
end
```

省略代码块:

``` ruby
require 'socket'

client = Socket.tcp('baidu.com', 80)
```

### 4.4 系统调用

```  bash
Socket#bind -> bind(2)
Socket#connect -> connect(2)
```

