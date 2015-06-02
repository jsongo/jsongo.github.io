---
layout: post_item
status: publish
published: true
title: OpenERP安装配置
author: jsongo
wordpress_id: 165
wordpress_url: http://jsongo.com/?p=165
date: '2013-12-08 02:50:01 +0800'
date_gmt: '2013-12-07 18:50:01 +0800'
categories: [draft]
tags: [openerp]
comments: []
---
先装postgresql:  
ubuntu上安装奶简单，sudo apt-get install postgresql，其它平台的怎么装就不说了。  
安装openerp时，先加下面的源（/etc/apt/sources.list）  

> deb http://nightly.openerp.com/7.0/nightly/deb/ ./  

然后更新下：  

> apt-get update  

安装：  

> apt-get install openerp  

如果这种方法不行，下载源码，直接运行sudo python setup.py install  
这样也可以直接安装。  
＝＝＝＝＝＝＝＝＝＝＝＝＝  
用postgresql账号运行来创建你自己的postgres账户：  

> sudo -u postgres -s createuser my_name&nbsp;-P  

然后再创建一个数据库  

> sudo -u postgres -s createdb mydb -O&nbsp;my_name  

上面用setup.py install安装之后，在install目前里面有一个openerp_server.conf，把它拷贝到/etc/openerp/目录下，没有这目录就建一个。  
用apt-get 安装的话，如果在系统里找不到这个conf文件，可以用下面这个：  
{% highlight nginx %}  
[option]  
; This is the password that allows database operations:  
; admin_passwd = admin  
db_host = False  
db_port = False  
db_user = False  
db_password = False  
logfile = /etc/openerp/openerp-server.log  
{% endhighlight %}  
然后运行openerp的服务端：  

> openerp-server -c /etc/openerp/openerp-server.conf &amp;  
