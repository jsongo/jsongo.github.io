---
layout: post_item
status: publish
published: true
title: snorby安装
author: jsongo
wordpress_id: 158
wordpress_url: http://jsongo.com/?p=158
date: '2013-12-04 03:31:29 +0800'
date_gmt: '2013-12-03 19:31:29 +0800'
categories: [security]
tags: [linux, security]
comments: []
---
Snorby是一个Ruby on Rails的Web应用程序，网络安全监控与目前流行的入侵检测系统（Snort的项目Suricata和Sagan）的接口。该项目的目标是创建一个免费的，开源和竞争力的网络监控应用，为私人和企业使用。  
Snorby的首页有关于snorby安装的教程：https://snorby.org/


{% highlight bash %}
git clone http://github.com/Snorby/snorby.git
{% endhighlight %}
git到服务器，进入，再bundle install  
当然要先安装bundle等  
如果报错：Gem files will remain installed in /var/lib/gems/1.9.1/gems/nokogiri-1.5.9 for inspection. Results logged to /var/lib/gems/1.9.1/gems/nokogiri-1.5.9/ext/nokogiri/gem_make.out  
那么安装一下下面几个包：  
{% highlight bash %}
apt-get install ruby1.8-dev ruby1.8 ri1.8 rdoc1.8 irb1.8  
apt-get install libreadline-ruby1.8 libruby1.8 libopenssl-ruby  
apt-get install libxslt-dev libxml2-dev  
gem install nokogiri
{% endhighlight %}
[这里](http://stackoverflow.com/questions/16028028/nokogiri-will-not-install-error-failed-to-build-gem-native-extension)有说明。  
进入config目录，cp一份database.yml，修改一下里面的数据库连接参数，然后就可以安装了  
{% highlight bash %}
bundle exec rake snorby:setup
{% endhighlight %}
装完后运行：  
{% highlight bash %}
bundle exec rails server -e production
{% endhighlight %}
可以用-p指定端口号，默认是3000，直接用ip访问：http://xxx.xxx.xxx.xxx:3000  
建议都用production模式运行：  
{% highlight bash %}
ruby script/delayed_job start RAILS_ENV=production
{% endhighlight %}
进入后，默认的登录用户名：snorby@snorby.org，密码：snorby  
进入后记得先修改密码！  
DONE  
有几篇文章可以参考：http://drops.wooyun.org/papers/653  
