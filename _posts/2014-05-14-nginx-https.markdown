---
layout: post_item
status: publish
published: true
title: nginx的https设置
author: jsongo
author_email: jsongo@qq.com
wordpress_id: 315
wordpress_url: http://jsongo.com/?p=315
date: '2014-05-14 01:22:14 +0800'
date_gmt: '2014-05-13 17:22:14 +0800'
categories: [security]
tags: [https]
comments: []
---
对于一些信息比较敏感的系统，我们可以考虑牺牲一下访问速度来保证它的安全性。https访问时，数据是加密的，在ssl通道中传输，别人就算抓了包也看不出什么信息。当然前段时间爆出的openssl漏洞另当别论，这个不是https协议的问题，是openssl的问题，不在本文的讨论范围内。


我的服务器上装了nginx 1.7.0的版本，openssl的heartbleed漏洞已经修复。（其它版本的话，1.5.12+, 1.4.7+，这些以上不受影响）我的nginx是我下的最新的源码来编译的。（可以在[这里](http://wangzhan.360.cn/heartbleed/)验证是否有这漏洞）
在nginx中设置https访问，分两步：

一、生成证书且签名

生成公钥
{% highlight bash %}
openssl genrsa -des3 -out server.pkey 1024
{% endhighlight %}
干掉密码，要不然每次重启服务都要重输密码：
{% highlight bash %}
openssl rsa -in server.pkey -out server.key
{% endhighlight %}
生成证书请求文件，&ldquo;也就是证书申请者在申请数字证书时由CSP(加密服务提供者)在生成私钥的同时也生成证书请求文件，证书申请者只要把CSR文件提交给证书颁发机构后，证书颁发机构使用其根证书私钥签名就生成了证书公钥文件，也就是颁发给用户的证书。&rdquo;（来自[CSR_百度百科](http://baike.baidu.com/view/1008928.htm#7)）
这一步要输入很多证书相关的信息。
{% highlight bash %}
openssl req -new -key server.key -out server.csr
{% endhighlight %}
最后用证书请求文件来给我们的证书签名：
{% highlight bash %}
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
{% endhighlight %}
到这里，我们已经生成了我们所需要的两个文件：server.crt server.key，证书和密钥。

可以新建一个/etc/ssl/nginx文件夹，把上面两个文件拷进去，修改所有者和读写属性：
{% highlight bash %}
sudo chown jsongo:jsongo server.crt server.key
sudo chmod 640 jsongo:jsongo server.key
sudo chmod 710 /etc/ssl/nginx
sudo chown jsongo:jsongo /etc/ssl/nginx
{% endhighlight %}

二、修改nginx的配置：
在server {} 外面加上：
{% highlight nginx %}
upstream webserver {
    server 127.0.0.1:8069 weight=1 fail_timeout=300s;
}
{% endhighlight %}
然后在server {}里加上配置https
{% highlight nginx linenos %}
server {
    # server port and name
    listen        443 default;
    server_name   oa.jsongo.com;
    # Specifies the maximum accepted body size of a client request,
    # as indicated by the request header Content-Length.
    client_max_body_size 200m;
    # ssl log files
    access_log    /etc/nginx/logs/openerp-access.log;
    error_log    /etc/nginx/logs/openerp-error.log;
    # ssl certificate files
    ssl on;
    ssl_certificate        /etc/ssl/nginx/server.crt;
    ssl_certificate_key    /etc/ssl/nginx/server.key;
    # add ssl specific settings
    keepalive_timeout    60;
    # limit ciphers
    ssl_ciphers            HIGH:!ADH:!MD5;
    ssl_protocols            SSLv3 TLSv1;
    ssl_prefer_server_ciphers    on;
    # increase proxy buffer to handle some OpenERP web requests
    proxy_buffers 16 64k;
    proxy_buffer_size 128k;
    location / {
        proxy_pass    http://webserver;
        # force timeouts if the backend dies
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;
        # set headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forward-For $proxy_add_x_forwarded_for;
        # Let the OpenERP web service know that we're using HTTPS, otherwise
        # it will generate URL using http:// and not https://
        proxy_set_header X-Forwarded-Proto https;
        # by default, do not forward anything
        proxy_redirect off;
    }
    # cache some static data in memory for 60mins.
    # under heavy load this should relieve stress on the OpenERP web interface a bit.
    location ~* /web/static/ {
        proxy_cache_valid 200 60m;
        proxy_buffering    on;
        expires 864000;
        proxy_pass http://webserver;
    }
}
{% endhighlight %}
最后，把所有的http请求都自动转向到https来
{% highlight nginx linenos %}
server {
    listen 80;
    server_name oa.jsongo.com;

    index index.html index.htm index.php;

    # open_log_file_cache max=1000 inactive=60s;

    add_header Strict-Transport-Security max-age=2592000;

    rewrite ^/.*$ https://$host$request_uri? permanent;

    location ~ .*\.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$ {
        expires      30d;
    }
    location ~ .*\.(js|css)?$ {
        expires      1h;
    }

}
{% endhighlight %}

从这篇文章中学来的：
[《Reverse SSL Proxy using NGINX with OpenERP v7 | Ubuntu 12.04 LTS》](http://www.schenkels.nl/2013/01/reverse-ssl-proxy-using-nginx-with-openerp-v7/)
