---
layout: post_item
status: publish
published: true
title: git对commit或tag签名
author: jsongo
date: '2015-05-10 23:19:21'
categories: [git]
tags: [git]
comments: []
---
高级玩法，一般不用签名，但如果你想装下逼格，加上你自己的签名，可以继续往下阅读。  
签名的好处是，你可以证明提交这个版本的确实是你本人提交的，而不是别人假借你的账户提交的代码。tag的版本也可以加上签名。打tag是一个很重要的行为，每个tag都表示它是一个可发布的版本，所以它很重要，不能随便打tag。  
这里用通用的gpg工具。  


先生成你的gpg密钥对，这个不在本文讨论范围，网上很多相关的介绍，我自己的PC上的重要文件，都被我用gpg加密了。  
{% highlight bash %}
gpg --list-secret-keys
{% endhighlight %}
可以列出你本地的所有的密钥，找到你要用来加密的那个，看sec开头的那行  
然后设置下全局提交用的签名密钥  
{% highlight bash %}
git config --global user.signingkey F325ECBE  
{% endhighlight %}
现在你可以用这个密钥来签名，并提交了。下面以打tag为例：  
{% highlight bash %}
git tag -a -s v1.0 -m '第一个对外发布的版本'  
{% endhighlight %}
用git show来查看版本，可以看到你的签名信息  

最后，可以验证这个标签的签名是谁的：
{% highlight bash %}
git tag -v v1.0
{% endhighlight %}
提交这个标签  
{% highlight bash %}
git push origin v1.0
{% endhighlight %}
