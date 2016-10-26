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

---

## 第五章：交换数据

- TCP 连接如同遗传连接了本地套接字和远程套接字的管子，可以通过管子发送数据
- 所有的数据都被编码成 TCP/IP 分组

### 流

- TCP 是一个基于流的协议
-  创建套接字，需要传入 :STREAM 选项
- TCP 连接提供了一个不间断的、有序的通信流。

演示伪代码：

``` ruby
# 下面的代码会在网络上发送 3 份数据，一次一份
data = ['a','b','c']

for piece in data
  write_to_connection(piece)
end

# 下面的代码在一次做作中读取全部数据
result = read_from_connection #=> ['a','b','c']
```

`流并没有消息边界的概念`：

- 客户端分别发送 3 份数据，服务器读取，是将其作为一份数据来接手，并不知道客户端是分批发送的数据

---


## 第六章：套接字读取

学习如何在套接字上传送数据

### 6.1 简单的读操作

- Ruby 的各种套接字以及 File 在 IO 中都有一个共同的父类。
- Ruby 中所有的 IO 对象(套接字、管道、文件...)都有一套通用的接口，支持 read、write、flush 等方法
- 抽象源自操作系统核心本身，底层的 `read(2)` `write(2)` 等系统调用都可以作用域文件、套接字、管道等之上

服务端：
``` ruby
require 'socket'

Socket.tcp_server_loop(4481) do |connection|
  # 从连接中读取数据最简单的方法
  puts connection.read

  # 完成读取之后关闭连接，让客户端知道不用再等待数据返回
  connection.close
end
```

客户端调用：

``` bash
echo ohi | nc lcoalhost 4481
```
  
### 6.2 没那么简单

- `EOF`:  end-of-file, 表示数据结尾
- 服务器的 read 会一直阻塞，直到客户端发完数据为止

客户端：

``` bash
 tail -f /var/log/system.log | nc -v localhost 4481
```


### 6.3 读取长度
- 解决阻塞的办法是，指定最小的读取长度，告诉服务器读取(read)特定的数据量

``` ruby
require 'socket'
one_kb = 1024 # 字节数

Socket.tcp_server_loop(4481) do |connection|

  # 以 1kb 为单位进行读取
  while data = connection.read(one_kb) do 
    puts data
  end

  # 完成读取之后关闭连接，让客户端知道不用再等待数据返回
  connection.close
end
```

### 6.4 阻塞的本质

- `read` 调用会一直阻塞，直到获取了完整长度的数据为止

解决 `read` 死锁的办法：

1. 客户端发完 500B 后再发送一个 ·EOF·
2. 服务器采用部分读取 (partial read) 的方式

### 6.5 EOF 事件

- 当在连接上调用 `read` 并接收到 EOF 事件时，就可以确定不会再有数据，可以停止读取了。
- `EOF` 代表 `end of file`(文件结束)
- `EOF` 并不代表某种字符序列，它更新一个状态事件(state even)
- 如果一个套接字没有数据可写，可以调用 `shutdown` 或 `close` 表示不再需要写入数据。这发送一个 `EOF` 事件给另一端进行读操作的进程
- 调用 `File#read` 时(同 `Socket#read` 的行为方式类似)，会一直进行数据读取，直到无数据为止。一旦读完文件，会受到一个 `EOF` 事件并返回已读取到的数据

``` ruby
# ./code/snippets/read_with_length.rb

require 'socket'
one_hundred_kb = 1024 * 100 # 字节数

Socket.tcp_server_loop(4481) do |connection|
  begin
    # 以 1kb 为单位进行读取
    while data = connection.readpartial(one_hundred_kb) do
        puts data
      end
    rescue EOFError
    end

    # 完成读取之后关闭连接，让客户端知道不用再等待数据返回
    connection.close
  end

```

客户端连接：

``` ruby
# ./code/snippets/write_with_eof.rb

require 'socket'
client = TCPSocket.new('localhost', 4481)
client.write('gekko')
client.close
```

### 6.6 部分读取

- 调用 `readpartial` 不会阻塞，而是立即返回可用数据
- `readpartial` 必须传递一个整数作为参数，来指定最大的长度
-  `readpartial` 最多读取到指定长度。如果指明 1kb 数据，但客户端发送了 500B，并不会阻塞，会立即将读到的数据返回 
-  当接收到 `EOF` 时 `read` 仅仅是返回， 而 `readpartial` 则会产生一个 `EOFError` 异常

``` ruby
# ./code/snippets/readpartial_with_length.rb

require 'socket'
one_hundred_kb = 1024 * 100 # 字节数

Socket.tcp_server_loop(4481) do |connection|
  begin
    # 以 1kb 为单位进行读取
    while data = connection.readpartial(one_hundred_kb) do
        puts data
      end
    rescue EOFError
    end

    # 完成读取之后关闭连接，让客户端知道不用再等待数据返回
    connection.close
  end

```

### 6.7 系统调用

``` bash
Socket#read -> read(2),行为类似 fread(3)
Socket#readpartial -> read(2)
```

---


## 第七章：套接字写操作

- 套接字写入数据，需要调用 `write` 方法
- 系统调用: `Socket#write` -> `write(2)`

``` ruby
require 'socket'

Socket.tcp_server_loop(4481) do |connection|

  # 向连接中写入数据的最简单的方法
  connection.write('Welcome!')
  connection.close
end
```

---

## 第八章：缓冲

### 8.1 写缓冲

- 调用 `write` 并返回，不代表数据已通过网络发送并被客户端套接字接收
- 调用 `write` 并返回，只表明已将数据提交给了 `Ruby` 的 `IO` 系统和底层的操作系统内核
- 在应用程序代码和实际的网络硬件之间至少还存在一个缓冲层
- TCP 套接字morning将 sync 设置为 true, 跳过了 ruby 的内部缓冲
- IO 缓冲是为了更好的性能

### 8.2 读写入多少数据

- 因为有缓冲区，我们可以一次写入所有的数据，由内核决定如何对数据进行分割或合并来调节性能
- 如果数据量很大的 write，可以将自己将数据分割，避免全部载入内存中

### 8.3 读缓冲

- 读操作同样会被缓冲
- 用 `read` 读取指定长度的数据，ruby 实际会接收大于制定长度的数据, ruby 多读的数据会被存储在 ruby 内部的读缓冲区
- 下次调用 `read`,ruby 会查看内部缓冲区数据，然后再通过内核请求更多的数据

### 8.4 该读取多少数据

- TCP 提供的是数据流，无法得知发送方到底发送了多少数据，读取长度只能靠猜测
- 指定读取长度时，内核会分配一定的内存
- 指定读取长度太大，会浪费内存资源；指定长度太小，会有大量系统调用开销
- Mongrel、Unicorn、Puma、Passenger 以及 Net::HTTP，采 `readpartial(1024*16)` 16KB 作为读取长度
- `redis-rb` 使用 1KB 作为读取长度

---

## 第 10 章：套接字选项

### 10.1 SO_TYPE

```  ruby
require 'socket'
socket = TCPSocket.new('google.com', 80)

# 获得一个描述套接字类型的 Socket::Option 实例
opt = socket.getsockopt(Socket::SOL_SOCKET, Socket::SO_TYPE)

# 将描述该选项的整数值同存储在 Socket::SOCK_STREAM 中的整数值进行比较
puts opt.int == Socket::SOCK_STREAM #=> true
puts opt.int == Socket::SOCK_DGRAM #=> false
```

简便方式：

``` ruby
require 'socket'
socket = TCPSocket.new('google.com', 80)

# 使用符号名,而不是常量
opt = socket.getsockopt(:SOCKET, :TYPE)
```

### 10.2 SO_REUSE_ADDR

- `SO_REUSE_ADDR` 选项告诉内核：如果服务器当前处于 TCP 的 `TIME_WAIT` 状态，即便另一个套接字要绑定(`bind`) 到服务器目前所使用的本地地址也无妨.
- TCPServer.new、Socket.tcp_server_loop 及其类似的方法默认都打开了此选项

示例代码：

``` ruby
require 'socket'
server = TCPServer.new('localhost', 4481)
server.setsockopt(:SOCKET, :REUSEADDR, true)

server.getsockopt(:SOCKET, :REUSEADDR) #=> true
```

### 10.3 系统调用

``` bash
Socket#setsockopt -> setsockopt(2)
Socket#getsockopt -> getsockopt(2)
```

---


## 第11章：非阻塞式 IO

### 11.1 非阻塞式读操作

**两种阻塞的读操作**
- `read` 会一直保持阻塞，直到接收到 `EOF` 或是获得指定的最小字节数为止
- `readpartial` 会立即返回所有的可用数据，但如果没有数据可用，那么 `readpartial`仍会陷入阻塞

**`Socket#read_nonblock` **:

- 非阻塞读操作，需要指定整数值，作为读取的最大字节数
- 如果可用数据小于最大字节数，则返回可用数据
- 没有数据可读，`read_nonblock` 调用仍然会立即返回，并产生一个 `Errno::EAGAIN`异常
- `Errno::EAGAIN`: 文件被标记用于非阻塞式 IO，无数据可读


>  `read_nonblock` 方法首先检查 `ruby` 的内部缓冲区中是否还有未处理的数据，如果有，则立即返回
>    `read_nonblock` 会询问内核是否有其他可用的数据可供 `select(2)`  读取,如果有，不管这些数据是在内核缓冲区还是网络中，他们都会被读取并返回
>   其他情况都会使 `read(2)` 阻塞并在 `read_nonblock` 中引发异常

``` ruby
require 'socket'

Socket.tcp_server_loop(4481) do |connection|
  begin
    puts connection.read_nonblock(4096)
  rescue Errno::EAGAIN => e
    IO.select([connection])
    retry
  rescue EOFError
    break
  end

  connection.close
end
```

### 11.2 非阻塞式写操作

- `write_nonblock` 会在出现阻塞的时候，返回部分写入的结果
- `write_nonblock` 的行为和系统调用 `write(2)`一样， 尽可能多的写入数据并返回写入的数量
- `write` 和 `write_nonblock` 不同，`write` 会多次调用 `write(2)` 写入所有请求的数据
-  `write_nonblock` 如果遇到阻塞会得到一个 `Errno::EAGAIN` 异常

``` ruby
./code/snippets/write_nonblock.rb
require 'socket'

client = TCPSocket.new('localhost', 4481)
payload = 'Lorem ipsum' * 100_000

written = client.write_nonblock(payload)
puts written < payload.size 
```

非阻塞，多次写入：

``` ruby
./code/snippets/retry_partial_write.rb

require 'socket'

client = TCPSocket.new('localhost', 4481)
payload = 'Lorem ipsum' * 100_000

begin
  loop do
    bytes = client.write_nonblock(payload)

    break if bytes >= payload.size
    puts "----#{bytes}"
    payload.slice!(0, bytes) # 删除已经写入的数据
    IO.select(nil, [client])
  end

rescue Errno::EAGAIN
  IO.select(nil, [client])
  retry
end
```

### 11.3 非拥塞式接收

- `accept` 只是从侦听队列中弹出一个连接
- `accept_nonblock` 在侦听队列为空时不会阻塞，只是产生一个 `Errno::EAGAIN`

### 11.4 非拥塞式连接

- `connect_nonblock` 不能立即发起到远程主机的连接，他会在后台继续执行操作并产生 `Errno::EINPROGRESS`

---

## 第 12 章：连接复用

- 连接复用指同时处理多个活动套接字，不是并行处理，无关多线程

示例代码：`./code/snippets/native_multiplexing.rb`

## 12.1 select(2)
- `IO.select` 的作用是接手若干个 `IO` 对象，告知哪个可以进行读写

``` go
# snippets/select_returns.rb
for_reading = [<TCPSocket>, <TCPSocket>, <TCPSocket>]
for_writing = [<TCPSocket>, <TCPSocket>, <TCPSocket>]

ready = IO.select(for_reading, for_writing, for_writing)

# 对于每个座位参数传入的数组均会返回一个数组
# 在这里， for_writing 中没有连接可写，for_reading 中有一个连接可读
p ready #=> [[<TCPSocket>], [], []]
```

- `IO.select` 可以使用 3 个数组作为参数：
  - 第一个参数是希望从中进行读取的 IO 对象数组
  - 第二个参数是希望进行写入的 IO 对象数组
  - 第三个是在异常条件下使用的 IO 对象数组，可以被忽略
  - `IO.select`  返回一个包含了3个元素的嵌套数组，分别对应它的参数列表

- `IO.select` 会阻塞，是一个同步方法调用
- `IO.select` 还有第四个参数，一个以秒为单位的超时值，可以避免 `IO.select` 永久的阻塞下去， 如果超时会返回 `nil`
- 可以传递纯 ruby 对象给 `IO.select` ，只要它们实现了 `to_io` 方法并返回一个 IO 对象

``` go
# snippets/select_timeout.rb
for_reading = [<TCPSocket>, <TCPSocket>, <TCPSocket>]
for_writing = [<TCPSocket>, <TCPSocket>, <TCPSocket>]

timeout = 10
ready = IO.select(for_reading, for_writing, for_writing, timeout)

# 在这里 `IO.select` 在 10 秒钟内没有检测到任何状态的改变
# 因此返回 nil, 而非嵌套数组
p ready #=> nil
```

## 12.2 读/写之外的事件
`IO.select` 监视套接字的读写状态

### 12.2.1 EOF
`EOF` 是 `end of file` ，如果在监视可读性时，接到 `EOF` ，该套接字会作为可读套接字数组的一部分被返回

### 12.2.2 accept

- 监视服务器套接字可读性时，如果收到接入连接，套接字可作为可读套接字数组的一部分返回


### 12.2.2 connect

- `connect_nonblock` 是非阻塞式连接，如果不能立刻完成连接，则会产生 `Errno::EIGPROGRESS`
- 使用 `IO.select` 了解后台连接是否已经完成
端口扫描器代码见 `./code/snippets/port_scanner.rb`

### 12.2.3 高性能复用

- `IO.select` 是 ruby 核心代码库，他是 ruby 进行复用唯一手段
- 大多数系统支持多种复用方法， `select(2)` 几乎是最古老，也是用的最少的
- `IO.select` 同它所监视的连接数呈线性关系，监视连接数越多，性能就越差
- `select(2)` 系统调用受到 `FD_SETSIZE`（文件描述符数量大小） 的限制，无法对编号大于 FD_SETSIZE(多数系统上是 1024)的文件描述符进行监视

- `poll(2)` 系统调用与 `select(2)` 仅限于表面不同
- `epoll(2)` 以及 BSD 的 `kqueue(2)` 系统调用比 `select(2)` 效果更好，性能更先进
- `EvenMachine` 倾向于使用 `epoll(2)` 以及 BSD 的 `kqueue(2)`
- ruby 的 gem `nio4r` 为 `select(2)`, `epoll(2)` 等提供了通用的接口

--- 

## 第 13 章：Nagle 算法

- Nagle 算法是一种默认用于所有的 TCP 连接的优化
- 这种优化适合那些不进行缓冲、每次只发送很小数据量的应用程序
- ruby 有缓冲，所以在 TCP 上实现的大部分常见协议会希望禁用 Nagle 算法

``` ruby
server.setsockopt(Socket::IPPROTO_TCP, Socket::TCP_NODELAY, 1)
```

---

## 第 14 章：消息划分

- 发送多条消息并复用连接，需要用某种方式表明消息之间的起止
- 多消息重用连接与 `HTTP keep-alive` 特性背后的理念一致，在多个请求间保持连接开放(包括客户端和服务器协商的划分消息的方法)，通过避免打开新的连接来节省资源。

**协议与消息**：

- 协议定义了应该如何格式化消息
- 比如：HTTP 协议既定义了消息边界(连续的新行)，也定义了用于消息内容(涉及请求行、头部等)的协议

### 14.1 使用新行

- 使用新行(newlines) 是一种划分消息的简单方法
- 使用 ruby `IO#gets` 和 `IO#puts` 可以发送带新行的消息
-  `IO#gets` 和 `IO#puts`  在不同的操作系统中使用的行分隔符不一样，需要注意兼容性问题
- 现实中使用新行划分消息的协议是 HTTP，用 `\r\n`

### 14.2 使用内容长度

划分指定内容长度(content length):

- 发送方先计算出消息的长度，使用 pack 将其转换成固定宽度的整数
- 消息接收方首先读取这个长度值，知道了消息的大小
- 然后接收方严格读取长度值所指定的字节数，获得完整的消息


代码详细见 cloudhash/server2.rb  cloudhash/client2.rb

---

## 第 15 章： 超时

如果套接字没能在 5 秒内完成数据写入，那就说明存在问题

### 15.1 不可用的选项

- ruby 标准库 `timeout` 提供了一种通用的超时机制 
- 操作系统提供了自带的套接字超时处理机制, ruby 1.9 之后 不能会用
- ruby 处理套接字超时建议使用 `IO.select`

### 15.2 IO.select

- 除了读取超时，连接/接收的超时都可以用 `IO.select` 处理

代码见 `snippet/read_timeout.rb`

``` ruby
require 'socket'
require 'timeout'

timeout = 5 # 秒

Socket.tcp_server_loop(4481) do |connection|

  begin
    # 发起一个初始化 read(2)。这一点很重要
    # 因为要求套接字上有被请求的数据，有数据可读时避免使用 select(2)
    connection.read_nonblock(4096)

  rescue Errno::EAGAIN
    # 监视连接是否可读
    if IO.select([connection], nil, nil, timeout)
      # IO.select 会将套接字返回，不过我们并不关心返回值
      # 不返回 nil 就意味着套接字可读
      retry
    else
      raise Timeout::Error  # 使用 timeout 只是为了用 Timeout::Error 常量
    end

  end

  connection.close
end
```

---

## 第 16 章： DNS 查询

### MRI 和 GIL

- `Global Interpreter Lock, GIL` 全局解释锁，确保 ruby 解释器只做一件有潜在危险的事。多线程环境中，当一个线程进行活动时，其它线程全部处于阻塞状态
- 如果一个线程进行阻塞式 IO, （例如一个阻塞式 read）, GIL 会释放 GIL 并让另一个线程继续执行
- 只要代码块用到了 C 语言扩展 API, GIL 会阻塞其它代码的运行
- ruby 的 DNS 查询使用了一个 C 语言扩展，可能会被长时间阻塞，MRI 就不会释放 GIL


resolv

- resolv 为 DNS 查询提供了一套纯 Ruby 的替代方案，是的 MRI 能够为长期阻塞的 DNS 查询释放 GIL
- ruby 标准库使用 `resolv-replace` 猴子不定来使用 resolv

``` ruby
require `resolv` # 库
reequire `resolv-replace` # 猴子补丁
```

---

## 第 17 章: SSL 套接字

- SSL 使用公钥加密提供了一套用于在套接字上进行安全的数据交换的机制
- 套接字可以升级为 SSL，但一个套接字不能同时进行 SSL 和非 SSL 通信
- ruby 中使用标准库的 openssl 实现套接字转为 SSL 套接字


## 第 18 章: SSL 套接字

- TCP 套接字数据提供了一种有序的数据流。
- 可以将 TCP 数据流想象成一个队列。套接字连接的一端向连接中写入数据，就相当于将数据入列。
- 数据经过若干阶段（本地缓冲、网络传输、远程缓冲），然后在接收端的套接字出列。
-  TCP 紧急数据，更多的时候被称作 “带外数据”(out-of-band data),支持将数据推到队列的前端，绕过其它已经在传输的数据，比便于另一端尽快接收

### 发送紧急数据

``` ruby
require 'socket'

socket = TCPSocket.new 'localhost', 4481

# 会用标准方法发送数据
socket.write 'first'
socket.write 'second'

# 发送紧急数据
socket.send '!',Socket::MSG_OOB
```
- `Socket#send` 将 `Socket::MSG_OOB` 常量作为标志。 OOB 指的就是带外数据
- 发送方和接收方需要合作才可以处理带外数据

## 接收紧急数据

``` ruby
require 'socket'

Socket.tcp_server_loop(4481) do |connection|

  # 优先接收紧急数据
  urgent_data = connection.recv(1, Socket::MSG_OOB)
  data = connection.readpartial(1024)
end
```

- 接收紧急数据，需要使用 `Socket#recv` 以及在发送紧急数据时用过的那个标志
- 紧急数据会优先于 “普通” 数据使用，即使写入的比 “普通” 数据晚
- 如果不存在未处理的紧急数据，调用 `connection.recv(1, Socket::MSG_OOB)` 会失败，并产生 `Errno::EINVAL`

## 局限

- TCP 实现对于紧急数据仅提供了有限的支持，一次只能发送一个字节的紧急数据。如果发送多个字节，只有最后一个字节会被视为紧急数据，之前的数据会被视为普通的 TCP 数据流

## 紧急数据和 IO.select

- 如果套接字接收到了紧急数据，它们会被包含在 `IO.select`  所返回数组的第三个元素中
- `IO.select` 会不停的报告有紧急数据，即便是所有的紧急数据已经处理完毕，所以需要特殊处理

## SO_OOBINLINE 选项

`SO_OOBINLINE` 套接字选项，允许在带内接收带外数据，启用后回一句写入次序从队列读出


## TCP Sockets 编程(20): 串行化

串行化架构处理流程：

1. 客户端连接
2. 客户端/服务器交换请求及响应
3. 客户端断开连接
4. 返回到步骤(1)

串行化的特点：简单化，没有锁，没有共享状态，处理完一个连接之后才能处理另一个，不能支持并发操作


## TCP Sockets 编程(21): 单连接进程

单连接进程事件流程：

1. 一个连接抵达服务器
2. 主服务器进程接受该练级
3. 衍生出一个和服务器一模一样的新子进程
4. 服务器进程返回步骤 1，由子进程并行处理连接

优点：

- 简单，能并行处理多个客户端
- for 提供了一个父进程的所有东西的副本，没有锁和竞争条件

缺点：

- 对 fork 出的子进程的数量没有施加限制，如果超出系统限制会崩溃
- 只有 Unix 系统才支持 fork，windows 或 JRuby 中没法使用 fork


## TCP Sockets 编程(22): 单连接线程

线程与进程：

- 生成(`spawn`): 线程的成本低于进程，进程生成需创建原始进程所拥有的一切资源的副本，多个线程共享内存，不需要创建副本
- 同步：因为线程共享内存，所以线程之间使用互斥量(mutex)、锁和同步访问。进程不需要这些
- 并行：解释器对当前执行环境使用了一个全局解释锁 `GIL`,所以多线程无法实现真正的并行, 在 `MRI` 中，只有进程才能实现真正的并发
  但ruby 中如果某个线程阻塞在 IO 上， ruby 能让其他的线程继续执行


使用线程注意：

- 套接字如果分配给一个实例变量，会在所有活动线程之间共享该实例的内部状态
- 使用线程进行套接字编程，必须让每个线程获得它自己的连接对象，这样可以减少麻烦

## TCP Sockets 编程(23): Preforking

Preforking 处理流程：

1. 主服务器进程创建一个侦听套接字
2. 主服务器进程衍生出一大批子进程
3. 每个子进程在共享套接字上接受连接，然后进行独立处理
4. 主服务器进程随时关注子进程

Preforking 优点：
- 多进程处理连接的负载均衡由操作系统处理
- 子进程完全隔离，每个进程都拥有包括 ruby 解释器在内的所有资源的副本，单个进程的故障不会影响其他进程。

缺点:
- 衍生进程越多，消耗的内存也越多

## TCP Sockets 编程(24): 线程池

- 线程池模式类似于 `preforking`
- 线程池在服务器启动后生产一批线程，将处理连接的任务交给独立线程来完成
