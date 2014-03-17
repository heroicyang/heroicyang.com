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

function test() {
  console.log(i); // 10
  console.log(foo); // 'bar'
}
test();
```

从运行结果不难看出，变量`i`在循环体结束之后仍然可以访问。而诸如Java、C#等语言中，循环结束之后便不能再访问到循环体中的变量了。继续看下面代码：

```
var name = 'heroic';

function foo() {
  var name = 'foo';
  console.log(name); // 'foo'
}

foo();
console.log(name); // 'heroic'
```

**结合两段代码可以知道，在JavaScript中是不存在块级作用域的，只存在函数作用域（也称“本地作用域”）和全局作用域。**而作用域中的变量我们分别称其为：局部变量和全局变量。  

前辈们（前端的长辈们，嚯嚯）常常反复在说，全局变量是魔鬼啊，魔鬼啊。。。（回声）那到底是嘛原因呢？咱接着往下说。

<!-- more -->

* JavaScript中如果不使用`var`关键字来声明变量，则该变量会成为全局变量。  
* 在整个作用域中，都可以访问并修改这些全局变量，不可控，BUG随之而来。  
* 局部变量的访问优先级高于全局变量，所以在某种特殊情况下（JavaScript变量声明提前特性），可能得不到我们想要的结果。

```
function foo() {
  name = 'foo';
}

function bar() {
  console.log(name); // 'foo'
  name = 'bar';
}

foo();
bar();
console.log(name); // 'bar'
```

上面这个例子很好理解，由于在`foo`中没有使用`var`关键字来声明`name`变量，所以可以在整个全局作用域中访问并修改该变量的值，在实际项目中必然会混乱不堪，引入不可控的BUG等等。接下来看看这个可能对于初学者来说有点迷糊的例子。

```
var name = 'heroic';

function foo() {
  console.log(name);
  var name = 'foo';
  console.log(name);
}

foo();
// undefined
// 'foo'
```

这就是前面提到的**声明提前**，JavaScript中变量和函数以及函数的参数的申明都是在一个类似于预编译的时期就做了，而在运行时期才是创建变量赋值表达式和函数表达式等。所以上面的代码等同于：

```
var name = 'heroic';

function foo() {
  var name;
  console.log(name);
  name = 'foo';
  console.log(name);
}

foo();
```