---
layout: post_item
status: publish
published: true
title: python的系统底层操作3——进程注入
author: jsongo
wordpress_id: 65
wordpress_url: http://jsongo.com/?p=65
date: '2013-09-25 16:53:31 +0800'
date_gmt: '2013-09-25 08:53:31 +0800'
categories: [python]
tags: [masterpiece, python]
comments: []
---

    为了能够取得在运行的某个进程在内存中的数据，就得附加到某个进程里面去。由进程的pid调用OpenProcess来取得进程的句柄，之后有了这个句柄就可以做很多事情啦！  


    本文中的代码是上一篇《python的系统底层操作2&mdash;调用函数》的扩展，也是要实现调试的功能。用DebugActiveProcess来附加到进程里面去，然后调用WaitForDebugEvent来等待调试事件的发生，一旦调试事件被触发，进程就被挂起，我们就可以操作进程中的数据，进行一些调试相关的处理，完成后，调用ContinueDebugEvent回到原进程，最后可以调用DebugActiveProcessStop与被调试的进程分离。  
代码：  
debugger.py  
{% highlight python %}  
# -*- coding: utf-8 -*-  
from ctypes import *  
from debug_define import *  
kernel32 = windll.kernel32  
class debugger():  
    def __init__(self):  
        self.h_process = None  
        self.pid = None  
        self.debugger_active = False  

    def loadExe(self, exe_path):  
        creation_flags = DEBUG_PROCESS  
        startupinfo = STARTUPINFO() #相当于c的alloc这么大一个空间，初始化为0或NULL  
        processinfo = PROCESS_INFORMATION() #同上  
        startupinfo.dwFlags = STARTF_USESHOWWINDOW  
        startupinfo.wShowWindow = SW_HIDE  
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
        # 打开这个进程  
        self.h_process = self.open_process(processinfo.dwProcessId)  
    # 由id取得进程  
    def open_process(self, pid):  
        h_progress = kernel32.OpenProcess(PROCESS_ALL_ACCESS, False, pid)  
        return h_progress  
    # 附加进程  
    def attach(self, pid):  
        self.h_process = self.open_process(pid)  
        # 调用DebugActiveProcess，附加到pid进程  
        if kernel32.DebugActiveProcess(pid):  
            self.debugger_active = True # 可以开始调试了  
            self.pid = int(pid)  
            self.run()  
        else:  
            print "进程附加失败！"  
    def run(self):  
        while self.debugger_active == True:  
            self.fire_debug_event()  
    # 等待触发调试事件。。  
    def fire_debug_event(self):  
        debug_event = DEBUG_EVENT()  
        continue_status = DBG_CONTINUE  
        # 接下去就是等待调试的事件被触发  
        if kernel32.WaitForDebugEvent(byref(debug_event), INFINITE): #一直等  
            # 到这里，说明调试的条件被触发了  
            # 。。。（在这里做些调试相关的事情）  
            raw_input("按回车键继续...")  
            # 这个调试事件处理结束，继续  
            self.debugger_active = False  
            kernel32.ContinueDebugEvent(debug_event.dwProcessId,  
                                        debug_event.dwThreadId,  
                                        continue_status)  
    # 解除对进程的附加  
    def detach(self):  
        if kernel32.DebugActiveProcessStop(self.pid):  
            print "完成调试，退出..."  
            return True  
        else:  
            print "调试过程中出错..."  
            return False  
{% endhighlight %}  
上面用到的常量、结构体、联合体等都在debug_define.py里面定义：  
{% highlight python %} 
# -*- coding: utf-8 -*-  
from ctypes import *  
# 定义window编程中常用的类型  
WORD = c_ushort  
DWORD = c_ulong  
LPBYTE = POINTER(c_ubyte)  
LPTSTR = POINTER(c_char)  
HANDLE = c_void_p  
PVOID     = c_void_p  
LPVOID    = c_void_p  
ULONG_PTR  = c_ulong  
SIZE_T    = c_ulong  
# 常量  
DEBUG_PROCESS = 0x0001  
CREATE_NEW_CONSOLE = 0x0010  
PROCESS_ALL_ACCESS = 0x001F0FFF  
INFINITE = 0xFFFFFFFF  
DBG_CONTINUE = 0x00010002  
STARTF_USESHOWWINDOW = 0x0001  
SW_HIDE = 0  
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
class EXCEPTION_RECORD(Structure):  
    pass  
EXCEPTION_RECORD._fields_ = [  
        ("ExceptionCode",DWORD),  
        ("ExceptionFlags",DWORD),  
        ("ExceptionRecord",POINTER(EXCEPTION_RECORD)),  
        ("ExceptionAddress",PVOID),  
        ("NumberParameters",DWORD),  
        ("ExceptionInformation",ULONG_PTR*15) # 最多允许15条错误信息  
    ]  
class EXCEPTION_DEBUG_INFO (Structure):  
    _fields_ = [  
        ("ExceptionRecord", EXCEPTION_RECORD),  
        ("dwFirstChance",DWORD)  
    ]  
class DEBUG_EVENT_UNION(Union):  
    _fields_ = [  
        ("Exception",EXCEPTION_DEBUG_INFO), #只用到这一个，下面的不用定义了  
#        ("CreateThread",CREATE_THREAD_DEBUG_INFO),  
#        ("CreateProcessInfo",CREATE_PROCESS_DEBUG_INFO),  
#        ("ExitThread",EXIT_THREAD_DEBUG_INFO),  
#        ("ExitProcess",EXIT_PROCESS_DEBUG_INFO),  
#        ("LoadDll",LOAD_DLL_DEBUG_INFO),  
#        ("UnloadDll",UNLOAD_DLL_DEBUG_INFO),  
#        ("DebugString",OUTPUT_DEBUG_STRING_INFO),  
#        ("RipInfo",RIP_INFO)  
    ]  
class DEBUG_EVENT(Structure):  
    _fields_ = [  
        ("dwDebugEventCode",DWORD),  
        ("dwProcessId",DWORD),  
        ("dwThreadId",DWORD),  
        ("u",DEBUG_EVENT_UNION)  
    ]  
{% endhighlight %}  
DEBUG_EVENT里面需要DEBUG_EVENT_UNION的联合体，而这个联合体则用到了EXCEPTION_DEBUG_INFO的结构体，然而这个结构体也里面还用到了EXCEPTION_RECORD结构体，这个结构体的定义比较特殊，因为它的_fields_里面引用了自身结构体的指针，所以分开定义好些。  
最后，我们来测试：  
{% highlight python %} 
# -*- coding: utf-8 -*-  
import debugger.debugger as debugger  
deb = debugger.debugger()  
pid = raw_input("请输入要调试（附加）的进程ID：")  
deb.attach(int(pid))  
deb.detach()  
{% endhighlight %}  
打开任务管理器，找一个自己打开的程序的进程，读取它的pid，运行上面的代码，输出的结果：  
请输入要调试（附加）的进程ID：5968  
按回车键继续...  
完成调试，退出...  
测试说明：win7，注入windows/system32里面的程序，测试了几个都失败了。试一下其它的进程，比如QQ，附加成功！失败的原因，可能是系统设置了保护，具体原因有朋友知道的，可以在下面回复一下！感谢！  
