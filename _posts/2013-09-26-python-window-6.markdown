---
layout: post_item
status: publish
published: true
title: python的系统底层操作6——硬断点
author: jsongo
wordpress_id: 83
wordpress_url: http://jsongo.com/?p=83
date: '2013-09-26 22:27:42 +0800'
date_gmt: '2013-09-26 14:27:42 +0800'
categories: [python]
tags: [python, masterpiece]
comments: []
---
前一篇分析了软断点的原理和它的实现。对于软断点，有时候会不起作用。因为有一些软件会检测自己在内存中运行的代码的CRC校验值，一旦检测失败就会进行&ldquo;自我了断&rdquo;，而软断点就是要修改内存中运行的代码，这对于这种软件是行不通的，只能用&ldquo;硬断点&rdquo;和内存断点。本文主要分析硬断点。  


寄存器中有几个专门用来调试的寄存器，DR0--DR7，前面四个DR0到DR3用于存储所设硬件断点的地址，所以你最多只能设置4个硬件断点。DR4和DR5是保留不用的寄存器，DR6是调试状态寄存器，它记录了一次断点触发时相关的调试事件类型的信息，DR7则是一个开关，它用来控制DR0到DR3的激活状况，这个比较重要。硬件断点有三个触发条件：  
当某一特定的内存地址被 1、读写时。2、写入时。3、执行时。  
DR7的结构如下图：  
![](/img/201212/20/001858ui86smvs7t33dvs1.jpg) 
前8位分别用来标识DR0到DR3是否激活，每个寄存器对应两位，分别表示L域和G域，即局部和全局断点，这两个其中只要有一个被设成1，断点就会起作用。8&mdash;15位比较神秘，我也不知道用来干嘛。16到31位，分成4个4位，每个4位中的前两位表示断点的类型，就是上面讲的三个触发条件，后两位表示所要设的断点指定的内存中的长度，这个一般设置1个字节长就可以了。  
如上所述，三种触发条件是用两位来表示的，而它分道扬镳值如下：00表示执行，01表示写入，11表示读写。  
后两们的长度，值对应关系为：00表示1字节，01表示2字节，11表示4字节。  
硬件断点使用的是INT1，1号中断，该中断事件一般是用于硬件调试和单步调试事件。  
代码实现部分：  
其实代码很多部分都和前面的一样，硬件断点的事件也是在fire_debug_event这个函数里面触发的。它的ExceptionCode是EXCEPTION_SINGLE_STEP。  
而当EXCEPTION_SINGLE_STEP触发时，也不一定是硬件断点，这个要根据Dr6寄存器来判断，如果Dr6的前四位有一个为1，且它对应的Dr0到Dr3也被激活，那么它就是硬件断点触发的，这时我们才让它继续执行。下面的代码中，我只让断点停一次，然后就把断点清掉了。  
设置断点要分几个步骤来，首先判断Dr0到Dr3是否有可用的，有就把内存地址传给第一个可用的，然后就是设置Dr7，先把低8位中相关的位设1，激活，然后再把后16位中，相应的断点类型和内存长度设置上。而且对于要调试进程的每一个线程都要进行相同的这些设置。去掉断点的时候，也要把上面这些步骤都回滚一遍。  
主要实现的文件：debugger.py  
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
        self.hw_breakpoints = {} # 所有的硬件断点数  
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
    # 枚举被调试进程的线程  
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
                # print "------>%d" % thread_entry.th32OwnerProcessID  
                if thread_entry.th32OwnerProcessID == self.pid:  
                    thread_list.append(thread_entry.th32ThreadID)  
                next = kernel32.Thread32Next(snapshot, # 获取下一个线程，参数一样  
                                                 byref(thread_entry))  
            kernel32.CloseHandle(snapshot)  
            return thread_list  
        else:  
            return False  
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
            self.fire_debug_event()  
    # 可以通过Dr6中的BS标志位来判断这个单步事件的触发原  
    def del_hw_breakpoint(self, slot):  
        # 为调试进程的所有线程都移除断点  
        for tid in self.enumerate_threads():  
            context = self.get_thread_context(tid)  
            # 把Dr0&mdash;&mdash;Dr3中相应的Dr清空  
            if slot == 0:  
                context.Dr0 = 0  
            elif slot == 1:  
                context.Dr1 = 0  
            elif slot == 2:  
                context.Dr2 = 0  
            elif slot == 3:  
                context.Dr3 = 0  
            # 把Dr7中相应的标志去掉  
            context.Dr7 &amp;= ~(1 << (slot*2)) # 把低8位中相应的值清空  
            context.Dr7 &amp;= ~(3 << (16+slot*4)) # 把Dr7中高16中相应的触发条件标志去掉  
            context.Dr7 &amp;= ~(3 << (16+slot*4)) # 把Dr7中高16中相应的长度标志去掉  
            # 改完，写回  
            h_thread = self.open_thread(tid)  
            kernel32.SetThreadContext(h_thread, byref(context))  
        # 上面已对所有相关的线程进行处理了，接着把它清掉  
        del self.hw_breakpoints[slot]  
        return True  
    def exception_handle_single_step(self):  
        # 如果是硬件断点触发的话，就会命中下面的一个if  
        if self.context.Dr6 &amp; 0x1 and self.hw_breakpoints.has_key(0):  
             slot = 0  
        elif self.context.Dr6 &amp; 0x2 and self.hw_breakpoints.has_key(1):  
             slot = 1  
        elif self.context.Dr6 &amp; 0x4 and self.hw_breakpoints.has_key(2):  
             slot = 2  
        elif self.context.Dr6 &amp; 0x8 and self.hw_breakpoints.has_key(3):  
             slot = 3  
        else: # 不是硬件断点的单步调试事件  
            continue_status = DBG_EXCEPTION_NOT_HANDLED  
        # 打印寄存器中的所有信息  
        print "slot：%d" % slot  
        self.print_cur_context()  
        # 触发一次就移除这个断点  
        if self.del_hw_breakpoint(slot):  
            continue_status = DBG_CONTINUE # 继续  
        print "硬件断点已移除"  
        return continue_status  
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
            # print "0x%08x" % debug_event.dwDebugEventCode  
            if debug_event.dwDebugEventCode == EXCEPTION_DEBUG_EVENT:  
                # 获取异常的信息  
                exception = debug_event.u.Exception.ExceptionRecord.ExceptionCode  
                self.exception_addr = debug_event.u.Exception.ExceptionRecord.ExceptionAddress  
                print "异常代码：%d，地址0x%08x" % (exception,self.exception_addr)  
                if exception == EXCEPTION_ACCESS_VIOLATION:  
                    print "EXCEPTION_ACCESS_VIOLATION"  
                elif exception == EXCEPTION_BREAKPOINT:  
                    print "EXCEPTION_BREAKPOINT"  
                elif exception == EXCEPTION_GUARD_PAGE:  
                    print "EXCEPTION_GUARD_PAGE"  
                elif exception == EXCEPTION_SINGLE_STEP:  
                    self.exception_handle_single_step()  
                    #print "EXCEPTION_SINGLE_STEP"  
            # 这个调试事件处理结束，继续  
            # self.debugger_active = False  
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
    # 得到线程的CONTEXT信息  
    def get_thread_context(self, thread_id=None, h_thread=None):  
        context = CONTEXT()  
        context.ContextFlags = CONTEXT_FULL | CONTEXT_DEBUG_REGISTERS  
        # 获得线程句柄  
        if h_thread is None:  
            self.h_thread = self.open_thread(thread_id)  
        if kernel32.GetThreadContext(self.h_thread, byref(context)):  
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
    def set_hw_breakpoint(self, addr, length, condition):  
        if length not in(1, 2, 4): # 长度，减1后表示：0是1字节，1是2字节，3是4字节  
            return False  
        length -= 1;  
        # 检查触发的条件是否有效  
        if condition not in(HW_ACCESS, HW_EXECUTE, HW_WRITE):  
            return False  
        # 找到还没被用的硬件断点  
        if not self.hw_breakpoints.has_key(0): # Dr0  
            available = 0  
        elif not self.hw_breakpoints.has_key(1): # Dr1  
            available = 1  
        elif not self.hw_breakpoints.has_key(2): # Dr2  
            available = 2  
        elif not self.hw_breakpoints.has_key(3): # Dr3  
            available = 3  
        else:  
            return False  
        # 在调试进程的每个线程里面都进行R7的设置  
        for tid in self.enumerate_threads():  
            context = self.get_thread_context(tid)  
            # Dr0--Dr3其中一个中写入地址  
            if available == 0:  
                context.Dr0 = addr  
            elif available == 1:  
                context.Dr1 = addr  
            elif available == 2:  
                context.Dr2 = addr  
            elif available == 3:  
                context.Dr3 = addr  
            # 设置低8位，使Dr0--Dr3其中的一个有效（激活）  
            context.Dr7 |= 1 << (available*2) # 设置一位就可以  
            # 设置高16位中相应的触发条件信息  
            context.Dr7 |= condition << (16+available*4)  
            # 设置高16位中相应的长度  
            context.Dr7 |= length << (18+available*4)  
            # 到这里，context已经改完了，把它写回寄存器  
            h_thread = self.open_thread(tid)  
            kernel32.SetThreadContext(h_thread, byref(context))  
        # 把这个硬件记录到hw_breakpoints里面，字段为本函数的参数  
        self.hw_breakpoints[available] = (addr, length, condition)  
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
定义的文件：debug_define.py  
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
DBG_EXCEPTION_NOT_HANDLED = 0x80010001  
# 自定义的硬件断点类型  
HW_ACCESS = 0x0011  
HW_EXECUTE = 0x0000  
HW_WRITE = 0x0001  
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
测试文件，testMyDebugger.py  
{% highlight python %}  
# -*- coding: utf-8 -*-  
import debugger.debugger as debugger  
from debugger.debug_define import HW_EXECUTE  
deb = debugger.debugger()  
pid = raw_input("请输入要调试（附加）的进程ID：")  
deb.attach(int(pid))  
printf_addr = deb.get_func_addr("msvcrt.dll", "printf")  
deb.set_hw_breakpoint(printf_addr, 1, HW_EXECUTE)  
deb.run()  
deb.detach()  
{% endhighlight %}  
