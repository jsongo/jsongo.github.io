---
layout: post_item
status: publish
published: true
title: python的系统底层操作1——基础
author: jsongo
wordpress_id: 59
wordpress_url: http://jsongo.com/?p=59
date: '2013-09-25 13:17:20 +0800'
date_gmt: '2013-09-25 05:17:20 +0800'
categories: [python]
tags: [masterpiece, python]
---
python中有一个特别强大的库，它赋予了python像C语言一样的底层操作能力：ctype库


{% highlight python %}

#encoding=utf-8
from ctypes import *
msvcrt = cdll.msvcrt
msg_str = "Hello, jasonling，凌云壮志"
msvcrt.printf("The string is: %s\n",msg_str)
ch_p = c_char_p("jasonling, Good Job!")
msvcrt.printf("%s\n",ch_p)
print ch_p

{% endhighlight %}

这是python兼容window的代码，cdll遵守参数从右向左入栈的调用约定，这种约定在x86架构上被大多数c编译器使用。

msvcrt = cdll.msvcrt获得C的API调用的句柄，然后用它直接调用C相关的函数，函数名基本一致。

python中要得到C里面的相关类型及各种指针，可以用如下的关系对应表对应起来：

![](/img/201212/13/223954zczisgiwiypssgs4.jpg)

上面用到的ch_p = c_char_p("jasonling, Good Job!")就是把这个字符串的指针赋给ch_p，把它print出来的结果是：c_char_p('jasonling, Good Job!')

表示它是一个字符串指针。上面程序的运行结果如下：

{% highlight python %}

c_char_p('jasonling, Good Job!')

The string is: Hello, jasonling，凌云壮志

jasonling, Good Job!

{% endhighlight %}

我们看到最后的那个print语句执行的结果是第一个打印出来的，因为它执行得比较快，另外两个打印语句printf还要去调用系统位于高地址处的api，所以返回的比较慢些。

引用传参：

如果要像C一样，在函数中传入指针作为参数，则可以用ctype提供的byref()函数来实现，如：myfunc( byref(param) )

结构体：

要像C那样调用底层的API，结构体和联合体是必不可少的

python定义结构体和联合体的方式如下：

{% highlight python %}
#encoding=utf-8
from ctypes import *

class Fraction(Structure):

    _fields_ = [
        ("numerator", c_int), #分子
        ("denominator", c_int) #分母
    ]

    def printValue(self):
        print str(self.numerator) + "/" + str(self.denominator)

class Figure(Union):

    _fields_ = [
        ("fig_int", c_int),
        ("fig_long", c_long),
        ("fig_char", c_char * 8)
    ]

    def printValue(self):
        if self.fig_int:
            print "%d" % self.fig_int
            print "%ld" % self.fig_long
            print "%s" % self.fig_char

# 测试struture

value1 = raw_input("分子：")
value2 = raw_input("分母：")
fra = Fraction(int(value1), int(value2))
fra.printValue()

# 测试Union
value1 = raw_input("输入数字：")
fig = Figure(int(value1))
fig.printValue()
{% endhighlight %}

输出结果：

{% highlight python %}
分子：1
分母：2
1/2
输入数字：97
97
97
a
{% endhighlight %}

定义的时候，还是用class关键字来定义。对于结构体，定义的时候是括号里用Structure，继承自这个类，表示这是一个结构体，联合体用Union，这些都要在里面定义一个_fields_。在_field_里面定义结构体或联合体的字段，赋值时直接传入。联合体里面，char*8，表示8个字节，传入数字，会被转化成相应ASCII码所对应的字母！

