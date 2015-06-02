---
layout: post_item
status: publish
published: true
title: redis监控
author: jsongo
wordpress_id: 205
wordpress_url: http://jsongo.com/?p=205
date: '2014-01-19 01:40:55 +0800'
date_gmt: '2014-01-18 17:40:55 +0800'
categories: [draft]
tags: [redis]
comments: []
---
监控redis的实时情况和总体数据，界面： 

![](/img/201401/redmon.png)

安装很简单，直接用gem: 
{% highlight bash %}
gem install redmon
{% endhighlight %}
然后运行redmon，它就会自己开一个端口，默认是4567，用浏览器直接访问就可以了。  
redmon的详细说明在：https://github.com/steelThread/redmon 
