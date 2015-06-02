---
layout: post_item
status: publish
published: true
title: mac上制作linux启动安装盘
author: jsongo
wordpress_id: 239
wordpress_url: http://jsongo.com/?p=239
date: '2014-02-19 23:23:28 +0800'
date_gmt: '2014-02-19 15:23:28 +0800'
categories: [draft]
tags: [mac]
comments: []
---

在mac上制作linux启动盘很简单，一个命令就可以了。
{% highlight bash %}
sudo dd if=/Volumes/jsongo/kali-linux-1.0.5-amd64.iso of=/dev/disk2 bs=1m
{% endhighlight %}
其中if=后面的参数是你的linux镜像文件的位置，of是你的盘，/dev/目录下，至于怎么确定是disk几，可以用下面的命令查看：

diskutil list
在列出的磁盘信息中，就可以找到你的u盘了。
