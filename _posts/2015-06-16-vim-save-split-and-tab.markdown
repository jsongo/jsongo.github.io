---
layout: post_item
status: publish
published: true
title: How to save the split view of vim
author: jsongo
date: '2015-06-16 22:57:02'
categories: [draft]
tags: [vi, vim]
comments: []
---
I'd like to use vim with several windows opening in split mode, as well as tab buffer.  
But every time I quit vim, I'll have to split it again to make the view I last edited in. So how to save the session.  


Thanks to https://github.com/dhruvasagar/vim-prosession this plugin. This makes it possible to save the session and reload it when I restart vim, silently, automatically.  
But, well, It may bring some problem. Since I have it installed, my file-tree-window will become rather narrow sometimes when I vertically split the window more than 2.  
Anyway, it solved my great problem. You could give a try.  
