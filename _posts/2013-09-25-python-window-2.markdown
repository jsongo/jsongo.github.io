---
layout: post_item
status: publish
published: true
title: python的系统底层操作2——调用函数
author: jsongo
wordpress_id: 62
wordpress_url: http://jsongo.com/?p=62
date: '2013-09-25 13:22:05 +0800'
date_gmt: '2013-09-25 05:22:05 +0800'
categories: [python]
tags: [masterpiece, python]
comments: []
---

上一篇中，了解了python调用C底层的能力，以此为基础，本文介绍win32 API的调用。  
比如调用win32的CreateProcess函数：  


{% highlight cpp %}  
BOOL CreateProcess(  
  LPCTSTR lpApplicationName,                 // name of executable module  
  LPTSTR lpCommandLine,                      // command line string  
  LPSECURITY_ATTRIBUTES lpProcessAttributes, // SD  
  LPSECURITY_ATTRIBUTES lpThreadAttributes,  // SD  
  BOOL bInheritHandles,                      // handle inheritance option  
  DWORD dwCreationFlags,                     // creation flags  
  LPVOID lpEnvironment,                      // new environment block  
  LPCTSTR lpCurrentDirectory,                // current directory name  
  LPSTARTUPINFO lpStartupInfo,               // startup information  
  LPPROCESS_INFORMATION lpProcessInformation // process information  
);  
{% endhighlight %} 

这里比较重要的是LPSTARUPINFO结构体和PROCESS_INFORMATION结构体，前者的定义如下：  

{% highlight cpp %}  
typedef struct _STARTUPINFO {  
    DWORD   cb;  
    LPTSTR  lpReserved;  
    LPTSTR  lpDesktop;  
    LPTSTR  lpTitle;  
    DWORD   dwX;  
    DWORD   dwY;  
    DWORD   dwXSize;  
    DWORD   dwYSize;  
    DWORD   dwXCountChars;  
    DWORD   dwYCountChars;  
    DWORD   dwFillAttribute;  
    DWORD   dwFlags;  
    WORD    wShowWindow;  
    WORD    cbReserved2;  
    LPBYTE  lpReserved2;  
    HANDLE  hStdInput;  
    HANDLE  hStdOutput;  
    HANDLE  hStdError;  
} STARTUPINFO, *LPSTARTUPINFO;  
{% endhighlight %} 

所以要用python定义一个这样的结构体，一看这么多参数，一个一个写，效率太低了，于是开了个IDLE，写了个方法，方便大家把上面定义中的各个字段转化成python的结构体定义中字段：  
{% highlight python %} 
def formatParam(s):  
        strs = s.split('\n')  
        rst = ''  
        for s in strs:  
                rst += re.sub('(\w+)(\s+)(\w+);','(\"\g<3>\",\g<1>),',s) + '\n'  
        return rst[:-2]  
{% endhighlight %}

要先import re。调用这个函数：  

{% highlight python %} 
rst=formatParam('''DWORD   cb;  
    LPTSTR  lpReserved;  
    LPTSTR  lpDesktop;  
    LPTSTR  lpTitle;  
    DWORD   dwX;  
    DWORD   dwY;  
    DWORD   dwXSize;  
    DWORD   dwYSize;  
    DWORD   dwXCountChars;  
    DWORD   dwYCountChars;  
    DWORD   dwFillAttribute;  
    DWORD   dwFlags;  
    WORD    wShowWindow;  
    WORD    cbReserved2;  
    LPBYTE  lpReserved2;  
    HANDLE  hStdInput;  
    HANDLE  hStdOutput;  
    HANDLE  hStdError;  
''')  
print rst  
{% endhighlight %}

这样就得到了：  
{% highlight python %} 
("cb",DWORD),  
    ("lpReserved",LPTSTR),  
    ("lpDesktop",LPTSTR),  
    ("lpTitle",LPTSTR),  
    ("dwX",DWORD),  
    ("dwY",DWORD),  
    ("dwXSize",DWORD),  
    ("dwYSize",DWORD),  
    ("dwXCountChars",DWORD),  
    ("dwYCountChars",DWORD),  
    ("dwFillAttribute",DWORD),  
    ("dwFlags",DWORD),  
    ("wShowWindow",WORD),  
    ("cbReserved2",WORD),  
    ("lpReserved2",LPBYTE),  
    ("hStdInput",HANDLE),  
    ("hStdOutput",HANDLE),  
    ("hStdError",HANDLE),  
{% endhighlight %}

拷到python代码里面就可以了！  
PROCESS_INFORMATION的定义如下：  
{% highlight cpp %}  
typedef struct _PROCESS_INFORMATION {  
    HANDLE hProcess;  
    HANDLE hThread;  
    DWORD dwProcessId;  
    DWORD dwThreadId;  
} PROCESS_INFORMATION;  
{% endhighlight %} 

这两个转成python代码后，放到一个文件debug_define.py里面，如下： 

{% highlight python %} 
# -*- coding: utf-8 -*-  
from ctypes import *  
# 定义window编程中常用的类型  
WORD = c_ushort  
DWORD = c_ulong  
LPBYTE = POINTER(c_ubyte)  
LPTSTR = POINTER(c_char)  
HANDLE = c_void_p  
# 常量，定义CreateProcess时要用到  
DEBUG_PROCESS = 0x0001;  
CREATE_NEW_CONSOLE = 0x0010;  
# CreateProcessA函数所需结构体  
class STARTUPINFO(Structure):  
    _fields_ = [  
        ("cb",DWORD),  
        ("lpReserved",LPTSTR),  
        ("lpDesktop",LPTSTR),  
        ("lpTitle",LPTSTR),  
        ("dwX",DWORD),  
        ("dwY",DWORD),  
        ("dwXSize",DWORD),  
        ("dwYSize",DWORD),  
        ("dwXCountChars",DWORD),  
        ("dwYCountChars",DWORD),  
        ("dwFillAttribute",DWORD),  
        ("dwFlags",DWORD),  
        ("wShowWindow",WORD),  
        ("cbReserved2",WORD),  
        ("lpReserved2",LPBYTE),  
        ("hStdInput",HANDLE),  
        ("hStdOutput",HANDLE),  
        ("hStdError",HANDLE)  
    ]  
class PROCESS_INFORMATION(Structure):  
    _fields_ = [  
        ("hProcess",HANDLE),  
        ("hThread",HANDLE),  
        ("dwProcessId",DWORD),  
        ("dwThreadId",DWORD)  
    ]  
{% endhighlight %}

STARTUPINFO结构详细的解释可以去查MSDN，这里主要讲两个参数，dwFlags 和 wShowWindow 。要使后者起作用，前者必须设为STARTF_USESHOWWINDOW，对应的值为0x0001，后者的值和nCmdShow的值定义是一样的，参考：http://www.lingyibin.com/forum.php?mod=viewthread&tid=118。这里把它设为0，即SW_HIDE，不显示窗口。这里做个简单的说明，本程序是一个简单的python调试器的雏形，它创建一个新进程，用它来加载另一个要调试的程序，所以我们这里创建出来的第一个进程是要隐藏窗口的。  
创建进程代码：  

{% highlight python %} 
# -*- coding: utf-8 -*-  
from ctypes import *  
# 引入之前定义的那两个结构体等数据  
from debug_define import *  
kernel32 = windll.kernel32  
class debugger:  
    def __init__(self):  
        pass  
    # 创建一个新进程，用来载入一个exe，调试  
    # exe_path 表示exe路径  
    def loadExe(self, exe_path):  
        creation_flags = DEBUG_PROCESS  
        startupinfo = STARTUPINFO() #相当于c的alloc这么大一个空间，初始化为0或NULL  
        processinfo = PROCESS_INFORMATION() #同上  
        startupinfo.dwFlags = 0x0001 # STARTF_USESHOWWINDOW  
        startupinfo.wShowWindow = 0 # SW_HIDE  
        startupinfo.cb = sizeof(startupinfo)  
        # 开始创建  
        if kernel32.CreateProcessA(exe_path, #lpApplicationName  
                                  None, #lpCommandLine  
                                  None, #lpProcessAttributes  
                                  None, #lpThreadAttributes  
                                  None, #bInheritHandles  
                                  creation_flags, #dwCreationFlags  
                                  None, #lpEnvironment  
                                  None, #lpCurrentDirectory  
                                  byref(startupinfo), #lpStartupInfo  
                                  byref(processinfo)): #lpProcessInformation  
            print "进程ID:%d" % processinfo.dwProcessId  
        else:  
            print "进程创建失败。"  
{% endhighlight %}

测试：  
{% highlight python %} 
# -*- coding: utf-8 -*-  
import debug.debugger as debugger  
debugger = debugger.debugger()  
debugger.loadExe("c:/Windows/System32/cmd.exe")  
{% endhighlight %}
