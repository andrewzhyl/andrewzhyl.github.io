---
layout: post
title:  "《Javascript 设计模式与开发实践》学习笔记"
date:   2015-08-05
description: '《Javascript 设计模式与开发实践》学习笔记'
category: notes
---

## 第 1 章：面向对象的 Javascript

- js 是通过原型委托的方式来实现对象与对象之间的继承
- 编程语言按照数据类型大体分为两类，一类是静态类型语言，另一类是动态类型语言。
- 静态语言在编译时便已确定变量的类型
- 动态语言的变量类型要到程序运行的时候，待变量被赋予某个值之后，才会具有某种类型

#### 静态类型

**优点：**

- 编译时就能发现类型不匹配的错误，提前避免运行时的错误
- 明确规定了数据类型，编译器会针对性的做优化，提高程序执行速度

**缺点：**

- 迫使程序员依照强契约来编写程序，为变量规定数据类型只是编写可靠性高程序的一种手段，而不是编写程序的目的
- 类型的声明也会增加更多的代码，细节会让程序员的精力从思考业务逻辑上分散开来

#### 动态类型

**优点：**

- 代码量少，更加简洁，更加易于阅读
- 程序员可以把精力更多的放在业务逻辑上
- 由于无需进行类型检测，我们可以尝试调用任何对象的任意方法，无需考虑它是否拥有

**缺点：**

- 无法保证变量的类型，从而在程序运行期有可能发生跟类型相关的错误

#### 鸭子类型

- 鸭子类型(`duck typing`)：如果它走起路来像鸭子，叫起来也是鸭子，那么它就是鸭子
- `面向接口编程，而不是面向实现编程`： 
	- 一个对象若有 `push` 和 `pop`,并且提供了正确的实现，它就可以被当做栈
	- 如果一个对象有 `length` 属性，也可以用下标来存取属性，那么对象可以被当做数组来用

#### 多态

- 多态含义：同一操作作用于不同的对象上面，可以产生不同的解释和不同的执行结果
- 开放封闭原则：对扩展开放，对修改封闭。
- 把不变的部分隔离出来，把可变的部分封装起来
- java 中通过继承，向上转型来实现多态，js 中不需要
- 多态的好处在于不必问对象是什么类型，根据得到的答案调用对象的某个行为

#### 封装

- 封装的目的是将信息隐藏。
- 封装数据：部分语言的对象系统中，封装数据是由词法解析来实现的，提供了 private、public、protected 等关键字来提供不同的访问权限
- js 中需要依赖变量的作用域来实现封装特性，只能模拟出 public 和 private 另种封装特性
- 封装实现：对象之间通过暴露的 API 接口来通信，其它对象和用户都不关心它的内部实现
- 封装类型：通过抽象类和接口来进行，把对象真正的类型隐藏在抽象类和接口之后，相比对象的类型，客户更关心对象的行为。比如工厂方法模式、组合模式等
- 封装变化：把容易变化的部分和稳定不变的部分隔离开来，在系统演变过程中，只需要替换已封装的容易变化的部分，保证程序的稳定性和可扩展性

#### javascript 中的原型继承

![js原型继承](/assets/images/js原型继承.png)


原型编程的基本原则:

- 所有的数据都是对象
- 要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它
- 对象会记住它的原型
- 如果对象无法响应某个请求，它会把这个请求委托给它自己的原型

**所有的数据都是对象：**

js 引入了两套类型机制：基本类型和对象类型。基本类型包括：undefined、number、boolean、string、function、object

**要得到一个对象，不是通过实例化类，而是找到一个对象作为原型并克隆它:**

- js 中没有类的概念，函数构造器可以用 `new` 运算符来创建对象，实际只是先克隆 `Object.prototype` 对象，再进行其他额外的操作。函数构造器也可以作为普通函数使用
- 对象有 `[[prototype]]` 指针指向正确的原型，在 chrome 和 firefox 中是 `_proto_`

``` javascript
function Person() {
  this.name = name;
};

Person.prototype.getName = function() {
  return this.name;
};

var objectFactory = function() {

  // 从 Object.prototype 上克隆一个空的对象
  var obj = new Object(),

  // 取得外部传入的构造器，此例是 Person
  Constructor = [].shift.call(arguments); 
  obj._proto_ = Constructor.prototype; // 指向正确的原型
}

var a = objectFactory(Person, 'seven');

console.log(a.name); // 输出  sven
console.log(a.getName()); // 输出:sven
console.log(Object.getPrototypeOf(a) === Person.prototype); // 输出： true

```

**对象会记住它的原型：**

- `对象的原型` 并不是对象有原型，而是对象的构造器有原型，对象将请求委托给它的构造器的原型
- js 给对象提供了一个名为 `_proto_` 的隐藏属性，对象的 `_proto_` 属性默认会指向它的构造器的原型对象，即 `[Constructor].prototype` ，在 chrome 和 firefox 中 `_proto_`  被公开出来

**如果对象无法响应某个请求，它会把这个请求委托给它自己的原型:**

- 原型继承的精髓：当一个对象无法响应某个请求的时候，它会顺着原型链把请求传递下去，知道遇到一个可以处理该请求的对象为止
- js 的对象都是从 `Object.prototype` 对象克隆而来，只能得到单一的继承关系，单对象构造器的原型可以动态的指向其它对象


## 原型继承的未来

- Peter Norvig 说： 设计模式是对语言不足的补充，如果要适用设计模式，不如去找一门更好的语言。
- 通过设置构造器的 `prototype` 的来实现原型继承的时候，除了根对象 `Object.prototype` 本身之外，任何对象都会有一个原型。
- 通过 `Object.create(null)` 可以创建出没有原型的对象，大多数浏览器提供了 `Object.create` 方法，`Object.create` 方法是原型模式的天然实现

## 第 2 章：this、call 和 apply

### this

`js` 的 this 总指向一个对象，具体指向哪个对象是在运行时基于函数的执行环境动态绑定的，而非函数被声明时的环境

#### this 的指向：

this 的指向可以分为 4 种情况：

- 作为对象的方法调用
- 作为普通函数调用
- 构造器调用
- `Function.prototype.call` 或 `Function.prototype.apply` 调用

**作为对象的方法调用：**

this 指向该对象

**作为普通函数调用**:

- `this` 指向 `window`(全局对象)
- `js` 处理元素的事件函数内部，callback 作为普通函数调用时，内部函数的，this 指向了 window，看代码：
- 
``` javascript
    document.getElementById('div1').onclick = function() {
        var that = this;
        console.log(this.id); // 输出 'div1'
        var callback = function() {
            alert(that.id);
        }
        callback();
    }
```

-  `ECMAScript 5` 的 `strict` 模式下， `this` 已被规定为不会指向全局对象，而是 `undefined`:

``` javascript
function func(){
	"use strict"; // 启用严格模式
	alert(this); // 输出： undefined
}
```

**构造器调用：**

- 使用 `new` 运算符调用函数时，该函数总会返回一个对象，`this` 指向构造器返回的这个对象
- 如果 `new` 调用构造器时，不能显示的指定返回值

#### 丢失的 this：

- 作为普通函数调用，函数内部的 this 会指向 window

以下代码会抛出异常，因为 `getId` 引用 `document.getElementById` ，调用 `getId`，此时就成了普通函数调用，函数内部的 this 指向了 widnow， 而不是原来的 `document`
``` javascript
    <div id="div1">我是一个 div</div>
    <script type="text/javascript">
    var getId = document.getElementById;
    getId('div1');
    </script>
```

### call 和 apply

- `ECMAScript3` 给 Function 的原型定义了两个方法，分别是 `Function.prototype.call` 和  `Function.prototype.apply`  

**call 和 apply 的区别：**

- `apply` 接受两个参数，第一个参数指定了函数体内 this 对象的指向，第二个参数为一个带下标的集合（数组或者类数组）
- `call` 传入参数数量不固定，第一个参数也是 this 指向，第二个往后的参数不定，依次传入
- `call` 是包装在 `apply` 上面的一颗语法糖

``` javascript
# 使用 call 或 apply 时，如果第一个参数为 null, 函数内的 this 会指向默认的宿主对象，浏览器你是 window

var func = function(a, b, c){
	alert( this ==== window); // 输出 true,如果启用严格模式，this 会是 null
}

func.apply(null, [1,2,3]);
```

**call 和 apply 的用途：**

- 改变 this 指向
- Function.prototype.bind

``` javascript
    Function.prototype.bind = function(context) {
        var self = this; // 保存原函数
        return function() { // 返回一个新的函数
            return self.apply(context, arguments); // 执行新的函数的时候，会把之前传入的 context 当做新函数体内的 this
        }
    }

    var obj = {
        name: 'seven'
    };

    var func = function() {
        console.log(this.name); // 输出 seven
    }.bind(obj);

    func();
```
- 借用其他对象的方法

## 第 3 章：闭包和高阶函数

js 设计之初参考了 LISP 的两大方言之一的 `Scheme`，引入了 `Lambda` 表达式、闭包、高阶函数等特性


### 闭包

**变量的作用域：**

- 在函数中声明一个变量时，如果变量前没有关键字 var，这个变量会成为全局变量
- js 中，函数可以用来创造函数作用域，函数内可以看到外面的变量，函数外无法看到函数内部的变量
- 当在函数中搜索一个变量时，如果函数内没有声明这个变量，那么会随着代码执行环境创建的作用域链往外层逐层搜索，一直搜索到全局对象为止

**变量的生存周期：**

- 全局变量的生命周期当然是永久的，除非主动销毁
- 函数内部用 var 生命的局部变量，会随着函数的调用结束而被销毁

闭包的例子：

``` javascript
    var nodes = document.getElementsByTagName('div');
    var len = nodes.length;
    for (var i = 0; i < len; i++) {
        (function(i) {
            nodes[i].onclick = function() {
                alert(i);
            }
        })(i);
    }
```

**闭包的更多作用：**

- 封装变量：把一些不需要暴露在全局的变量封装成 “私有变量”
- 延续局部变量的寿命：

``` javascript
  // var report = function(src) {
  //     var img = new Image();
  //     img.src = src;
  // }

  // img 变量用闭包封闭起来，可以解决请求丢失的问题:
    var report = (function() {
        var imgs = [];
        return function(src) {
            var img = new Image();
            imgs.push(img);
            img.src = src;
        }
    })();
```

**闭包和面向对象设计：**

闭包相关代码：

``` javascript
var extend = function() {
    var value = 0;
    return {
      call: function() {
        value++;
        console.log(value);
      }
    }
  };

  var extend = extend();
  extend.call(); // 输出 1
  extend.call(); // 输出 1
  extend.call(); // 输出 1
```

面向对象的写法：

``` javascript
var extend = {
        value: 0,
        call: function() {
            this.value++;
            console.log(this.value);
        }
    }

    // var extend = extend();
    extend.call(); // 输出 1
    extend.call(); // 输出 1
    extend.call(); // 输出 1
```

原型继承写法：

``` javascript
    var Extend = function() {
        this.value = 0;
    }

    Extend.prototype.call = function() {
        this.value++;
        console.log(this.value);
    }

    var extend = new Extend();
    extend.call(); // 输出 1
    extend.call(); // 输出 2
    extend.call(); // 输出 3
```

**闭包和内存管理：**

- 局部变量本应该在函数退出时被解除引用，但如果局部变量被封闭在闭包形成的环境中，那么可以局部变量可以一直生存下去。如果需要回收变量，可以手动设置变量为 `null`
- 在 IE 浏览器中，由于 BOM 和 DOM 中的对象是使用 C++ 以 COM 对象的方式实现的，而 COM 对象的垃圾收集机制采用的是`引用计数策略`。在`引用计数策略`垃圾回收机制中，如果两个对象之间形成了循环引用，那么这两个对象都无法被回收
- 解决循环引用带来的内存泄露问题，只需要把循环引用中的变量设为 null 即可

### 高阶函数

高阶函数至少满足以下条件：

- 函数可以作为参数被传递；
- 函数可以作为返回值输出


**单例模式：**

``` javascript
 var getSingle = function(fn) {
        var ret;
        return function() {
            return ret || (ret = fn.apply(this, arguments));
        }
    }

    var getScript = getSingle(function(){
        return document.createElement('script');
    })

    var script1 = getScript();
    var script2 = getScript();

    console.log(script1 === script2);
```


**高阶函数实现 AOP：**

- AOP(面向切面编程)的主要作用是把一些跟核心业务逻辑模块无关的功能抽离出来，这些跟业务逻辑无关的功能通常包括日志统计、安全控制、异常处理等。把这些功能抽离出来，再通过“动态织入”的方式掺入业务逻辑模块中。

``` javascript
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

**柯里化(currying):***

- `currying` 又称部分求值。函数首先会接受一些参数，接受了这些参数之后，该函数并不会立即求值，而是继续返回另外一个函数，到最后真正需要求值时，所有参数都会被一次性用于求值

``` javascript

var currying = function(fn){
    var args = [];

    return function(){
        if(arguments.length === 0){
            return fn.apply(this, args);
        }else{
            [].push.apply(args, arguments);
            return arguments.callee;
        }
    };

};
```

**uncurrying:**

``` javascript
// uncurrying 实现
Function.prototype.uncurrying = function() {
    var self = this;
    return function() {
        var obj = Array.prototype.shift.call(arguments);
        return self.apply(obj, arguments);
    };
};
```

**函数节流:**

函数被频繁调用的场景：

- `window.onresize` 事件，浏览器窗口大小被改变的时候
- `mousemove` 事件
- 上传进度

函数节流的代码实现：

``` javascript
var throttle = function(fn, interval) {
    var _self = fn, // 保存需要被延迟执行的函数引用
        timer, // 定时器
        firstTime = true; // 是否是第一次调用

    return function() {
        var args = arguments,
            __me = this;

        if (firstTime) { // 如果是第一次调用，不需要延迟执行
            _self.apply(__me, args);
            return firstTime = false;
        }

        if(timer){ // 如果定时器还在，说明前一次延迟执行还没有完成
            return false;
        }

        timer = setTimeout(function(){ // 延迟一段时间执行
            clearTimeout(timer);
            timer = null;
            _self.apply(__me, args);
        }, interval || 500);
    };
};

window.onresize = throttle(function(){
    console.log(1);
}, 500);
```

函数节流的原理：

将即将被执行的函数用 `setTimeout` 延迟一段时间执行。如果延迟执行还没有完成，则忽略接下来的调用


**分时函数：**

通过定时调用 `setInterval`，可以将大量数据的处理分批进行

``` javascript
var timeChunk = function(ary, fn, count) {
    var obj, t;
    
    var start = function() {

        //  Math.min(int a, int b)方法用法实例教程 - 此方法返回a和b中的较小的, 避免 count 大于 ary.length 的 bug
        for (var i = 0; i < Math.min(count || 1, ary.length); i++) {
            var obj = ary.shift();
            fn(obj);
        }
    };


    return function() {
        t = setInterval(function() {
            if (ary.length === 0) { // 如果全部节点都已经被创建好
                return clearInterval(t);
            }
            start();
        }, 200); // 分批执行的时间间隔，也可以用参数的方式
    }
}
```

**惰性加载函数：**

addEvent 函数内有分支判断，第一次进入条件分支后，函数内部会重写这个函数

``` javascript
    // 方案 3，惰性载入函数方案

    var addEvent = function(elem, type, handler) {
        if (window.addEventListener){
            addEvent = function(elem, type, handler){
                elem.addEventListener(type, handler, false)
            }
        }else if(window.attachEvent){
            addEvent = function(elem, type, handler){
                elem.attachEvent('on' + type, handler);
            }
        }
        addEvent(elem, type, handler);
    };

    var div = document.getElementById('div1');
    addEvent(div, 'click', function(){
        alert('hello');
    });

    addEvent(div, 'click', function(){
        alert('world');
    });

    addEvent(div, 'click', function(){
        alert('andrew');
    });
```

## 第 4 章：单例模式

单例模式定义：
保证一个类仅有一个实例，并提供一个访问它的全局访问点。

**代理实现单例模式：**

``` javascript
<script type="text/javascript">
var CreateDiv = function(html) {
    this.html = html;
    this.init();
};

CreateDiv.prototype.init = function() {
    var div = document.createElement('div');
    div.innerHTML = this.html;
    document.body.appendChild(div);
};

var ProxySingletonCreateDiv = (function() {
    var instance;
    return function(html) {
        if (!instance) {
            instance = new CreateDiv(html);
        }
        return instance;
    }
})();

var a = new ProxySingletonCreateDiv('sven1');
var b = new ProxySingletonCreateDiv('sven2');
alert(a === b); // true
</script>
```

**javascript 中的的单例模式：**

- javascript 是无类(`clas-free`)语言，面向对象的单例模式不适用
- 单例模式的核心：确保只有一个实例，并提供全局访问。


### 降低全局变量命名污染的方式:

**1. 使用命名空间：**


``` javascript
//  对象字面量命名空间
var namespace1 = {
    a: function() {
        alert(1);
    },
    b: function() {
        alert(2);
    }
};

// 动态创建命名空间
var Myapp = {};

Myapp.namespace = function(name) {
    var parts = name.split('.');
    var current = Myapp;
    for (var i in parts) {
        if (!current[parts[i]]) {
            current[parts[i]] = {};
        }
        current = current[parts[i]];
    }
};

Myapp.namespace('event');
Myapp.namespace('dom.style');

console.dir(Myapp);

// 上述代码等价于：
var Myapp = {
    event: {},
    dom: {
        style: {}
    }
}
```

2. 使用闭包封装私有变量

``` javascript
// 使用闭包封装私有变量
var user = (function() {
    var _name = 'seven',
        _age = 29;

    return {
        getUserInfo: function() {
            return _name + '-' + _age;
        }
    }
})();
```


**惰性单例：**

- 在需要的时候才创建对象实例

通用惰性单例示例代码：

``` javascript
<button id="loginbtn">打开百度</button>
<script type="text/javascript">
var getSingle = function(fn) {
    var result;
    return function() {
        return result || (result = fn.apply(this, arguments));
    };
};


var createSingleIframe = getSingle(function(){
    var iframe = document.createElement('iframe');
    document.body.appendChild(iframe);
    return iframe;
});

document.getElementById('loginbtn').onclick = function() {
    var loginLayer = createSingleIframe();
    loginLayer.src = "http://baidu.com";
}
</script>
```

## 第 5 章：策略模式

- 策略模式的定义：定义一系列的算法，把它们一个个封装起来，并且使它们可以相互转换。
- 基于策略模式的程序至少由两部分组成:
	- 第一部分是一组策略类，策略类封装了具体的算法，并负责具体的计算过程
	- 第二部分是环境类 `Context`，`Context` 接受客户的请求，随后把请求委托给某一个策略类
- 策略模式可以消除程序中的大片的条件分支语句，这是对象多态性的体现
- `Peter Norvig` 说过："在函数作为一等对象的语言中，策略模式是隐形的。strategy 就是值为函数的变量"

**策略模式的优缺点：**

优点：

- 策略模式利用组合、委托和多态等技术和思想，可以有效的避免多重条件选择语句
- 策略模式提供了对开放-封闭原则的完美支持，将算法封装在独立的 strategy 语句中，使得它们易于切换，易于理解，易于扩展
- 策略模式中的算法也可以复用在系统的其它地方，从而避免许多重复的复制粘贴工作
- 在策略模式中利用组合和委托来让 `Context` 拥有执行算法的能力，这也是继承的一种更轻便的替代方案

缺点：

- 策略模式会在程序中增加许多策略类或者策略对象
- 使用策略模式，strategy 要向客户暴露它的所有实现，这是违反最少知识原则的

## 第 6 章：代理模式

- 代理模式是为一个对象提供一个代用品或占位符，以便于控制对它的访问
- 生活中的场景：明星的经纪人
- 代理模式的关键：当客户不方便直接访问一个对象或不满足需要的时候，提供一个替身对象来控制对这个对象的访问，客户实际访问替身对象。替身对象对请求做出一些处理之后，再把请求转交给本体对象
- `单一职责：` 就一个类（通常也包括对象和函数等）而言，应该仅有一个引起它变化的原因
- 代理和本体接口的一致性：
	- 用户可以放心的请求代理，他只关心是否能得到想要的结果
	- 在任何使用本体的地方都可以替换成并使用代理
- 编写业务代码时，不需要猜测是否需要使用代理模式，当真正发现不方便直接访问某个对象时，再编写代理也不迟

虚拟代理实现图片预加载：

``` javascript
    <script type="text/javascript">
    var myImage = (function() {
        var imgNode = document.createElement('img');
        document.body.appendChild(imgNode);
        return function(src) {
            imgNode.src = src;
        };
    })();

    // 没有改变或增加 myImage 的接口
    // 通过代理给系统增加了新的行为，符合开放封闭原则
    var proxyImage = (function() {
        var img = new Image;
        img.onload = function() {
            var src = this.src;
            setTimeout(function() {
                myImage(src);
            }, 1000);
        }

        return function(src) {
            myImage("/images/loading.gif");
            img.src = src;
        };

    })();

    // proxyImage('http://ww1.sinaimg.cn/large/aeb19806gw1ermlpunhdej20rs15owon.jpg')
    proxyImage("http://bbsimg0.dahe.cn/Day_141111/1038_14484479_acc705d86c0bc0a.jpg");

    </script>
```


## 第 7 章：迭代器模式

- 迭代器模式：是指提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示

内部迭代器：

``` javascript
var each = function(ary, callback){
  for( var i = 0, l = ary.length; i < l; i++){
    callback.call(ary[i], i, ary[i]); // 把下标和元素当作参数传给 callback 参数
  }
};
```

外部迭代器：

``` javascript
var Iterator = function(obj) {
    var current = 0;

    var next = function() {
        current += 1;
    };

    var isDone = function() {
        return current >= obj.length;
    };

    var getCurrItem = function() {
        return obj[current];
    };

    return {
        next: next,
        isDone: isDone,
        getCurrItem: getCurrItem
    }
};
```

## 第 8 章：发布订阅模式

- 发布订阅模式又叫观察者模式，它定义对象之间的一种一对多的关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知。
- js 开发中，一般用事件模型来代替传统的发布-订阅模式
- 发布订阅模式可以广泛应用于异步编程中，替代传递回调函数的方案
- 发布订阅模式可以取代对象之间硬编码的通知机制，一个对象不用再显式得调用另外一个对象的某个接口，松耦合的联系在一起，不清楚彼此细节，但不影响通信。
- `DOM 事件`就是使用发布-订阅模式

**自定义事件：**

- 首先要指定好谁充当发布者（比如售楼部）；
- 然后给发布者添加一个缓存列表，用于存放回调函数以便于通知订阅者（售楼部的花名册）
- 最后发布消息时，发布者会遍历这个缓存列表，依次触发里面存放的订阅者回调函数（遍历花名册，挨个发短信）

发布订阅模式的通用实现：

``` javascript
var event = {
    clientList: [],
    listen: function(key, fn) {
        if (!this.clientList[key]) { // 如果还没有订阅过此类消息，给该类消息创建一个缓存列表
            this.clientList[key] = [];
        }
        this.clientList[key].push(fn); // 订阅的消息添加进消息缓存列表
    },
    trigger: function() {
        var key = Array.prototype.shift.call(arguments), // 取出消息类型
            fns = this.clientList[key]; //取出该类消息对应的回调函数集合

        if (!fns || fns.length === 0) { // 如果没有订阅该消息，则返回
            return false;
        }

        for (var i = 0, fn; fn = fns[i++];) {
            fn.apply(this, arguments); // arguments 是发布消息时带上的参数
        }
    }
};

// 给对象动态安装发布-订阅功能
var installEvent = function(obj) {
    for (var i in event) {
        obj[i] = event[i];
    }
};
```

## 第 9 章：命令模式

- 命令模式中的命令指的是一个执行某些特定事情的指令
- 命令模式的应用场景是：有时候需要向某些对象发送请求，但是并不知道请求的接受者是谁，也不知道被请求的操作是什么
- 命令模式是一种松耦合的设计方式，使命令发送者和请求接收者能够消除彼此之间的耦合关系
- 设计模式的主题总是把不变的事物和变化的事物分离开来

js 命令模式示例：

``` javascript
var button1 = document.getElementById('button1');

var setCommand = function(button, command) {
    button.onclick = function(){
        command.execute();
    }
};

var MenuBar = {
    refresh: function() {
        console.log('刷新菜单目录');
    }
};

var SubMenu = {
    add: function() {
        console.log('增加子菜单');
    },
    del: function() {
        console.log('删除子菜单');
    }
};

var RefreshMenuBarCommand = function(receiver) {
    return {
        execute: function() {
            receiver.refresh();
        }
    };
};

var refreshMenuBarCommand  = RefreshMenuBarCommand(MenuBar);
setCommand(button1, refreshMenuBarCommand);
```

## 第 10 章：组合模式

- 组合模式是用小的子对象来构建更大的对象，而小的子对象本身也许是由更小的“孙对象”构成的
- 组合模式将对象组合成树形结构，以表示 “部分-整体” 的层次结构
- 组合模式通过对象多态性表现，使得用户对单个对象和组合对象的使用具有一致性

- 组合模式注意的地方：
	- 组合模式不是父子关系：组合模式是 `HAS-A`(聚合)的关系
	- 对叶对象操作的一致性：组合模式要求组合对象和叶对象拥有相同的接口，对一组叶对象的操作必须具有一致性
	- 双向映射关系
	- 用职责链模式提高组合模式性能

- 如何使用组合模式：
	- 表示对象的部分-整体层次结构。
	- 客户希望统一对待树中的是所有对象，忽略组合对象和叶对象的区别
- 组合模式的每个对象的区别只有在运行时才会显现，会使代码难以理解
- 如果通过组合模式创建了太多的对象，那么这些对象可能会让系统负担不起


组合模式示例代码：

``` javascript
var closeDoorCommand = {
    execute: function() {
        console.log('关门');
    },
    add: funciton() {
        throw new Error('叶对象不能添加子节点');
    }
};

var openPcCommand = {
    execute: function() {
        console.log('开电脑');
    },
    add: funciton() {
        throw new Error('叶对象不能添加子节点');
    }
};

var openQQCommand = {
    execute: function() {
        console.log('登录 QQ');
    },
    add: funciton() {
        throw new Error('叶对象不能添加子节点');
    }
};

var MacroCommand = function() {
    return {
        commandsList: [],
        add: function(command) {
            this.commandsList.push(command)
        },
        execute: function() {
            for (var i = 0, command; command = this.commandsList[i++];) {
                command.execute();
            }
        }
    }
};

var macroCommand = MacroCommand();
macroCommand.add(closeDoorCommand);
macroCommand.add(openPcCommand);
macroCommand.add(openQQCommand);

macroCommand.execute();
```

## 第 11 章：模板方法模式

- 模板方法是只需要使用集成就可以实现的非常简单的模式
- 模板方法由两部分组成，第一部分是抽象父类，第二部分是具体的实现子类
- 父类中封装了子类的算法框架，包括实现一些公共方法以及封装子类中所有方法的执行顺序。子类通过继承这个抽象类，也继承了整个算法结构，并且可以选择重写父类的方法

**抽象类：**

- 模板方法模式是一种严重依赖抽象类的设计模式
- 抽象类不可以实例化
- 依赖倒置原则： 
	- A.高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。
	- B.抽象不应该依赖于具体实现，具体实现应该依赖于抽象。
- 继承抽象类的所有子类都将拥有跟抽象类一致的接口方法，抽象类的主要作用就是为它的子类定义这些公共接口

模板方法示例代码：

``` javascript
var Beverage = function() {};

Beverage.prototype.boilWater = function() {
    console.log('把水煮沸');
};

Beverage.prototype.brew = function() {}; // 空方法，应该由子类重写
Beverage.prototype.pourInCup = function() {}; // 空方法，应该由子类重写
Beverage.prototype.addCondiments = function() {}; // 空方法，应该由子类重写

Beverage.prototype.customerWantsCondiments = function() {
    return true; // 默认需要调料
};

Beverage.prototype.init = function() {
    this.boilWater();
    this.brew();
    this.pourInCup();
    if (this.customerWantsCondiments()) {
        this.addCondiments();
    }
};

var Coffee = function() {};

Coffee.prototype = new Beverage();

Coffee.prototype.brew = function() {
    console.log('用沸水冲泡咖啡');
};

Coffee.prototype.pourInCup = function() {
    console.log('把咖啡倒进杯子');
};

Coffee.prototype.addCondiments = function() {
    console.log('加糖和牛奶');
};

Coffee.prototype.customerWantsCondiments = function() {
    return window.confirm('请问需要调料吗？');
};


var coffee = new Coffee();
coffee.init();
```

**好莱坞原则：**

- 好莱坞对待新人的：`不要来找我，我会给你打电话`
- 高层组件调用底层组件
- 使用场景：
	- 发布-订阅模式
	- 回调函数

非继承代码示例：

``` javascript
var Beverage = function(params) {

    var boilWater = function() {
        console.log('把水煮沸');
    };

    var brew = params.brew || function() {
        throw new Error('必须传递 brew 方法');
    };

    var pourInCup = params.pourInCup || function() {
        throw new Error('必须传递 pourInCup 方法');
    };

    var addCondiments = params.addCondiments || function() {
        throw new Error('必须传递 addCondiments 方法');
    };

    var customerWantsCondiments = params.customerWantsCondiments || function() {
        return true;
    };

    var F = function() {};

    F.prototype.init = function() {
        boilWater();
        brew();
        pourInCup();
        if (customerWantsCondiments()) {
            addCondiments();
        }
    };

    return F;
};

var Tea = Beverage({
    brew: function() {
        console.log('用沸水浸泡茶叶');
    },
    pourInCup: function() {
        console.log('把茶倒进杯子');
    },
    addCondiments: function() {
        console.log('加柠檬');
    },
    customerWantsCondiments: function(){
        return window.confirm('请问需要加调料吗？')
    }
});

var tea = new Tea();
tea.init();
```


## 第 12 章：享元模式(flyweight)

- 享元模式是一种用于性能优化的模式, `fly` 是苍蝇的意思，`flyweight` 意为蝇量级。
- 享元模式的核心是运用共享技术来有效支持大量细粒度的对象
- 如果因为创建大量类似的对象导致内存占用过高，享元模式可以节省内存
-享元模式是一种用时间换空间的优化模式

享元模式代码示例：

``` javascript
var Model = function(sex, underwear) {
    this.sex = sex;
    this.underwear = underwear;
}

Model.prototype.takePhoto = function() {
    console.log('sex=' + this.sex + ' underwear=' + this.underwear);
};

var maleModel = new Model('male'),
    femaleModel = new Model('female');

for (var i = 1; i <= 50; i++) {
    maleModel.underwear = 'underwear' + i;
    maleModel.takePhoto();
};

for (var i = 1; i <= 50; i++) {
    femaleModel.underwear = 'underwear' + i;
    femaleModel.takePhoto();
};
```


**内部状态与外部状态：**

- 享元模式要求将对象的属性划分为内部状态和外部状态（状态在这里指属性）
- 享元模式的目的是尽量减少共享对象的数量
- 划分内部和外部状态的指引：
	- 内部状态存储于对象内部
	- 内部状态可以被一些对象共享
	- 内部状态独立于具体的场景，通常不会改变
	- 外部状态取决于具体的场景，并根据场景而变化，外部状态不能被共享


对象池代码示例：

``` javascript
var objectPoolFactory = function(createObjFn) {
    var objectPool = [];

    return {
        create: function() {
            var obj;
            // var obj = objectPool.length === 0 ?
            // createObjFn.apply(this, arguments) : objectPool.shift();
            if (objectPool.length === 0) {
                obj = createObjFn.apply(this, arguments);
            } else {
                obj = objectPool.shift();
            }

            return obj;
        },
        recover: function(obj) {
            objectPool.push(obj);
            console.log(objectPool.length);
        }
    }
};

var iframeFactory = objectPoolFactory(function() {
    var iframe = document.createElement('iframe');
    document.body.appendChild(iframe);

    iframe.onload = function() {
        iframe.onload = null; // 防止 iframe 重复加载的 bug
        iframeFactory.recover(iframe); // iframe 加载完成之后回收节点
    }
    return iframe;
});

var iframe1 = iframeFactory.create();
iframe1.src = 'http://baidu.com';

var iframe2 = iframeFactory.create();
iframe2.src = 'http://qq.com';

setTimeout(function() {
    var iframe3 = iframeFactory.create();
    iframe3.src = 'http://163.com';
}, 3000);
```

## 第 13 章：职责链模式

- 职责链模式的定义是：使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系，将这些对象连成一条链，并沿着这条链传递该请求，知道有一个对象处理它为止。 


职责链代码示例：

``` javascript

var order500 = function(orderType, pay, stock) {
    if (orderType === 1 && pay === true) {
        console.log('500 元定金预购,得到 100 优惠劵');
    } else {
        return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
    }
};

var order200 = function(orderType, pay, stock) {
    if (orderType === 2 && pay === true) {
        console.log('200 元定金预购,得到 50 优惠劵');
    } else {
        return 'nextSuccessor'; // 我不知道下一个节点是谁，反正把请求往后面传递
    }
};

var orderNormal = function(orderType, pay, stock) {
    if (stock > 0) {
        console.log('普通购买，无优惠劵');
    } else {
        console.log('手机库存不足');
    }
};


var Chain = function(fn) {
    this.fn = fn;
    this.successor = null;
};

Chain.prototype.setNextSuccessor = function(successor) {
    return this.successor = successor;
};

Chain.prototype.passRequest = function() {
    var ret = this.fn.apply(this, arguments);

    if (ret === 'nextSuccessor') {
        return this.successor && this.successor.passRequest.apply(this.successor, arguments);
    }
    return ret;
}

var chainOrder500 = new Chain( order500 );
var chainOrder200 = new Chain( order200 );
var chainOrderNormal = new Chain( orderNormal );

chainOrder500.setNextSuccessor( chainOrder200 );
chainOrder200.setNextSuccessor( chainOrderNormal );

chainOrder500.passRequest(1, true, 500); // 输出：500 元定金预购,得到 100 优惠劵
chainOrder500.passRequest(2, true, 500); // 输出：200 元定金预购,得到 50 优惠劵
chainOrder500.passRequest(3, true, 500); // 输出：普通购买，无优惠劵
chainOrder500.passRequest(1, false, 0); // 输出：手机库存不足
```

**职责链中的优缺点：**

- 最大优点就是解耦了请求发送者和 N 个接收者之间的复杂关系
- 可以手动设置起始节点，请求并不是非得从链中的第一个节点开始传递
- 职责链中大部分节点并没有起到实质性的作用，他们仅仅是让请求传递下去，从性能方面考虑，我们要避免过长的职责链带来的性能损耗


AOP 实现职责链：

``` javascript
Function.prototype.after = function( fn ){
    var self = this;
    return function(){
        var ret = self.apply( this, arguments );
        if( ret === 'nextSuccessor'){
            return fn.apply(this, arguments);
        }

        return ret;
    }
}; 
```

## 第 14 章：中介者模式

- 中介者模式的作用就是解除对象与对象之间的紧耦合关系。
- 增加一个中介者对象后，所有的相关对象都通过中介者对象来通信，而不是互相引用，所以当一个对象发生改变时，只需要通知中介者对象即可
- 中介者模式使网状的多对多关系变成了相对简单的一对多关系

![中介者模式](/assets/images/中介者模式.png)

现实的中介者：

- 机场指挥塔
- 博彩公司


小结：

- 迪密特法则叫最少知识原则，是指一个对象应该尽可能少地了解另外的对象(类似不合陌生人说话)
- 中介者模式是迎合迪米特法则的一种实现
- 中介者模式里，对象之间几乎不知道彼此的存在，只能通过中介者对象来互相影响对方
- 如果对象之间的复杂耦合度导致调用和维护出现困难，而且耦合度随项目的变化呈指数增长曲线，那么就可以考虑用中介者模式来重构代码


## 第 15 章：装饰者模式

- 给对象动态得增加职责的方式称为装饰者模式
- 装饰者模式能够在不改变对象自身的基础上，在程序运行起劲给对象动态的添加职责
- GOF 原想把装饰者(`decorator`) 模式称为包装器(`wrapper`)模式
- 装饰者模式将一个对象嵌入另一个对象之中，实际上相当于这个对象被另一个对象包装起来，形成一条包装链，请求随着包装链一次传递到所有的对象


js 版本的装饰者示例：

``` javascript
var plane = {
    fire: function() {
        console.log('发射普通子弹');
    }
}
var missileDecorator = function() {
    console.log('发射导弹');
};
var AtomDecorator = function() {
    console.log('发射原子弹');
};

var fire1 = plane.fire;
plane.fire = function() {
    fire1();
    missileDecorator();
};

var fire2 = plane.fire;
plane.fire = function() {
    fire2();
    AtomDecorator();
};

plane.fire();
```

## 第 16 章：状态模式

- 状态模式的关键是区分事务内部的状态，事物内部状态的改变往往会带来事物的行为改变
- 优点：
	- 状态模式定义了状态和行为之间的关系，并将它们封装在一个类里，通过增加新的状态类，很容易增加新的状态和转换
	- 避免 `Context` 无限膨胀，状态切换的逻辑被分布在状态类中，也去掉了 `Context` 中原本过多的条件分支
	- 用对象代理字符串来记录当前状态，使得状态的切换更加一目了然
	- `Context` 中的请求动作和状态类中封装的行为可以非常容易地独立变化而互不影响
- 缺点：
	- 定义许多状态类，增加了很多对象
	- 逻辑分散，无法在一个地方理解整个状态机的逻辑
- 状态和策略的类图几乎一模一样，但意图不同，策略的各个策略类之间没有任何联系，状态模式中状态之间有 “改变行为” 的切换规定

### 有限状态机

- http://www.ruanyifeng.com/blog/2013/09/finite-state_machine_for_javascript.html
- https://github.com/jakesgordon/javascript-state-machine

有限状态机的实践：

- 游戏开发中，游戏 API 的逻辑编写，比如游戏格斗任务有走动、攻击、防御、跌倒、跳跃等多种状态
- 比如 TCP 请求有建立连接、监听、关闭等状态

## 第 17 章：适配器模式

- 适配器的别名是包装器(`wrapper`)，这是一个相对简单的模式
- 适配器应用场景：当我们试图调用模块或者对象的某个接口时，却发现这个接口的格式不符合目前的需求，可以创建适配器，将原接口转换为客户希望的另一个接口，客户只需要和适配器打交道
- 现实中的适配器：港式插头转换器、电源适配器、USB 转换口
- 适配器模式主要用来解决两个已有接口之间不匹配的问题，它不考虑这些接口是怎样实现的，也饿不考虑它们奖励可能会如何演化。适配器模式不需要改变已有的接口，就能够使它们协同作用
- 装饰者模式和代理模式也不会改变原有对象的接口，但装饰者模式的作用是为了给对象增加功能。
	- 装饰链模式常常形成一条长的装饰链
	- 适配器模式通常只包装一次
	- 代理模式是为了控制对对象的访问，通常也只包装一次
- 外观模式的作用倒是和适配器比较相似，有人把外观模式看成一组对象的适配器，但外观模式最显著的特点是定义了一个新的接口


## 第 18 章：单一职责原则

- 单一职责原则（`SRP`）的职责被定义为“引起变化的原因”，就一个类而言，应该有且仅有一个引起它变化的原因
- `SRP` 原则体现为：一个对象（方法）只做一件事情
- `SRP` 原则是最简单也是最难正确运用的原则之一，因为并不是所有的职责都应该一一分离

## 第 19 章：最少知识原则

- 最少知识原则(`LKP`)说的是一个软件实体应当尽可能少地与其他实体发生相互作用
- 软件实体是一个广义概念，不仅包括对象，还包括系统、类、模块、函数、变量等
- `LKP` 要求设计中尽量减少队形之间的联系
- 最少知识原则在设计模式中体现得最多的地方是中介者模式和外观模式
- 外观模式的作用：
	- 为一组子系统提供一个简单遍历的访问入口
	- 隔离客户与复杂子系统之间的联系，客户不用去了解子系统的细节
- 最少知识原则又称为迪米特法则
- 最少知识原则可以减少对象之间的依赖，但也有可能增加一些庞大到难以遵守的第三者对象

## 第 19 章：开放-封闭原则

- 开放-封闭原则定义：软件实体（类、模块、函数）等应该是可以扩展的，但是不可修改
- 遵守开放-封闭原则的规律，最明显的就是找出程序中将要发生变化的地方，然后将变化封装起来
- 编写开放-封闭原则代码的方式：
	- 放置挂钩：放置挂钩(hook)也是分离变化的一种方式，我们在程序有可能发生变化的地方放置一个挂钩，挂钩的返回结果决定了程序的下一步走向
	- 使用回调函数
- 挑出最容易发生变化的地方，然后构造抽象来封闭这些变化
- 在不可避免发生修改的时候，尽量修改那些相对容易修改的地方，比如修改配置文件比修改源码简单方便


## 第 21 章：接口和面向接口编程

- 接口是对象能响应的请求的集合
- 抽象类的作用：
	- 向上转型
	- 建立一些契约
- 面向接口编程，而不是面向实现编程
- 抽象类和 interface 的作用：
	- 通过向上转型来隐藏对象的真正类型，以表现对象的多态性
	- 约定类与类支架按的一些契约行为


## 第 22 章：代码重构

- 提炼函数
	- 避免出现超大函数
	- 独立出来的函数有助于代码复用
	- 独立出来的函数更容易被覆写
	- 独立出来的函数如果拥有一个良好的命名，它本身就起到了注释的作用




