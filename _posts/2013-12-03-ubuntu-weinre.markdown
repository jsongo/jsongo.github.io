---
layout: post_item
status: publish
published: true
title: ubuntu上配置weinre，远程调试页面脚本
author: jsongo
wordpress_id: 151
wordpress_url: http://jsongo.com/?p=151
date: '2013-12-03 01:49:20 +0800'
date_gmt: '2013-12-02 17:49:20 +0800'
categories: [draft]
tags: [linux, js]
comments: []
---
先安装nodejs  
{% highlight bash %}
apt-get install nodejs  
{% endhighlight %}


安装npm（nodejs package management） 
{% highlight bash %} 
apt-get install npm  
{% endhighlight %}
再安装weinre  
{% highlight bash %} 
sudo npm -g install weinre  
{% endhighlight %}
至此，运行：node weinre  就可以直接把weinre服务打开了，默认绑定了8080端口，可以指定：  
weinre --httpPort 8081 --boundHost -all-  
后面加上boundHost 参数是因为不加的话，外网访问不到  
如果node weinre报错，没有找到命令，那就换成nodejs，或者cp一下nodejs成node：
{% highlight bash %}   
cp /usr/bin/nodejs /usr/bin/node  
{% endhighlight %}
DONE  
