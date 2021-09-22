---
layout: post
title:  "js 、ruby、golang：AOP 实现"
date:   2021-08-06
description: 'js 、ruby、golang：AOP 实现'
category: notes
---

## AOP 介绍

AOP(Aspect-Oriented Programming)，中文叫面向切面编程，主要作用是把一些跟核心业务逻辑模块无关的功能抽离出来，这些跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。把这些功能抽离出来，再通过“动态织入”的方式掺入业务逻辑模块中。

## ruby 版

**第一种：**

```ruby
module BarkLogger
  def bark
    puts "Logging #@name's bark!"
    super
  end
end

class Dog
  prepend BarkLogger

  def initialize(name)
    @name = name
  end

  def bark
    puts "#@name the dog says bark!"
  end
end

Dog.new("Rufus").bark
```

**第二种：**

给原方法增加日志

```ruby
class Module
  def add_logging(*method_names)
    method_names.each do |method_name|
      original_method = instance_method(method_name)
      define_method(method_name) do |*args,&blk|
        puts "logging #{method_name}"
        original_method.bind(self).call(*args,&blk)
      end
    end
  end
end

class A
  add_logging :test
end
```

**ruby 第三种**

- 通过 `alias_method` 
- `Active Support` 中有 `alias_method_chain` 环绕别名构建器更方便实现

## js 版本

给原函数增加 `before` 和 `after`

```js
Function.prototype.before = function(beforefn) {
        var __self = this; // 保存原函数的引用
        return function() { // 返回包含了原函数和新函数的代理函数
            beforefn.apply(this, arguments); // 执行新函数，修正 this
            return __self.apply(this, arguments); // 执行原函数
        }
    }
    Function.prototype.after = function(afterfn) {
        var __self = this; // 保存原函数的引用
        return function() { // 返回包含了原函数和新函数的代理函数
            var ret = __self.apply(this, arguments) // 执行原函数
            afterfn.apply(this, arguments); // 执行新函数，修正 this
            return ret;
        }
    }

    var func = function() {
        console.log(2);
    }
    func = func.before(function() {
        console.log(1)
    }).after(function() {
        console.log(3);
    });

    func();
```

## golang 版本

简单的 http 服务中间件

```go
// basic-middleware.go
package main

import (
    "fmt"
    "log"
    "net/http"
)

func logging(f http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        log.Println(r.URL.Path)
        f(w, r)
    }
}

func foo(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "foo")
}

func bar(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "bar")
}

func main() {
    http.HandleFunc("/foo", logging(foo))
    http.HandleFunc("/bar", logging(bar))

    http.ListenAndServe(":8080", nil)
}
```