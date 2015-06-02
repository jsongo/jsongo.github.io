---
layout: post_item
status: publish
published: true
title: "通过apache访问Ruby on Rails服务（redmine）"
author: jsongo
wordpress_id: 119
wordpress_url: http://jsongo.com/?p=119
date: '2013-11-30 01:42:27 +0800'
date_gmt: '2013-11-29 17:42:27 +0800'
categories: [draft]
tags: [linux]
comments: []
---
本文拿redmine来举例  
1、用wget到http://www.redmine.org/projects/redmine/wiki/Download下载最新的redmine  
解压到某个目录，如/var/redmine  
其实这个里面就有安装说明：INSTALL这个文件，说明特别详细


2、安装其它的一些工具  
mysql  
rubygems（现在装ruby都自带安装了）  
bundler（方便指安装一些包，安装：gem install bundler） 

3、在mysql中创建一个数据库，db_redmine（名字任取）  
最好不要用root用户，可以新建一个用户，然后把这个库的权限都给它  
{% highlight bash %}
grant all privileges on db_redmine.* to jsongo;
{% endhighlight %} 

4、安装一些相关的依赖库  
{% highlight bash %}
bundle install --without development test postgresql sqlite rmagick 
{% endhighlight %} 
其中without表示不安装后面的几个模块，其它的都装  

5、进redmine目前修改配置  
{% highlight bash %}
cd config  
cp database.yml.example database.yml  
{% endhighlight %} 
修改database.yml中的production部分：  
{% highlight bash %}
production:  
  adapter: mysql  
  database: db_redmine  
  host: localhost  
  username: jsongo  
  password: xxx  
  encoding: utf8
{% endhighlight %}
继续修改configuration.yml，没有就从.example后缀名的文件复制过来  
修改production部分，可以配置相应的smtp服务器，比如smtp.qq.com等等，不改也行  

6、修改redmine中几个目录的权限（非必须，看情况而定）  
{% highlight bash %}
sudo chown -R redmine:redmine files log tmp public/plugin_assets  
sudo chmod -R 755 files log tmp public/plugin_assets
{% endhighlight %}

7、Generate a session store secret  
redmine把session信息都存在cookie里，此时需要生成一个密钥来保证它的安全  
{% highlight bash %}
Generate a session store secret
{% endhighlight %}
这个命令在redmine根目录下执行，它会生成一个config/initializers/secret_token.rb  

8、生成初始数据库里的默认表：  
{% highlight bash %}
rake db:migrate RAILS_ENV="production"
{% endhighlight %}
或  
{% highlight bash %}
RAILS_ENV=production rake db:migrate
{% endhighlight %}
插入缺少数据  
{% highlight bash %}
RAILS_ENV=production rake redmine:load_default_data
{% endhighlight %}

9、启动redmine  
{% highlight bash %}
ruby script/rails server -e production
{% endhighlight %}
默认会在3000端口打开  
（不过我测试时，在命令行可以wget访问到页面，但用外网ip访问不到，所以就有下面的配置）  
###########################  
上面就算装完redmine了  
接下去和apache2整合  
1、先装passenger  
{% highlight bash %}
gem install passenger  
passenger-install-apache2-module
{% endhighlight %}
2、在apache2中，加入passenger模块，详见我的另一篇文章：  
[《在ubuntu apache2上添加模块》](/post/draft/2013/ubuntu-apache2-add-module/)

3、通过apache的反向代理隐藏端口号  
详见另一文：[《ubuntu上配置apache的反向代理》](http://jsongo.com/2013/11/ubuntu-reverse-proxy/) 
&nbsp;  
参考：http://www.cnblogs.com/baizhantang/archive/2012/12/20/2827061.html  
