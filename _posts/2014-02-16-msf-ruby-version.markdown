---
layout: post_item
status: publish
published: true
title: msf更换ruby版本
author: jsongo
wordpress_id: 231
wordpress_url: http://jsongo.com/?p=231
date: '2014-02-16 23:20:10 +0800'
date_gmt: '2014-02-16 15:20:10 +0800'
categories: [draft]
tags: [msf, ruby]
comments: []
---
msf和ruby2.0兼容貌似不是很好，在mac安装时，运行msfconsole总是运行不起来。网上查了下，是和ruby2.0的兼容问题。官方是声称可以支持2.0，不过是有不少问题。所以干脆把版本切换回1.9

先安装rvm，即ruby version manager，用来管理ruby的版本的。教程很多，不列举了。

然后运行rvm 1.9.3-p484 --default

写这篇文章时，msf最新开发版引用的是ruby的这个版本。

之后再bundle install一下，再运行msf，可能之前的问题就消失了！

