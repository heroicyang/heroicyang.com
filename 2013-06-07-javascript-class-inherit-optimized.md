title: 关于JavaScript中的继承（三）：再谈类式继承
date: 2013-06-07 22:23:30
tags: [javascript,inherit]
---
在[《关于JavaScript中的继承（一）：类式继承》](/2013/05/30/learn-javascript-inherit-class/)中已经基本上实现了类式继承，但仍然还存在一些问题，接下来对之前的实现进一步进行完善。  

{% codeblock lang:javascript %}
var Parent = function (name) {
  this.name = name || 'heroic';
};
Parent.prototype.print = function () {
  console.log('name: ', this.name);
};

var parent = new Parent();

var Child = function (name) {
  Parent.apply(this, arguments);
};
Child.prototype = parent;
Child.prototype.setChildAge = function (age) {
  this.age = age;
};

parent.setChildAge(10);
console.log(parent.age);  // 10
{% endcodeblock %}

正如代码所见，在为子对象原型添加自己独有方法的时候，父对象也受到了影响，这可不是期望的结果。  

{% codeblock lang:javascript %}
var child = new Child('child');
console.log(child.name);  // 'child'

delete child.name;
console.log(child.name);  // 'heroic'
{% endcodeblock %}

同样如代码所示，`child`对象持有了两个`name`属性，一个是通过构造函数拷贝的，另一个是原型链上的，当删除掉本身的`name`属性后，便访问到了原型链上的了。对于这个问题，解决方案很简单直接：  

{% codeblock lang:javascript %}
/* ... */
Child.prototype = Parent.prototype;  // 只继承父对象原型链上的属性
/* ... */
delete child.name;
console.log(child.name);  // undefined
{% endcodeblock %}

但是这种方法也并没有解决最开始的那个问题，即添加或删除子对象原型上的属性时，会一并反映到父对象中。这个时候就需要用到[《关于JavaScript中的继承（二）：原型式继承》](/2013/06/02/learn-javascript-inherit-proto/)中用到的临时构造函数了。  

<!-- more -->

{% codeblock lang:javascript %}
var inherit = function (subClass, superClass) {
  var F = function () {};
  F.prototype = superClass.prototype;
  subClass.prototype = new F;
};

// 然后将 `Child.prototype = Parent.prototype` 调整为
inherit(Child, Parent);

// 增加`Child`自己的方法之后
Child.prototype.setChildAge = function (age) {
  this.age = age;
};

parent.setChildAge(10);  // TypeError: Object [object Object] has no method 'setChildAge'
{% endcodeblock %}

不再影响父对象的行为了，而且还可以为`inherit`方法增加子对象访问父对象行为的特性。  

{% codeblock lang:javascript %}
var inherit = function (subClass, superClass) {
  var F = function () {};
  F.prototype = superClass.prototype;
  subClass.prototype = new F;
  subClass.prototype._super = superClass.prototype;
};

// 然后重写了子对象的`print()`方法，但依然还是想重用父对象的
Child.prototype.print = function () {
  console.log('before print...balabala...');
  this._super.print.call(this);
};

var child = new Child('child');
child.print();  // 'before print...balabala...'
                // 'name:  child'
{% endcodeblock %}

最后，如果说这个类式继承模式还有哪点不够完美的话，那就是在子对象继承父对象之后，子对象的构造函数指向被改写了。  

{% codeblock lang:javascript %}
console.log(child.constructor === Parent);  // true
{% endcodeblock %}

没办法，只有在继承的最后，把`constructor`修正回来就是。  

{% codeblock lang:javascript %}
var inherit = function () {
  var F = function () {};

  return function (subClass, superClass) {
    F.prototype = superClass.prototype;
    subClass.prototype = new F;
    subClass.prototype._super = superClass.prototype;
    subClass.prototype.constructor = subClass;
  };
}();

/* ... */
console.log(child.constructor === Child);  // true
console.log(child.constructor === Parent);  // false
{% endcodeblock %}

同时，也如[《关于JavaScript中的继承（二）：原型式继承》](/2013/06/02/learn-javascript-inherit-proto/)中提到的那样，利用闭包来减少每次调用`inherit()`都会生成一个临时构造函数的开销。  

写在最后，类式继承为我们带来了JavaScript中不存在的完整的类的概念，这对于从面向对象语言转过来的程序员来说，可能是很好的方式。但是它也有可能让我们忽略了JavaScript真正的原型式继承。不过这些模式都没有好与坏之分，应该在适合的场景使用合适的方法才是。