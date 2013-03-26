---
layout: post
title: "Backbone之旅：前端MVC架构初体验（上）"
date: 2012-08-08 00:48
meta: true
comments: true
categories: JavaScript
tags: [javascript,backbone,mvc]
---
最近一段时间来，才算是真正的开始深入学习`JavaScript`，收获颇丰。也首次领略了前端`MVC`架构的风采，现在前端`MVC`的类库和框架越来越多，在经过初步的评估之后，决定先学习备受推崇的`Backbone`。 
 
以前自己做的一些`Web`应用，基本上都是按照非常传统的方式：1.服务器端渲染模板；2.利用`jQuery`的`ajax`进行异步数据交换。所以首次接触前端架构类的东西，难免有点无从下手。经过几天的奋战，以及参阅国外大牛们的各种`Tutorial`之后，终于拨开迷雾，缕了些头绪，自己也试着从传统的方式过渡（重构）出了所谓的架构性的代码。  

整个重构的过程让我受益良多，所以决定再认真的记录一遍，加深自己的印象，也再确认一遍自己是否真的搞明白了，文章应该会比较长。  

<!-- more -->

首先，先上一段所谓的传统式的代码。
{% codeblock lang:javascript %}
$(function(){
  $('#new-todo form').submit(function(e){
    e.preventDefault();
    var that = this;

    $.ajax({
      url: '/add',
      type: 'POST',
      dataType: 'json',
      data: { todoContent: $(this).find('textarea').val() },
      success: function(data){
        $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
        $(that).find('textarea').val('');
      }
    });
  });
});
{% endcodeblock %}  
另外，也上一张关于这篇文章中涉及到的`HTML`结构图，方便参照。由于文章稍长，我想如果直接在这里插图的话会影响阅读，所以就只给出[图片链接](http://img.heroicyang.com/to-backbone-tutorial.png)了。 
 
上面的一段代码我想应该都是大家非常熟悉的做法，因为我是一个伪前端攻城湿，所以我以前的代码中无不充斥着类似的、一堆一堆这样的代码。看上去貌似挺好的啊，也没啥问题，程序跑得倍儿棒。但是就这么短短的一段代码，它可干了不少事情：监听页面事件、用户事件、网络事件，接收用户的输入、执行网络的I/O、解析服务端返回的数据、动态生成`HTML`结构，可谓是包罗万象啊，就这么短短的一段代码就解释了整个`Web`应用程序的本质。  

所以即便是这么一个小小的应用，逻辑和架构上都已经臃肿了，完全违反了咱们软件开发中的“单一职责原则”。如果是一个大应用，那估计就如乱麻------剪不断理还乱了。所以，改变迫在眉睫。 
 
确实咱的要求也不高，如果把它搞成这样，其实咱就满足了：

1. 在`$(document).ready`当中只保留一些应用程序的初始化代码即可，即应用的启动程序。  
2. 干掉乱如麻的逻辑，使得其符合咱们的“单一职责原则”，方便测试。  
3. 减小`ajax`和`DOM`的耦合，其实也算是第2条。  

OK，动手。按照最基本的重构方式，咱先把`ajax`分离到一个方法里面去。  
{% codeblock lang:javascript %}
var addTodo = function(){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: $('#new-todo').find('textarea').val() },
    success: function(data){
      $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
      $('#new-todo').find('textarea').val('');
    }
  });
};

$(function(){
  $('#new-todo form').submit(function(e){
    e.preventDefault();
    
    addTodo();
  });
});
{% endcodeblock %}  
但是，在`ajax`所在的方法中，`data`和`success`属性仍然保留了对`DOM`的依赖，于是接下来将其调整为函数的参数来传递。  
{% codeblock lang:javascript %}
var addTodo = function(options){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: options.todoContent },
    success: options.success
  });
};

$(function(){
  $('#new-todo form').submit(function(e){
    e.preventDefault();
    
    addTodo({
      todoContent: $('#new-todo').find('textarea').val(),
      success: function(data){
        $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
        $('#new-todo').find('textarea').val('');
      }
    });
  });
});
{% endcodeblock %}  
好像OK了，不过此时`addTodo()`方法暴露在全局环境内，任何人都可以呼之欲来。我可不想当屌丝，作为一个富有上进心的、想成为一个合格前端攻城湿的我，还是给`addTodo()`方法加个命名空间吧。  
{% codeblock lang:javascript %}
var TodoList = function(){};

TodoList.prototype.add = function(options){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: options.todoContent },
    success: options.success
  });
};

$(function(){
  var todoList = new TodoList();

  $('#new-todo form').submit(function(e){
    e.preventDefault();
    
    todoList.add({
      todoContent: $('#new-todo').find('textarea').val(),
      success: function(data){
        $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
        $('#new-todo').find('textarea').val('');
      }
    });
  });
});
{% endcodeblock %}  
现在`submit`事件只依赖一个`todoList`变量了，而且最重要的是现在的`submit`事件中只关注`DOM`操作了，干脆大刀阔斧的把它移到外层去。于是咱们引入视图`View`了。  
{% codeblock lang:javascript %}
var TodoList = function(){};

TodoList.prototype.add = function(options){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: options.todoContent },
    success: options.success
  });
};

var NewTodoView = function(options){
  var todoList = options.todoList;

  $('#new-todo form').submit(function(e){
    e.preventDefault();
    
    todoList.add({
      todoContent: $('#new-todo').find('textarea').val(),
      success: function(data){
        $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
        $('#new-todo').find('textarea').val('');
      }
    });
  });
};

$(function(){
  var todoList = new TodoList();
  new NewTodoView({ todoList: todoList });
});
{% endcodeblock %}  
恩，现如今`$(document).ready`中就简洁得只剩我们之前所说的应用启动代码了。虽然代码已经组件化了，也工作得很好，但是仍然有需要重构的地方。`NewTodoView`目前看上去都不怎么像一个对象的行为，所以继续重构之。  
{% codeblock lang:javascript %}
var TodoList = function(){};

TodoList.prototype.add = function(options){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: options.todoContent },
    success: options.success
  });
};

var NewTodoView = function(options){
  this.todoList = options.todoList;

  $('#new-todo form').submit($.proxy(this.addTodo, this));
};

NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();
    
  this.todoList.add({
    todoContent: $('#new-todo').find('textarea').val(),
    success: function(data){
      $('#todo-list ul').append('<li>' + data.todoContent + '</li>');
      $('#new-todo').find('textarea').val('');
    }
  });
};

$(function(){
  var todoList = new TodoList();
  new NewTodoView({ todoList: todoList });
});
{% endcodeblock %}  
这里用到了`jQuery`中的`$.proxy()`方法来解决`this`作用域的问题，玩`JavaScript`的童鞋们应该都很了解作用域这个东东。接下来，咱干点有关洁癖的事情，鉴于要保证代码的清晰、方便阅读，咱把`success`里面的行为采用`callback`的形式来完成。  
{% codeblock lang:javascript %}
/*前面不变*/
NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();
  var that = this;

  this.todoList.add({
    todoContent: $('#new-todo').find('textarea').val(),
    success: function(data){
      that.appendTodo(data.todoContent);
      that.clearTextArea();
    }
  });
};

NewTodoView.prototype.appendTodo = function(todoContent){
  $('#todo-list ul').append('<li>' + todoContent + '</li>');
};

NewTodoView.prototype.clearTextArea = function(){
  $('#new-todo').find('textarea').val('');
};
/*后面也不变*/
{% endcodeblock %}  
至此，重构的第一个版本其实就算得上大功告成了，已经达到前面提出的三大方针政策。文章果然比较长，所以我决定还是分成了上、下两节，当前这篇中完全没涉及到`backbone`，所以到此就打住了，敬请关注下回分解。
