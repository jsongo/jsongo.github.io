---
layout: post_item
status: publish
published: true
title: select多个字段，但distinct一个字段的方法
author: jsongo
author_login: jsono
author_email: jsongo@qq.com
wordpress_id: 305
wordpress_url: http://jsongo.com/?p=305
date: '2014-05-09 15:30:25 +0800'
date_gmt: '2014-05-09 07:30:25 +0800'
categories: [draft]
tags: [mysql, error]
---
<p>比如select出a,b两个字段，但只想distinct a，方法：</p>
<p>select a,b,count(distinct a) from xxx group by a;</p>
<p>这样就可以达到目的。</p>
