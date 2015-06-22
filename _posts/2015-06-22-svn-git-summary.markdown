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
`[]` in commands below means the part is optional.  

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
With git, you cannot take co for short of clone. You'll have to type the whole word, as well as the other commands in git, except `rm` or `mv` which are special and can only be written in this way.   
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

<a id="sec3"></a>

> 3# How to update files

## [1]. with svn
{% highlight bash %}
svn up
{% endhighlight %}
`up` is short for `update`.

## [2]. with git
{% highlight bash %}
git fetch origin master
{% endhighlight %}
or
{% highlight bash %}
git pull origin master
{% endhighlight %}

<a id="sec4"></a>

> 4# How to delete files

If you delete in some ways without svn/git command, named `rm test.py`, by accident, svn will know nothing about that. It's not aware of the changes you've made to its files. You have to explicitly tell it. And next time you run `svn up` will it check the difference of his file-tree from last version, and will it fetch the deleted files back, without warning you what it have done. You'll know nothing until you check the logs.  
But git won't act that way. It'll warn you with red mark and you'll clearly see what had happended.

## [1]. with svn
{% highlight bash %}
svn del xxx
{% endhighlight %}

## [2]. with git
{% highlight bash %}
git rm xxx
{% endhighlight %}

<a id="sec5"></a>

> 5# How to check info

Check the current working status and info, branch or tag or trunk info, log info, etc.

## [1]. with svn

* current status
{% highlight bash %}
svn st
{% endhighlight %}
`st` is short for `status`.

* current info
{% highlight bash %}
svn info
{% endhighlight %}

* log info  
{% highlight bash %}
svn log [file_name]
{% endhighlight %}

* trunk or branches or tags info  
It's boring for the commands are similar.  
{% highlight bash %}
svn ls svn://xxx/path/trunks
svn ls svn://xxx/path/branch
svn ls svn://xxx/path/tags
{% endhighlight %}

## [2]. with git

* current status and info
{% highlight bash %}
git status
{% endhighlight %}
{% highlight bash %}
git show
{% endhighlight %}
This command will only show you the last modifications at your local repo.  
And for git, there is still many infos of current remote repos, which I'll talked about later in the [Remote operation](#sec9). 
* log info
{% highlight bash %}
git log [file_name]
{% endhighlight %}

* branches or tags info  
{% highlight bash %}
git branch      # which will show you the local branch info
git branch -v   # which will show you the remote branch info
git branch -a   # which will show you both the local and remote branch info
git tag         # tag info
{% endhighlight %}
And `git show` can also be used to show the info of some branch or tag. Try it by yourself.  
Trunk in svn means the main branch, as is the same as master in git.  

<a id="sec6"></a>

> 6# How to compare files

## [1]. with svn
{% highlight bash %}
svn diff [-rxxx:yyy] [file_name]
{% endhighlight %}

## [2]. with git
{% highlight bash %}
git diff [file_name]
{% endhighlight %}
This command of git will only compare the files before commited with the lastest version locally. 
{% highlight bash %}
git diff --cached
{% endhighlight %}
This time, it compare the local newest version with HEAD which means the lastest version remotely.
{% highlight bash %}
git diff HEAD
{% endhighlight %}
And this will compare the working directory with the remote newest version.  
[Here](http://git-scm.com/docs/git-diff) are the detail use of git diff.   

<a id="sec7"></a>

> 7# How to merge

## [1]. with svn
{% highlight bash %}
svn merge svn://xxx/other_path
{% endhighlight %}

## [2]. with git 
{% highlight bash %}
git merge other_branch
{% endhighlight %}

<a id="sec8"></a>

> 8# Branch and Tags

## [1]. with svn

* create branch
{% highlight bash %}
svn cp svn://xxx/path/trunk svn://xxx/path/branches/branch_xxx -m 'xxx'
{% endhighlight %}

* create tag
{% highlight bash %}
svn cp svn://xxx/path/trunk svn://xxx/path/tags/tag_xxx -m 'xxx'
{% endhighlight %}

Find that? They are nearly the same. Actually, branches, tags, and trunk, they are just like different folders. In [Why we use git](#sec15) below, I'll tell you the big difference between svn and git. Imagine you've got 10,000 files in your trunk or branch (say you've got only one branch). Gonna checkout the trunk and branch? Sorry, svn tell you, you have to download 20,000 files. >_< Painful...

command to checkout the branch or tag? The same too.
{% highlight bash %}
svn co svn://xxx/path/branches/branch_xxx
{% endhighlight %}

## [2]. with git

* create branch
{% highlight bash %}
git branch the_name -m 'xxx'
{% endhighlight %}

* create tag
{% highlight bash %}
git tag the_name -m 'xxx'
{% endhighlight %}

* pull the branch
{% highlight bash %}
git checkout -b newBrach origin/master
{% endhighlight %}

<a id="sec9"></a>

> 9# Remote operation

## [1]. with svn

## [2]. with git

<a id="sec10"></a>

> 10# Roll back

## [1]. with svn

## [2]. with git

<a id="sec11"></a>

> 11# How to resolve conflict

## [1]. with svn

## [2]. with git

<a id="sec12"></a>

> 12# How to relocate

## [1]. with svn

## [2]. with git

<a id="sec13"></a>

> 13# How to set external links

## [1]. with svn

## [2]. with git

<a id="sec14"></a>

> 14# Why we use svn

## [1]. with svn

## [2]. with git

<a id="sec15"></a>

> 15# Why we use git
## [1]. with svn

## [2]. with git
