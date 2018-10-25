---
title: 函数 f(x)
date: 2018-10-17 16:45:23
categories: web前端
tags: [JavaScript]
comments: false
---

上一篇文章我学习了对象 这篇文章大家和我一起来学习函，其实函数也是一个对象

<!-- more -->
#### 函数的声明与表达式

函数是JavaScript中的一等对象，这意味着可以把函数想起它值一样传递。一个常见的用法是把匿名函数作为毁掉函数传递到异步函数中

##### 函数声明
``` JavaScript
function Fun() {} 
```
上面的方法会在执行前被解析，因此它存在在与当前文档的任意一个地方，即使在函数在定义体的上方被调用也是对的
``` JavaScript
Fun() // 正常运行，因为foo在代码运行前已经被创建
function Fun() {} 
```
##### 函数赋值表达式
``` JavaScript
var Fun =  function() {} //  这个是把一个匿名函数赋值给变量 Fun

// 下面由于 var 定义了一个声明语句，对变量 foo 的解析是在代码运行之前， 因此 foo 变量在代码运行时已经被定义过了  
foo; // 'undefined'
foo(); // 出错：TypeError
var foo = function() {};
// 但是由于赋值语句只在运行时执行，因此在相应代码执行之前，foo 的值缺省为 undefined。
```

##### 命名函数的赋值表达式
有一个特殊的情况，就是将命名函数赋值给一个变量
``` JavaScript
var foo = function bar(){
	bar() // 正常运行
}
bar() // 出错：ReferenceError
bar 函数声明外是不可见的，是应为我们把函数赋值给了 Foo；然而在 bar 内部依然可见。这是由于 JavaScript的命名处理所导致，函数名在函数内总是可见的。
```
注意:在IE8及IE8以下版本浏览器bar在外部也是可见的，是因为浏览器对命名函数赋值表达式进行了错误的解析， 解析成两个函数 foo 和 bar

#### this的工作原理
在大家时间工作中大家经常会弄错 this的范围，现在我们来了解一下this的工作原理

JavaScript 有一套完全不同于其他语言的对 this 的处理机制。在五中不同的情况下，this 指向的各不相同 (ES6 的尖括号函数解放了this ）

##### 全局范围内的 this
``` JavaScript
this； // 当在全部范围内使用 this，它将会指向全局对象。
```
当在全部范围内使用 this，它将会指向全局对象。
##### 函数调用
``` JavaScript
foo(); // 这里 this 也会指向全局对象。
```
##### 方法调用
``` JavaScript
test.foo(); // 这个例子中，this 指向 test 对象。
```
##### 调用构造函数
``` JavaScript
new foo();  // 如果函数倾向于和 new 关键词一块使用，则我们称这个函数是构造函数。 在函数内部，this 指向新创建的对象。
```

##### 调用构造函数
``` JavaScript
function foo(a, b, c) {}
var bar = {};
foo.apply(bar, [1, 2, 3]); // 数组将会被扩展，如下所示
foo.call(bar, 1, 2, 3); // 传递到foo的参数是：a = 1, b = 2, c = 3
```

当使用 Function.prototype 上的 call 或者 apply 方法时，函数内的 this 将会被显式设置为函数调用的第一个参数。

因此函数调用的规则在上例中已经不适用了，在foo 函数内 this 被设置成了 bar。

注意: 在对象的字面声明语法中，this 不能用来指向对象本身。 因此 var obj = {me: this} 中的 me 不会指向 obj，因为 this 只可能出现在上述的五种情况中。 译者注：这个例子中，如果是在浏览器中运行， obj.me 等于 window 对象。

##### 常见误解 
尽管大部分的情况都说的过去，不过第一个规则（译者注：这里指的应该是第二个规则，也就是直接调用函数时，this 指向全局对象） 被认为是JavaScript语言另一个错误设计的地方，因为它从来就没有实际的用途。
```JavaScript
Foo.method = function() {
    function test() {
		// this 将会被设置为全局对象（译者注：浏览器环境中也就是 window 对象）
		console.log(this)
    }
    test();
}
```
一个常见的误解是 test 中的 this 将会指向 Foo 对象，实际上不是这样子的。
为了在 test 中获取对 Foo 对象的引用，我们需要在 method 函数内部创建一个局部变量指向 Foo 对象。
```JavaScript
Foo.method = function() {
    var that = this;
    function test() {
        // 使用 that 来指向 Foo 对象
    }
    test();
}
```
that 只是我们随意起的名字，不过这个名字被广泛的用来指向外部的 this 对象。 在 闭包 一节，我们可以看到 that 可以作为参数传递。

##### 方法的赋值表达式
另一个看起来奇怪的地方是函数别名，也就是将一个方法赋值给一个变量。
```JavaScript
var test = someObject.methodTest;
test();
```
上例中，test 就像一个普通的函数被调用；因此，函数内的 this 将不再被指向到 someObject 对象。
然 this 的晚绑定特性似乎并不友好，但这确实是基于原型继承赖以生存的土壤。
```JavaScript
function Foo() {
	console.log('Foo',this)
}
Foo.prototype.method = function() {
	console.log('Foo.prototype.method',this)
};

function Bar() {
	console.log('Bar',this)
}
Bar.prototype = Foo.prototype;

new Bar().method();
```
当 method 被调用时，this 将会指向 Bar 的实例对象。

#### 闭包和引用
闭包是 JavaScript 一个非常重要的特性，这意味着当前作用域总是能够访问外部作用域中的变量。 因为函数是 JavaScript 中唯一拥有自身作用域的结构，因此闭包的创建依赖于函数。

##### 模拟私有变量
```JavaScript
function Counter(start) {
    var count = start;
    return {
        increment: function() {
            count++;
        },

        get: function() {
            return count;
        }
    }
}

var foo = Counter(4);
foo.increment();
foo.get(); // 5
```
这里，Counter 函数返回两个闭包，函数 increment 和函数 get。 这两个函数都维持着 对外部作用域 Counter 的引用，因此总可以访问此作用域内定义的变量 count。

##### 为什么不可以在外部访问私有变量

因为 JavaScript 中不可以对作用域进行引用或赋值，因此没有办法在外部访问 count 变量。 唯一的途径就是通过那两个闭包。
```JavaScript
var foo = new Counter(4);
foo.hack = function() {
	console.log('--- 上 ---',count)
	count = 1337;
	console.log('--- xiao ---',count)
};
```
上面的代码不会改变定义在 Counter 作用域中的 count 变量的值，因为 foo.hack 没有 定义在那个作用域内。它将会创建或者覆盖全局变量 count。

##### 循环中的闭包
一个常见的错误出现在循环中使用闭包，假设我们需要在每次循环中调用循环序号
```JavaScript
for(var i = 0; i < 10; i++) {
    setTimeout(function() {
        console.log('--- 1111 ---',i);  
    }, 1000);
}
```
上面的代码不会输出数字 0 到 9，而是会输出数字 10 十次。
当 console.log 被调用的时候，匿名函数保持对外部变量 i 的引用，此时 for循环已经结束， i 的值被修改成了 10.
为了得到想要的结果，需要在每次循环中创建变量 i 的拷贝。

##### 避免引用错误

为了正确的获得循环序号，最好使用 匿名包装器（译者注：其实就是我们通常说的自执行匿名函数）。
```JavaScript
for(var i = 0; i < 10; i++) {
    (function(e) {
        setTimeout(function() {
            console.log(e);  
        }, 1000);
    })(i);
}
```
外部的匿名函数会立即执行，并把 i 作为它的参数，此时函数内 e 变量就拥有了 i 的一个拷贝。

当传递给 setTimeout 的匿名函数执行时，它就拥有了对 e 的引用，而这个值是不会被循环改变的。

有另一个方法完成同样的工作，那就是从匿名包装器中返回一个函数。这和上面的代码效果一样

#### arguments 对象

JavaScript 中每个函数内都能访问一个特别变量 arguments。这个变量维护着所有传递到这个函数中的参数列表。
arguments 变量不是一个数组（Array）。 尽管在语法上它有数组相关的属性 length，但它不从 Array.prototype 继承，实际上它是一个对象（Object）。
注意: 由于 arguments 已经被定义为函数内的一个变量。 因此通过 var 关键字定义 arguments 或者将 arguments 声明为一个形式参数， 都将导致原生的 arguments 不会被创建。

因此，无法对 arguments 变量使用标准的数组方法，比如 push, pop 或者 slice。 虽然使用 for 循环遍历也是可以的，但是为了更好的使用数组方法，最好把它转化为一个真正的数组。

##### 转化为数组
下面的代码将会创建一个新的数组，包含所有 arguments 对象中的元素
```JavaScript
Array.prototype.slice.call(arguments);
// 这个转化比较慢，在性能不好的代码中不推荐这种做法
```

##### 传递参数
下面是将参数从一个函数传递到另一个函数的推荐做法。
```JavaScript
function foo() {
    bar.apply(null, arguments);
}
function bar(a, b, c) {
    // 干活
}
```
另一个技巧是同时使用 call 和 apply，创建一个快速的解绑定包装器。
```JavaScript
function Foo() {}
Foo.prototype.method = function(a, b, c) {
    console.log(this, a, b, c);
};

// 创建一个解绑定的 "method"
// 输入参数为: this, arg1, arg2...argN
Foo.method = function() {

    // 结果: Foo.prototype.method.call(this, arg1, arg2... argN)
    Function.call.apply(Foo.prototype.method, arguments);
};
```
译者注：上面的 Foo.method 函数和下面代码的效果是一样的:
```JavaScript
Foo.method = function() {
    var args = Array.prototype.slice.call(arguments);
    Foo.prototype.method.apply(args[0], args.slice(1));
};
```
##### 自动更新
arguments 对象为其内部属性以及函数形式参数创建 getter 和 setter 方法。

因此，改变形参的值会影响到 arguments 对象的值，反之亦然。
```JavaScript
function foo(a, b, c) {
    arguments[0] = 2;
    a; // 2                                                           

    b = 4;
    arguments[1]; // 4

    var d = c;
    d = 9;
    c; // 3
}
foo(1, 2, 3);
```
##### 性能真相

不管它是否有被使用，arguments 对象总会被创建，除了两个特殊情况 - 作为局部变量声明和作为形式参数。
arguments 的 getters 和 setters 方法总会被创建；因此使用 arguments 对性能不会有什么影响。 除非是需要对 arguments 对象的属性进行多次访问。

ES5 提示: 这些 getters 和 setters 在严格模式下（strict mode）不会被创建。

译者注：在 MDC 中对 strict mode 模式下 arguments 的描述有助于我们的理解，请看下面代码：
```JavaScript
// 阐述在 ES5 的严格模式下 `arguments` 的特性
function f(a) {
  "use strict";
  a = 42;
  return [a, arguments[0]];
}
var pair = f(17);
console.assert(pair[0] === 42);
console.assert(pair[1] === 17);
```
然而，的确有一种情况会显著的影响现代 JavaScript 引擎的性能。这就是使用 arguments.callee。
```JavaScript
function foo() {
    arguments.callee; // do something with this function object
    arguments.callee.caller; // and the calling function object
}

function bigLoop() {
    for(var i = 0; i < 100000; i++) {
        foo(); // Would normally be inlined...
    }
}
```
上面代码中，foo 不再是一个单纯的内联函数 inlining（译者注：这里指的是解析器可以做内联处理）， 因为它需要知道它自己和它的调用者。 这不仅抵消了内联函数带来的性能提升，而且破坏了封装，因此现在函数可能要依赖于特定的上下文。
因此强烈建议大家不要使用 arguments.callee 和它的属性。


ps: 这篇文章学到函数的 arguments 对象，下篇文章我，会写一下我学函数的 构造函数，及函数的作用域与命名空间。
