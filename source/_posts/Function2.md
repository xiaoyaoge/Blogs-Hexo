---
title: 函数 f(x) —— 构造函数及作用域与命名空间
date: 2018-10-24 17:07:23
categories: web前端
tags: [JavaScript]
comments: false
---

这一篇文章继续学习JavaScript函数，函数在JavaScript中的地位是一等公民，所以掌握JavaScript函数是非常有必要的。

<!-- more -->
#### 构造函数
JavaScript 中的构造函数和其它语言中的构造函数是不同的。 通过 new 关键字方式调用的函数都被认为是构造函数。

在构造函数内部 - 也就是被调用的函数内 - this 指向新创建的对象 Object。 这个新创建的对象的 prototype 被指向到构造函数的 prototype。

如果被调用的函数没有显式的 return 表达式，则隐式的会返回 this 对象 - 也就是新创建的对象。
```JavaScript
function Foo() {
    this.bla = 1;
}

Foo.prototype.test = function() {
    console.log(this.bla);
};

var test = new Foo();
```
上面代码把 Foo 作为构造函数调用，并设置新创建对象的 prototype 为 Foo.prototype。

显式的 return 表达式将会影响返回结果，但仅限于返回的是一个对象。
```JavaScript
function Bar() {
    return 2;
}
new Bar(); // 返回新创建的对象

function Test() {
    this.value = 2;

    return {
        foo: 1
    };
}
new Test(); // 返回的对象
```
译者注：new Bar() 返回的是新创建的对象，而不是数字的字面值 2。 因此 new Bar().constructor === Bar，但是如果返回的是数字对象，结果就不同了，如下所示
```JavaScript
function Bar() {
    return new Number(2);
}
new Bar().constructor === Number
```
译者注：这里得到的 new Test()是函数返回的对象，而不是通过new关键字新创建的对象，因此：
```JavaScript
(new Test()).value === undefined
(new Test()).foo === 1
```
如果 new 被遗漏了，则函数不会返回新创建的对象。
```JavaScript
function Foo() {
    this.bla = 1; // 获取设置全局参数
}
Foo(); // undefined
```
虽然上例在有些情况下也能正常运行，但是由于 JavaScript 中 this 的工作原理， 这里的 this 指向全局对象。

##### 工厂模式
为了不使用 new 关键字，构造函数必须显式的返回一个值。
```JavaScript
function Bar() {
    var value = 1;
    return {
        method: function() {
            return value;
        }
    }
}
Bar.prototype = {
    foo: function() {}
};

new Bar();
Bar();
```
上面两种对 Bar 函数的调用返回的值完全相同，一个新创建的拥有 method 属性的对象被返回， 其实这里创建了一个闭包。

还需要注意， new Bar() 并不会改变返回对象的原型（译者注：也就是返回对象的原型不会指向 Bar.prototype）。 因为构造函数的原型会被指向到刚刚创建的新对象，而这里的 Bar 没有把这个新对象返回（译者注：而是返回了一个包含 method 属性的自定义对象）。

在上面的例子中，使用或者不使用 new 关键字没有功能性的区别。

译者注：上面两种方式创建的对象不能访问 Bar 原型链上的属性，如下所示：
```JavaScript
var bar1 = new Bar();
typeof(bar1.method); // "function"
typeof(bar1.foo); // "undefined"

var bar2 = Bar();
typeof(bar2.method); // "function"
typeof(bar2.foo); // "undefined"
```
##### 通过工厂模式创建新对象
我们常听到的一条忠告是不要使用 new 关键字来调用函数，因为如果忘记使用它就会导致错误。

为了创建新对象，我们可以创建一个工厂方法，并且在方法内构造一个新对象。
```JavaScript
function Foo() {
    var obj = {};
    obj.value = 'blub';

    var private = 2;
    obj.someMethod = function(value) {
        this.value = value;
    }

    obj.getPrivate = function() {
        return private;
    }
    return obj;
}
```
虽然上面的方式比起 new 的调用方式不容易出错，并且可以充分利用私有变量带来的便利， 但是随之而来的是一些不好的地方。
1.会占用更多的内存，因为新创建的对象不能共享原型上的方法。
2.为了实现继承，工厂方法需要从另外一个对象拷贝所有属性，或者把一个对象作为新创建对象的原型。
3.放弃原型链仅仅是因为防止遗漏 new 带来的问题，这似乎和语言本身的思想相违背。

##### 总结
虽然遗漏 new 关键字可能会导致问题，但这并不是放弃使用原型链的借口。 最终使用哪种方式取决于应用程序的需求，选择一种代码书写风格并坚持下去才是最重要的。

#### 作用域与命名空间
尽管 JavaScript 支持一对花括号创建的代码段，但是并不支持块级作用域； 而仅仅支持 函数作用域。
```JavaScript
function test() { // 一个作用域
    for(var i = 0; i < 10; i++) { // 不是一个作用域
        // count
    }
    console.log(i); // 10
}
```
译者注：如果 return 对象的左括号和 return 不在一行上就会出错。
注意: 如果不是在赋值语句中，而是在 return 表达式或者函数参数中，{...} 将会作为代码段解析， 而不是作为对象的字面语法解析。如果考虑到 自动分号插入，这可能会导致一些不易察觉的错误。
```JavaScript
// 译者注：下面输出 undefined
function add(a, b) {
    return 
        a + b;
}
console.log(add(1, 2));
```
JavaScript 中没有显式的命名空间定义，这就意味着所有对象都定义在一个全局共享的命名空间下面。

每次引用一个变量，JavaScript 会向上遍历整个作用域直到找到这个变量为止。 如果到达全局作用域但是这个变量仍未找到，则会抛出 ReferenceError 异常。

##### 隐式的全局变量
```JavaScript
// 脚本 A
foo = '42';

// 脚本 B
var foo = '42'
```
上面两段脚本效果不同。脚本 A 在全局作用域内定义了变量 foo，而脚本 B 在当前作用域内定义变量 foo。
再次强调，上面的效果完全不同，不使用 var 声明变量将会导致隐式的全局变量产生。
```JavaScript
// 全局作用域
var foo = 42;
function test() {
    // 局部作用域
    foo = 21;
}
test();
foo; // 21
```
在函数 test 内不使用 var 关键字声明 foo 变量将会覆盖外部的同名变量。 起初这看起来并不是大问题，但是当有成千上万行代码时，不使用 var 声明变量将会带来难以跟踪的 BUG。
```JavaScript
// 全局作用域
var items = [/* 数组 */];
for(var i = 0; i < 10; i++) {
    subLoop();
}

function subLoop() {
    // subLoop 函数作用域
    for(i = 0; i < 10; i++) { // 没有使用 var 声明变量
        // 干活
    }
}
```
外部循环在第一次调用 subLoop 之后就会终止，因为 subLoop 覆盖了全局变量 i。 在第二个 for 循环中使用 var 声明变量可以避免这种错误。 声明变量时绝对不要遗漏 var 关键字，除非这就是期望的影响外部作用域的行为。
##### 局部变量
JavaScript 中局部变量只可能通过两种方式声明，一个是作为函数参数，另一个是通过 var 关键字声明。
```JavaScript
// 全局变量
var foo = 1;
var bar = 2;
var i = 2;

function test(i) {
    // 函数 test 内的局部作用域
    i = 5;

    var foo = 3;
    bar = 4;
}
test(10);
```
foo 和 i 是函数 test 内的局部变量，而对 bar 的赋值将会覆盖全局作用域内的同名变量。
##### 变量声明提升（Hoisting）
JavaScript 会提升变量声明。这意味着 var 表达式和 function 声明都将会被提升到当前作用域的顶部。
```JavaScript
bar();
var bar = function() {};
var someValue = 42;

test();
function test(data) {
    if (false) {
        goo = 1;

    } else {
        var goo = 2;
    }
    for(var i = 0; i < 100; i++) {
        var e = data[i];
    }
}
```
上面代码在运行之前将会被转化。JavaScript 将会把 var 表达式和 function 声明提升到当前作用域的顶部。
```JavaScript
// var 表达式被移动到这里
var bar, someValue; // 缺省值是 'undefined'

// 函数声明也会提升
function test(data) {
    var goo, i, e; // 没有块级作用域，这些变量被移动到函数顶部
    if (false) {
        goo = 1;

    } else {
        goo = 2;
    }
    for(i = 0; i < 100; i++) {
        e = data[i];
    }
}

bar(); // 出错：TypeError，因为 bar 依然是 'undefined'
someValue = 42; // 赋值语句不会被提升规则（hoisting）影响
bar = function() {};

test();
```
没有块级作用域不仅导致 var 表达式被从循环内移到外部，而且使一些 if 表达式更难看懂。

在原来代码中，if 表达式看起来修改了全局变量 goo，实际上在提升规则被应用后，却是在修改局部变量。

如果没有提升规则（hoisting）的知识，下面的代码看起来会抛出异常 ReferenceError。
```JavaScript
// 检查 SomeImportantThing 是否已经被初始化
if (!SomeImportantThing) {
    var SomeImportantThing = {};
}
```
实际上，上面的代码正常运行，因为 var 表达式会被提升到全局作用域的顶部。
```JavaScript
var SomeImportantThing;

// 其它一些代码，可能会初始化 SomeImportantThing，也可能不会

// 检查是否已经被初始化
if (!SomeImportantThing) {
    SomeImportantThing = {};
}
```
译者注：在 Nettuts+ 网站有一篇介绍 hoisting 的文章，其中的代码很有启发性。
```JavaScript
// 译者注：来自 Nettuts+ 的一段代码，生动的阐述了 JavaScript 中变量声明提升规则
var myvar = 'my value';  

(function() {  
    alert(myvar); // undefined  
    var myvar = 'local value';  
})();  
```