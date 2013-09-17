title: JavaScript闭包
date: 2013-07-27 18:20:35
tags: [javascript, closure]
---
闭包是JavaScript中的重要特性之一，大多数用过JavaScript的程序员也基本上都接触过闭包，不管是否知道或了解闭包这个概念。比如在用jQuery的时候：

``` lang:javascript
var count = 0;
$('.btn').onclick = function(e) {
  count += 1;
};
```

闭包，[维基百科]的解释是：指引用了自由变量的函数。而我个人认为前端大牛[johnhax](http://weibo.com/haxy)的解释更加容易理解：闭包就是内部函数能访问外部的变量。

所以，要理解闭包，只要理清楚**变量作用域**这个概念就差不多了。我也把对**变量作用域**的一些个人理解记录在了前两篇文章中，故这里就只简单说说一个函数它可以访问哪些作用域中的变量：

1. 该函数自己内部声明的变量
2. global作用域中的全局变量
3. 如果该函数是内部函数，那它还可以访问其外部函数内声明的变量

而对于第三点，就是闭包的行为了，用一个简单的例子来说明。

<!-- more -->

{% codeblock lang:javascript %}
function foo() {
  var i = 0;
  function bar() {
    i += 1;
    console.log(i);
  }
  return bar;
}

var test = foo();
test(); // 1
test(); // 2
{% endcodeblock %}

简单解读下这段代码：

1. `bar`函数是内部函数，即定义在`foo`函数内部
2. `foo`函数调用之后会将`bar`函数的引用作为返回值
3. 全局作用域中有变量引用了`bar`函数，即`bar`函数还处于活动状态

于是在`foo`函数已经被调用结束之后，其内部的`i`变量仍然没被销毁，而且在每次调用`bar`函数之后，其值是一直在递增的。  

如果用作用域链来解释，那就更加清晰明了了：  

1.代码载入全局执行环境，开始运行。

{% codeblock lang:javascript %}
globalScopeChain = {
  test: undefined,
  foo: [Function]
};

fooScopeChain = [globalScopeChain];
{% endcodeblock %}

2.调用`foo`函数。

{% codeblock lang:javascript %}
// 首先，`foo`函数的作用域链变为：
fooScopeChain = [{
  arguments: [],
  i: 0,
  bar: [Function]
}, {
  test: undefined,
  foo: [Function]
}];

// 声明了`bar`函数，并初始化其作用域链为：
barScopeChain = [fooScopeChain];
{% endcodeblock %}

3.`foo`函数调用结束，并将`test`变量指向`foo`执行返回的`bar`函数的引用。

{% codeblock lang:javascript %}
// 此时全局作用域链变为：
globalScopeChain = [{
  test: [Function],
  foo: [Function]
}];
{% endcodeblock %}

4.第一次调用`test`，也就是`bar`函数。`bar`函数作用域链上有变量`i`，且值为`0`。

{% codeblock lang:javascript %}
// 作用域上进行变量查找，找到后做增量赋值操作，所以`i = 1`
barScopeChain = [{
  arguments: []
}, {
  i: 0,
  bar: [Function]
}, {
  test: undefined,
  foo: [Function]
}];
{% endcodeblock %}

5.第二次调用`test`，与第4步相同。只是本次调用时，其作用域链上的`i`值已经在上一次调用后变成`1`了，所以增量赋值操作之后，得到结果为`2`。  

因此，只要理解了作用域链，再来看闭包就很清晰了。不过对于闭包的解释，我仍然推荐开篇所提到的：**内部函数能访问外部的变量**，简单易懂。  

[维基百科]: http://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)