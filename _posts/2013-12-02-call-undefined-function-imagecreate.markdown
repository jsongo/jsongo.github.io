---
layout: post_item
status: publish
published: true
title: Call to undefined function ImageCreate错误的解决
author: jsongo
wordpress_id: 125
wordpress_url: http://jsongo.com/?p=125
date: '2013-12-02 01:00:28 +0800'
date_gmt: '2013-12-01 17:00:28 +0800'
categories: [draft]
tags: [linux]
comments: []
---
主要是因为缺少gd库，装一下就ok了：  
apt-get install php5-gd  
DONE  
