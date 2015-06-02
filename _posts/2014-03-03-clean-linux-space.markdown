---
layout: post_item
status: publish
published: true
title: "清理linux服务器上的磁盘空间"
author: jsongo
wordpress_id: 241
wordpress_url: http://jsongo.com/?p=241
date: '2014-03-03 23:35:11 +0800'
date_gmt: '2014-03-03 15:35:11 +0800'
categories: [draft]
tags: [linux]
comments: []
---
今天服务器反应异常地慢，有些进程甚至挂了。各种找原因，最后发现是磁盘满了，很多IO操作都被阻塞了。


必须得清理磁盘。linux上清理磁盘空间不像windows那样，装个什么什么管家/卫士就可以一键搞定，而且服务器还是远程的，只能命令操作，麻烦得很。临时学习了几个操作，总结一下：

{% highlight bash %}
df -i  /home
{% endhighlight %}

*查看inode数量*，越大说明大文件越多。因为每个inode节点的大小一般是128字节或256字节。inode节点的总数，在格式化时就给定，一般是每1KB或每2KB就设置一个inode。假定在一块1GB的硬盘中，每个inode节点的大小为128字节，每1KB就设置一个inode，那么inode table的大小就会达到128MB，占整块硬盘的12.8%。（摘自http://www.ruanyifeng.com/blog/2011/12/inode.html）

*接下去就要查找大文件在哪*，并把它干掉或压缩。

{% highlight bash %}
du -hs /home
{% endhighlight %}

查看home目录大小，如果比较小，再看看其它地方，如：/var 、/www、/usr等等，只要把上面的home换成你想看的目录，再一层一层往里找，直到定位到比较大的而且你认为没怎么用的文件，清除掉就可以省出很多空间。

如果要查看的目录里文件太多，可以按条件找，如当前目录及其子目录下查找大于100M的文件：

{% highlight bash %}
du | awk '$1>100000 {print $2}'
{% endhighlight %}

这样就可以把当前目录及其子目录中的大于100M的文件都抓出来。

文件太多的时候，要一眼看出来还真不容易，换算成兆M，再从大到小排个序：

{% highlight bash %}
du | awk '$1>100000 {print $1/1000,$2}' | sort -nr
{% endhighlight %}

这样从上到下从小到大排列，一目了然。

 

