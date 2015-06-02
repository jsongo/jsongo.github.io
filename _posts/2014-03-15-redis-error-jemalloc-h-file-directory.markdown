---
layout: post_item
status: publish
published: true
title: 'redis编译报错jemalloc/jemalloc.h: No such file or directory'
author: jsongo
wordpress_id: 262
wordpress_url: http://jsongo.com/?p=262
date: '2014-03-15 11:12:39 +0800'
date_gmt: '2014-03-15 03:12:39 +0800'
categories: [draft]
tags: [redis]
comments: []
---
确认你安装了jemalloc

然后：

make MALLOC=libc
