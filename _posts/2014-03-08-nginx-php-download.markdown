---
layout: post_item
status: publish
published: true
title: nginx中的php文件老是变成下载的一种情况
author: jsongo
wordpress_id: 245
wordpress_url: http://jsongo.com/?p=245
date: '2014-03-08 20:34:49 +0800'
date_gmt: '2014-03-08 12:34:49 +0800'
categories: [draft]
tags: [nginx]
comments: []
---
不是site-available里面没加上location .\php$，这个问题很好解决。也不是因为没装fast-cgi，或者fast-cgi没启动。


遇到这个问题在网上搜了半天，最后还是stack-overflow帮了大忙http://stackoverflow.com/questions/20668886/nginx-and-fastcgi-downloads-php-files-instead-of-processing-them

里面指示了用户把default_type改成text/html，这是可以解决问题，不过这个方法有过暴力，拆东墙补西墙的感觉。其实主要是因为/etc/nginx/mime.types这个文件里面没有php类型的对应关系导致服务器不认识这个类型。加一句php类型的对应关系就可以解决问题了。

