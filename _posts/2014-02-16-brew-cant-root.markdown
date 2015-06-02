---
layout: post_item
status: publish
published: true
title: brew 报错，无法用root执行
author: jsongo
wordpress_id: 229
wordpress_url: http://jsongo.com/?p=229
date: '2014-02-16 20:30:39 +0800'
date_gmt: '2014-02-16 12:30:39 +0800'
categories: [draft]
tags: []
comments: []
---
mac下brew install 报错

Password:
Error: Cowardly refusing to `sudo brew install'
You can use brew with sudo, but only if the brew executable is owned by root.
However, this is both not recommended and completely unsupported so do so at
your own risk.

原因是这个brew的权限不正确
修改一下这个brew的权限chown root:wheel /usr/local/bin/brew
