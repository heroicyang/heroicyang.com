---
layout: post
title: "JavaScript Event Loop 浅析"
date: 2012-08-28 22:52
meta: true
comments: true
tags: [javascript,javascript timer,event-loop]
---
最近在学习`Nodejs`的过程中深入的了解了`异步编程`这个概念，为了更好的使用`Nodejs`，这些概念不可不知。在以前作为一个`JavaScript`用户的时候，完全是不知道它是怎么运行的，对好些概念也是“知其然不知其所以然”。  

对于客户端的`JavaScript`和`Nodejs`来说其实差距不是很大，这回就从客户端方面来说说`Event Loop`这个概念吧，算是`异步编程`的一个切入点吧。其实`jQuery`的作者John Resig在几年前就写了一篇好文章[How JavaScript Timers Work](http://ejohn.org/blog/how-javascript-timers-work/)，来讲述`timer`和`事件`在浏览器中是怎样工作的，我也是通过这篇文章才“知其所以然”。  

###问题场景
先来看看一段代码：  
{% codeblock lang:html %}
<a href="#" id="doBtn">do something</a>
<div id="status"></div>
<script type="text/javascript">
  void function() {
    var doBtn = document.getElementById('doBtn')
      , status = document.getElementById('status');

    doBtn.onclick = function(e) {
      e.preventDefault();

      status.innerText = 'doing...please wait...';  //开始啦
      sleep(10000);  //模拟一个耗时较长的计算过程，10s
      status.innerText = 'done';  //完成啦
    };
  }();

  function sleep(ms) {
    var start = new Date();
    while (new Date() - start <= ms) {}
  }
</script>
{% endcodeblock %}

上面代码主要想完成一个功能：按钮被点击时------>显示一个状态告知用户正在干一些事情------>开始干------>事情干完后状态变更为已完成。  
<!-- more -->

看上去没问题，应该是可以工作的，于是在浏览器运行这个页面。可是现实总是残忍的，没有符合预期效果。当点击按钮之后，浏览器就冻结了，用于显示状态的`div`并没有显示，界面上也没有“doing...”这个提示；经过`10s`之后，浏览器回过神了，代表耗时较长的计算已经结束，此时用于显示状态的`div`显示“done”。  

究其原因：JavaScript引擎是单线程的。而此时还有必要再了解下浏览器内核都有哪些主要的常驻线程，才能解上面的疑惑。浏览器内核常驻线程大致包含以下：  

1. 浏览器GUI渲染线程
2. JavaScript引擎线程
3. 浏览器定时触发器线程
4. 浏览器事件触发线程
5. 浏览器http异步请求线程

而GUI渲染线程和JavaScript引擎线程是互斥的，JavaScript执行时GUI渲染线程是挂起的，页面将停止一切的解析和渲染行为。上面的3、4、5类线程也会产生不同的异步事件。看下面这张图就应该比较直观了。

![JavaScript-Event-Loop](http://img.heroicyang.com/js-event-loop.png)

因为JavaScript引擎是单线程的，所以代码都是先压到队列，然后由引擎采用先进先出的方式运行。事件处理函数、timer执行函数也会排到这个队列中，然后利用一个无穷回圈，不断从队头取出函数执行，这个就是`Event Loop`。  

接下来还是继续用图来说明上面的代码为什么没有达到预期效果。

![](http://img.heroicyang.com/js-event-loop-1.png)

于是结果就只看到了"done"。  

###怎样解决？
使用`setTimeout()`，下面是修改后的`onclick`事件处理函数：

{% codeblock lang:javascript %}
doBtn.onclick = function(e) {
  e.preventDefault();

  status.innerText = 'doing...please wait...';  //开始啦
  
  setTimeout(function() {
    sleep(10000);  //模拟一个耗时较长的计算过程，10s
    status.innerText = 'done';  //完成啦
  }, 0);  // 0ms delay
};
{% endcodeblock %}

为什么这样就解决了呢？还是用上面的队列的图来解释。

![](http://img.heroicyang.com/js-event-loop-2.png)