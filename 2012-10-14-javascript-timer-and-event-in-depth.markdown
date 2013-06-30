---
layout: post
title: "深入理解JavaScript定时器（续）"
date: 2012-10-14 22:50
meta: true
comments: true
tags: [javascript,javascript timer]
---
对于浏览器端来说，大多数事件都是异步的，但是有部分事件却不是，这部分事件称做**同步事件**，因此它们都是立即执行的，完全不理会前几篇文章中所提到的**事件队列**。以及浏览器的渲染、重绘等操作，也会打乱之前我们好不容易所建立起来的**事件队列**的概念。不过，本篇将会陆续不断的把这些坑给填上。  

##同步事件

###DOM改变事件(DOM Mutation events)

下面的Demo便用于说明同步事件之一的`DOM Mutation events`（注：该事件不支持Chrome浏览器）。

{% codeblock lang:html %}
<a href="http://heroicyang.com/">
  heroicyang.com
</a>
<script type="text/javascript">
  var anchor = document.getElementsByTagName('a')[0];
  anchor.onclick = function(e) {
    alert('in onclick');
    this.setAttribute('href', '#');
    alert('out onclick');
    return false;
  };
  if (anchor.addEventListener) {  //Firefox, Opera
    anchor
      .addEventListener('DOMAttrModified', onpropchange, false);
  } else if (anchor.attachEvent) {  //IE
    anchor
      .attachEvent('onpropertychange', onpropchange);
  }
  
  function onpropchange() {
    alert('onpropchange');
  }
</script>
{% endcodeblock %}

<!-- more -->
当`click`事件触发时，其处理的顺序依次为：

1. alert `in onclick`  
2. 超链接的属性立即被改变，并alert `onpropchange`  
3. 继续执行`onclick`事件处理程序中剩下的 `alert('out onclick');`  

![](http://img.heroicyang.com/synchronous-mutation-events.png)  
关于`DOM Mutation events`，详情请参见：  
[https://developer.mozilla.org/en-US/docs/DOM/Mutation_events](https://developer.mozilla.org/en-US/docs/DOM/Mutation_events)  
[http://www.w3.org/TR/DOM-Level-3-Events/#events-mutationevents](http://www.w3.org/TR/DOM-Level-3-Events/#events-mutationevents)  

###嵌套的DOM事件

在浏览器端，有一些方法会立即触发某类事件，而这类事件也是同步的。比如`element.focus()`，下面是演示代码。  
{% codeblock lang:javascript %}
<input type="button" value="click me">
<input type="text">
<script type="text/javascript">
  var btn = document.getElementsByTagName('input')[0]
    , text = document.getElementsByTagName('input')[1];

  btn.onclick = function(e) {
    console.log('in onclick');
    text.focus();
    console.log('out onclick');
  };

  text.onfocus = function(e) {
    console.log('onfocus');
  };
</script>
{% endcodeblock %}  
执行结果如下：

![](http://img.heroicyang.com/synchronous-focus-event.png)  

常规情况下，事件处理都是一个一个执行的，而我们也就假定一个事件开始时，前一个事件是执行完毕了的。而以上这些同步事件不仅打破了我们的常规认识，还会给我们带来一些负面效应。不过我们依旧可以使用[上一篇](http://heroicyang.com/2012/09/22/javascript-timer-in-depth)中所使用的`setTimeout(func, 0)`来解决。  

##JavaScript执行与页面渲染

大多数浏览器中，JavaScript的执行和页面渲染是互斥的，于是JavaScript执行时，浏览器就不会做任何的页面渲染。比如下面的Demo...  
{% codeblock lang:javascript %}
<!DOCTYPE HTML>
<html lang="en-US">
<head>
  <meta charset="UTF-8">
  <title>JavaScript执行与页面渲染</title>
  <style type="text/css">
    #container {
      width: 200px; 
      height: 100px; 
      background-color: #A00000; 
      margin-bottom: 10px;
    }
  </style>
</head>
<body>
  <div id="container"></div>
  <input type="button" value="run" id="run">
  <script type="text/javascript">
    var runBtn = document.getElementById('run')
      , container = document.getElementById('container');
    
    runBtn.onclick = function(e) {
      for (var i = 0xA00000; i < 0xFFFFFF; i++) {
        container.style.backgroundColor = '#' + i.toString(16);
      }
    };
  </script>
</body>
</html>
{% endcodeblock %}
<iframe src="http://sample.heroicyang.com/repaint.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; width: 90%; height:150px; padding: 1px;"></iframe>

运行上面的Demo后，大多数浏览器都会假死了，直到`container`的背景颜色变更为`#FFFFFF`后才恢复。而有的浏览器(如Firefox)还会弹出警告，告知JavaScript没有响应，是终止还是等待。但是Opera却能正常运行，并不断更改背景颜色。因此不同浏览器对页面渲染和JavaScript执行的实现方式是不一样的。  

关于这方面有很大的学问，还需要继续学习，慢慢摸索。So...这个就点到为止了。

###模式对话框的同步调用

浏览器提供的如`alert`等的模式对话框是同步调用的，所以当这类对话框工作时，会停止`JavaScript线程`，当然如页面渲染等活动也将被冻结。继续下面的Demo…当运行代码下面的`iframe`中的进度条后，无论是点击`主窗体中的alert`按钮，还是点击`iframe中的alert`按钮，都会导致进度条挂起。  

<input type="button" value="主窗体中的alert" onclick="alert('主窗体对话框');">
{% codeblock lang:javascript %}
<div id="container" style="width: 0px; height: 20px; background-color: #A00000;"></div>
<input type="button" value="run" id="run">
<input type="button" value="stop" id="stop">
<input type="button" value="iframe中的alert" onclick="alert('iframe中的对话框');">
<script type="text/javascript">
  var runBtn = document.getElementById('run')
    , stopBtn = document.getElementById('stop')
    , container = document.getElementById('container');
  var timer = null;

  runBtn.onclick = function(e) {
    timer = setInterval(function() {
      var style = container.style;
      style.width = (parseInt(style.width) + 2) % 400 + 'px';
    }, 50);
  };
  stopBtn.onclick = function(e) {
    clearInterval(timer);
  };
</script>
{% endcodeblock %}
<iframe src="http://sample.heroicyang.com/modal-sync.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; width: 90%; height:70px; padding: 1px;"></iframe>  

因此，浏览器所提供的`alert`、`confirm`、`prompt`这三类模式对话框，都会阻塞`JavaScript线程`和`UI线程`。  

**依旧，Opera有一点点的例外。。。**  

在Opera中，点击`主窗体中的alert`按钮不会阻塞`iframe`中的进度条。。。又打破我们的常规认识啊：同一个页面上，`iframe`是和主窗体同一个线程的。但Opera的设计并非如此。。。  

##当脚本需要花很长的时间干复杂的工作时

类似的就是前面那个阻塞我们浏览器的，频繁更改`container`背景颜色的例子。最后，我们还是用[上一篇文章中](http://heroicyang.com/2012/09/22/javascript-timer-in-depth)的`setTimeout(func, 0)`来解决它吧。  
{% codeblock lang:javascript %}
<div id="container"></div>
<input type="button" value="run" id="run">
<input type="button" value="stop" id="stop">
<script type="text/javascript">
  var runBtn = document.getElementById('run')
    , stopBtn = document.getElementById('stop')
    , container = document.getElementById('container');
  var i = 0xA00000, timer = null;

  runBtn.onclick = function(e) {
    function run() {
      timer = setTimeout(run, 0);
      container.style.backgroundColor = '#' + i.toString(16);

      if (i++ == 0xFFFFFF) stop();
    }
    timer = setTimeout(run, 0);
  };
  stopBtn.onclick = stop;

  function stop() {
    clearTimeout(timer);
  }
</script>
{% endcodeblock %}
<iframe src="http://sample.heroicyang.com/heavy-jobs.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; width: 90%; height:150px; padding: 1px;"></iframe>  

最后，总结一下`setTimeout(func, 0)`的使用场景吧：
  
1. 让浏览器渲染当前的变化  
2. 避免长时间运行的脚本  
3. 流程控制  
4. 等等等等。。。