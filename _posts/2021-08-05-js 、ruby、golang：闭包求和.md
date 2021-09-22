---
layout: post
title:  "js 、ruby、golang：闭包求和"
date:   2021-08-05
description: 'js 、ruby、golang：闭包求和'
category: notes
---

## 闭包概念介绍

闭包变量的作用域：

- js 中，函数可以用来创造函数作用域，函数内可以看到外面的变量，函数外无法看到函数内部的变量
- 当在函数中搜索一个变量时，如果函数内没有声明这个变量，那么会随着代码执行环境创建的作用域链往外层逐层搜索，一直搜索到全局对象为止

闭包变量的生命周期：

- 全局变量的生命周期当然是永久的，除非主动销毁
- 函数内部用 var 生命的局部变量，会随着函数的调用结束而被销毁

闭包的作用：

- 封装变量：把一些不需要暴露在全局的变量封装成 “私有变量”
- 延续局部变量的寿命：

## js 版闭包求和

``` javascript
var getSum = (function() {
	sum = 0
	return function(v) {
		sum += v
		return sum
	}
})()
for( i = 0; i < 10; i++){
    console.log("0 + 1 + ... + "+ i +" = " + getSum(i))
}
```

## ruby 版闭包计数器

ruby 中使用 `lambda` 实现

``` ruby
def addr
  sum = 0
  return ->(x) { sum += x; sum }
end

get_sum = addr
10.times do |i|
  puts "0 + 1 + ... + #{i} = #{get_sum.call(i)}"
end
```

## golang 版闭包求和

``` go
func Adder() func(int) int {
	sum := 0
	return func(v int) int {
		sum += v
		return sum
	}
}

func main() {
	var getSum = Adder()
	for i := 0; i < 10; i++ {
		fmt.Printf("0 + 1 + ... + %d = %d\n", i, getSum(i))
	}
}
```

## 三种语言的区别

- js 中可以使用匿名函数，函数可以作为参数和函数值
- ruby 版本需要借助 `lambda` 实现
- go 中也可以用匿名函数，但只能在函数内部
- 三个版本都需要注意函数调用结束后闭包内的私有变量会被销毁