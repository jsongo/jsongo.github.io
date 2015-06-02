---
layout: post_item
status: publish
published: true
title: 'ImportError: No module named MySQLdb（python）'
author: jsongo
wordpress_id: 176
wordpress_url: http://jsongo.com/?p=176
date: '2013-12-13 03:10:39 +0800'
date_gmt: '2013-12-12 19:10:39 +0800'
categories: [draft]
tags: [python]
comments: []
---
python连接数据库时，报错：  


> import MySQLdb  
> ImportError: No module named MySQLdb

没有安装python的mysql模块，ubuntu上运行：  

> apt-get install python-mysqldb  

其它系统：  
easy_install mysql-python (mix os)  
pip install mysql-python (mix os)  
apt-get install python-mysqldb (Linux Ubuntu, ...)  
cd /usr/ports/databases/py-MySQLdb &amp;&amp; make install clean (FreeBSD)  
yum install MySQL-python (Linux Fedora, CentOS ...)  
