---
layout: post_item
status: publish
published: true
title: python的系统底层操作5&mdash;软断点
author: jsongo
wordpress_id: 77
wordpress_url: http://jsongo.com/?p=77
date: '2013-09-26 00:32:47 +0800'
date_gmt: '2013-09-25 16:32:47 +0800'
categories: [python]
tags: [python]
comments: []
---
&nbsp;&nbsp;进（线）程中的数据可能是时刻变化的，为了能得到它们内部的数据，或者你想知道当代码运行到某些地方时，内存或CPU里面都有些什么数据，这时你就要先让进（线）程停下来，这个在前面的博客中已经提到，更进一步地，我们要它在我们想要的地方停下来，这就要涉及到断点了。平时经常写代码调代码的朋友对断点应该很熟悉了，下文会简单地分析这些断点的分类及原理。


1、软断点  
假如我们要访问的指令地址为0x748cc5b9，指令8BC3（mov eax, ebx），为了使CPU执行到此地址的命令时，停下来，我们把8BC3的前一个字节替换成0xCC，它是INT3中断指令，它告诉CPU暂停执行当前的进程，替换后，变成CCC3，把原来的8B保存到另一个变量中，要在中断后继续执行就把8B替换回去。  
根据这个原理，我们可以给出一个指令的具体地址，让线程跑到这里的时候停下来。当然，随便说一个地址也不一定会起作用，因为它可能是数据地址甚至不可访问的地址等等，所以这种情况比较少，当然你也可以用一个其它的工具去得到某个指令所在的内存地址，不过我们一般是给出一个函数的首地址，让CPU在这个函数要执行时，停下来。GetProcAddress，这个API可以帮我们得到某个函数的首地址：  
{% highlight cpp %}    
FARPROC GetProcAddress(  
    HMODULE hModule, 	// DLL模块句柄  
    LPCSTR lpProcName 	// 函数名  
);  
{% endhighlight %}   
在MSDN中搜不到具体的定义，不过它很好理解，它带两个参数，第一个是模块的句柄，第二个是函数名，字符串。对于模块的句柄，可以用GetModuleHandle来取得。它只有一个参数，就是模块名，很简单。  
接下来的事就是从得到的这个函数首地址中，取得第一个字节，把它存起来，然后替换成0xCC。之后再把它替换回去。  
原理很简单，下面的例子，在系统的printf函数首地址下个断点，直接上代码：  
具体实现，debugger.py  
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
        self.h_process = None
        self.context = None
        self.breakpoints = {} # 所有的断点数
        self.first_breakpoint = True
        
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
            # self.run()
        else:
            print "进程附加失败！"
            
        
    def run(self):
        while self.debugger_active == True:
            print '--- *** ---'
            self.fire_debug_event()
    
    # 等待触发调试事件。。
    def fire_debug_event(self):
        debug_event = DEBUG_EVENT()
        continue_status = DBG_CONTINUE
        
        # 接下去就是等待调试的事件被触发
        if kernel32.WaitForDebugEvent(byref(debug_event), INFINITE): #一直等
            # 到这里，说明调试的条件被触发了
            # 。。。（在这里做些调试相关的事情）
            self.debug_event = debug_event
            self.h_thread = self.open_thread(debug_event.dwThreadId)
            self.context = self.get_thread_context(h_thread=self.h_thread)
            # print "被调试的进程：%d，线程：%d" % (debug_event.dwProcessId,debug_event.dwThreadId)
            print "0x%08x" % debug_event.dwDebugEventCode
            if debug_event.dwDebugEventCode == EXCEPTION_DEBUG_EVENT:
                # 获取异常的信息
                exception = debug_event.u.Exception.ExceptionRecord.ExceptionCode
                self.exception_addr = debug_event.u.Exception.ExceptionRecord.ExceptionAddress
                print "异常代码：%d，地址0x%08x" % (exception,self.exception_addr)
                if exception == EXCEPTION_ACCESS_VIOLATION:
                    print "EXCEPTION_ACCESS_VIOLATION"
                elif exception == EXCEPTION_BREAKPOINT:
                    continue_status = self.exception_handle_breakpoint()
                    # print "EXCEPTION_BREAKPOINT"                    
                elif exception == EXCEPTION_GUARD_PAGE:
                    print "EXCEPTION_GUARD_PAGE"
                elif exception == EXCEPTION_SINGLE_STEP:
                    print "EXCEPTION_SINGLE_STEP"  
            # 这个调试事件处理结束，继续
            # self.debugger_active = False
            kernel32.ContinueDebugEvent(debug_event.dwProcessId,
                                        debug_event.dwThreadId,
                                        continue_status)
    
    # 控制断点事件
    def exception_handle_breakpoint(self):
        print "断点地址：0x%08x" % self.exception_addr
        # 打印出CPU中的数据
        self.print_cur_context()
        if not self.breakpoints.has_key(self.exception_addr):
            if self.first_breakpoint:
                self.first_breakpoint = False
                print "第一个断点地址"
                return DBG_CONTINUE
        else:
            print "用户定义的断点"
            # 把INT3断点去掉
            self.write_process_memory(self.exception_addr,
                       self.breakpoints[self.exception_addr])
            
            # 把EIP往回调动一个字节，因为它多读了一个INT3中断0xCC
            self.context.Eip -= 1
            
            kernel32.SetThreadContext(self.h_thread,byref(self.context))
            
            continue_status = DBG_CONTINUE

        return continue_status
    
    # 解除对进程的附加
    def detach(self):
        if kernel32.DebugActiveProcessStop(self.pid):
            print "完成调试，退出..."
            return True
        else:
            print "调试过程中出错..."
            return False
        
    # 打开线程
    def open_thread(self, thread_id):
        h_thread = kernel32.OpenThread(THREAD_ALL_ACCESS,
                                       None,
                                       thread_id)
        if h_thread is not None:
            return h_thread
        else:
            print "获取线程ID时出错！"
            return False
    
    # 得到线程的CONTEXT信息
    def get_thread_context(self, thread_id=None, h_thread=None):
        context = CONTEXT()
        context.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS
        
        # 获得线程句柄
        if h_thread is None:
            self.h_thread = self.open_thread(thread_id)
        if kernel32.GetThreadContext(h_thread, byref(context)):
            # kernel32.CloseHandle(h_thread)
            return context
        else:
            return False
    
    # 打印寄存器信息（CONTEXT）
    def print_cur_context(self):
        thread_context = self.context
        if thread_context:
            # 打印信息
            print "=====================================" 
            print "[**] 线程ID: %d" % self.h_process
            print "[**] EAX: 0x%08x" % thread_context.Eax 
            print "[**] EBX: 0x%08x" % thread_context.Ebx 
            print "[**] ECX: 0x%08x" % thread_context.Ecx 
            print "[**] EDX: 0x%08x" % thread_context.Edx 
            print "===="
            print "[**] ESI: 0x%08x" % thread_context.Esi 
            print "[**] EDI: 0x%08x" % thread_context.Edi 
            print "===="
            print "[**] EBP: 0x%08x" % thread_context.Ebp 
            print "[**] ESP: 0x%08x" % thread_context.Esp 
            print "[**] EIP: 0x%08x" % thread_context.Eip 
            print "====================================="
    
    # 读取线程内存中的数据
    def read_process_memory(self, addr, length):
        data = ""
        read_buf = create_string_buffer(length)
        cnt = c_ulong(0)
        
        if kernel32.ReadProcessMemory(self.h_process,
                                          addr,
                                          read_buf,
                                          length,
                                          byref(cnt)):
            data += read_buf.raw
            return data 
        else:
            return False
        
    # 往线程内存中写数据
    def write_process_memory(self, addr, data):
        cnt = c_ulong(0)
        length = len(data)
        c_data = c_char_p(data[cnt.value])
        
        if kernel32.WriteProcessMemory(self.h_process,
                                       addr,
                                       c_data,
                                       length,
                                       byref(cnt)):
            return True
        else:
            return False
    
    # 设置断点
    def set_breakpoint(self, addr):
        if not self.breakpoints.has_key(addr): # 在这个地址上还没加过断点
            try:
                # 把该地址当前的字节数据备份一下
                ori_byte = self.read_process_memory(addr, 1)
                # 写入INT3中断，0xCC
                self.write_process_memory(addr,"\xCC")
                # 记录到breakpoints字典里面
                self.breakpoints[addr] = ori_byte
            except:
                return False
        return True
    
    # 要下断点，得找一个地址，一般是一个函数的首地址
    # 获取函数首地址的方法：
    def get_func_addr(self, dllName, # dllName函数所在dll模块的名称
                      funcName): # funcName函数名称
        handle = kernel32.GetModuleHandleA(dllName)
        addr = kernel32.GetProcAddress(handle, funcName)
        kernel32.CloseHandle(handle)
        print "得到的地址：0x%08x" % addr
        return addr 
{% endhighlight %}   
然后我们自己写个简单的循环打印的程序，里面调用系统的printf函数，专用来调试测试  
{% highlight python %}  
from ctypes import *  
import time  
msvcrt = cdll.msvcrt  
cnt = 0  
while True:  
    msvcrt.printf("going...%d\n" , cnt)  
    time.sleep(3)  
    cnt += 1  
{% endhighlight %}  

测试文件，testMyDebugger.py： 

{% highlight python %}  
# -*- coding: utf-8 -*-  
import debugger.debugger as debugger  
deb = debugger.debugger()  
pid = raw_input("请输入要调试（附加）的进程ID：")  
deb.attach(int(pid))  
printf_addr = deb.get_func_addr("msvcrt.dll", "printf")  
deb.set_breakpoint(printf_addr)  
deb.run()  
deb.detach()  
{% endhighlight %} 

还有结构体、联合体、常量等的定义，在debug_define.py里面：  
{% highlight python %} 
# -*- coding: utf-8 -*-
from ctypes import *

# 定义window编程中常用的类型
WORD = c_ushort
DWORD = c_ulong
LPBYTE = POINTER(c_ubyte)
LPTSTR = POINTER(c_char)
HANDLE = c_void_p
PVOID = c_void_p
LPVOID = c_void_p
ULONG_PTR = c_ulong
SIZE_T = c_ulong
BYTE = c_ubyte

# 常量
DEBUG_PROCESS = 0x0001 
CREATE_NEW_CONSOLE = 0x0010
PROCESS_ALL_ACCESS = 0x001F0FFF
THREAD_ALL_ACCESS   = 0x001F03FF
INFINITE = 0xFFFFFFFF
DBG_CONTINUE = 0x00010002
STARTF_USESHOWWINDOW = 0x0001
SW_HIDE = 0
TH32CS_SNAPTHREAD = 0x00000004
CONTEXT_FULL = 0x00010007 # 得到所有的CPU寄存器
CONTEXT_DEBUG_REGISTERS = 0x00010010 # CPU的调试寄存器

# Exception
EXCEPTION_DEBUG_EVENT = 0x0001
EXCEPTION_ACCESS_VIOLATION = 0xC0000005
EXCEPTION_BREAKPOINT = 0x80000003
EXCEPTION_GUARD_PAGE = 0x80000001
EXCEPTION_SINGLE_STEP = 0x80000004

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
 
class THREADENTRY32(Structure):
    _fields_ = [
        ("dwSize",DWORD),
        ("cntUsage",DWORD),
        ("th32ThreadID",DWORD),
        ("th32OwnerProcessID",DWORD),
        ("tpBasePri",DWORD), # 本来是LONG，它和DWORD一样的位数
        ("tpDeltaPri",DWORD),  # LONG
        ("dwFlags",DWORD)
    ]
    
# Used by the CONTEXT structure
class FLOATING_SAVE_AREA(Structure):
    _fields_ = [   
        ("ControlWord", DWORD),
        ("StatusWord", DWORD),
        ("TagWord", DWORD),
        ("ErrorOffset", DWORD),
        ("ErrorSelector", DWORD),
        ("DataOffset", DWORD),
        ("DataSelector", DWORD),
        ("RegisterArea", BYTE * 80),
        ("Cr0NpxState", DWORD),
	]
class CONTEXT(Structure):
    _fields_ = [
        ("ContextFlags", DWORD),
        ("Dr0", DWORD),
        ("Dr1", DWORD),
        ("Dr2", DWORD),
        ("Dr3", DWORD),
        ("Dr6", DWORD),
        ("Dr7", DWORD),
        ("FloatSave", FLOATING_SAVE_AREA),
        ("SegGs", DWORD),
        ("SegFs", DWORD),
        ("SegEs", DWORD),
        ("SegDs", DWORD),
        ("Edi", DWORD),
        ("Esi", DWORD),
        ("Ebx", DWORD),
        ("Edx", DWORD),
        ("Ecx", DWORD),
        ("Eax", DWORD),
        ("Ebp", DWORD),
        ("Eip", DWORD),
        ("SegCs", DWORD),
        ("EFlags", DWORD),
        ("Esp", DWORD),
        ("SegSs", DWORD),
        ("ExtendedRegisters", BYTE * 512)
    ]
{% endhighlight %}   
结果：  

> 请输入要调试（附加）的进程ID：8516  
> 得到的地址：0x748cc5b9  
> --- *** ---  
> 0x00000003  
> --- *** ---  
> 0x00000006  
> --- *** ---  
> 0x00000006  
> --- *** ---  
> .........  
> .........  
> ... #这里一共有22个0x00000006  
> --- *** ---  
> 0x00000002  
> --- *** ---  
> 0x00000001  
> 异常代码：2147483651，地址0x76fd000c  
> 断点地址：0x76fd000c  
> =====================================  
> [**] 线程ID: 148  
> [**] EAX: 0x7efda000  
> [**] EBX: 0x00000000  
> [**] ECX: 0x00000000  
> [**] EDX: 0x7705f85a  
> ====  
> [**] ESI: 0x00000000  
> [**] EDI: 0x00000000  
> ====  
> [**] EBP: 0x0258ff88  
> [**] ESP: 0x0258ff5c  
> [**] EIP: 0x76fd000d  
> =====================================  
> 第一个断点地址  
> --- *** ---  
> 0x00000004  
> --- *** ---  
> 0x00000001  
> 异常代码：2147483651，地址0x748cc5b9  
> 断点地址：0x748cc5b9  
> =====================================  
> [**] 线程ID: 148  
> [**] EAX: 0x0027fb6c  
> [**] EBX: 0x0027fb01  
> [**] ECX: 0x00000001  
> [**] EDX: 0x0027facc  
> ====  
> [**] ESI: 0x0027fad4  
> [**] EDI: 0x00378ad8  
> ====  
> [**] EBP: 0x0027fad8  
> [**] ESP: 0x0027fac8  
> [**] EIP: 0x748cc5ba  
> =====================================  

用户定义的断点  
--- *** ---  
打印出来的下面这种值，  
--- *** ---  
0x00000003  
是DEBUG_EVENT的dwDebugEventCode字段，它的值有：  
{% highlight cpp %}    
#define EXCEPTION_DEBUG_EVENT 1  
#define CREATE_THREAD_DEBUG_EVENT 2  
#define CREATE_PROCESS_DEBUG_EVENT 3  
#define EXIT_THREAD_DEBUG_EVENT 4  
#define EXIT_PROCESS_DEBUG_EVENT 5  
#define LOAD_DLL_DEBUG_EVENT 6  
#define UNLOAD_DLL_DEBUG_EVENT 7  
#define OUTPUT_DEBUG_STRING_EVENT 8  
#define RIP_EVENT 9  
{% endhighlight %}   
它的定义在WinBase.h里面，和WinNT.h同一目录，具体地址C:\Program Files\Microsoft SDKs\Windows\v7.0A\Include（我的机上上的路径，各大机子不尽相同），找不到的话，我上传了：金山快盘附件：WinBase.h（333.15KB）  
&nbsp;&nbsp;大家可以对应上面这些值去看程序运行的结果，先创建调试进程，再加载相应的模块，再创建调试线程，然后触发异常，结束线程，最后再触发一次断点。这里为什么会有两个断点触发呢？在上面的exception_handle_breakpoint函数中，对第一次触发事件直接跳过，第二次触发时，再把原字节写回INT3中断0xCC所在的地方，所以就有两次触发，当然你也可以在第一次触发时把INT3断点信息替换掉。上面提到过&ldquo;异常&rdquo;这两个字，断点也是一种异常，这些异常的定义在WinBase.h里面，从78行开始。而它们的值则在WinNT.h里面定义，从2005行开始，两个文件综合起来查询来写这个程序。  
