---
layout: post_item
status: publish
published: true
title: PHP生成二维码（phpQRcode）
author: jsongo
wordpress_id: 127
wordpress_url: http://jsongo.com/?p=127
date: '2013-12-02 01:09:30 +0800'
date_gmt: '2013-12-01 17:09:30 +0800'
categories: [draft]
tags: [php]
comments: []
---
不用google的开放接口来生成，可以用php-QRcode  
地址：[http://phpqrcode.sourceforge.net](http://phpqrcode.sourceforge.net/)  
网站提供了一些demo，最简单的一个就是先引入qrlib.php，然后加上：  


{% highlight php %}  
QRcode::png('Hello, this is jsongo.');  
{% endhighlight %}  
更繁杂的示例可以参考：[http://phpqrcode.sourceforge.net/examples/index.php](http://phpqrcode.sourceforge.net/examples/index.php) 
可以做出各种各样形形色色、五花八门的二维码，功能很强大。  
