title: 关于《JavaScript Web Application》中的bind方法
date: 2013-06-24 21:04:39
tags: [javascript]
---
去年底就草草了了的翻过{% link 《JavaScript Web Application》 http://book.douban.com/subject/10733304/ true %}这本书，当时就遇到好几个地方没怎么看明白，当时也没有去深究，就只是简单的过一遍这本书。由于此前JavaScript方面的知识匮乏，所以把这本书就缓在阅读队列里了，最近才轮到。其间读了好几本基础和进阶的书，所以这次读这本书，很多之前不明白的地方就豁然明了了。不过好记性不如烂笔头，没准下次忘记了呢。  

记得此前书中对ES5 `bind()`方法的实现就是一个让我感到迷糊的地方（中文版P16）：

{% codeblock lang:javascript %}
if (!Function.prototype.bind) {
  Function.prototype.bind = function (context) {
    var slice = [].slice
      , args = slice.call(arguments, 1)
      , self = this
      , nop = function () {}  // ① `nop`（函数）的作用？
      , bound = function () {
          // ② 为什么要做`instanceof`判断？
          return self.apply(this instanceof nop ? this : (context || {}),
                              args.concat(slice.call(arguments)));
      };

    // ③ 为什么要设置它俩的原型？
    nop.prototype = self.prototype;
    bound.prototype = new nop();

    return bound;
  };
}
{% endcodeblock %}

> `bind()`：用来动态改变函数调用时其内部`this`对象的引用，使目标函数基于正确的上下文进行调用。jQuery中实现为`$.proxy()`方法，ES5则自带了。

上面代码中① ② ③都是我此前不明白的地方，不过在这次阅读的过程中，就豁然开朗了。

<!-- more -->

让我来实现这个方法的话，或许应该是这样子的：

{% codeblock lang:javascript %}
// 取名为`bind1`好了，方便后面对比
Function.prototype.bind1 = function (context) {
  var slice = [].slice
    , args = slice.call(arguments, 1)
    , self = this;

  return function () {
    return self.apply(context || {}, args.concat(slice.call(arguments)));
  };
};
{% endcodeblock %}

也就是不会引入`nop`这个空函数，不会做`intanceof`判断，不会有原型设置。当然，这个实现是没有啥大问题的，不就是改变函数调用时的上下文环境么，貌似妥妥的呢。  

但是现在仔细想想，这个实现是没错，但不完善。因为JavaScript中一个函数除了可以做普通的函数调用以外，还充当了构造函数的作用，所以就可能会有下面这样的代码。  

{% codeblock lang:javascript %}
var obj = {
  name: 'test'
};
function func () {
  console.log([].slice.call(arguments), this);
}
var fun = func.bind(obj, 1, 2, 3)
  , fun1 = func.bind1(obj, 1, 2, 3);

fun(4);  // [ 1, 2, 3, 4 ] { name: 'test' }
var newFun = new fun(4);  // [ 1, 2, 3, 4 ] {}

fun1(4);  // [ 1, 2, 3, 4 ] { name: 'test' }
var newFun1 = new fun1(4);  // [ 1, 2, 3, 4 ] { name: 'test' }

console.log(newFun instanceof fun);  // true
console.log(newFun instanceof func);  // true

console.log(newFun1 instanceof fun1);  // true
console.log(newFun1 instanceof func);  // false
{% endcodeblock %}

如此可见，在实现`bind()`方法的时候，除了考虑一般的函数调用以外，还得考虑其结合`new`关键字使用时的构造函数的特性。因此以我在第一次看这本书时的认知，看不明白也就不奇怪了，嚯嚯。。  

那就来继续解最开始的那三个圈住的迷惑：

- ① `var nop = function () {}` ：这并是一个普通的函数，而是一个构造函数，把它用作返回的`bound`函数的原型，以便于代码中做`instanceof`判断。
- ② 鉴于前面的解释，这里做`instanceof`判断就无可厚非了。
- ③ 先设置临时构造函数的原型为调用`bind()`方法时的上下文对象的原型，即目标函数的原型。然后将临时构造函数的一个实例设置为`bound`函数的原型，原型就链起来了。这样，在对bind后的函数进行`new`关键字操作的时候，就可以获得原目标函数的特性了。

> 关于继承和原型，可以参见我的前几篇[文章](http://heroicyang.com/2013/06/07/javascript-class-inherit-optimized/)。

最后，再补一个例子吧：

{% codeblock lang:javascript %}
var Person = function (name) {
  this.name = name;
};
Person.prototype.printName = function () {
  console.log(this.name);
};
var obj = { name: 'test' };

var PersonA = Person.bind(obj, 'heroic')
  , PersonB = Person.bind1(obj, 'heroic');

var p1 = new PersonA()
  , p2 = new PersonB();

p1.printName();  // 'heroic'
p2.printName();  // TypeError: Object [object Object] has no method 'printName'
{% endcodeblock %}