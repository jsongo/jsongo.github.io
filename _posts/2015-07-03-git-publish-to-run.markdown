---
layout: post_item
status: publish
published: true
title: git代码发布方法
author: jsongo
date: '2015-07-03 00:23:35'
categories: [git]
tags: [git]
comments: []
---
现公司内部代码发布规范（简短版）

1. Tag版本号规范：  
所有的Tag版本号都分三部分，如v1.3.0，由点号分成三部分，第一部分v1表示大版本，功能上有非常大的变化才增加这个数字，如目前是给达人用的版本，之后要扩展成全面开放的、普通用户也能用的能用版本时，就是v2。第二部分x.3.x表示一次正式发布，目前这次是第二次正式发布，不过由于历史这里是3，请忽略这个问题。第三部分x.x.0表示一次正式发布里的小发布，比如修改一个严重的bug等，0为第一次。


2. 发布方法：  
[1] 每次发布必须用tag发布，不能用分支或master发布。  
[2] 打tag时，建议用-s参数签名，确保这个tag的真实可靠。  
[3] 打tag时，请用-a参数来标识它为Annotated。普通tag只是一个指向某个commit的指针，Annotated Tag 则会被当成是一次比较正规的单独的“版本”，可以用git show或git log看到。  
[4] 打tag时，请用-m参数来注明这个tag更新的主要功能。  

具体操作示例：  
{% highlight bash %}
git tag -s -a v1.3.1 -m '修改页面风格，增加当地活动、向导服务、交通服务，修复之前录入版本的两个交互问题'
{% endhighlight %}
其中，如果用-s签名遇到问题，请查阅：《[git对commit或tag签名](http://jsongo.github.io/post/git/2015/sign-a-git-commit/)》  

3. 服务器侧的发布操作  
目前git在服务器端配置了代理，可以拉取内网的代码进行发布，发布步骤：  
{% highlight bash %}
git checkout -b tag-v1.3.1 v1.3.1
{% endhighlight %}
上面的命令做的事情：拉取远程v1.3.1版本的Tag到服务器上，并在本地repo中新建分支 tag-v1.3.1，然后切换当前目录的代码到这个分支的代码上来。  

如果遇到以下问题：  

> fatal: 不能同时更新路径并切换到分支’tag-v1.3.1’。  
> 您是想要检出 ‘v1.3.1′ 但其未能解析为提交么？  

可以更新一下远程信息：
{% highlight bash %}
git remote update
{% endhighlight %}
如果还不行，更新不到最新tag，那就明文指定这个tag名，fetch下来：  
{% highlight bash %}
git fetch origin v1.3.1
git remote update
{% endhighlight %}
这样基本就可以解决这类问题。
