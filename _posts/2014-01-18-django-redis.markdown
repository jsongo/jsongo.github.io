---
layout: post_item
status: publish
published: true
title: django+redis
author: jsongo
wordpress_id: 200
wordpress_url: http://jsongo.com/?p=200
date: '2014-01-18 01:39:59 +0800'
date_gmt: '2014-01-17 17:39:59 +0800'
categories: [draft, python]
tags: [python, redis]
comments: []
---
需要django-redis插件：https://django-redis.readthedocs.org/  
不过安装它之前得先安装redis-py：https://github.com/andymccurdy/redis-py/  
git clone下来后，运行python setup.py install 就ok了


装完redis-py后，要验证是否成功，可以运行一下下面几个命令：  
{% highlight python %}
>>> import redis 
>>> r = redis.StrictRedis(host='localhost', port=6379, db=0) 
>>> r.set('foo', 'bar') 
True 
>>> r.get('foo') 
'bar' 
{% endhighlight %}
接下去安装django-redis：https://github.com/niwibe/django-redis  
python setup.py install  
装完之后，修改一下设置：vi /usr/local/lib/python2.7/dist-packages/django/conf/global_settings.py  
{% highlight python %}
CACHES = { 
    "default": { 
        "BACKEND": "redis_cache.cache.RedisCache", 
        "LOCATION": "127.0.0.1:6379:1", 
        "OPTIONS": { 
            "CLIENT_CLASS": "redis_cache.client.DefaultClient", 
        } 
    } 
} 
{% endhighlight %}
如果是多个机器或多个端口，则用下面的配置： 
{% highlight python %}
CACHES = { 
    "default": { 
        "BACKEND": "redis_cache.cache.RedisCache", 
        "LOCATION": [ 
            "127.0.0.1:6379:1", 
            "127.0.0.1:6379:2", 
        ], 
        "OPTIONS": { 
            "CLIENT_CLASS": "redis_cache.client.ShardClient", 
        } 
    } 
} 
{% endhighlight %}
考虑到容灾的话，用下面配置（第一个失效了，自动切换到之后几个）： 
{% highlight python %}
CACHES = { 
    "default": { 
        "BACKEND": "redis_cache.cache.RedisCache", 
        "LOCATION": "127.0.0.1:6379:1/127.0.0.2:6379:1", 
        "OPTIONS": { 
            "CLIENT_CLASS": "redis_cache.client.SimpleFailoverClient", 
        } 
    } 
} 
{% endhighlight %}
