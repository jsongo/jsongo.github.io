---
layout: post_item
status: publish
published: true
title: "用htaccess禁止访问目录"
author: jsongo
wordpress_id: 280
wordpress_url: http://jsongo.com/?p=280
date: '2014-04-08 23:44:40 +0800'
date_gmt: '2014-04-08 15:44:40 +0800'
categories: [draft]
tags: [apache]
comments: []
---
apache服务器中，不想让用户直接访问某个文件夹，只需要在这个文件夹里面新建一个.htaccess文件，然后写入deny from all

这样直接访问就访问不到这个文件夹了。

