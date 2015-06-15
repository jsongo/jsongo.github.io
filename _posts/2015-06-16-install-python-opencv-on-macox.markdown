---
layout: post_item
status: publish
published: true
title: install python opencv on mac ox
author: jsongo
date: '2015-06-16 00:57:04'
categories: [draft]
tags: [mac]
comments: []
---
Of course we'll refer to the great HomeBrew.  
There is detail here: https://jjyap.wordpress.com/2014/05/24/installing-opencv-2-4-9-on-mac-osx-with-python-support/.  
But if you're in a hurry, then this post will help you.


Before this I'm sure you've got homebrew installed on you mac.  
Go ahead.  
Add homebrew/science which is where OpenCV is located using:  
{% highlight bash %}
brew tap homebrew/science
{% endhighlight %}
And then, Let's see whether opencv can be searched using:  
{% highlight bash %}
brew info opencv
{% endhighlight %}
And if you've got problems during these two steps, try to update homebrew using:  
{% highlight bash %}
brew update
{% endhighlight %}
More unfortunately, this fails again, you'll have to take this step to force this update working.  
{% highlight bash %}
cd /usr/local
git fetch
git reset --hard origin/master
brew update
{% endhighlight %}

So, back to where we break. We now can run install.  
{% highlight bash %}
brew install opencv
{% endhighlight %}
And it will really take some time.  

After opencv is installed, it's not finished yet. There is a last step, make the soft link of the lib file.
{% highlight bash %}
cd /Library/Python/2.7/site-packages/
ln -s /usr/local/Cellar/opencv/2.4.11_1/lib/python2.7/site-packages/cv.py cv.py
/usr/local/Cellar/opencv/2.4.11_1/lib/python2.7/site-packages/cv2.so cv2.so
{% endhighlight %}
And of course, when you're reading this post, the version of opencv may have been changed. So the path up there may be differet. Change it according to your opencv installed.

Done.
