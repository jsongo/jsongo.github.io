---
layout: post_item
status: publish
published: true
title: postgressql学习
author: jsongo
wordpress_id: 173
wordpress_url: http://jsongo.com/?p=173
date: '2013-12-11 03:16:24 +0800'
date_gmt: '2013-12-10 19:16:24 +0800'
categories: [draft]
tags: [sql]
comments: []
---
psql连接上去，后面可以跟库名  
\\l 可以列出所有的数据库（list）  
\\c 可以切换当前数据库  
\\d 列出当前所有的表和序列  
\\dt 列表当前所有的表  
\\d table_name 可以查看这个表的所有字段详细  


select pg_column_size('asadfasdf'); 可以查看存储某个元素值所需的字节数  
select count(*) from base_language_export; 和mysql一样，查行数  
select pg_database_size('adsds'); 查询adsds这个数据库的总大小  
select pg_size_pretty(pg_database_size('adsds')); 整洁地打印出来  
\\q 退出  
