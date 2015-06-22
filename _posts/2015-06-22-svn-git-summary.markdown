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
[1. How to pull files](#sec1)  
[2. How to add files and commit](#sec2)  
[3. How to update files](#sec3)  
[4. How to delete files](#sec4)  
[5. How to check info](#sec5)  
[6. How to compare files](#sec6)  
[7. How to merge](#sec7)  
[8. Branch and Tags](#sec8)  
[9. Remote operation](#sec9)  
[10. Roll back](#sec10)  
[11. How to resolve conflict](#sec11)  
[12. How to relocate](#sec12)  
[13. How to set external links](#sec13)  
[14. Why we use svn](#sec14)  
[15. Why we use git](#sec15)  

---
First of all, say you're familiar with svn, there's something different you've got to know about git. Git has two repositories for every project you clone to your local directory. One is the remote repo which can be shared with different people, the other is local repo which could only be access by yourself but could be used when you're disconnected from the internet.  
This post is mainly for coders, so, below I'll talk about codes instead of files.  
<a id="sec1"></a>

> 1# How to pull files  

## [1]. with svn
{% highlight bash %}
svn co svn://xxx/path [local_dir]
{% endhighlight %}
`co` is short for commit.  
But if you just want to pull files without `.svn` directories, just do exporting.  
{% highlight bash %}
svn export svn://xxx/path [local_dir]
{% endhighlight %}
And you can export any version of files. The simple syntax is:
{% highlight bash %}
svn export [-r version] svn://xxx/path [local_dir]
{% endhighlight %}
In some cases, if you submit your code for the first time, svn will ask for your name and password. There is way to void that. You could use `--username xxx --password xxx` params when you run the svn commands. But it's not recommended to type your password in the command line. So, don't do it. Only use `--username`.  

## [2]. with git
{% highlight bash %}
git clone https://xxx/path.git [local_dir]
{% endhighlight %}
With git, you cannot take co for short of clone. You'll have to type the whole word, as well as the other commands in git, except `rm` which is special and can only be written in this way.   
When cloned, the default remote tag is origin, and the default branch master.  
Well, with git, clone will pull down the code of all the branches, which is also the case of svn. So with git, if you only want the codes on branch of master, method below will do it. And this is the recommended way of pulling code in git.  
{% highlight bash %}
git init
git remote add origin https://xxx/path.git
git fetch origin master
{% endhighlight %}
And the last command can be replaced with `git pull origin master`. But they're different. `fetch` only means copying the code to local while `pull` will additionally merge the fetched code with existing file at local. There's a detailed disscussion about that, named [GIT: FETCH AND MERGE, DON’T PULL](http://longair.net/blog/2009/04/16/git-fetch-and-merge/).  

<a id="sec2"></a>

> 2# How to add files and commit  

## [1]. with svn
{% highlight bash %}
svn add xxx  (if all, run `svn add *`)
svn ci -m 'xxx'
{% endhighlight %}
`ci` is short for commit, you can type `commit` too. They're the same.  

## [2]. with git
{% highlight bash %}
git add xxx  (if all, run `git add --all`)
git commit -m 'xxx'
{% endhighlight %}
Notice that, after commit, it's not finish yet. The changes stay at local repo and haven't been sync with the remote git server.  You've got to run push command to push the changes up there.  
{% highlight bash %}
git push -u origin master
{% endhighlight %}

> 3# How to update files
