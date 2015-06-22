---
layout: post_item
status: publish
published: true
title: Briefly intro of svn and git
author: jsongo
date: '2015-06-22 12:08:41'
categories: [tools]
tags: [masterpiece]
comments: []
---
&nbsp;&nbsp;This is the summary of how I use svn and now git.  
I've been using svn for the past three years and git for several months, which are exactly not long enough to write a deepgoing article about these two tools. Well, if you're new to one of them, this post may be of some use. I'll just talk about how, not much why. Any question, leave some comments at the following comment box, I'll answer it as soon as I find it, and as long as I Know How To Reply >_<. Anyway, there may be some mistakes below. Tell me when you find it. Much appreciate about that.  


&nbsp;&nbsp;So now let's begin. And this is the catalogue:  
1. How to pull files  
2. How to add files  
3. How to update files  
4. How to delete files  
5. How to check info  
6. How to compare files  
7. How to merge  
8. Branch and Tags  
9. Remote operation  
10. Roll back  
11. How to resolve conflict  
12. How to relocate  
13. How to set external links  
* 14. Why we use svn  
* 15. Why we use git  

---

> 1. How to pull files  

## [1]. with svn
{% highlight bash %}
svn co svn://xxx/path [local_dir]
{% endhighlight %}
But if you just want to pull files without {% highlight raw %}.svn{% endhighlight %} directories, just do exporting.  
{% highlight bash %}
svn export svn://xxx/path [local_dir]
{% endhighlight %}
And you can export any version of files. The simple syntax is:
{% highlight bash %}
svn export [-r version] svn://xxx/path [local_dir]
{% endhighlight %}
In some cases, if you submit your code for the first time, svn will ask for your name and password. There is way to void that. You could use {% highlight raw %}--username xxx --password xxx{% endhighlight %} params when you run the svn commands. But it's not recommended to type your password in the command line. So, don't do it. Only use `--username`.

## [2]. with git

> 2. How to add files 
