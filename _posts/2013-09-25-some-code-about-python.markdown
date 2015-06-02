---
layout: post_item
status: publish
published: true
title: python的几个语法收集
author: jsongo
wordpress_id: 55
wordpress_url: http://jsongo.com/?p=55
date: '2013-09-25 01:50:43 +0800'
date_gmt: '2013-09-24 17:50:43 +0800'
categories: [python]
tags: [python, masterpiece]
comments: []
---
1、map的用法：  
{% highlight python %}  
map(lambda item:item*2, [1,2,3])  
{% endhighlight %}  


2、拷贝  
{% highlight python %}  
from copy import copy<br />
x=[1,2,3,4]<br />
y=copy(x)<br />
x.append(5)  
{% endhighlight %}  
但这个只是浅拷贝，y的结果没变，也就是说x的改变不会影响到y了。所以，如果x里面的元素还是一个集合的话，那就得用深拷贝了：  
{% highlight python %}  
from copy import deepcopy<br />
x=[1, {'a': 11}, 3]<br />
y=deepcopy(x)<br />
x[1]['a']=111  
{% endhighlight %}  
