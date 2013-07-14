title: JavaScript变量作用域
date: 2013-07-14 22:11:36
tags: [javascript]
---
所谓作用域，也就是变量和函数起作用的区域，不同的语言有着不同的实现。而在JavaScript中，这也是往往让人迷糊的地方，也是JavaScript中必须理解的特性之一。  

首先来看看下面的代码：

```
for (var i = 0; i < 10; i++) {
    var foo = 'bar';
}
console.log(i); // 10
console.log(foo); // 'bar'
```

从运行结果不难看出，变量`i`在循环体结束之后仍然可以访问。而诸如Java、C#等语言中，循环结束之后便不能再访问到循环体中的变量了。**所以，在JavaScript中是不存在块级作用域的。**了解这一点非常重要，也可以避免很多BUG的产生。  

**JavaScript中只存在函数作用域和全局作用域。**

```
var name = 'heroic';

function foo() {
  var name = 'foo';
  console.log(name); // 'foo'
}

foo();
console.log(name); // 'heroic'
```