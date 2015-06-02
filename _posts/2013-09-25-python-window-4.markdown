---
layout: post_item
status: publish
published: true
title: python的系统底层操作4&mdash;寄存器
author: jsongo
wordpress_id: 68
wordpress_url: http://jsongo.com/?p=68
date: '2013-09-25 17:06:10 +0800'
date_gmt: '2013-09-25 09:06:10 +0800'
categories: [python]
tags: [masterpiece, python]
comments: []
---

上一篇博客中， 我们可以新建一个进程，附加另一个正在运行的进程，使原来的挂起。而对于调试一个进程，我们最感兴趣的就是能看到它运行的汇编代码和寄存器上的信息，特别是后者，是我们了解当前进程运行环境的关键，而且我们可以根据寄存器上的EIP得到当前正在运行的代码所在的内存地址。。


本文将用python语言来获取正在运行的一个进程的寄存器信息。。 前面我们已经可以获得一个进程的句柄了，而对于我们所说的CPU状态是相对线程来说的，所以这里还要去获取在这个进程下运行的线程句柄。OpenThread函数可以帮我们办到：  
{% highlight cpp %}   
HANDLE WINAPI OpenThread(  
  __in  DWORD dwDesiredAccess,  
  __in  BOOL bInheritHandle,  
  __in  DWORD dwThreadId  
);  
{% endhighlight %}    
我本地MSDN地址 第一个参数，用THREAD_ALL_ACCESS，它的值是0x001F03FF，表示获取所有控制权限，其它选项可以查MSDN。有的MSDN版本没有这些相应的数值，怎么办呢？查winNT.h，这个文件在：C:\Program Files\Microsoft SDKs\Windows\v7.0A\Include 这个目录下，当然路径名不一定完全相同，读者根据这个自己查找。如果懒得找或找不到的话，我上传了：金山快盘附件：WinNT.h（491.31KB） 第二个参数NULL，第三个参数就是这个线程的ID，所以下面我们就要想办法去获取这个TID。 进程没有获取它内部线程的方法，怎么办？遍历所有正在运行的线程，找出其中PID（所在进程的ID）值与当前进程PID相等的线程。 CreateToolhelp32Snapshot ，这个API可以帮助我们实现线程的枚举，在kernel32里面：  
{% highlight cpp %}   
HANDLE WINAPI CreateToolhelp32Snapshot(  
  __in  DWORD dwFlags,  
  __in  DWORD th32ProcessID  
);  
{% endhighlight %}    
我本地MSDN地址 第一个参数用TH32CS_SNAPTHREAD，值0x00000004，表示枚举所有的thread。第二个指当前进程的PID。 返回的是一个HANDLER，通过Thread32First()来迈出整个线程枚举的第一步：  
{% highlight cpp %}   
BOOL WINAPI Thread32First(  
  __in     HANDLE hSnapshot,  
  __inout  LPTHREADENTRY32 lpte  
);  
{% endhighlight %}    
我本地MSDN地址 第一个参数就是上面取得的句柄，第二个参数是一个结构体，定义如下：  
{% highlight cpp %}   
typedef struct tagTHREADENTRY32 {  
  DWORD dwSize;  
  DWORD cntUsage;  
  DWORD th32ThreadID;  
  DWORD th32OwnerProcessID;  
  LONG  tpBasePri;  
  LONG  tpDeltaPri;  
  DWORD dwFlags;  
} THREADENTRY32, *PTHREADENTRY32;  
{% endhighlight %}    
我本地MSDN地址 对于这个结构体，我们在意的就三个字段：dwSize，这个我们赋给它这个结构体的大小就行，th32ThreadID和th32OwnerProcessID，这两个是Thread32First函数帮我们填进去的，分别表示当前枚举到的线程TID和它所在的进程PID。我们根据这个进程的PID来判断，等于当前进程的PID，则把线程TID加入到一个队列里面。为什么要等于当前进程呢？因为我们当前的进程（即我们运行的这个代码）它已经附加到要调试的进程里面了。 这里，我们得到了First Thread，第一个线程，接着，我们再用Thread32Next来往下遍历。得到的线程同上处理。最后会输出一个线程队列，这个就是我们想要的线程的集合。 接下去我们的工作就是OpenThread，到里面去看看，获取我们想要的寄存器上的数据。这里就要用到GetThreadContext来获得：  
{% highlight cpp %}   
BOOL WINAPI GetThreadContext(  
  __in     HANDLE hThread,  
  __inout  LPCONTEXT lpContext  
);  
{% endhighlight %}    
我本地MSDN地址 第一个参数是OpenThread得到的线程的句柄，第二个参数就麻烦了。它是CONTEXT结构体的指针，这个结构体在MSDN中查不到完整的定义，怎么办？ 直接看winNT.h源码。对于32位和64位的系统，它们的定义还不一样，想想也知道，因为它们的CPU工作原理有点差异。。不过，64位是兼容32位的，所以说，我们用32位的结构就可以了。 从winNT.h源码中摘录出32位的定义，如下：  
{% highlight cpp %}   
typedef struct _CONTEXT {  
    //  
    // The flags values within this flag control the contents of  
    // a CONTEXT record.  
    //  
    // If the context record is used as an input parameter, then  
    // for each portion of the context record controlled by a flag  
    // whose value is set, it is assumed that that portion of the  
    // context record contains valid context. If the context record  
    // is being used to modify a threads context, then only that  
    // portion of the threads context will be modified.  
    //  
    // If the context record is used as an IN OUT parameter to capture  
    // the context of a thread, then only those portions of the thread's  
    // context corresponding to set flags will be returned.  
    //  
    // The context record is never used as an OUT only parameter.  
    //  
    DWORD ContextFlags;  
    //  
    // This section is specified/returned if CONTEXT_DEBUG_REGISTERS is  
    // set in ContextFlags.  Note that CONTEXT_DEBUG_REGISTERS is NOT  
    // included in CONTEXT_FULL.  
    //  
    DWORD   Dr0;  
    DWORD   Dr1;  
    DWORD   Dr2;  
    DWORD   Dr3;  
    DWORD   Dr6;  
    DWORD   Dr7;  
    //  
    // This section is specified/returned if the  
    // ContextFlags word contians the flag CONTEXT_FLOATING_POINT.  
    //  
    FLOATING_SAVE_AREA FloatSave;  
    //  
    // This section is specified/returned if the  
    // ContextFlags word contians the flag CONTEXT_SEGMENTS.  
    //  
    DWORD   SegGs;  
    DWORD   SegFs;  
    DWORD   SegEs;  
    DWORD   SegDs;  
    //  
    // This section is specified/returned if the  
    // ContextFlags word contians the flag CONTEXT_INTEGER.  
    //  
    DWORD   Edi;  
    DWORD   Esi;  
    DWORD   Ebx;  
    DWORD   Edx;  
    DWORD   Ecx;  
    DWORD   Eax;  
    //  
    // This section is specified/returned if the  
    // ContextFlags word contians the flag CONTEXT_CONTROL.  
    //  
    DWORD   Ebp;  
    DWORD   Eip;  
    DWORD   SegCs;              // MUST BE SANITIZED  
    DWORD   EFlags;             // MUST BE SANITIZED  
    DWORD   Esp;  
    DWORD   SegSs;  
    //  
    // This section is specified/returned if the ContextFlags word  
    // contains the flag CONTEXT_EXTENDED_REGISTERS.  
    // The format and contexts are processor specific  
    //  
    BYTE    ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];  
} CONTEXT;  
typedef CONTEXT *PCONTEXT;  
{% endhighlight %}    
位于winNT.h的4193行开始的位置。 这里做个说明：作者的系统是win7 64位的，对于xp/win8等系统，以及32位的系统，这个文件可能会有所差异。 MSDN查不到详细信息的结构体最麻烦，你还得去找它里面的每一个字段所对应的值。还好上面的CONTEXT我们只需要给它的第一个字段赋值就可以了，其它字段GetThreadContext会帮我们填充。第一个字段：ContextFlags，在源码中查到，它的值：  
{% highlight cpp %}   
#define CONTEXT_AMD64   0x100000  
// end_wx86  
#define CONTEXT_CONTROL (CONTEXT_AMD64 | 0x1L)  
#define CONTEXT_INTEGER (CONTEXT_AMD64 | 0x2L)  
#define CONTEXT_SEGMENTS (CONTEXT_AMD64 | 0x4L)  
#define CONTEXT_FLOATING_POINT  (CONTEXT_AMD64 | 0x8L)  
#define CONTEXT_DEBUG_REGISTERS (CONTEXT_AMD64 | 0x10L)  
#define CONTEXT_FULL (CONTEXT_CONTROL | CONTEXT_INTEGER | CONTEXT_FLOATING_POINT)  
#define CONTEXT_ALL (CONTEXT_CONTROL | CONTEXT_INTEGER | CONTEXT_SEGMENTS | CONTEXT_FLOATING_POINT | CONTEXT_DEBUG_REGISTERS)  
#define CONTEXT_XSTATE (CONTEXT_AMD64 | 0x20L)  
#define CONTEXT_EXCEPTION_ACTIVE 0x8000000  
#define CONTEXT_SERVICE_ACTIVE 0x10000000  
#define CONTEXT_EXCEPTION_REQUEST 0x40000000  
#define CONTEXT_EXCEPTION_REPORTING 0x80000000  
{% endhighlight %}    
这里我们需要的值有两个： CONTEXT_FULL = 0x00010007 # 得到所有的CPU寄存器CONTEXT_DEBUG_REGISTERS = 0x00010010 # CPU的调试寄存器 到这里，基本上完事了，不过，还要记得CloseHandle(线程) ，有始有终。 最后我们把一些常用的寄存器里面的当前值打出来。所有的代码如下，和前面几篇博客一样，三个文件，一个定义结构体等，一个实现，一个测试。 定义，debug_define.py:  
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
实现，debugger.py:  
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
    def enumerate_threads(self):  
        thread_entry = THREADENTRY32()  
        thread_list = []  
        snapshot = kernel32.CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD,  
                                                     self.pid)  
        # 上面这个函数要注意help第一个字母是小写CreateToolhelp32Snapshot  
        if snapshot is not None:  
            thread_entry.dwSize = sizeof(thread_entry)  
            # 遍历  
            # 先取得第一个线程  
            next = kernel32.Thread32First(snapshot, byref(thread_entry))  
            # 循环获取属于本进程的线程  
            while next:  
                print "------&amp;gt;%d" % thread_entry.th32OwnerProcessID  
                if thread_entry.th32OwnerProcessID == self.pid:  
                    thread_list.append(thread_entry.th32ThreadID)  
                next = kernel32.Thread32Next(snapshot, # 获取下一个线程，参数一样  
                                                 byref(thread_entry))  
            kernel32.CloseHandle(snapshot)  
            return thread_list  
        else:  
            return False  
    def get_thread_context(self, thread_id=None, h_thread=None):  
        context = CONTEXT()  
        context.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS  
        # 获得线程句柄  
        h_thread = self.open_thread(thread_id)  
        if kernel32.GetThreadContext(h_thread, byref(context)):  
            kernel32.CloseHandle(h_thread)  
            return context  
        else:  
            return False  
{% endhighlight %}    
测试，testMyDebugger.py:  
{% highlight python %}  
# -*- coding: utf-8 -*-  
import debugger.debugger as debugger  
deb = debugger.debugger()  
pid = raw_input("请输入要调试（附加）的进程ID：")  
deb.attach(int(pid))  
list = deb.enumerate_threads()  
# 对于上面获得的列表中的每个线程，打印出它的寄存器信息  
if list:  
    for thread in list:  
        thread_context = deb.get_thread_context(thread)  
        if thread_context:  
            # 打印信息  
            print "====================================="  
            print "[**] 线程ID: %d" % thread  
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
deb.detach()  
{% endhighlight %}  

> 测试结果： 请输入要调试（附加）的进程ID：11912 按回车键继续... ------>0 .......... # 这里省略很长的打印出来的进程号。。。 ------>12804 ===================================== [**] 线程ID: 13124 [**] EAX: 0x007d3f80 [**] EBX: 0x00000000 [**] ECX: 0x00000000 [**] EDX: 0x00000000 ==== [**] ESI: 0x007c31b8 [**] EDI: 0x007c3188 ==== [**] EBP: 0x0018fea8 [**] ESP: 0x0018fe8c [**] EIP: 0x75de78d7 ===================================== ===================================== [**] 线程ID: 9224 [**] EAX: 0x7716f85a [**] EBX: 0x00000000 [**] ECX: 0x00000000 [**] EDX: 0x00000000 ==== [**] ESI: 0x00000000 [**] EDI: 0x00000000 ==== [**] EBP: 0x00000000 [**] ESP: 0x02abfff0 [**] EIP: 0x770e01b4 ===================================== 完成调试，退出...
