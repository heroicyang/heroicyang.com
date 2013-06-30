title: Mongodb中随机的查询文档记录
date: 2013-06-20 21:59:46
tags: [mongodb]
---
在实际应用场景中，几乎都会有随机获取数据记录的需求。而这个需求在Mongodb却不是很好实现，就目前而言，大致上有三种解决方案：

1. 先计算出一个从`0`到记录总数之间的随机数，然后采用`skip(yourRandomNumber)`方法。
2. 为每一条记录增设`random`字段，插入数据时赋值为`Math.random()`，查询时采用`$gte`和`$lte`。
3. 借助Mongodb对地理空间索引（`geospatial indexes`）的支持，从而可以在第二种方法的基础上来实现随机记录的获取。

因为Mongodb是不建议使用`skip`方法的，所以这里就略去第一种方法吧。

### 方法二

{% codeblock lang:javascript %}
> db.twitter.save({ username: 'heroic', random: Math.random(), content: 'balabala0...' })
> db.twitter.save({ username: 'heroic', random: Math.random(), content: 'balabala1...' })
> db.twitter.save({ username: 'heroic', random: Math.random(), content: 'balabala2...' })
> db.twitter.save({ username: 'heroic', random: Math.random(), content: 'balabala3...' })
> db.twitter.save({ username: 'heroic', random: Math.random(), content: 'balabala4...' })
/* more records... */

/* create index */
> db.twitter.ensureIndex({ username: 1, random: 1 })

> rand = Math.random()
> result = db.twitter.findOne({ username: 'heroic', random: { $gte: rand } })
> if (result == null) {
>   result = db.twitter.findOne({ username: 'heroic', random: { $lte: rand } })
> }
{% endcodeblock %}

### 方法三

{% codeblock lang:javascript %}
> db.twitter.save({ username: 'heroic', random: [Math.random(), 0], content: 'balabala0...' })
> db.twitter.save({ username: 'heroic', random: [Math.random(), 0], content: 'balabala1...' })
> db.twitter.save({ username: 'heroic', random: [Math.random(), 0], content: 'balabala2...' })
> db.twitter.save({ username: 'heroic', random: [Math.random(), 0], content: 'balabala3...' })
> db.twitter.save({ username: 'heroic', random: [Math.random(), 0], content: 'balabala4...' })
/* more records... */

/* create index */
> db.twitter.ensureIndex({ username: 1, random: '2d' })

> result = db.twitter.findOne({ username: 'heroic', random: { $near: [Math.random(), 0] } })
{% endcodeblock %}

更多关于Mongodb地理空间索引资料，请参见[这里](http://docs.mongodb.org/manual/core/2d/)。

目前这几种方案似乎都不是很理想，但是也没有其他办法了，所以广大程序员们就相约到Mongodb的官方jira提了相应的需求，但是目前仍然没有任何的响应。可以参见[这里](https://jira.mongodb.org/browse/SERVER-533)，围观一下。