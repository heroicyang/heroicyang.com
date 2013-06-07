title: 关于JavaScript中的继承（二）：原型式继承
date: 2013-06-02 22:52:19
tags: [javascript, inherit]
---
JavaScript中本没有类，所以凡事也不要强求，强扭的瓜总是不填的嘛。继承也无非就是一个对象拥有另外一个对象的特性，所以完全不需要复杂的去模拟面向对象中的类式继承，而借助JavaScript独特的原型机制就可以实现了。同样，需要创建一个对象时也不再采用模拟**类-对象**的方式，而是直接使用JavaScript的对象字面量就好了。因为对于对象来说我们关系的无非也就是它具备哪些属性、有哪些行为而已。  

{% codeblock lang:javascript %}
var parent = {
  printName: function () {
    console.log(this.name || 'parent');
  }
};

var child = {};  // 暂时这样
child.name = 'child';  // 在很多情况下我们并不希望继承原对象自己的属性，而是在我们需要时，直接添加就好
child.printName();  // 重点的是我们希望child具备打印自己名字的行为
{% endcodeblock %}

上面的代码目前还不能工作，[上一篇文章](http://heroicyang.com/2013/05/30/learn-javascript-inherit-class/)中谈到过JavaScript中对象属性查找机制，如果`child`对象本身没有`printName()`这个方法，那我们保证其原型上有这个方法，它就能正常工作了。  

{% codeblock lang:javascript %}
/* var parent... */

var object = function (obj) {
  var F = function () {};
  F.prototype = obj;
  return new F;
};

// 改变child对象的创建方式，不直接采用字面量了，而是用`object`方法来创建
var child = object(parent);
child.name = 'child';
child.printName();  // 'child'
{% endcodeblock %}

通过一个`object()`函数，我们就完全可以实现一个对象从另一个对象继承了。其原理很简单，即在`object()`函数内部创建一个临时的构造函数，然后修改这个构造函数的原型，最后再返回这个构造函数的一个实例。

<!-- more -->

当然，`parent`对象也并不一定要使用对象字面量，你可以选择任何你能想到的创建对象的方式，如构造函数等。  

{% codeblock lang:javascript %}
function Parent () {
  this.name = 'parent';
};
Parent.prototype.printName = function () {
  console.log(this.name);
};

var parent = new Parent();
var child = object(parent);
child.printName();  // 'parent'
{% endcodeblock %}

正如前面所述，我们可能并不想继承原对象自己的属性，如上面的例子我们也并不希望它打印出`parent`。  

原型式继承的规则就是：对象从对象继承，不管父对象是如何而来。而`Parent.prototype`也是一个对象，所以  

{% codeblock lang:javascript %}
var child = object(Parent.prototype);
child.printName();  // undefined
{% endcodeblock %}

改写之后，打印`child`自己的名字时得到的是`undefined`，因为创建`child`对象后，还没有给它赋予任何的属性，所以这恰是我们想要的结果。  

而在`ECMAScript 5`中，原型式继承已经成为了语言特性，为我们增加了`Object.create()`方法，也就是说可以不用自己实现上面的`object`方法了。而且`Object.create()`方法更加强劲、适用。  

{% codeblock lang:javascript %}
var parent = {
  name: 'parent',
  printName: function () {
    console.log(this.name);
  }
};

// 第二个参数接收一个对象，而该对象的属性将作为`child`对象的属性
var child = Object.create(parent, {
  name: { value: 'child' }
});

child.printName();  // 'child'
{% endcodeblock %}

最后，再谈谈最开始实现的`object()`方法吧，其实还可以做一点点简单的优化。那就是每次调用`object()`方法时都要创建一个临时的代理构造函数`F`，而事实上仅需要创建一次就足够了。  

{% codeblock lang:javascript %}
var object = function () {
  var F = function () {};
  
  return function (obj) {
    F.prototype = obj;
    return new F;  
  };
}();
{% endcodeblock %}

我们把代理构造函数放到一个立即执行的函数中创建，然后这个函数返回一个新的、真正实现继承逻辑的函数。