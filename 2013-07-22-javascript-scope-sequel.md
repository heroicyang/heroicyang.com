title: JavaScript变量作用域（续）
date: 2013-07-22 22:14:06
tags: [javascript, scope]
---
接[上篇](http://heroicyang.com/2013/07/14/javascript-variable-scope/)，已经大致明确了以下几点：

1. JavaScript没有块级作用域，只有函数(局部)作用域和全局作用域
2. 函数中未使用`var`关键字声明的变量会成为全局变量
3. 同名时局部变量访问优先级高于全局变量 
4. JavaScript具有变量声明提前的特性

接下来根据上篇留下的最后一段代码，继续谈谈变量作用域。

{% codeblock lang:javascript %}
var name = 'global';

function foo() {
  var name = 'foo';
  bar();
}

function bar() {
  console.log(name);
}

foo();
{% endcodeblock %}

这段代码最终会在控制台打印出`'global'`，而并非`'foo'`。可以看出，函数运行时能访问到的作用域是它被定义时的作用域，不是被调用时的作用域。

每当谈及JavaScript作用域的时候，基本上都会提到“词法作用域”、“执行环境”、“活动对象”、“作用域链”这几个概念，而了解这些概念将有助于理解JavaScript中的闭包。我也谈谈我对此的理解，如误欢迎指正，不胜感激。

<!-- more -->

###词法作用域

词法作用域，也称静态作用域，也即是说函数的作用域是定义时决定的，而非运行时。最开始的代码就阐明了这一点，所以在源码时期就可以通过分析得出一个函数的作用域。JavaScript正是基于词法作用域的语言。

###执行环境

JavaScript需要一个环境来运行，比如客户端的浏览器和服务端的Node.js。而执行环境有分为**全局执行环境**和**局部执行环境**。

####全局执行环境

在浏览器中，全局执行环境为`window`对象。因此当JavaScript代码运行时，所有的全局变量和函数都作为了`window`对象的属性和方法创建。全局执行环境关联了一个**作用域链**，包含了全局执行环境中的变量对象。

####局部执行环境、活动对象

每个函数在定义时，都会关联一个初始的**作用域链**，将当前执行环境作用域链上的变量对象附加到这个关联的**作用域链**上。每个函数都是有自己的执行环境的，也就是局部执行环境。当一个函数被调用，进入局部执行环境。随之创建了**活动对象**，包含`arguments`和局部变量，然后将其附加到作用域链的前端（即下标为0）。在函数执行之后，退出局部执行环境，把控制权交给之前的执行环境（可能是全局执行环境，也有可能是另一个局部执行环境）。

###作用域链

前面已经反复提到了作用域链，也基本解释了作用域链，它差不多等同于一个对象列表或链表。为了更好的理解作用域链，还是结合最开始的那段代码来进行解释，不然有点不是很好理清楚。  


{% codeblock lang:javascript %}
var name = 'global';

function foo() {
  var name = 'foo';
  bar();
}

function bar() {
  console.log(name);
}

foo();
{% endcodeblock %}

*1.代码运行开始，在全局执行环境中，全局环境关联了一个作用域链，变量对象包含`name`、`foo`、`bar`。然后`foo`函数和`bar`函数也各自关联了自己作用域链。*

{% codeblock lang:javascript %}
globalScopeChain = {
    name: 'global',
    foo: [Function],
    bar: [Function]
};

fooScopeChain = [globalScopeChain]
barScopeChain = [globalScopeChain]
{% endcodeblock %}

*2.调用`foo`函数，进入`foo`函数的局部执行环境，创建活动对象包含`arguments`、`name`，并将活动对象添加到作用域链的前端。*

{% codeblock lang:javascript %}
fooScopeChain = [
  {
    arguments: [],
    name: 'foo'
  },
  {
    name: 'global',
    foo: [Function],
    bar: [Function]
  }
]
{% endcodeblock %}
    
*3.调用`bar`函数，进入`bar`函数的局部执行环境，创建活动对象包含`arguments`，然后也将活动对象添加到作用域链的前端。*

{% codeblock lang:javascript %}
barScopeChain = [
  {
    arguments: []
  },
  {
    name: 'global',
    foo: [Function],
    bar: [Function]
  }
]
{% endcodeblock %}
    
而变量的查找，就是在整个作用域链上进行，从第一个对象开始，直到作用域链的顶端（即全局）。这也就是实例代码中打印出`'global'`的原因。