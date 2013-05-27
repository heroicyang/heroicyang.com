---
layout: post
title: "Backbone之旅：前端MVC架构初体验（下）"
date: 2012-08-09 00:59
meta: true
comments: true
tags: [javascript,backbone,mvc]
---
继[《Backbone之旅：前端MVC架构初体验（上）》](http://heroicyang.com/2012/08/08/to-backbone-tutorial-1)，上篇中最后的代码已经完全达到最初提出的几点要求，现在就结合`Backbone`提供的能力，来继续精简代码。最后的目标就是将上篇中的代码全部重构为`Backbone`的`MVC`模式。  

上篇中最后一次改造就已经使用到了`callback`的方式，所以我们索性再加上`Event`机制吧，因为`Backbone`内置了这个能力。  
{% codeblock lang:javascript %}
var events = _.clone(Backbone.Events);

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

  events.on('todo:add', this.appendTodo, this);
  events.on('todo:add', this.clearTextArea, this);

  $('#new-todo form').submit($.proxy(this.addTodo, this));
};

NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();

  this.todoList.add({
    todoContent: $('#new-todo').find('textarea').val(),
    success: function(data){
      events.trigger('todo:add', data.todoContent);
    }
  });
};

/*后面不变*/
{% endcodeblock %}  
现在既然调用`add()`时传入的`success`属性已经完全不涉及到`DOM`操作了，而是单纯的事件触发，那完全可以把这个行为放置到`TodoList`原型的`add()`方法中去了，这样重用性更高。  

<!-- more -->
{% codeblock lang:javascript %}
/* … */
TodoList.prototype.add = function(todoContent){
  $.ajax({
    url: '/add',
    type: 'POST',
    dataType: 'json',
    data: { todoContent: todoContent },
    success: function(data){
      events.trigger('todo:add', data.todoContent);
    }
  });
};
/* … */
NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();

  this.todoList.add($('#new-todo').find('textarea').val());
};
/* … */
{% endcodeblock %}  
接下来，咱看看在`NewTodoView`这个视图中事件订阅所触发的对应方法`appendTodo()`和`clearTextArea()`中，涉及到的是处在同一级别的不同的`DOM`元素节点，也就是说在`NewTodoView`这个视图中，我们处理了两个`DOM`元素，这似乎和我们之前提到的“单一职责原则”相违背了，所以还有待进一步的改进。  

我们分别把新增`Todo`的视图和负责展示`Todo Item`的视图分开定义，使其符合“单一职责原则”。  
{% codeblock lang:javascript %}
/* 前面不变 */
var NewTodoView = function(options){
  this.todoList = options.todoList;

  events.on('todo:add', this.clearTextArea, this);

  $('#new-todo form').submit($.proxy(this.addTodo, this));
};

NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();

  this.todoList.add($('#new-todo').find('textarea').val());
};

NewTodoView.prototype.clearTextArea = function(){
  $('#new-todo').find('textarea').val('');
};

/* 用于展示Todo Item */
var TodoView = function(){
  events.on('todo:add', this.appendTodo, this);
};

TodoView.prototype.appendTodo = function(todoContent){
  $('#todo-list ul').append('<li>' + todoContent + '</li>');
};

/* 应用程序启动 */
$(function(){
  var todoList = new TodoList();
  new NewTodoView({ todoList: todoList });
  new TodoView();
});
{% endcodeblock %}  
现在每个`View`里面只依赖一个顶层的`HTML Element`了，而在各自的`View`里面多次使用到了`$('#new-todo')`这样的代码，所以干脆将其在初始化的时候作为`View`的一个属性来提供吧。  
{% codeblock lang:javascript %}
/* 前面依旧不变 */
var NewTodoView = function(options){
  this.todoList = options.todoList;
  this.el = $('#new-todo');  //定义一个el属性ß

  events.on('todo:add', this.clearTextArea, this);

  this.el.find('form').submit($.proxy(this.addTodo, this));
};

NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();

  this.todoList.add(this.el.find('textarea').val());
};

NewTodoView.prototype.clearTextArea = function(){
  this.el.find('textarea').val('');
};

var TodoView = function(){
  this.el = $('#todo-list');
  events.on('todo:add', this.appendTodo, this);
};

TodoView.prototype.appendTodo = function(todoContent){
  this.el.find('ul').append('<li>' + todoContent + '</li>');
};
/* 后面不变 */
{% endcodeblock %}  
此时观察发现，两个`View`当中还保留着对`DOM`节点的依赖，其重用度依然不高，于是可采用实例化`View`的时候传入`el`参数来解决这个问题。  
{% codeblock lang:javascript %}
/* 前面不变 */
var NewTodoView = function(options){
  this.todoList = options.todoList;
  this.el = options.el;

  events.on('todo:add', this.clearTextArea, this);

  this.el.find('form').submit($.proxy(this.addTodo, this));
};
/* NewTodoView的原型方法也不变 */

var TodoView = function(options){
  this.el = options.el;
  events.on('todo:add', this.appendTodo, this);
};
/* TodoView的原型方法也不变 */

/* 初始化View的时候传入el */	
$(function(){
  var todoList = new TodoList();
  new NewTodoView({ el: $('#new-todo'), todoList: todoList });
  new TodoView({ el: $('#todo-list') });
});
{% endcodeblock %}  
`View`中我们频繁使用到了`jQuery`的`find()`方法来查找`View`所在`el`下面的子元素，所以可以考虑将这作为`View`的特性来提供，于是我们为`View`定义这样一个名叫`$`方法，然后替换掉`this.el.find()`这样的写法。  
{% codeblock lang:javascript %}
/* … */
var NewTodoView = function(options){
  this.todoList = options.todoList;
  this.el = options.el;

  events.on('todo:add', this.clearTextArea, this);

  this.$('form').submit($.proxy(this.addTodo, this));
};

NewTodoView.prototype.addTodo = function(e){
  e.preventDefault();

  this.todoList.add(this.$('textarea').val());
};

NewTodoView.prototype.clearTextArea = function(){
  this.$('textarea').val('');
};

NewTodoView.prototype.$ = function(selector){
  return this.el.find(selector);
};

var TodoView = function(options){
  this.el = options.el;
  events.on('todo:add', this.appendTodo, this);
};

TodoView.prototype.appendTodo = function(todoContent){
  this.$('ul').append('<li>' + todoContent + '</li>');
};

TodoView.prototype.$ = function(selector){
  return this.el.find(selector);
};
/* … */
{% endcodeblock %}  
上面的代码越来越多了，看上去好像咱是干的坏事，而不是往好的方向发展啊。是的，如果每个`View`都有很多自己的特性（方法），那向上面这样着实太痛苦了。看样子是时候请出`Backbone`提供的`View`特性了。OK，把我们自己的`View`转移到`Backbone`的。  
{% codeblock lang:javascript %}
/* … */
var NewTodoView = Backbone.View.extend({
  initialize: function(options){
    this.todoList = options.todoList;
    this.el = options.el;

    events.on('todo:add', this.clearTextArea, this);

    this.$('form').submit($.proxy(this.addTodo, this));
  },
  addTodo: function(e){
    e.preventDefault();

    this.todoList.add(this.$('textarea').val());
  },
  clearTextArea: function(){
    this.$('textarea').val('');
  },
  $: function(selector){
    return this.el.find(selector);
  }
});

var TodoView = Backbone.View.extend({
  initialize: function(options){
    this.el = options.el;
    events.on('todo:add', this.appendTodo, this);
  },
  appendTodo: function(todoContent){
    this.$('ul').append('<li>' + todoContent + '</li>');
  },
  $: function(selector){
    return this.el.find(selector);
  }
});
/* … */
{% endcodeblock %}  
由于`Backbone`的`View`已经提供了我们实现的`$()`方法的能力，也叫`$`（这也是之前我们自己命名的原因）；同时`Backbone`的`View`也提供了`this.el`的能力，所以可以把它们从代码中显示的移除了。  
{% codeblock lang:javascript %}
/* … */
var NewTodoView = Backbone.View.extend({
  initialize: function(options){
    this.todoList = options.todoList;

    events.on('todo:add', this.clearTextArea, this);

    this.$('form').submit($.proxy(this.addTodo, this));
  },
  addTodo: function(e){
    e.preventDefault();

    this.todoList.add(this.$('textarea').val());
  },
  clearTextArea: function(){
    this.$('textarea').val('');
  }
});

var TodoView = Backbone.View.extend({
  initialize: function(options){
    events.on('todo:add', this.appendTodo, this);
  },
  appendTodo: function(todoContent){
    this.$('ul').append('<li>' + todoContent + '</li>');
  }
});
/* 启动代码依然不变 */
{% endcodeblock %}  
现在可以回过头来看看`ajax`那部分了，由于`Backbone`提供了`Model`的能力，这个就是用于和服务端打交道的，所以将长长的`ajax`代码改写为这一方式。  
{% codeblock lang:javascript %}
/* … */
var Todo = Backbone.Model.extend({
  url: '/add'
});

var TodoList = function(){};

TodoList.prototype.add = function(todoContent){
  var todo = new Todo();
  todo.save({ todoContent: todoContent },{
    success: function(model, data){
      events.trigger('todo:add', data.todoContent);
    }
  });
};
/* … */
{% endcodeblock %}  
同时，`Backbone`中还提供了一个`Collection`的概念，也就是`Model`的集合，比如我们这个案例中，每次创建单条的`Todo`，然后形成`Todo List`。当然，我们的任何数据都应该是以多条记录的方式存在的。所以，我们同时将上面的`TodoList`的实现改为`Collection`。  

而且，`Backbone`的`Collection`已经支持了`Event`机制，所以我们也无需自定义`events`了，于是开头的`events`变量也一并移除了。  
{% codeblock lang:javascript %}
var Todo = Backbone.Model.extend({
  url: '/add'
});

var TodoList = Backbone.Collection.extend({
  add: function(todoContent){
    var todo = new Todo(),
        that = this;
    todo.save({ todoContent: todoContent },{
      success: function(model, data){
        that.trigger('add', data.todoContent);
      }
    });
  }
});

var NewTodoView = Backbone.View.extend({
  initialize: function(options){
    this.todoList = options.todoList;

    this.todoList.on('add', this.clearTextArea, this);

    this.$('form').submit($.proxy(this.addTodo, this));
  },
  addTodo: function(e){
    e.preventDefault();

    this.todoList.add(this.$('textarea').val());
  },
  clearTextArea: function(){
    this.$('textarea').val('');
  }
});

var TodoView = Backbone.View.extend({
  initialize: function(options){
    this.todoList = options.todoList;
    this.todoList.on('add', this.appendTodo, this);
  },
  appendTodo: function(todoContent){
    this.$('ul').append('<li>' + todoContent + '</li>');
  }
});

$(function(){
  var todoList = new TodoList();
  new NewTodoView({ el: $('#new-todo'), todoList: todoList });
  new TodoView({ el: $('#todo-list'), todoList: todoList });
});
{% endcodeblock %}  
`Collection`提供了一个名叫`create()`的方法，其可以根据`Collection`的`Model`属性创建一个`Model`的实例，并执行`Model`的`save()`方法。所以我们的`TodoList`中的`add()`方法已经可以废去了。我们只需为`TodoList`提供`Model`属性的值即可，然后在`NewTodoView`的`addTodo()`方法中，替换`this.todoList.add()`方法为`this.todoList.create()`。  
{% codeblock lang:javascript %}	
/* … */
var TodoList = Backbone.Collection.extend({
  model: Todo
});

var NewTodoView = Backbone.View.extend({
  initialize: function(options){
    this.todoList = options.todoList;

    this.todoList.on('add', this.clearTextArea, this);

    this.$('form').submit($.proxy(this.addTodo, this));
  },
  addTodo: function(e){
    e.preventDefault();

    this.todoList.create({ todoContent: this.$('textarea').val() });  //替换为create方法
  },
  clearTextArea: function(){
    this.$('textarea').val('');
  }
});
/* … */
{% endcodeblock %}  
这时，我们的`Model`、`Collection`、`View`都已经齐上阵了。由于`Backbone`的`View`已经内置`collection`属性，使得我们可以设置、获取`View`对应的`Collection`，所以我们完全无需手动在`View`的内部来定义一个`todoList`的变量了。  
{% codeblock lang:javascript %}
/* … */
var NewTodoView = Backbone.View.extend({
  initialize: function(options){
    this.collection.on('add', this.clearTextArea, this);

    this.$('form').submit($.proxy(this.addTodo, this));
  },
  addTodo: function(e){
    e.preventDefault();

    this.collection.create({ todoContent: this.$('textarea').val() });
  },
  clearTextArea: function(){
    this.$('textarea').val('');
  }
});

var TodoView = Backbone.View.extend({
  initialize: function(options){
    this.collection.on('add', this.appendTodo, this);
  },
  appendTodo: function(todo){
    this.$('ul').append('<li>' + todo.get('todoContent') + '</li>');
  }
});

$(function(){
  var todoList = new TodoList();
  new NewTodoView({ el: $('#new-todo'), collection: todoList });
  new TodoView({ el: $('#todo-list'), collection: todoList });
});
{% endcodeblock %}  
至此，完整的基于`Backbone`的`Model`、`Collection`、`View`模式就构建好了。如果说还有什么瑕疵的话，应该就是一些表层功夫了，那就是咱们的`HTML Element`的`append`了，需要做一些过滤，比如用户输入`JavaScript`代码那就糟糕了。  
{% codeblock lang:javascript %}
this.$('ul').append('<li>' + todo.get('todoContent') + '</li>');
调整为
this.$('ul').append('<li>' + todo.escape('todoContent') + '</li>');
{% endcodeblock %}  
这样就Perfect了。文章忒长了点，但是为了从一个`0`变成一个`1`，我想应该还是很有意思的。