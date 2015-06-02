---
layout: post_item
status: publish
published: true
title: "入侵检测(IDS)和snort"
author: jsongo
wordpress_id: 160
wordpress_url: http://jsongo.com/?p=160
date: '2013-12-05 01:02:43 +0800'
date_gmt: '2013-12-04 17:02:43 +0800'
categories: [security]
tags: [security]
comments: []
---
&nbsp;&nbsp;IDS，instrution detection system，入侵检测系统在如今越来越受关注，做一个站点，即使你很小心，做了很多安全防护，但难免还是会受到攻击。被攻击了还不知道别人是怎么得逞的，不知道今后怎么去防御，这就很悲哀。  
&nbsp;&nbsp;网络入侵检测系统NIDS，在web站点的安全防护中是缺一不可的环节。snort就是其中之一，它同时还有一定程度上的防御功能，也起到了IPS(instrution protection system)的作用。用snort来保护站点，分析网络的实时情况。  


&nbsp;&nbsp;之前一篇文章《[snorby安装](/post/security/2013/snorby-ubuntu/)》讲过snorby，它为snort提供更加直观的前端展示。一般的教程是结合snort和BASE来做，BASE，即Basic Analysis and Security Engine ，是snort老牌的前端展示工具，不过感觉界面有点丑，不知道功能和snorby相比如何，没用过，不过光从体验上和用户友好度来说，snorby也是个很不错的选择。  
安装：  
这里有snorby官网的简单安装教程：https://github.com/Snorby/snorby/wiki/Installing-Snort  
这里有篇文章讲得比较详细：http://www.aboutdebian.com/snort.htm  
其实在ubuntu上的安装很简单：apt-get install snort，搞定  
再编辑一下/etc/snort/snort.conf，修改一下数据库连接：  

> output database: alert, mysql, user=root password=password dbname=snorby host=localhost

数据库和snorby同用一个，但snorby和snort建的表在各自的安装过程中都会冲突（目前的最新版本是这样），虽如此，还是有办法解决的，用snorby的安装过程来建表，然后mysqldump导出备份，再清空所有的表。然后按下面的过程安装snort-mysql模块建表，完成后，再把之前备份的表再次导入就可以了。  
**snort-mysql模块的安装配置：**  
用apt-get来安装snort时，是没有把mysql编译进去的，这时需要安装snort-mysql模块，然后重新配置。  
apt-get install snort-mysql  
安装完后，要运行dpkg-reconfigure配置一下，但运行后会报错，错误信息会指引你先去做如下操作：  
1、cd /usr/share/doc/snort-mysql，  
zcat create_mysql.gz | mysql -u用户名 -p 数据库名  
2、rm /etc/snort/db-pending-config  
这两个步骤完成后，就可以运行：dpkg-reconfigure snort-mysql来重新配置snort-mysql。  
至此snort就可以正常工作了，如果start出现错误，《[Setting Up A Snort IDS on Debian Linux](http://www.aboutdebian.com/snort.htm)》这篇文章的下半部分会引导你解决网卡的问题。  

**结合barnyard2的配置：**

> output unified2: filename snort.out, limit 128

重启一下snort服务就ok了  
如果要重新详细地配置snort，运行：dpkg-reconfigure snort  
