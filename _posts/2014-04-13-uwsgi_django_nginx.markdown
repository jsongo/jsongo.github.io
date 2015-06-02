---
layout: post_item
status: publish
published: true
title: uwsgi+django+nginx，报internal server error的可能原因
author: jsongo
wordpress_id: 282
wordpress_url: http://jsongo.com/?p=282
date: '2014-04-13 13:23:51 +0800'
date_gmt: '2014-04-13 05:23:51 +0800'
categories: [draft]
tags: [python, nginx]
comments: []
---
1、如果不是用--vhost形式跑的报这个错，换用这个虚拟模式试试


2、查看目录权限

3、nginx的配置项：

uwsgi_param UWSGI_SCRIPT abc_script;

uwsgi_param UWSGI_CHDIR /etc/nginx/html/abc;

UWSGI_CHDIR后面没/。

UWSGI_SCRIPT的名字不能跟项目名一样（这个问题搞了我n久）

