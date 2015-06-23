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
**But, git isn't better or worse than Subversion, it's different.**  
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
This post is mainly for coders, so, below I'll talk about codes instead of files in some places.  
`[]` in commands below means the part is optional.  
'repo' is short for 'repository' which performs the server and storage for svn or git.  
Also, there is a different meaning of 'checkout' between svn and git.  
In svn, 'checkout' means pulling down the codes while in git, it means swithing among branches in your working directory. I'll talk more about that bellow.   

<a id="sec1"></a>

> ## 1# How to pull files  

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

> ## 2# How to add files and commit  

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

> ## 3# How to update files

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

> ## 4# How to delete files

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

So how to delete a remote repo ?
with svn, you could just do this:
{% highlight bash %}
svn del xxx
{% endhighlight %}
Year, it's the same as files. Because there are only directories and files in svn repo. But in git, you'll have to do it like this:
{% highlight bash %}
git push origin :my_branch
{% endhighlight %}
Interesting, right? Push nothing to the my_branch of remote 'origin'. It means to delete a repo. But notice that, you can't simply delete master in such an easy way, it is protected for the default branch. You've got to do some settings at the remote platform, for example, GitHub or GitLab.  Change the default branch from master to other branch, and then could you delete the master.  

<a id="sec5"></a>

> ## 5# How to check info

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
<a id="branchRemote"></a>
{% highlight bash %}
git branch      # which will show you the local branch info
git branch -r   # which will show you the remote branch info
git branch -a   # which will show you both the local and remote branch info
git tag         # tag info
{% endhighlight %}
And `git show` can also be used to show the info of some branch or tag. Try it by yourself.  
Trunk in svn means the main branch, as is the same as master in git.  

<a id="sec6"></a>

> ## 6# How to compare files

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

> ## 7# How to merge

## [1]. with svn
{% highlight bash %}
svn merge svn://xxx/other_path
{% endhighlight %}

## [2]. with git 
{% highlight bash %}
git merge other_branch
{% endhighlight %}

<a id="sec8"></a>

> ## 8# Branch and Tags

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
git tag -s the_name -m 'xxx'
{% endhighlight %}
`-s` is a signature operation. You can read more about it [here](http://pragmatictim.blogspot.jp/2012/11/creating-signed-tags-in-git-from-scratch.html). And a commit can also be signitured using `-S`. Learn more [here](https://ariejan.net/2014/06/04/gpg-sign-your-git-commits/).   
Now you can push it using:
{% highlight bash %}
git push origin tag_name
{% endhighlight %}
If you wanna push all tags, just use this:
{% highlight bash %}
git push origin --tags
{% endhighlight %}

* pull the branch
{% highlight bash %}
git checkout -b newBrach origin/master
{% endhighlight %}

<a id="sec9"></a>

> ## 9# Remote operation

Actually, svn doesn't seperate what remote or local operation is. As the old saying goes, it's the same. 

## [1]. with svn
So let's skip this for it's the same step as operation on trunk. Such as:
{% highlight bash %}
svn ls svn://xxx/path
{% endhighlight %}

## [2]. with git
(Ok, I may make some mistakes below for I'm not that familiar with tags operation with git. So if you find some, just tell me. I'll be much thankful to correct them. )
{% highlight bash %}
git remote -v     # To check the remote name and urls, ie. origin https://xxx/path.git
git remote show   # To display the information about current remote name
git remote add test_repo https://xxx/other_path.git   # To add another remote repo info which you can push your code there.
{% endhighlight %}
And it's awesome that in one directory you could manage several remote repo. Just imagine what you can do with it. Pull codes form one place, modify it, then submit to another place. Cool, isn't it? What you have to do is add a remote repo with one command. Of course, you can delete remote repo with `git remote rm test_repo`, rename with `git remote rename test_repo new_repo`.  
Remote branch discussed above can be reached [here](#branchRemote).   
But when we talk about tags, it's not that simple. 'Cause the tags are some kind of  protected. It's different from the branches including the master. With the lastest version of git, remote tags can't be listed with `git branch -a` or `git branch -r`. (My version is '1.9.5 (Apple Git-50.3)' ). But you can list you local tags using:
{% highlight bash %}
git tag -l
{% endhighlight %}
And `git show tag_version` is working. Checkout using: 
{% highlight bash %}
git checkout -b local_tag_name remote_tag
{% endhighlight %}
Then you can operate on it like a branch, ie. `git checkout local_tag_name`, except that you cannot commit to the remote existing tag. If you try to, it'll throw out an exception with the words of 'tag already exists in the remote'. Actually you can make by force with params of `--force` or `-f`. But I recommend you not doing that unless you really have to and really know what you're doing.  
If you checkout to a tag, the `git branch` command will tell you that your current working copy is 'detached from tag_name_xxx'.  
You can make a tag of the history version too, using:
{% highlight bash %}
git tag -a tag_version history_label 
{% endhighlight %}
'history_label' is the SHA-1 checksum git generated for every commit you made. You can find it with `git log` as well as other methods.  
Anyway, you can operate freely on your local tags. For example, deleting using:
{% highlight bash %}
git tag -d your_local_tag_name
{% endhighlight %}
This will only affect your local repo. 

<a id="sec10"></a>

> ## 10# Roll back

## [1]. with svn
Most of the time, we only have to discard local modification to solve problems.  
{% highlight bash %}
svn revert [--recursive] file_or_dir_name
{% endhighlight %}
But there's still some time, that we commit by accident or find the recent commits of no use. Then we could solve this problem by merging.  
{% highlight bash %}
svn merge --dry-run -r:version_new:version_old http://xxx/path
svn merge -r:version_new:version_old http://xxx/path
svn commit -m "Reverted to revision version_old."
{% endhighlight %}
The first command about will perform a dry run and show you what the merge will produce. Learn more about this [here](https://aralbalkan.com/1381/).  

## [2]. with git
To discard local modifications recently made before commit or to rollback to a particular version
{% highlight bash %}
git reset --hard <tag/branch/commit id>
{% endhighlight %}
Excellent. It's really a rollback operation instead of several different steps in svn. Rather convenient and clear.  

<a id="sec11"></a>

> ## 11# How to resolve conflict

## [1]. with svn
When updated, svn will mark the conflicts with different files and with '======' inside files. So search for the conflict parts, modify them and delete the backup files created by svn. Then run:  
{% highlight bash %}
svn resolved file_name
{% endhighlight %}
This will tell svn that the conflicts in this file is resolved.  

## [2]. with git
Git will sometimes produce conflicts when merging. Modify the marked parts like svn and the submit. Git will find that it's resolved and you don't have to tell it.   

<a id="sec12"></a>

> ## 12# How to set external links

An external link means you can set a different svn repo link in your sub-directory.  
For example, you're working in dir_a, but you want your sub-directory, dir_b, to synchronized with the code from another repo, then you cat set a external link in dir_b.  

dir_a     <-- codes form svn://xxx/path1  
  |-- dir_b     <-- codes form svn://xxx/path2  
  |-- ...  
  ...  

## [1]. with svn
{% highlight bash %}
svn propset svn:externals 'dir_name svn://xxx/path' .
svn up
{% endhighlight %}

## [2]. with git
That's simple. Step into your sub-directory, and run `git clone https://xxx/path.git`, and it's done.  

<a id="sec13"></a>

> ## 13# How to relocate

## [1]. with svn
{% highlight bash %}
svn sw --relocate svn://url1/path1 svn://url2/path2
{% endhighlight %}
And 'sw' is short for switch.  

## [2]. with git
{% highlight bash %}
git remote rm origin
git remote add origin https://xxx/new_path.git
{% endhighlight %}

<a id="sec14"></a>

> ## 14# Why we use svn

1. Svn is much simpler and easier.  
2. `svn co` can pull a single directory deep inside a sub-tree which git fails to.  
3. Git is hard to learn.  
4. Git has a really scathing authentication rules. For example, you have to generate a pair of rsa keys and set the pub-key to the remote platform and then can submit your codes for the first time.  
5. In window, git tools is not as many nor as good as svn.  

<a id="sec15"></a>

> ## 15# Why we use git

<a>1#</a> If you're trapped somewhere disconnected from the internet, or to say you're on the plane, basement or elevator, you can still commit, or revert, etc.  

<a>2#</a> Branch is really branch, and tag is really tag.  
&nbsp;&nbsp;It's only a normal directory of branches or tags for svn. They're not different from trunk actually. But this is not the case for git. They're seperated and quite different for they've got different commands to reach different goals.  

<a>3#</a> Svn operates by files while git by elements.  
&nbsp;&nbsp;When you have one trunk or master and several branches or tags, svn stores them separately in different directories so it has several copies of a file. But git only stores the master files, as for branches or tags, git stores the difference between them and master.  
On the other hand, did you find that '.svn' is quite smaller then '.git'?  That's the result of the different ways svn and git store the files info.  

<a>4#</a> Imagine the remote repo being destroyed and the remote server can't roll back to save the repo.  
&nbsp;&nbsp;What would happend to svn and git? How will they resume the remote repo?  
Sorry, svn can't make it. It has to build a new remote repo, delete your local '.svn' directories, and then submit your local code to the new-build repo with all the history lost. You'll never restore a history version.  
And what about git? You could still work. Submitting, rolling back, merging, deleting, branches or tags creating, etc, except sharing with others.   
You don't have to ask the system manager for history and you're the manager. You have everything locally. Anyone who has ever pulled down the code can rebuild the remote repo easily. Take the three steps bellow: 
{% highlight bash %}
git remote rm origin
git remote add origin https://new_repo_path
git rebase
{% endhighlight %}
As you can see, there is an interesting command named `rebase`. It's magical. It can restore all the histories you've got to some branch. [Here](http://git-scm.com/docs/git-rebase) is the document for it.  
So from this aspect of view, svn is 'single' while git is 'distributed'. There are backups everywhere for git.  

<a>5#</a> See how svn and git do rollback operation [above](#sec10).  

<a>6#</a> If you've changed many files, svn will commit all of them if you run `svn ci`. So what will git do?  
&nbsp;&nbsp;You can only submit the files you want using `git add` and then `git submit`. It's rather helpful when you just want to submit some files instead of all.  

<a>7#</a> With git, you can manage many branches and remote repos in one directory.   
&nbsp;&nbsp;Quite convenient. Ok, this is my favorite part of git.  
Switch from different branches or tags using:
{% highlight bash %}
git checkout xxx
{% endhighlight %}
Here I'm gonna talk about another interesting command of `git stash`.  
Imagine one case that you're in middle of something, your leader comes in, and ask you to fix one emergency bug. Normally what will you do? Commit your current changes to a temporary branch and return to your original branch to fix the bug. But with 'stash' you can simplify these steps.  
{% highlight bash %}
git stash     # now your working directory has the codes of HEAD.
# ... do some fixing ... and when you're finished, just do: 
git stash pop
{% endhighlight %}
Yeah, that's all.  
Now if you're still not clear about what `git stash` is, check [this](http://git-scm.com/docs/git-stash) for details.  

<a>8#</a> 'svn update' will change your local codes even when there are conflicts.  
&nbsp;&nbsp;It marks the conflicts by modifying you local codes without asking and so forces you to resolve it.  
As for git, `git pull` refuses to overwrite your local codes. Only when you run `git merge` will git modify your codes. And the remained step are discribed [above](sec11).   

<a>9#</a> Ignore files.  
&nbsp;&nbsp;It's simple for git to ignore some kinds of files when submitting using '.gitignore' file. But for svn, there is no easy way to make it.  

<a>10#</a> Platform and UI interaction.  
&nbsp;&nbsp;Almost all of us have heard about GitHub or GitLab. There are too many convenient things we can use for developing. You know, visual interaction is clearer and easier.  
