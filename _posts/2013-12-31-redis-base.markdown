---
layout: post_item
status: publish
published: true
title: redis简单操作
author: jsongo
wordpress_id: 187
wordpress_url: http://jsongo.com/?p=187
date: '2013-12-31 01:58:37 +0800'
date_gmt: '2013-12-30 17:58:37 +0800'
categories: [draft]
tags: [redis]
comments: []
---
Redis的命令可以在[http://redis.io/commands](http://redis.io/commands)这里面查  
一、string  
1、set name json   把name这个key的值设置成'json'  
2、get name 取值  
  del name  删除


3、setnx  在不存在的时候设置值set not exist  
4、setex 设置值的同时设置有效期  
  如setex name 5 json，设置name值为json，有效时间为5秒  
5、setrange name 4 go  从第4个字符开始设置（替换）成后面的字符串：jsongo  
6、mset name1 jsongo name2 kitty   指设置  
  同样也有msetnx，msetex  
  和mget  
7、getrange name 0 6   取name值，从下标0取到6  
8、incr num  把num的值增加1  
  如果num不存在，则把它设成1  
9、incrby num 6     把num增加6  
  同样也有decr和decrby  
10、append name @jsongo.com      追加  
11、strlen  字符长度  

二、hash  
1、hset user:001 name jsongo    设置一个user对象，key 为001, name属性为jsongo  
2、hget user:001 name  
3、同样也有hsetnx  
  hmset user:002 name jsongo age 10 sex 1  
4、hincrby user:002 age 3    把user:002的age属性加3  
5、hlen user:002    返回user:002对象的属性个数  
6、hdel user:002 age   删除对象的age属性  
7、hkeys user:002    遍历对象的所有属性  
  当然也有hvals   用来得到对象的所有属性值  
8、hgetall  取得所有的key和value，一起列出来  

三、bit操作  
1、bitcount 计算一个key的bit长度  
2、bitset mykey 7 1      把mykey的第8位设置成1（注意这里的位置是从左算到右的）  
3、bitop or b a "\x06"     对a和\x06进行or操作，结果存到b里。除了or，还有and ,xor, not  

四、list操作  
1、lpush mylist "abc"   往mylist里推入abc，从下往上，类似stack  
2、lrange mylist 1 -1    从下标为1的取到最后一个（从上往下取的）  
3、rpush mylist "json"   往mylist里从上面插入  
4、lset mylist 1 "hello"   把mylist里面的第2个元素改成hello，从上往下算的  
5、linsert mylist before abc new_value    往mylist里abc的上面插入new_value  