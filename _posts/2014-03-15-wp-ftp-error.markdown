---
layout: post_item
status: publish
published: true
title: wp中出现插件安装提示ftp登录的问题排查（nginx）
author: jsongo
wordpress_id: 254
wordpress_url: http://jsongo.com/?p=254
date: '2014-03-15 01:52:54 +0800'
date_gmt: '2014-03-14 17:52:54 +0800'
categories: [draft]
tags: [wordpress, error, nginx]
---
安装、卸载插件时，就直接提示FTP登录，那你不用想了，就是权限的问题。

首先看word-press目录及里面的所有文件的owner是不是你配置的那个owner。nginx的这个配置在/etc/nginx/nginx.conf里面，第一行就是要运行nginx的用户，这个用户必须跟你的wp目录的所有者一致。不一样就修改，可以用chown修改目录权限，也可以直接修改nginx的这个配置，然后重启nginx就可以。如果这两个地方一致了（比如都改成jsongo用户），你就可以再去试下。


还不行，那继续。看fast-cgi的运行者有没有问题。

ps aux \| grep php-fpm

当然，如果你用的fast-cgi是*spawn-fcgi*那就grep这个。建议将spawn-fcgi换成php5-fpm，因为后者是作为php的一个插件来开发的，与php结合更紧密，性能比前者优越些，特别是在处理高并发的时候。

如果上面抓到的php-fpm进程不是上面的jsongo用户在运行的话，那么，修改/etc/php5/fpm/pool.d/www.conf 这个文件中的user和group配置，改成jsongo，然后service php5-fpm restart。

再试试 ，这次应该OK了，再不ok，那你真没救了。

