title: 关于JavaScript中的继承（一）：类式继承
date: 2013-05-30 22:19:21
tags: [javascript,inherit]
---
“继承”是面向对象中的一个概念，说开去无非就是代码重用的一种方式罢了。  

虽然JavaScript并不是一门真正的面向对象语言，甚至连类的概念都没有。但得益于构造器的存在，在JavaScript中是可以完全模拟出 **类-对象** 行为的。如：  

{% codeblock lang:javascript %}
var person = new Person();
{% endcodeblock %}

看上去除了变量声明时不是强类型之外，完全与面向对象如出一辙。所以谈及继承时，大家首推的也是一种叫“类式继承”的手法了。  
### 类式继承之基于原型链

{% codeblock lang:javascript %}
var Parent = function (name) {
  this.name = name || 'heroic';
};
Parent.prototype.printName = function () {
  console.log(this.name);
};
var Child = function (name) {};

inherit(Child, Parent);
// or
Child.inherit(Parent);
{% endcodeblock %}

<!-- more -->

上面的伪代码是类式继承的理想状态，但`inherit`方法并不存在，需要由自己实现。  

{% codeblock lang:javascript %}
var inherit = function (subClass, superClass) {
  subClass.prototype = new superClass();
};
// 为了实现 Child.inherit(Parent) 这样的效果
Function.prototype.inherit = function (superClass) {
  this.prototype = new superClass();
};
{% endcodeblock %}

和计划中的一样，子对象不仅继承了父对象的属性，也继承了父对象原型上的方法。  

{% codeblock lang:javascript %}
var child = new Child();
child.name = 'test';
child.printName();  // 'test'
{% endcodeblock %}

但是这种方式却存在一些问题：

1. 对象属性的写操作是直接发生的，即对象如果不存在这个属性，则为该对象创建这个属性，并为其赋值；如果存在，则直接为其赋新值。而对于对象属性的读操作，则完全不一样了：首先查找对象本身是否有该属性，有则返回，没有则查找其原型链，直到找到该属性为止，如果到了原型链的最顶层(Object)都没找到，则返回`undefined`。由于存在这种读写的不对等性，我们都不会采取从父对象继承属性，而是直接为子对象添加属性即可，而需要继承的方法则放到原型上。
2. 且上面的实现中，我们无法完成这样的初始化：`var child = new Child('test')`  

利用原型链实现的类式继承先放一边，为了解决在初始化就能传入参数的问题，便产生了一种叫“借用构造函数”方式的继承。  
### 类式继承之借用构造函数

{% codeblock lang:javascript %}
var Parent = function (name) {
  this.name = name || 'heroic';
  this.printName = function () {
    console.log(this.name);
  };
}

var Child = function (name) {
  Parent.call(this, name);
  // 或者在多参数的情况下
  // Parent.apply(this, arguments);
}

var child = new Child('test');
child.printName();  // 'test'

var parent = new Parent();
parent.printName();  // 'heroic'
{% endcodeblock %}

首先来谈谈这种机制相对于第一种的优点，Talk is cheap, Show me the code.  

{% codeblock lang:javascript %}
var Parent = function () {
  this.tags = ['.NETer'];
}
var parent = new Parent();

var ChildA = function () {};
ChildA.prototype = parent;

var ChildB = function () {
  Parent.apply(this, arguments);
};

var child_a = new ChildA();
child_a.tags.push('Javaer');

var child_b = new ChildB();
child_b.tags.push('Pythoner');

console.log(child_a.tags.join(', '));  //.NETer, Javaer
console.log(child_b.tags.join(', '));  //.NETer, Pythoner

console.log(parent.tags.join(', '));  //.NETer, Javaer
// WTF...Why is my tags contains `Javaer` ?
{% endcodeblock %}

显而易见，借用构造函数方式在继承时是采取一份单独的拷贝，而原型链方式则是指向同一个引用。（但是由此可见，原型链上的属性或方法不会在每个实例中都创建一次。）  

接下来则是谈谈缺陷了。

{% codeblock lang:javascript %}
var Parent = function () {};
Parent.prototype.papapa = function () {
  console.log('pa pa pa...');
};

var Child = function () {
  Parent.apply(this, arguments);
};

var child = new Child();
child.papapa();  // TypeError: Object [object Object] has no method 'papapa'
// Yeah, you're too young, so...
{% endcodeblock %}

借用构造函数其实是在构造时，通过改写方法调用上下文来实现属性的拷贝，所以并未涉及到`prototype`，所以就没有办法继承原型了。  

由于原型链上的属性或方法不会在每个实例中都创建一次，所以是我们放置需要重用的属性和方法的理想地方；而借用构造函数则可以使子对象拥有自己一份独立的拷贝，不存在意外改写父对象属性的风险。所以两者互补产生了第三种比较完美的继承方式。  
### 类式继承之组合模式

{% codeblock lang:javascript %}
var Parent = function (name) {
  this.name = name || 'heroic';
  this.tags = ['coder'];
};
Parent.prototype.print = function () {
  console.log('name: ', this.name, ', tags: ', this.tags.join(', '));
};

var parent = new Parent();

var Child = function (name) {
  Parent.apply(this, arguments);
};
Child.prototype = parent;

var child = new Child('test');
child.tags.push('player');

child.print();  // name:  test , tags:  coder, player
parent.print();  // name:  heroic , tags:  coder
{% endcodeblock %}

近乎完美的实现，子对象继承了父对象的成员，但拥有自己的一份拷贝，不会担心修改自己而影响到父对象；子对象也复用了父对象原型中的方法；且子对象也可以传递任意参数给父对象的构造函数。可谓是面向对象中“类式继承”的准确诠释。