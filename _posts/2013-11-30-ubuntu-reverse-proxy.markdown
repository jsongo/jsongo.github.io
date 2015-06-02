---
layout: post_item
status: publish
published: true
title: ubuntu上配置apache的反向代理
author: jsongo
wordpress_id: 116
wordpress_url: http://jsongo.com/?p=116
date: '2013-11-30 01:00:55 +0800'
date_gmt: '2013-11-29 17:00:55 +0800'
categories: [draft]
tags: [apache]
comments: []
---
把一个有个性端口号的服务转成一个二级域名，这个就是apache反向代理擅长干的事情。  
ubuntu上的配置比较麻烦一些。  


步骤：  
1、a2enmod proxy  
用这个命令把apache的反向代码功能给打开  
同时要确保mod_proxy 和mod_proxy_http都已开启，没有的话，用a2enmod加载一下。  

2、proxy开启后，配置一下proxy.conf  
{% highlight html %}  
<Proxy *>  
 AddDefaultCharset off  
 Order deny,allow  
 Allow from all  
 #Allow from .example.com  
</Proxy>  
{% endhighlight %}  

3、配置/etc/apache2/sites-available/下面相应的域名对应的文件，加入：  
{% highlight html %}  
<VirtualHost *:80>  
LoadModule proxy_module /usr/lib/apache2/modules/mod_proxy.so # 没有这个可能会报500错误  
ServerAdmin jsongo@jsongo.com  
ServerName xxx.jsongo.com  
ServerAlias xxx.jsongo.com  
# !!! Be sure to point DocumentRoot to 'public'!  
DocumentRoot /var/xxx/public/  
<Directory />  
# This relaxes Apache security settings.  
AllowOverride all  
# MultiViews must be turned off.  
Options -MultiViews  
Options -Indexes  
</Directory>  
ProxyPass / http://localhost:8008/  
ProxyPassReverse / http://localhost:8008/  
ProxyPreserveHost off  
</VirtualHost>  
{% endhighlight %}  
service apache2 restart  
DONE  
