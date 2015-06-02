---
layout: post_item
status: publish
published: true
title: apt-get 出现 "subprocess installed post-installation script returned error exit status 1"
author: jsongo
wordpress_id: 111
wordpress_url: http://jsongo.com/?p=111
date: '2013-11-29 02:24:13 +0800'
date_gmt: '2013-11-28 18:24:13 +0800'
categories: [draft]
tags: []
comments: []
---
cd /var/lib/dpkg/info  
把相应的报错的包都干掉就ok了  
再不行就clean，升级  
apt-get autoclean  
apt-get autoremove  
apt-get update  
apt-get upgrade  
DONE  
