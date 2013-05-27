layout: photo
title: 折腾了个新主题
date: 2013-05-22 23:36:51
photos:
- http://img.heroicyang.com/hexo-theme-modernist-index.png
tags: [hexo,theme,modernist]
---
前几天偶然发现了一个Github Page主题：[Modernist](http://orderedlist.github.io/modernist/)，觉着很舒服。作者将其开源在Github上的，所以我就fork下来自己折腾了个[Hexo](https://github.com/tommy351/hexo)的主题。  

根据[light](https://github.com/tommy351/hexo-theme-light)主题的结构修改而来，但是去掉了侧边栏，改成一栏，所以随之也就没有了light主题的那些widget了。不过我增加了国内的[多说评论框](http://duoshuo.com/)的配置，以及更好的响应式支持。  

主题已经放到[Github](https://github.com/heroicyang/hexo-theme-modernist)上了。为了更好的使用这个主题，建议clone我fork的[hexo](https://github.com/heroicyang/hexo)项目到本地，使用`/path/to/hexo/bin/hexo`来代替之前安装的全局`hexo`命令，方法见下面。我主要修改了代码块高亮（highlight）生成，以及修复了始终会生成Read more链接的BUG，不过我会尽快发起pull request到hexo的。  

另外，如果需要使用多说评论框，可以使用我的自定义多说评论框样式，主要保持了和Modernist theme的样式统一。请猛戳[这里](https://gist.github.com/heroicyang/5644407)。

{% codeblock lang:bash %}
git clone https://github.com/heroicyang/hexo /path/to/hexo
cd /path/to/hexo
npm install
# 方法1
ln -s /path/to/hexo/bin/hexo /usr/bin/hexo
hexo your command (new/generate/deploy…)
# 方法2
/path/to/hexo/bin/hexo your command (new/generate/deploy…)
{% endcodeblock %}