---
layout: post_item
status: publish
published: true
title: "uwsgi报错libgcc_s.so.1 must be installed for pthread_cancel to work"
author: jsongo
author_login: jsono
author_email: jsongo@qq.com
wordpress_id: 301
wordpress_url: http://jsongo.com/?p=301
date: '2014-04-27 02:05:16 +0800'
date_gmt: '2014-04-26 18:05:16 +0800'
categories: [draft]
tags: [error]
comments: []
---
<p>在uwsgi上跑django时，如果操作了数据库，就有可能会导致标题中说的错误。</p>
<p>其实很简单，把--limit-as 改大一些就可以了，如1000</p>
