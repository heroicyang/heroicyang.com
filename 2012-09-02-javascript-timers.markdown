---
layout: post
title: "理解JavaScript定时器：setTimeout和setInterval"
date: 2012-09-06 22:56
meta: true
comments: true
categories: JavaScript
tags: [javascript,javascript timer]
---
**定时器其实并不是`JavaScript`提供的，而是由浏览器（对于前端来说）提供的。**所以`setTimeout()`和`setInterval()`这两个方法均是通过浏览器的顶层对象`window`进行调用，可能平时大家在使用的过程中也会省去`window`而直接使用这两个方法。  

这两个方法所接收的参数都一样：  
{% codeblock lang:javascript %}
setTimeout(func|code, delay);
setInterval(func|code, delay);
{% endcodeblock %}  
这两个方法总是被简单的认为：在多少毫秒之后就执行里面的函数或者每间隔多少毫秒就执行里面的函数，基于这种理解的话会遇到很多匪夷所思的坑。而结合[上篇文章](http://heroicyang.com/2012/08/28/javascript-event-loop.html)中所提到的执行队列来解释的话，很多疑问都可以迎刃而解。
  
前者：在指定的毫秒数后，将定时任务处理函数（`func|code`）添加到执行队列的队尾。  

后者：按照指定的周期（以毫秒计），将定时任务处理函数（`func|code`）添加到执行队列的队尾。  
<!-- more -->
下面分别使用了`setInterval`和`setTimeout`来实现同一个功能，可运行查看效果。 
 
<iframe src="http://sample.heroicyang.com/timer.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; height:80px;"></iframe>

这是相应的源代码：<a href="http://code.heroicyang.com/timer.html" target="_blank">传送门</a>  

**接下来继续填`setInterval`的坑。**  

假设定时器的上一个回调执行完到下一个回调开始的这段时间为时间间隔，那么对于`setTimeout`来说，这个时间间隔理论上是应该`>=delay`；而对于`setInterval`来说，这个时间间隔理论上是应该`<=delay`的。

但事实总会有出人意料的地方，`setInterval`就是那个制造意外的东西。   

以下是常规的代码：   
{% codeblock lang:javascript %}
var endTime = null;
setInterval(count, 200);
function count() {
  var elapsedTime = endTime ? (new Date() - endTime) : 200;
  i++;
  console.log('current count: ' + i + '.' + 'elapsed time: ' + elapsedTime + 'ms');
  endTime = new Date();
}
{% endcodeblock %}    
其执行结果也比较符合理论时间，见下图。

![](http://img.heroicyang.com/setInterval1.png)   

接下来修改代码，让`count()`方法的执行时间变长一点：  
{% codeblock lang:javascript %}
function count() {
  var elapsedTime = endTime ? (new Date() - endTime) : 200;
  i++;
  console.log('current count: ' + i + '.' + 'elapsed time: ' + elapsedTime + 'ms');
  sleep(100); //sleep 100ms
  endTime = new Date();
}
{% endcodeblock %}
执行结果如下：

![](http://img.heroicyang.com/setInterval2.png)

结合执行队列，可以用下图对上面两种情况进行直观的解释：

![](http://img.heroicyang.com/setInterval1-explain.png)   

接下来再次修改代码，让`count()`方法的执行时间更长，设定为`setInterval`中`delay`的`2`倍，即`400ms`：  
{% codeblock lang:javascript %}
function count() {
  var elapsedTime = endTime ? (new Date() - endTime) : 200;
  i++;
  console.log('current count: ' + i + '.' + 'elapsed time: ' + elapsedTime + 'ms');
  sleep(400); //sleep 400ms
  endTime = new Date();
}
{% endcodeblock %}  
其执行效果变为如下：

![](http://img.heroicyang.com/setInterval3.png)  

意外发生了，每个回调之间的时间间隔竟然没有了，或者说缩短到非常小的间隔。事情大概是这样的：如果`setInterval`的定时时间到了，而前一个回调还没有执行完时，就会把这次的回调放在执行队列的队尾；如果`setInterval`的定时时间已经多次触发，而此时最前一个回调仍然还在执行，那么就会丢弃掉本次的回调。还是用图来直观说明吧。  

这是回调处理时间比定时时间稍微长一点点的情况：

![](http://img.heroicyang.com/setInterval2-explain.png)  

这是回调处理时间比定时时间长很多的情况：

![](http://img.heroicyang.com/setInterval3-explain.png)  

**所以，如果使用`setInterval`的话，其时间间隔总是让人捉摸不定。而使用`setTimeout`嵌套，则完全可以解决这个问题，还我们一个固定的时间间隔。**