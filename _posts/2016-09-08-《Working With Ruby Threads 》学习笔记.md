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

代码在：`chapter05/block_io_demo1.rb`

``` ruby
require 'open-uri'
3.times.map do 
  Thread.new do
    open('http://zombo.com') 
  end
end.each(&:value)
```

运行以上代码，假设我们已经生成了所有的线程，他们都试图获取 `GIL` 来执行代码，Thread A 获得了 `GIL`,它创建了一个套接字并且试图打开一个连接到 `zombo.com`,这是线程 A 等待响应，并释放了 `GIL`, 线程 B 将获得 `GIL` 并且和线程 A 执行同样的步骤


### Why?

There are three reasons that the GIL exists(几种 GIL 存在的原因 ):

1.  为了在竞争条件下保护 MRI 核心部件
  竞争条件会引起很多问题，这同样的问题会出现在 MRI的 C 内核, ,最简单的办法就是减少竞争的数量，防止多个线程同时运行
2. To facilitate the C extension API(为了便于使用 C 扩展 API)
    只要代码块用到了 C 语言扩展 API, GIL 会阻塞其它代码的运行，因为 C 扩展可能不是线程安全的，GIL 的存在保证了线程安全
3. To reduce the likelihood of race conditions in your Ruby code(尽可能的减少竞争条件)


### Misconceptions

#### 错误1: `Myth: the GIL guarantees your code will be thread-safe.`(GIL 保证你的代码是线程安全的)

- 这个观点是错误的
- GIL 只是大大减少并行的可能性，但并不能阻止竞争条件的发生，所以 GIL 不会保证线程安全

代码在：`chapter04/unsafe_counter.rb`

``` ruby
counter = 0
5.times.map do
  Thread.new do
    temp = @counter

    # 加入以下这行，将会导致结果出错，因为 IO 阻塞时，线程会释放 GIL，导致两个线程的 @counter 值相同
    # puts  temp 
    temp = temp + 1
    @counter = temp
  end
end.each(&:join)
puts @counter

```

- 以上代码对于 @counter 的增加等同于 `+=`
- 两个线程有可能同时竞争，对于 @counter 进行赋值，结果可能会少于 5，特别是在有 IO 阻塞，或者 JRuby 以及 Rubinius 的环境下会出现


#### 错误2:`Myth: the GIL prevents concurrency`

- GIL 阻止了并行(parallel) 执行 ruby 代码，但并不会阻止并发，这是术语的错误
- 并发是可能发生的，甚至在单核 CPU 的环境，也会给每一个线程分配资源
- 重要点：GIL 允许多个线程同时发生 IO 阻塞，这意味着可以在 ` IO-bound` 的情况下并行的执行代码


## 第六章：Real Parallel Threading with JRuby and Rubinius

- `JRuby and Rubinius don't have a GIL` JRuby 和 Rubinius 没有 GIL

### Proof

代码见 `chapter06/prime.rb`计算素数， MRI 没有 JRuby 和  Rubinius 快

使用 1.8.7 的版本

``` ruby
require 'benchmark'

def prime_sieve_upto(n)
  all_nums = (0..n).to_a
  all_nums[0] = all_nums[1] = nil
  all_nums.each do |p|

    #jump over nils
    next unless p

    #stop if we're too high already
    break if p * p > n

    #kill all multiples of this number
    (p*p).step(n, p){ |m| all_nums[m] = nil }
  end

  #remove unwanted nils
  all_nums.compact
end


primes = 1_000_000
iterations = 10
num_threads = 5
iterations_per_thread = iterations / num_threads

Benchmark.bm(15) do |x|
  x.report('single-threaded') do
    iterations.times do
      prime_sieve_upto(primes)
    end
  end
  x.report('multi-threaded') do
    num_threads.times.map do
      Thread.new do
        iterations_per_thread.times do
          prime_sieve_upto(primes)
        end
      end
    end.each(&:join)
  end
end

```


ree-1.8.7-2012.02
```
                     user     system      total        real
single-threaded  5.660000   0.060000   5.720000 (  5.725174)
multi-threaded   6.110000   0.110000   6.220000 (  6.228208)
```

MRI ruby  1.9.3-p551

```
                      user     system      total        real
single-threaded   3.450000   0.060000   3.510000 (  3.531772)
multi-threaded    3.660000   0.080000   3.740000 (  3.760532)
```

MRI ruby 2.0.0-p598

```
                      user     system      total        real
single-threaded   3.630000   0.080000   3.710000 (  3.726324)
multi-threaded    3.680000   0.090000   3.770000 (  3.808694)
```

MRI ruby 2.0.0-p648

```
                      user     system      total        real
single-threaded   3.210000   0.060000   3.270000 (  3.276048)
multi-threaded    3.330000   0.080000   3.410000 (  3.402474)
```

MRI ruby  2.1.0

```
                      user     system      total        real
single-threaded   2.360000   0.070000   2.430000 (  2.422242)
multi-threaded    2.390000   0.070000   2.460000 (  2.462325)
```

MRI ruby  2.2.3：

```
                      user     system      total        real
single-threaded   2.300000   0.070000   2.370000 (  2.361750)
multi-threaded    2.410000   0.080000   2.490000 (  2.482332)
```

jruby-9.0.4.0：
``` 
                      user     system      total        real
single-threaded   7.740000   0.280000   8.020000 (  2.676519)
multi-threaded   11.760000   0.230000  11.990000 (  3.064823)
```

MRI ruby 还是一直在进步，Rubinius 就没测试了，装的好慢，可恨的 `GFW`


### So... how many should you use?

真是应用的或许不是很清晰，可能某处是  IO-bound, 某处是  CPU-bound，也可能都不是，而是 memory-bound，或者也可能在任何地方也并没有最大化消耗资源

以 rails 应用作为例子：

- 与数据库之间的通信，与客户端通信，调用外部服务，大多数的机会是出现  IO-bound. 
- 另一方面会调用 CPU, 比如 渲染 HTML 模板， 或者转换数据到 JSON 文件

`the only way to a surefire answer is to measure`:
通过不同的线程数量去运行代码，然后分析测量结果，不通过测量，我们不能找到争取的答案

## 第七章：How Many Threads Are Too Many?

为了从并发获益，我们必须把一个问题拆分为可以同时运行的较小的任务，如果一个问题有不可分割的重要任务，那么使用并发也不能有更多的性能增益

- 唯一有保证的方法是 measure(测量) 和 compare(比较):
- 方法是：尝试在单核 CPU 上用一个线程，然后再试着用 5 个线程，比较两个的执行结果，然后改进它
- 新人一般会解决任务会以为更多的线程会比较快


### ALL the threads

```
1.upto(10_000) do |i|
  Thread.new { sleep }
  puts i
end

以上代码输出：
1
2
...
2043
2044
2045
2046
chapter06/spawning_threads.rb:2:in `initialize': can't create Thread: Resource temporarily unavailable (ThreadError)
```
- 这是因为 ruby 对一个进程可产生的线程有数量的硬限制
- ree-1.8.7 以及在 linux 系统上，可以产生至少 10000 的线程，但是我们并不可能用到


#### Context Switching

- 虽然每个线程都需要很少的内存开销，4 核 CPU 只能并行执行 4 个线程，会有大量线程阻塞在 IO 以及大量线程处于空闲状态，线程调度器需要开销去管理这些线程
- 虽然需要增加开销，但是产生比内核数量更多的线程也是有意义，因为 IO-bound 会释放 GIL，允许并行执行，而 CPU-bound 在非 MRI 的环境下是可以并行执行的，并且单核 CPU 也是可以实现并发的

#### IO-bound


- 如果代码执行调用 web 请求的外部服务，有更好的网络连接速度，程序会更快
- 如果代码有大量读写硬盘操作，有支持更快读写的硬盘，程序会因为硬件升级而更快
- 以上两条是 IO-bound 的情况，因为需要从 IO 设备等待响应，产生比内核更多的线程是有意义的

代码例子见：`./chapter06/io_bound.rb` 

- 如果 IO 操作延迟比较高，我们需要更多的线程去解决 sweet spot, 因为线程多，阻塞等待的时间会更长，如果 IO 延迟比较低，那么我们需要更少的线程去解决 sweet spot,因为等待时间少，线程释放也会很快


### CPU-bound

... 待续


## 第八章：Thread safety

### What's really at stake?

When your code isn't thread-safe, the worst that can happen is that your underlying data becomes incorrect
当你的代码不是线程安全的，这最坏的情况会发生，你的基础数据会变得不正确

- If your code is 'thread-safe,' that means that you can run your code in a multi- threaded context and your underlying data will be safe.(基础数据安全)
- If your code is 'thread-safe,' that means that you can run your code in a multi- threaded context and your underlying data remains consistent.(基础数据保持一致)
-  If your code is 'thread-safe,' that means that you can run your code in a multi- threaded context and the semantics of your program are always correct.(程序在语义上正确)


### The computer is oblivious

- The computer is unaware of thread-safety issues.

### Is anything thread-safe by default?

- any concurrent modifications to the same object are not thread- safe.


## 第九章：Protecting Data with Mutexes

### Mutual exclusion

- If you wrap some section of your code with a mutex, you guarantee that no two threads can enter that section at the same time.(如果你用 mutex 包含一段代码，在同一时间不会有两个线程同时进入)
- Until the owning thread unlocks the mutex, no other thread can lock it(一直到所属的线程解锁前，没有其它线程能锁定)


``` ruby
# 通用的 mutex 使用方式
mutex.synchronize do 
  shared_array << nil
end
```


### The contract

- 注意： `the mutex is shared among all the threads` 互斥在所有线程中共享


### Making key operations atomic
- 使用 mutex 互斥所的操作需要具有原子性，不然会出现错误的结果

### Mutexes and memory visibility

- mutexes carry an implicit `memory barrier`(互斥锁能实现内存屏障)
- 程序在运行时内存实际的访问顺序和程序代码编写的访问顺序不一定一致，这就是内存乱序访问, `Memory barrier` 能够让 CPU 或编译器在内存访问上有序

### Mutex performance

-  mutexes inhibit parallelism(互斥锁抑制并行)
-  `GIL` 和互斥锁的行为一样，在同一时间只能有一个线程执行代码
- restrict the critical section to be as small as possible, while still preserving the safety of your data（互斥所限制的部分应该尽可能的小，并且同时保证数据安全性）,限制部分更小，那么可以让其它更多的代码并行执行,就是所谓的`finer-grained mutex`细粒度互斥锁

## 第 10 章: Signaling Threads with Condition Variables

### The API by example

- `ConditionVariable#wait` 会 unlock mutex，并使线程进入 sleep
-  `ConditionVariable#signal` 发信号后，第一个等待线程会获取 mutex，并且继续执行

代码在：chapter10/xkcd_printer.rb

### Broadcast

- `ConditionVariable#signal`

重开1个正在等待状态变量的线程。重开的线程将尝试ConditionVariable#wait所指的mutex锁。若有等待状态的线程的话，就返回该线程。除此之外将返回nil 。

- `ConditionVariable#broadcast` 

重开所有正在等待状态变量的线程。重开的线程将尝试`ConditionVariable#wait` 所指的 mutex 锁

## 第 11 章: Thread-safe Data Structures


- 阻塞队列使用放在共享对象内部的 mutex，而不是全局的，对象共享给各个线程，这个有每个共享对象保证自己的并发读写正确
- 书中的 `BlockingQueue` 使用 `ConditionVariable`，如果在队列为空的情况下，让线程进入 sleep
- `Queue` 是 ruby 标准库提供的唯一线程安全的数据结构，它是通过 `require 'thread'` 加载的，它也是阻塞队列
- ruby 的 `Array` 和 `Hash` 不是线程安全的， Jruby 以及 java 的也不是，在单线程中使用线程安全数据结构会降低性能，但是 java 有替代品
- 在 ruby 中，要使用线程安全的 Array 和 Hash，可以用  `thread_safe` rubygem 中的 `ThreadSafe::Array` `ThreadSafe::Hash`

## 第 12 章：Writing Thread-safe Code

- Idiomatic Ruby code is most often thread-safe Ruby code
惯用的(Idiomatic)Ruby 代码往往是线程安全的代码
- Avoid mutating globals 避免修改全局，全局变量会在所有线程中共享
  - 任何只有一个共享实例的东西都是全局的。比如：Constants（常量），AST(abstract syntax trees)，类变量，类方法
  - modifying the AST at runtime is almost always a bad idea, especially when multiple threads are
involved.（在运行时修改 AST 往往是坏主意，特别是在多线程环境下）
   - In other words, it's expected that the AST will be modified at startup time(换句话说，AST 的修改最好在程序启动时)
- Create more objects, rather than sharing one(创建更多对象，而不是共享一个)
  - **Thread-locals:**
    多个线程创建多个连接，对于少量的多线程是合适的，但对于并发较高的多线程不太适合，开销过大，改用线程池是比较合适的
  ``` ruby
  # Instead of
   $redis = Redis.new
   # use
   Thread.current[:redis] = Redis.new
  ```
  -  **Resource pools:**
  一个线程池将打开多个连接，或者是需要在多线程中共享的资源，当一个线程要使用一个连接时，它会要求连接池拿出一个连接，线程池负责检查连接是否可用并提供给线程使用，保证线程安全，当线程执行完成后，将连接放回连接池内
  connection_pool rubygem：https://github.com/mperham/connection_pool
  - **Avoid lazy loading（避免延迟加载）：**
    `autoload`是延迟并在运行时加载，在 MRI ruby 中不是线程安全的，Jruby 中是线程安全的
    rails3 中 `autoload`  也不是线程安全的，需要启用 `config.threadsafe!`，在 rails4 中是线程安全
  - **Prefer data structures over mutexes：**（优先考虑线程安全的数据结构，而不是互斥锁）
    互斥锁 mutex 是很难用好的，你需要决定很多问题：
    - 互斥的粒度粗细
    - 哪些代码应该在关键部分
    - 会不会引发死锁
    - 需要一个单个实例锁还是全局锁
   大多数程序员并不熟悉 mutex，所以使用线程安全的数据结构就可以避免使用互斥锁的诸多顾虑，`you simply don't need tocreate any mutexes in your code.`（你根本不需要在你的代码中创建任何的互斥锁）
    - **Finding bugs:**
      尽管你已经遵循所有的最佳实践，但还是会有莫名其妙的 bug 出现，并且可能非常难以去追踪或重现，最好的办法是去阅读源代码
      最常见的问题就是全局的引用，所以你可以试着用 2 个线程去同时访问，通过这样的实践，问题的原因可能会突然浮现


... 未完