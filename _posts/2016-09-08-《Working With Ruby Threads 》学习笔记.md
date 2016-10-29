---
layout: post
title:  "《Working With Ruby Threads》学习笔记"
date:   2016-09-21 09:08:00
description: '《Working With Ruby Threads》'
category: notes
---


## Introduction

### why care?

- 在多核 CPU上，代码必须构建在充分利用多核的架构上才能跑的更快

### The promise of multi-threading

- 多进程 copy 内存，多线程共享内存
- 多线程比多进程开销更小，多线程可以有更多的并发单元
- 多线程必须基于线程安全

## 第1章: You're Always in a Thread


``` bash
$ irb
> Thread.main
=> #<Thread:0x007fdc830677c0 run>
> Thread.current == Thread.main => true
```

- `Thread.main` 总是指向主线程
- 主线程退出，其它线程也会终止，并且 ruby 进程会退出

``` bash
$ irb
> Thread.main
=> #<Thread:0x007fdc830677c0 run>
> Thread.current == Thread.main => true
```

## 第2章：Threads of Execution

### Shared address space

- 线程共享一个作用域
- 所有 ruby 的线程会映射为一个 native，操作系统线程


``` bash
$ top -l1 -pid 8409 -stats pid,th
```
以上命令可以查看进程 id 为 8409 的线程数量

### Non-deterministic context switching(非确定的环境切换)

In order to provide fair access, the thread scheduler can 'pause' a thread at any time, suspending its current state
为了提供公平的访问，线程调度能在任意时间 “暂停” 一个线程，暂停它的当前状态

`||=` 语句不是线程安全的，因为线程可能在任何时间被阻止，如果 A 线程运行 `||=` 获得了初始值并且暂停，可能会出现失去 B 线程赋值的情况

``` ruby
# This statement
results ||= Queue.new

# when broken down, becomes something like
if @results.nil? 
 temp = Queue.new 
 @results = temp
end
```

A race condition involves two threads racing to perform an operation on some shared state.
一个竞争条件是在共享状态下，包含两个线程竞争去执行一个同样的操作

- **`重要原则：`**Any time that you have two or more threads trying to modify the same thing at the same time, you're going to have issues.
- This is because the thread scheduler can interrupt a thread at any time.

针对重要原则的两个策略：

1) don't allow concurrent modification
2) protect concurrent modification

## 第三章：Lifecycle of a Thread

### Thread.new

``` ruby
Thread.new { ... }
Thread.fork { ... } 
Thread.start(1, 2) { |x, y| x + y }
```
Thread.new 及其别名方法

### Thread#join

- Once you've spawned a thread, you can use #join to wait for it to finish
- Without #join, the main thread would exit before the sub-thread can execute its block. Using #join provides a guarantee in this situation.
- Calling #join on the spawned thread will join the current thread of execution with the spawned one
- 使用 #join 的时候，异常会在 #join 的时候才抛出，主线程也会执行但不会立即输出

### Thread#status

`Thread#value` 的几个可能值:

- `run`: Threads currently running have this status.
- `sleep`: Threads currently sleeping, blocked waiting for a mutex, or waiting on
IO, have this status.(线程当前睡眠状态，阻塞等待一个同步锁，或者 IO)
- `false`: Threads that finished executing their block of code, or were successfully killed, have this status.(线程已经执行完毕，或者成功被杀掉)
- `nil`: Threads that raised an unhandled exception have this status. wwrt | 33(线程抛出一个未处理的异常，会有返回状态)
- `aborting`: Threads that are currently running, yet dying, have this status(线程当前运行中，但是死掉了了)


### Thread.stop

- 这个方法会使线程进入 sleep 状态，然后告诉线程调度器去执行另一个线程
- 线程会一直处于 sleep 状态，直到调用 `Thread#wakeup`

``` ruby
require 'thread'

thread = Thread.new do
  Thread.stop
  puts "Hello there"
end

# wait for the thread trigger its stop
puts "----" until thread.status == 'sleep'

thread.wakeup
thread.join

# 输出------
# ----
# ----
...
# ----
# ----
# ----
# Hello there
# [Finished in 1.6s]
```

### Thread.pass

`Thread.pass` 类似于 `Thread.stop` 但是他仅仅是让线程调度器去调度另一个线程，不会使当前线程处于 sleep


### Avoid Thread#raise

- 不推荐使用这个方法，因为会有严重的问题

### Avoid Thread#kill

- 不推荐使用这个方法，跟 `Thread#raise` 一样，会有严重的问题


## 第四章：Concurrent != Parallel

- concurrent and parallel are not the same thing
  1. Do multiple threads run your code concurrently? Yes.
  2. Do multiple threads run your code in parallel? Maybe.

- 单核 CPU 执行多个任务是并发的，但是并不一定有顺序执行快
- 多核 CPU 执行多个任务是并行的，但是也可以因为某个任务出现问题，然后由其它的线程或进程接管
- 并行一定是并发的，并发不一定是并行

### You can't guarantee anything will be parallel

- making it execute in parallel is out of your hands. That responsibility is left to the underlying thread scheduler(你亲手让程序并行执行，但是具体的并行的责任是交给底层的调度器来执行的)
- 多核 CPU 系统中执行多线程程序，也有可能会在一个 CPU 内核执行，这是由线程调度器决定的
- 线程采用公平排队的方式，所有的线程都可以或多或少的使用可用的资源，但是不能有代码来决定

扩展阅读
- https://blog.golang.org/concurrency-is-not-parallelism
- https://blog.engineyard.com/2011/ruby-concurrency-and-you


## 第五章：The GIL and MRI

- MRI allows concurrent execution of Ruby code, but prevents parallel execution of Ruby code

### The global lock

`GIT` 别名：`Global Interpreter Lock`， `GVL (Global VM Lock)`，  `Global Lock`

- 每个 MRI 进程都仅有一个 `GIL`,多个进程都有它自己的 `GIL`
- 进程中产生多个线程，这些线程会共享 `GIL`
- ruby 多线程中，单一线程会在任意给定时间获得 `GIL`，其它线程需要等待它释放`GIT`
- MRI ruby 不能够实现并行
- 即使是没有 `GIL` 的语言，比如 JAVA,使用多线程也会需要有对相同的公共资源进行访问和修改，如果需要加锁控制，也不能利用到多核并行
- 利用多进程实现并行，是 ruby 常用的方式

### The special case: blocking IO(特殊情况：IO 阻塞)

- ruby 中有 `GIL` 会阻止并行执行，但是 `IO 阻塞` 会释放 `GIL`
- MRI doesn't let a thread hog the GIL when it hits blocking IO(当它触发阻塞 IO 的时候，MRI 不会让线程贪婪占用 GIL)
- 因为 ruby 有 `GIL` ，所以它等于去除了操作系统并行执行的能力，但是等于所有的情况下都不能并行

``` ruby
require 'open-uri'
3.times.map do 
  Thread.new do
    open('http://zombo.com') 
  end
end.each(&:value)
```

运行以上代码，假设我们已经生成了所有的线程，他们都试图获取 `GIL` 来执行代码，Thread A 获得了 `GIL`,它创建了一个套接字并且试图打开一个连接到 `zombo.com`,这是线程 A 等待响应，并释放了 `GIL`, 线程 B 将获得 `GIL` 并且和线程 A 执行同样的步骤


There are three reasons that the GIL exists(几种 GIL 存在的原因 ):

1.  为了在竞争条件下保护 MRI 核心部件
  竞争条件会引起很多问题，这同样的问题会出现在 MRI的 C 内核, ,最简单的办法就是减少竞争的数量，防止多个线程同时运行
2. To facilitate the C extension API(为了便于使用 C 扩展 API)
    只要代码块用到了 C 语言扩展 API, GIL 会阻塞其它代码的运行，因为 C 扩展可能不是线程安全的，GIL 的存在保证了线程安全
3. To reduce the likelihood of race conditions in your Ruby code(尽可能的减少竞争条件)


...未完