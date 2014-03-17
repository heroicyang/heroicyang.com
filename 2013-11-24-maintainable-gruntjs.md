title: 编写可维护的 Gruntfile.js
date: 2013-11-24 20:19:27
tags: [javascript, grunt, gruntjs]
---
使用[Grunt](http://gruntjs.com/)已经有很长一段时间了，不得不感叹其社区的壮大，各种插件层出不穷。而在这期间我也换过几种方式来组织`Gruntfile.js`，但都不是很理想，直到前段时间看到[load-grunt-tasks](https://npmjs.org/package/load-grunt-tasks)这个插件以及[More maintainable Gruntfiles](http://www.thomasboyt.com/2013/09/01/maintainable-grunt.html)这篇文章后，我就把项目中的`Gruntfile.js`都按照该文章作者所述的方式重新组织了一遍。

我就暂且把这种方式用自己的文字记录一下并分享给正在使用`Grunt`的同学们吧，不过本文也不算是对《More maintainable Gruntfiles》的翻译呐，毕竟我E文太差~

<!-- more -->

### load-grunt-tasks 插件

首先介绍下`load-grunt-tasks`这个插件。

我们一般都会把所有用到的插件以及插件的配置写到`Gruntfile.js`里面，对于小项目来说这个文件最终或许不是很大，但是对于大项目、有很多配置或者很多自定义任务的项目来说，最后这个文件都会变得越来越长，维护起来就成了麻烦。比如下面这样：

```javascript
module.exports = function(grunt) {
  grunt.initConfig({
    pkg: grunt.file.readJSON('package.json'),
    concat: {
      options: {
        separator: ';'
      },
      dist: {
        src: ['src/**/*.js'],
        dest: 'dist/<%= pkg.name %>.js'
      }
    },
    uglify: {
      options: {
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
      },
      dist: {
        files: {
          'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
        }
      }
    },
    qunit: {
      files: ['test/**/*.html']
    },
    jshint: {
      files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      options: {
        globals: {
          jQuery: true,
          console: true,
          module: true,
          document: true
        }
      }
    },
    watch: {
      files: ['<%= jshint.files %>'],
      tasks: ['jshint', 'qunit']
    }
  });

  grunt.loadNpmTasks('grunt-contrib-uglify');
  grunt.loadNpmTasks('grunt-contrib-jshint');
  grunt.loadNpmTasks('grunt-contrib-qunit');
  grunt.loadNpmTasks('grunt-contrib-watch');
  grunt.loadNpmTasks('grunt-contrib-concat');

  grunt.registerTask('test', ['jshint', 'qunit']);
  grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);
};
```

这是一个很标准的`Gruntfile.js`，显然也算是很简短的了，但是看起来也有点累觉不爱。于是`load-grunt-tasks`出来帮我们解决了一部分问题。

它会自动读取并加载项目`packge.json`文件中`devDependencies`配置下以`grunt-*`开头的依赖库。于是乎我们就可以用一行代码来搞定上面代码中很多行的`loadNpmTasks`了。

```javascript
require('load-grunt-tasks')(grunt);
// 就代替了以下全部
grunt.loadNpmTasks('grunt-contrib-uglify');
grunt.loadNpmTasks('grunt-contrib-jshint');
grunt.loadNpmTasks('grunt-contrib-qunit');
grunt.loadNpmTasks('grunt-contrib-watch');
grunt.loadNpmTasks('grunt-contrib-concat');
```

### Gruntfile.js 继续廋身

`load-grunt-tasks`插件替`Gruntfile.js`省去了那些反复书写的方法调用，接下来就是将整个`Gruntfile.js`变得干净清爽的步骤了。那就是把上面的各种`config`分离出去，让它们各自代表自己是属于哪个插件，而不是一口气全写在一起。当然，还有各种用`registerTask`方法定义的自定义任务，也该单独放到相应的文件中。

#### 自定义任务迁移

首先，在项目根目录下建一个名为`tasks`的目录，在这个目录下来编写各种自定义任务。可以一个任务一个 js 文件，也可以多个简单任务在一个 js 文件，看个人喜好吧。然后在`Gruntfile.js`中用一行代码来载入这些自定义任务：

```javascript
grunt.loadTasks('tasks');  // 即刚刚新建目录的名称
```

#### 配置项迁移

然后再在这个目录下新建一个名为`options`的子目录(tasks/options)，来存放之前说的那些`config`们。为每一类`config`建一个 js 文件，并以配置项节点名作为文件名称，比如下面这样：

```
tasks
└── options
    └── concat.js
    └── uglify.js
    └── qunit.js
    └── jshint.js
```

然后在每个文件中导出对应的配置项，拿`concat.js`来说：

```javascript
module.exports = exports = {
  options: {
    separator: ';'
  },
  dist: {
    src: ['src/**/*.js'],
    dest: 'dist/<%= pkg.name %>.js'
  }
};
```

最后在`Gruntfile.js`里用`require`将配置逐个引入即可，也可以封装一个函数来做这件事情。

```javascript
function loadConfig(configPath) {
  var config = {};

  glob.sync('*', { cwd: configPath })
    .forEach(function(configFile) {
      var prop = configFile.replace(/\.js$/, '');
      config[prop] = require(path.join(__dirname, configPath, configFile));
    });

  return config;
}
```

再改写`Gruntfile.js`中`initConfig`的调用即可。

```javascript
var _ = require('lodash');
var config = {
  pkg: grunt.file.readJSON('package.json')
};
_.extend(config, loadConfig('./tasks/options/'));
grunt.initConfig(config);
```

### 写在最后

于是乎在每个项目中`Gruntfile.js`几乎一致，而且也几乎不会再变更。`Gruntfile.js`、自定义任务、任务配置项各司其职，需要变化时只需对相应文件做出调整即可。

就在前些天，又一位 GitHuber 将这个思路封装成了一个库：[load-grunt-config](https://github.com/firstandthird/load-grunt-config)，感兴趣的同学可以看看。

最终的`Gruntfile.js`可以查看这个例子：[https://github.com/heroicyang/cnodeclub/blob/master/Gruntfile.js](https://github.com/heroicyang/cnodeclub/blob/master/Gruntfile.js)

### 参考资料

load-grunt-tasks: [https://npmjs.org/package/load-grunt-tasks](https://npmjs.org/package/load-grunt-tasks)  
More maintainable Gruntfiles: [http://www.thomasboyt.com/2013/09/01/maintainable-grunt.html](http://www.thomasboyt.com/2013/09/01/maintainable-grunt.html)