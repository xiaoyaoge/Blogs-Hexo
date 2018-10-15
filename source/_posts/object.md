---
title: JavaScript 对象
date: 2018-10-15 13:25:24
categories: web前端
tags: [JavaScript]
comments: false
---
文章以一句 ‘回首往事，不因虚度年华而悔恨，也不因碌碌无为而羞耻’ 开头送给自己也送给各位...
### 对象的使用和属性
javascript中所有的变量都可以当成对象来使用，但是有两个除外，null 和 undefined。
<!-- more -->
``` javascript
false.toString(); // 'false'
[1, 2, 3].toString(); // '1,2,3'
'123'.length; // 3

function Foo(){}
Foo.bar = 1;
Foo.bar; // 1
```
大家常常误解数字的字面值（literal）不能当做是对象使用。这是因为 JavaScript 解析器的一个错误， 它试图将点操作符解析为浮点数字面值的一部分。

```javascript
2.toString(); // 出错：SyntaxError
```
其实有很多的方法可以让数字的字面量看起来像对象
```javascript
2..toString(); // "2" 第二个点号可以解析
2 .toString(); // "2"  点前面有个空格
(2).toString(); // "2" 括号中的的数字会被先计算
(2+1).toString(); // "3" 括号中的的数字会被先计算
```
##### 对象作为数据类型
JavaScript的对象可以作为[哈希表](https://en.wikipedia.org/wiki/Hash_table)使用,只要是用来保存命名的键与值的对应关系。

使用对象的字面语法 - {} - 可以创建一个简单的对象。这个新建的对象从Object.prototype 继承下面，没有任何自定义属性。
```javascript
var Dog = {} // 空对象

var Cat = { name:'小黑'} // 一个新对象 拥有一个值为小黑的自定义属性 'name'
```
##### 访问属性
javascrippt 对象有两种方式来访问属性，点操作符和中括号操作符。
```JavaScript
var Dog = { name: '小黑'}
Dog.name // 小黑
Dog['name'] // 小黑

var getName = 'name'
Dog[getName] // 小黑
Dog.123 //  会 SyntaxError
Dog['1234'] // 可以正常执行
```
这两种语法是等价的，但是中括号操作符在下面两种情况下依然有效 
1.动态设置属性
2.属性名不是一个有效的变量名。（注意点：比如属性名中包含空格，或者属性名是 JS 的关键词 在 JSLint 语法检测工具中，点操作符是推荐做法）

##### 删除属性
删除属性唯一的方法是使用 delete 操作符，将对象属性设置为undefined或者null都不能真正删除属性，而仅仅是移除了属性和值的关联。
```javascript
var obj = {
	bor: 1,
	foo: 2,
	baz: 3
};
obj.bor = undefined;
obj.foo = null;
delete obj.baz

for(var i in obj ){
	if(obj.hasOwnProperty(i)){
		console.log(i, ' --> ' + obj[i])
	}
}
// 输出的结果
bor --> undefined
foo --> null
```
从上面输出的结果看，只有 baz 被真正的删除，所以就没有出现在输出结果中

##### 属性名的语法
```JavaScript
var test = {
	'case': 'I am a keyword so I must be notated as a string'
	delete: 'I am a keyword too so me' // 出错：SyntaxError
}
```
对象的属性名可以使用字符串或普通字符声明。但是由于JavaScript解释器的另一个错误设计，上面的的第二种声明方式在ECMAScript5 之前会抛出 SyntaxError 的错误
这个错误的原因是 delete 是 JavaScript 语言的一个关键词；因此为了在更低版本的 JavaScript 引擎下也能正常运行， 必须使用字符串字面值声明方式。

#### 原型
JavaScript不包含传统的类继承模型，而是使用porttype原型模型。
虽然这经常被当做是JavaScript的缺点被提及，其实基于原型的继承模型比传统的类继承还要强大，实现传统的类继承模很简单，但是实现JavaScript中的原型继承则要困难的多 (It is for example fairly trivial to build a classic model on top of it, while the other way around is a far more difficult task.)
由于JavaScript是唯一一个被广泛使用的基于原型继承的语言，所以理解两种继承模式的差异是需要一定时间的。

传统类继承和JavaScript原型继承的差异

第一个不同这出在于 JavaScript使用的是原型链的继承方式。
```javascript
function Foo() {
    this.value = 42;
}
Foo.prototype = {
    method: function() {}
};

function Bar() {}

// 设置Bar的prototype属性为Foo的实例对象
Bar.prototype = new Foo();
Bar.prototype.foo = 'Hello World';

// 修正Bar.prototype.constructor为Bar本身
Bar.prototype.constructor = Bar;

var test = new Bar() // 创建Bar的一个新实例

// 原型链
test [Bar的实例]
    Bar.prototype [Foo的实例] 
        { foo: 'Hello World' }
        Foo.prototype
            {method: ...};
            Object.prototype
                {toString: ... /* etc. */};
```
上面的例子中，test 对象从 Bar.prototype 和Foo。prototype 继承下来；因此，它能访问 Foo 的原型方法 method。同时，他也能够访问那个定义在原型上的 Foo 实例属性 value。 需要注意的是 new Bar() 不会创造出一个新的 Foo 实例，而是重复使用它原型上的那个实例； 因此，所有的 Bar 实例都会共享相同的 value 属性。 
注意: 简单的使用 Bar.prototype = Foo.prototype 将会导致两个对象共享相同的原型。 因此，改变任意一个对象的原型都会影响到另一个对象的原型，在大多数情况下这不是希望的结果。

##### 属性查找
当查找一个对象的属性是，JavaScript 会向上遍历原型链，知道找到给定名称的属性为止。

到查找到达原型链的顶部 - 也就是 Object.prototype - 但是仍然没有找到指定属性，就会返回 undefined

##### 原型属性
当原型属性用来创建原型链时，可以把任何类型的值赋给它（prototype）。 然而将原子类型赋给 prototype 的操作将会被忽略。
```JavaScript
function Foo() {}
Foo.prototype = 1; // 无效
```
而将对象赋值给 prototype，正如上面的例子所示，将会动态的创建原型链。

##### 性能
如果一个属性在原型链的上端，则对于查找时间将带来不利影响。特别的，试图获取一个不存在的属性将会遍历整个原型链。

并且，当使用 for in 循环遍历对象的属性时，原型链上的所有属性都将被访问。

##### 扩展内置类型的原型
一个错误特性被经常使用，那就是扩展 Object.prototype 或者其他内置类型的原型对象。

这种技术被称之为 monkey patching 并且会破坏封装。虽然它被广泛的应用到一些 JavaScript 类库中比如 Prototype, 但是我仍然不认为为内置类型添加一些非标准的函数是个好主意。

扩展内置类型的唯一理由是为了和新的 JavaScript 保持一致，比如 Array.forEach。

注意：这是编程领域常用的一种方式，称之为 Backport，也就是将新的补丁添加到老版本中。

##### 总结
在写复杂的 JavaScript 应用之前，充分理解原型链继承的工作方式是每个 JavaScript 程序员必修的功课。 要提防原型链过长带来的性能问题，并知道如何通过缩短原型链来提高性能。 更进一步，绝对不要扩展内置类型的原型，除非是为了和新的 JavaScript 引擎兼容。

#### hasOwnProperty 函数
为了判断一个对象是否包含自定义属性而不是原型链上的属性， 我们需要使用继承自 Object.prototype 的 hasOwnProperty 方法。

注意: 通过判断一个属性是否 undefined 是不够的。 因为一个属性可能确实存在，只不过它的值被设置为 undefined。

hasOwnProperty 是 JavaScript 中唯一一个处理属性但是不查找原型链的函数。
```JavaScript
// 修改Object.prototype
Object.prototype.bar = 1; 
var foo = {goo: undefined};

foo.bar; // 1
'bar' in foo; // true

foo.hasOwnProperty('bar'); // false
foo.hasOwnProperty('goo'); // true
```
只有 hasOwnProperty 可以给出正确和期望的结果，这在遍历对象的属性时会很有用。 没有其它方法可以用来排除原型链上的属性，而不是定义在对象自身上的属性。
##### hasOwnProperty 作为属性
JavaScript 不会保护 hasOwnProperty 被非法占用，因此如果一个对象碰巧存在这个属性， 就需要使用外部的 hasOwnProperty 函数来获取正确的结果。
```JavaScript
var foo = {
    hasOwnProperty: function() {
        return false;
    },
    bar: 'Here be dragons'
};

foo.hasOwnProperty('bar'); // 总是返回 false

// 使用其它对象的 hasOwnProperty，并将其上下文设置为foo
({}).hasOwnProperty.call(foo, 'bar'); // true
```

##### 结论
当检查对象上某个属性是否存在时，hasOwnProperty 是唯一可用的方法。 同时在使用 for in loop 遍历对象时，推荐总是使用 hasOwnProperty 方法， 这将会避免原型对象扩展带来的干扰。

#### for in 循环
和 in 操作符一样，for in 循环同样在查找对象属性时遍历原型链上的所有属性。
```javascript
// 修改 Object.prototype
Object.prototype.bar = 1;

var foo = {moo: 2};
for(var i in foo) {
    console.log(i); // 输出两个属性：bar 和 moo
}
```
由于不可能改变 for in 自身的行为，因此有必要过滤出那些不希望出现在循环体中的属性， 这可以通过 Object.prototype 原型上的 hasOwnProperty 函数来完成。
##### 使用 hasOwnProperty 过滤
```JavaScript
// foo 变量是上例中的
for(var i in foo) {
    if (foo.hasOwnProperty(i)) {
        console.log(i);
    }
}
```
这个版本的代码是唯一正确的写法。由于我们使用了 hasOwnProperty，所以这次只输出 moo。 如果不使用 hasOwnProperty，则这段代码在原生对象原型（比如 Object.prototype）被扩展时可能会出错。

一个广泛使用的类库 Prototype 就扩展了原生的 JavaScript 对象。 因此，当这个类库被包含在页面中时，不使用 hasOwnProperty 过滤的 for in 循环难免会出问题。

##### 总结
推荐总是使用 hasOwnProperty。不要对代码运行的环境做任何假设，不要假设原生对象是否已经被扩展了。