---
layout: post_item
status: publish
published: true
title: git全新版本代码如何提交（不覆盖原代码）
author: jsongo
date: '2015-06-01 13:01:14'
categories: [draft]
tags: [git]
comments: []
---
有时会遇到这种情况，在做一个项目时，要改版，发现原来的代码不能用了，全新的搭了一套框架，重新开始写，这时的代码跟原来的就完全不一样了，提交的话，会直接把原来的代码覆盖掉，虽然从历史版本中可以找回，但也很麻烦。


最好的办法就是，把原代码移到branch上，再把最新的代码提交到master

操作：

到原代码的目录里，执行
{% highlight bash %}
git push origin master:version-1
{% endhighlight %}
这样就把原master的代码复制了一份到分支versin-1去了，这样你就可以放心地覆盖原代码了。

删除master的代码，不过不能用下面这种暴力的方式
{% highlight bash %}
git push origin :master
{% endhighlight %}
因为它是要直接删除master分支，是不会执行成功的，会报错，除非你到gitlab或github的项目管理处把默认的分支从master切换成其它的分支。

正确的做法是，在原目录执行上面操作，删除掉原master的代码 ，然后提交上去：
{% highlight bash %}
git add --all
git commit -m 'delete old version'
git push origin master
{% endhighlight %}

这样就可以删除掉原master里的代码，为你提交新代码做准备。

到新代码目录执行
{% highlight bash %}
git init
git remote add origin http://xxx/xxx.git
git pull origin master
git add --all
git commit -m 'commit new version'
git push origin master
{% endhighlight %}
这样你就可以完成整个代码分支的迁移工作
