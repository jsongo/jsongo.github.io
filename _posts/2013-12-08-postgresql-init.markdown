---
layout: post_item
status: publish
published: true
title: postgresql刚安装完登录不上的问题
author: jsongo
wordpress_id: 168
wordpress_url: http://jsongo.com/?p=168
date: '2013-12-08 03:17:30 +0800'
date_gmt: '2013-12-07 19:17:30 +0800'
categories: [draft]
tags: []
comments: []
---
因为没有密码，postgresql不让登录，所以要先给postgresql的默认账户postgres设置一个密码。  
先修改/etc/postgresql/9.1/main/pg_hba.conf，加入  


> local all  all trust

这一行，然后重启一下postgresql：  
/etc/init.d/postgresql restart  
再以postgres的身份来运行psql  
sudo -u postgres psql  
这样就可以进去了。  
设置密码，在psql命令行内执行：  

> alter user postgres with encrypted password \'你的密码\'; 

就可以把postgres的密码改成你的密码了  
再回去把pg_hba.conf里面刚刚多加的那行去掉，加入（原来有修改就行）：  

> local all all md5

重启postgresql就ok了  
