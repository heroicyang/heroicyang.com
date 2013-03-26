---
layout: post
title: "像黑客一样写博客"
date: 2012-08-26 23:24
meta: true
comments: true
tags: blog
---
“像黑客一样写博客”，这个标题是网络上对`Octopress`（其实应该是`Jekyll`）的评价，而这一评价是来自一篇几年前的[文章](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html)。当我将自己的博客抹掉并重新开始的时候，我也准备以这个标题来作为这次的新起点。  

其实早就有换掉`WordPress`的想法，一是因为它太臃肿了，我只是想简单的写写博客，用不着那么多强大的功能；二是它对插入代码的支持让我绝望了，每次用`Markdown`写好文章，复制其`HTML`到`WordPress`之后，都要调整好半天的样式；三是我之前的博客中太多的碎碎念之类的水文了，可谓杂、乱，并不像一个记录技术的博客。综合这些借口，我每次登录到`WordPress`后台都没有再写文章的激情。  
<!-- more -->
而之前我也和[fiture](http://veryb.us/)聊到用[Nodejs](http://heroicyang.com/tags/nodejs/)来重新写个博客，也当学习练手。经过间歇性的聊聊之后，我提议说不弄数据库了，直接将文章以`Markdown`的格式`push`到`Github`吧等等的初步想法。当时我们觉得还不错，因为也没了解过有没有这样的东西存在，之后我下意识地搜了下。结果发现了`OctoPress`这个东西，了解之后感觉完全正合我意啊，所以二话不说就直接给换上了。  

于是我总算是终于弃掉了`WordPress`，用上`Octopress`这个高级货，可谓得心应手。  

其实我也就冲着我认为的这几点优势：

1. 直接使用`Markdown`写文章
2. 全站静态化，根据`Markdown`生成文章的静态页面
3. 所以直接在`Terminal`把文章`push`到我的[Github](https://github.com/heroicYang/heroicyang.com)上即可，有版本管理真好
4. 然后加之`Github Page`的支持，虽然有一些些小问题，比如`缓存`，但瑕不掩瑜
5. 整个写作过程和写代码的过程是一致的，符合码农的行为习惯，也就是所谓的“像黑客一样写博客”

这样，就只需要一个`Markdown`编辑器（推荐[Mou](http://mouapp.com/)和[Sublime Text 2](http://www.sublimetext.com/2)），再配合终端的`git`命令就OK了，其余的都不用管了，交给第三方去。比如：评论系统我就采用了国内的[多说](http://duoshuo.com/)；然后用[Dropbox](http://dropbox.com/)来保存文章中会用到的图片，因为`Dropbox`被`GFW`认证，所以再利用[Nginx](http://en.wikipedia.org/wiki/Nginx)做个[反向代理](http://en.wikipedia.org/wiki/Reverse_proxy)。一切都妥妥的了。  

以前博客的那些废柴文章都不要了，不过还是做了个备份，算是纪念好了。而把近期写的3篇与[JavaScript](http://heroicyang.com/tags/javascript/)相关的文章转移了过来，重新开始技术博客的历程，见证成长。
