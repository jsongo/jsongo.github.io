---
layout: post_item
status: publish
published: true
title: "在ubuntu apache2上添加模块"
author: jsongo
wordpress_id: 113
wordpress_url: http://jsongo.com/?p=113
date: '2013-11-29 03:11:47 +0800'
date_gmt: '2013-11-28 19:11:47 +0800'
categories: [draft]
tags: [linux]
comments: []
---
在Ubuntu上，apache2的配置目录结构跟以往不太一样，变化很大，要加个自定义的模块很不容易。  
自己测试了下，总结了几个步骤：  
比如以前要配置passenger（让ruby on rails能在apache上运行的模块）  


> LoadModule passenger_module /var/lib/gems/1.9.1/gems/passenger-4.0.26/buildout/apache2/>mod_passenger.so    
> &nbsp;&nbsp;PassengerRoot /var/lib/gems/1.9.1/gems/passenger-4.0.26    
> &nbsp;&nbsp;PassengerDefaultRuby /usr/bin/ruby1.9.1

这个在ubuntu apache2上配置如下：    
先到/etc/apache2/mods-available下新建两个文件：    
vi passenger.load    
写入    

> LoadModule passenger_module /var/lib/gems/1.9.1/gems/passenger-4.0.26/buildout/apache2/mod_passenger.so  

vi passenger.conf    
写入    

> PassengerRoot /var/lib/gems/1.9.1/gems/passenger-4.0.26  
> PassengerDefaultRuby /usr/bin/ruby1.9.1  

然后再运行：a2enmod passenger    
最后 service apache2 restart 重启一下就可以了    
    
