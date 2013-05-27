---
layout: post
title: "JavaScript中创建类的方式"
date: 2012-08-19 01:05
meta: true
comments: true
tags: [javascript,javascript class]
---
现在`JavaScript`这门技术已经到了一个引爆点，一年前我对它的了解都只停留在肤浅的网页客户端脚本语言，只会简单的玩玩`jQuery`和`ExtJs`，其实都算不上开发者，而是一个`JavaScript`用户。但今年的目标是做一个合格的前端攻城湿，所以恶补是必须的。  

在`JavaScript`中是其实不存在所谓“类”的概念，因为它并不是面向对象的语言。在面向对象中，一个最常见的说法就是：“类”是“对象”的模板，基本上都是采用语言内置的`Class`或`class`关键字来定义“类”。而`JavaScript`不存在这个概念，所以也没有提供类似的关键字（虽然`class`是`JavaScript`的关键字，但是至今都没有实现，只是被保留而已）。  

因此，在`JavaScript`中创建类就唯有使用模拟的方式，而模拟的手法多种多样，何时采用何种方式最合适，需视情况而定。以下就记录下常见的几种模式。
<!-- more -->
###一.工厂模式
工厂方法是设计模式中非常基础的，也被广泛用于面向对象编程中。而在`JavaScript`中，通过工厂方法即能模拟出类的行为。  
{% codeblock lang:javascript %}
function createPerson(name, sex, …) {
  var obj = {};
  obj.name = name;
  obj.sex = sex;
  …
  obj.getName = function() {
    return this.name;
  };
  return obj;
}
var personA = createPerson('heroicYang', 'male');
var personB = createPerosn('路人甲', 'male');
{% endcodeblock %}  
通过这样类似的工厂方法，就可以创建出多个相似的对象了，但是这样的方式其抽象度极低。面向对象编程中，对象是可以检测出类型的，但是采用上面这种方式，是没有办法进行对象类型识别的。
###二.构造函数模式
其实，这应该是很常见的模式了，很多书上基本上一来就是讲这个的，更狠点的可能就只讲这个…
{% codeblock lang:javascript %}
function Person(name, sex) {
  this.name = name;
  this.sex = sex;
  …
  this.getName = function() {
    return this.name;
  }
}
var personA = new Person('heroic', 'male');
var personB = new Person('路人甲', 'male');
{% endcodeblock %}  
这种模拟类方式的特点就是:
 
1. 没有显示的创建对象   
2. 直接将属性和方法赋给了`this`对象    
3. 没有`return`字句

在使用这种方式时，创建对象则必须使用`new`关键字。当然，好处就是完全解决了对象类型识别问题。
###三.原型模式
原型应该是`JavaScript`中一个很有意思，当然也是很有用的一个概念了。接下来用原型模式来模拟类。
{% codeblock lang:javascript %}  
function Person() {}
Person.prototype = {
  name: null,
  sex: null,
  getName: function() {
    return this.name;
  }
};
var personA = new Person;
personA.name = 'heroic';
personA.sex = 'male';
var personB = new Person;
personB.name = '路人甲';
personB.sex = 'male';
{% endcodeblock %}
###四.组合使用构造函数和原型模式
由于只用原型模式的话，会带来一些问题，所以常规情况下，都是采用组合构造函数和原型模式来创建类，这也是使用率最高的一种方式。
{% codeblock lang:javascript %}
function Person(name, sex) {
  this.name = name;
  this.sex = sex;
  …
}
Person.prototype.getName = function() {
  return this.name;
};
…
var personA = new Person('heroic', 'male');
var personB = new Person('路人甲', 'male');
personA.getName(); //"heroic"
personB.getName(); //"路人甲"
{% endcodeblock %}
###五.寄生构造函数模式
这种模式和工厂模式非常相似。
{% codeblock lang:javascript %}
function SpecialArray() {
  var values = new Array();
  values.push.apply(values, arguments);
    
  values.toPipString = function() {
    return this.join('|');
  }
  return values;
}
var test = new SpecialArray('1', '2', '3');
test.toPipString(); // "1|2|3"
{% endcodeblock %}  
这种模式主要用来扩展一些对象的行为，而又不会对这个对象造成污染。当然，上面的代码也是可以直接为`Array.prototype`原型对象添加一个`toPipString()`方法来完成的，但是这样就造成了对`JavaScript`原生对象的污染。