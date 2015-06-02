---
layout: post_item
status: publish
published: true
title: shell基础 笔记
author: jsongo
wordpress_id: 181
wordpress_url: http://jsongo.com/?p=181
date: '2013-12-22 03:54:27 +0800'
date_gmt: '2013-12-21 19:54:27 +0800'
categories: [draft]
tags: [redis]
comments: []
---
1、case流程控制  
{% highlight bash %}  
case "$key" in  
[a-z])  
echo "小写字母"  
;;  
[A-Z])  
echo "大写字母"  
;;  
*)  
echo "其它字符"  
esac  
{% endhighlight %}  


2、for循环  
{% highlight bash %}  
for $key in list  
do  
....  
done  
{% endhighlight %}  
如 可以for num in 12 33 35  
取文件内容 for word in $(cat /etc/host.conf)  
$()用来执行其它指令，并把结果返回  
例子：  
{% highlight bash %}  
for name in $(cat user.txt)  
do  
useradd $name  
echo "123456" | passwd -stdin $name  
chage -d 0 $name #让用户的密码过期，强制让其重新设置密码  
done  
{% endhighlight %}  
----------------------------  
2、if判断  
{% highlight bash %}  
if 条件  
then ...  
fi  
{% endhighlight %}  
很简单，其中的条件，每个判断可以用[ ]来包起来  
例子：  
{% highlight bash %}  
if [! -d '/home/json'] #-d用来判断是否是一个文件夹  
then  
echo "文件夹不存在"  
fi  
{% endhighlight %}  

then可以并列一个else，多if的可以用elif  
--------------------------  
3、字符串

* 字符串截取  
{% highlight bash %}  
str="/etc/host.conf"  
dirname $str     #得到/etc  
basename $str    #得到host.conf  
#用substr：  
expr substr $str 6 10      从位置6截取10个长度，得到host.conf。注意这里是从1开始算起的  
echo ${str:5:10}   一样的结果，这里是从0开始算起的。可空  
{% endhighlight %}  
* 替换  
{% highlight bash %}  
echo ${str/old/new}    str的值不变。这个只会替换一个，要全部替换，使用下面的：  
echo ${str//old/new}  使用两个/     在vi里面也有类似的，不过它表示全部替换时是用/g  
{% endhighlight %}  
* 生成随机串  
{% highlight bash %}  
head -1 /dev/urandom | md5sum | cut -b 0-9  
{% endhighlight %}  
cut中0可省略。/dev/urandom是用来产生随机字符的设备。md5sum得到md5摘要，结果后面有空格和横杠，用cut来截取需要的。  

4、条件测试  
test 或者[条件]  
[-d /home/abc] && echo YES      如果/home/abc是目录，则打印出YES  
参数有：  
-e 是否存在exist  
-d 是否是目录  
-f 是否是文件  
-x 是否可执行  
-r 是否可读  
-w 是否可写  
另外数值比较：  
-eq 等于    -ne 不等于     -gt  大于     -lt 小于     -ge   大于等于     -le 小于等于  
例子：  
{% highlight bash %}  
[$(who | wc -l) -eq 2] && echo YES  
{% endhighlight %}  
* 字符串比较 =和!=

5、其它  
（1）find /etc -name \"\*.conf\" -type f | wc -l        统计/etc目录下所有的conf后缀的文件（算出个数）  
（2）标准错误输出：  
2>、2>>      将错误信息覆盖、追加到文件中  
（3）正常输出和错误输出一起  
&>、&>>  
（4）||来隔开两个命令，那么第一个执行如果成功，第二个就不会执行。  
mkdir /home/abc 2>/dev/null && echo \"创建成功\"  
换成||就echo \"执行失败\"  
（5）分隔符来分隔多个命令，则前面的执行完后，会回车再执行第二个  
（6）双引号：允许引用和转义  
单引号禁止引用、转义  
$(命令) 或 \`命令\`  
特殊的反撇号引起来的表示执行后用结果替换当前位置  
（7）系统变量  
可以用    env | awk -F= \'/=/ {print $1}\'    来查看  
（8）脚本特殊变量  
$? 前一条命令的状态值，0为执行正常  
$0 脚本自身的程序名  
$1-9  第1-9个参数  
$* 命令行的所有参数  
$# 命令行的参数个数  
（9）运算  
{% highlight bash %}  
expr 1+1  
#乘法要注意，expr 1\*2  
x=11; y=22; echo $[x+y]  
{% endhighlight %}  
$[]放中括号里面的变量不用再加$  
自增自减等用let         let x++        let x+=2           let后的变量不用再加$  
（10）随机数 $RANDOM  
值默认是从0到32767  
（11）生成序列，像python里面的range，shell里面用seq  
seq 3   输出1 2 3  
seq 2 4  输出2 3 4  
seq 3 2 10  中间的数表示step，输出3 5 7 9  
（12）小数运算  
使用bc  
{% highlight bash %}  
echo "23.31-18.97" | bc  
#用除法时可以用scale来限定小数的位数：  
echo "scale=4;10/3" | bc  
{% endhighlight %}  
