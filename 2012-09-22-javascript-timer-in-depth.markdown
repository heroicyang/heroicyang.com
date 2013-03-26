---
layout: post
title: "深入理解JavaScript定时器"
date: 2012-09-22 18:57
meta: true
comments: true
categories: JavaScript
tags: [javascript,javascript timer]
---
对于浏览器内部，大部分操作都是异步的生成事件并添加到`JavaScript引擎线程`的队列中，然后由`JavaScript引擎线程`进行调度执行。因此浏览器的很多事件都是和`JavaScript`相结合的，但是也有一些内部的限制。  

首先我们非常确定`JavaScript`是单线程的，对于浏览器来说，一个窗体中只有一个`JavaScript引擎线程`。而其他的行为，如：渲染、下载等是由单独的线程进行管理的，且具有不同的优先级。  

##异步事件
前面提到大多数事件都是异步的，触发的时候就将回调函数添加到事件队列。浏览器提供了一个内部的回路，也就是之前所谈到的`Event Loop`，由它来负责检查队列和处理事件、执行函数等。详细可参考我的[前一篇博文](http://heroicyang.com/2012/08/28/javascript-event-loop.html)。而`setTimeout`和`setInterval`也是将其需要执行的函数添加到事件队列。  

**事实上，大多数交互和活动都得通过事件循环。**  
<!-- more -->  
##事件重叠

一些情况下，会有多个事件在同一时间附加到事件队列里。  

比如，`click`事件就会产生两个额外的事件：`mousedown`和`mouseup`。其中，`mouseup`和`click`事件会同时被添加到事件队列；而`mousedown`事件则很有可能会和另外一个事件重叠：`focus`。  

##setTimeout(func, 0)奇巧淫技

再一次解释关于`0ms`的误解：如果当前时钟周期内执行队列空闲，则立即执行该定时器，将回调函数加入到事件队列；然后等待下一个时钟周期，再执行该回调函数。不妨来看看下面的测试。
  
这段代码在我的浏览器中执行结果如下：

![](http://img.heroicyang.com/setTimeout-Measure.png)

在我本地的`Nodejs`环境中执行结果如下：  

![](http://img.heroicyang.com/setTimeout-Measure-Nodejs.png)  

上面的这个测试只是想说明`setTimeout(func, 0)`定时任务的回调函数执行时间是有延迟的，而并不是所谓的立即执行。  

因此，我们可以利用`setTimeout(func, 0)`来解决事件重叠所产生的负面效果，修正执行顺序。 

###奇巧淫技之一：模拟浏览器的事件捕获 
众所周知，浏览器的DOM事件都是采用冒泡的方式，只有个别浏览器是支持事件捕获的。而在实际的开发过程中可能存在需要事件捕获的需求，要求子元素的事件在父元素触发之后才能触发。为了兼容各个浏览器，我们不能使用事件捕获，而`setTimeout(func, 0)`在这个时候就很乐意帮忙了。  
{% codeblock lang:javascript %}
<input type="button" value="click" id="cbtn">
<div id="result"></div>
<script type="text/javascript">
  var cbtn = document.getElementById('cbtn')
    , result = document.getElementById('result');

  cbtn.onclick = function(e) {
    setTimeout(function() {
      result.innerHTML += 'input click, ';
    }, 0);
  };

  document.body.onclick = function(e) {
    result.innerHTML += 'body click -> ';
  };
</script>
{% endcodeblock %}  
点击查看运行效果：  
<iframe src="http://sample.heroicyang.com/setTimeout01.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; width: 90%; height:80px; padding: 1px;"></iframe>

###奇巧淫技之二：让浏览器更好的工作
大多数情况下，我们可以在浏览器的默认行为之前对事件进行处理，但是有时我们按照常规的思路去做的时候，往往事与愿违。比如下面的例子。  
{% codeblock lang:javascript %}
<input type="text" id="wordInput">
<script type="text/javascript">
  var wordInput = document.getElementById('wordInput');
  wordInput.onkeypress = function(e) {
    this.value = this.value.toUpperCase();
  };
</script>
{% endcodeblock %}  
看似一个很简单的需求：每输入一个字符，就将其转换为大写。但是上面的代码完全没有按照指示去做，不信你试试看：  
<iframe src="http://sample.heroicyang.com/setTimeout-toUpper-01.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; height:45px; padding: 1px;"></iframe>  

如果没有下一次输入，文本框中的小写字母永远都不会转换为大写。Why? 因为浏览器在`keypress`事件处理的时候，还没有将我们输入的值添加到文本框。于是乎换一个事件来handle然后再处理吧，既然键按下的时候还木有值，那就等键弹起来之后再处理。  
{% codeblock lang:javascript %}
<input type="text" id="wordInput">
<script type="text/javascript">
  var wordInput = document.getElementById('wordInput');
  wordInput.onkeyup = function(e) {
    this.value = this.value.toUpperCase();
  };
</script>
{% endcodeblock %}  
运行试试吧。
<iframe src="http://sample.heroicyang.com/setTimeout-toUpper-02.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; height:45px; padding: 1px;"></iframe>  

大概似乎是可行了，可是仔细观察就看出问题了。`keyup`事件触发时，文本框已经具备完整的值了，但先是一个小写的值，键完全释放之后转变为大写。这不科学…这太丑陋...  

是时候关门放出`setTimeout(func, 0)`了。。。
{% codeblock lang:javascript %}
<input type="text" id="wordInput">
<script type="text/javascript">
  var wordInput = document.getElementById('wordInput');
  wordInput.onkeypress = function(e) {
    var self = this;
    setTimeout(function() {
      self.value = self.value.toUpperCase();
    }, 0);
  };
</script>
{% endcodeblock %}  
<iframe src="http://sample.heroicyang.com/setTimeout-toUpper-03.html" style="border: 1px solid #DDD; border-radius: 3px; background: #F8F8F8; height:45px; padding: 1px;"></iframe>  

已经完美了。`keypress`事件触发时，将转换大写的操作添加到事件队列，紧接着浏览器添加我们输入的值，然后近乎0延迟的执行我们的转换大写操作函数。  

上面两个小案例只是冰山一角，so...合理利用`setTimeout(func, 0)`，明天更美好！