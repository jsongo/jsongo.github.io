---
layout: post_item
status: publish
published: true
title: nginx把资源存入redis访问
author: jsongo
wordpress_id: 269
wordpress_url: http://jsongo.com/?p=269
date: '2014-03-16 02:04:09 +0800'
date_gmt: '2014-03-15 18:04:09 +0800'
categories: [draft]
tags: [nginx, redis]
comments: []
---

一些特别常用的资源，每次访问都得去读IO，量小还好，如果访问量超大的时候，IO估计会成为瓶颈。如果把资源内容放Redis里，配置nginx来访问，同时还可以gzip。


有两个模块都可以用：

1、HttpRedis

可以到https://github.com/HoopCHINA/ngx_http_redis 这上面去下载，然后重新编译nginx，./configure时带上"--add-module=你的目录/ngx_http_redis " ，编译完替换掉/usr/sbin/nginx 重启。

nginx相应网站文件配置（默认default）：

{% highlight nginx %}
location /redis {
  gzip on;
  redis_gzip_flag 1;
  set $redis_key $uri;
  redis_pass 127.0.0.1:6379;
  default_type text/html;
  error_page 404 = @fetch;
}
location @fetch {
  include /etc/nginx/conf/fastcgi_params;
  default_type text/html;
  root /var/www/res;
  try_files $uri $uri/ =404;
}
{% endhighlight %}

把/redis这个路径配置成用来取资源文件，如果遇到redis里面没有的，就到/var/www/res/redis/下面去找相应的文件，如果再找不到就返回 404。

测试。

往redis里插入数据：set "/redis/hello" "Hello, World"

然后用http://jsongo.com/redis/hello来访问就可以取出相应的内容来。


2、HttpRedis2Module

HttpRedis的功能十分简单，如果你需要更强大的就用HttpRedis2Module，它可以和redis更紧密结合，充分使用redis的大部分功能。

可以在这里下载https://github.com/agentzh/redis2-nginx-module，同样的也要重新编译，带上--add-module=... ，然后使用新nginx重启。

或者可以下载安装OpenResty ，地址http://openresty.org/，它里面集成了很多第三方模块，使nginx成为一个更加强大的web容器。

HttpRedis2Module的配置和使用可以参考这里：https://github.com/agentzh/redis2-nginx-module#synopsis，功能特别强大
