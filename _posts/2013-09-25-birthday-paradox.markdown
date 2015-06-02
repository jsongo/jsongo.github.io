---
layout: post_item
status: publish
published: true
title: "计算生日悖论(python)"
author: jsongo
wordpress_id: 35
wordpress_url: http://jsongo.com/?p=35
date: '2013-09-25 01:02:30 +0800'
date_gmt: '2013-09-24 17:02:30 +0800'
categories: [python]
tags: [python, masterpiece]
comments: []
---
闲着无聊，突然想到生日悖论，写个代码去验证一下。  
多少人在一个房间，才能使得至少两个人同一天生日的概率大于1/2？


第二个人不同于第一个的概率是1-1/365，第三个人和前两个不同的概率是1-2/365... ...  
前面这些人都不同的概率是(1-1/365)(1-2/365)...(1-n/365)，当这个值小于1/2的时候，也就是说当前面n个人都不相同的概率小于1/2的时候，那么至少两个人相同的概率就大于1/2，于是：  
{% highlight python %}  
i=1  
rt = 1  
while rt >= 0.5:  
    rt *= (1-i/365.0)  
    i += 1  
{% endhighlight %}  
再打印出i，它就是我们要找的值。  
