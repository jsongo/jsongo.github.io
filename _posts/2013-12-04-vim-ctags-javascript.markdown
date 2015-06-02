---
layout: post_item
status: publish
published: true
title: vim中强化ctags对javascript的支持
author: jsongo
wordpress_id: 155
wordpress_url: http://jsongo.com/?p=155
date: '2013-12-04 01:55:55 +0800'
date_gmt: '2013-12-03 17:55:55 +0800'
categories: [draft]
tags: [vi, vim]
comments: []
---
到http://www.vim.org/scripts/script.php?script_id=1491下载javascript.vim，它可以更好地支持oop风格的js语法高亮。  
把它拷贝到.vim/syntax里面就可以了  
vi .vimrc，加入如下一行：  


{% highlight vim %}
au BufNewFile,BufRead *.jsm set filetype=javascript &nbsp; &nbsp;"set new file extension jsm as javascript file.
{% endhighlight %}
用来把.jsm文件识别成javascript  
新建~/.ctags，加入：  
{% highlight vim %}
--langdef=Javascript  
--langmap=Javascript:.js.jsm  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([A-Za-z0-9._$]+)[[:blank:]]*[:=][[:blank:]]*new[[:blank:]]+Object\(/\2/o,object/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([A-Za-z0-9._$]+)[[:blank:]]*[:=][[:blank:]]*\{/\2/o,object/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])(^[^\?][[:blank:]]*)([A-Za-z0-9_]+)[[:blank:]]*[:][[:blank:]]*[A-Za-z0-9._$'"()]+/\3/m,member/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([A-Za-z0-9._$]+)[[:blank:]]*[:=][[:blank:]]*new[[:blank:]]+Array\(/\2/a,array/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([A-Za-z0-9._$]+)[[:blank:]]*[:=][[:blank:]]*\[/\2/a,array/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([^! ]+[^= ]+)[[:blank:]]*=[[:blank:]]*[^""]'[^'']*/\2/s,string/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])([A-Za-z0-9._$()]+)[[:blank:]]*[:=][[:blank:]]*function[[:blank:]]*\(/\2/f,function/  
--regex-JavaScript=/(^|^[^\/*]+[[:blank:]])function[[:blank:]]+([A-Za-z0-9._$]+)[[:blank:]]*([^)])/\2/f,function/
{% endhighlight %}
然后往.vimrc里再加一行：  
{% highlight vim %}
let g:tlist_javascript_settings = 'javascript;s:string;a:array;o:object;f:function;m:member'
{% endhighlight %}
然后就可以打开一个js文件来看看效果了，  
DONE  
参考：http://www.huangwei.me/blog/2010/11/30/improve-vim-javascript-edit/  
还有一篇也不错：http://hi.baidu.com/whqvzhjoixbbdwd/item/e82e38f36ee06a1da62988e9  
