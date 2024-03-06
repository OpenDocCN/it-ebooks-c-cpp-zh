# Catchpoint

# Catchpoint

# 让 catchpoint 只触发一次

# 让 catchpoint 只触发一次

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void) {
    pid_t pid;
    int i = 0;

    for (i = 0; i < 2; i++)
    {
        pid = fork();
        if (pid < 0)
        {
            exit(1);
        }
        else if (pid == 0)
        {
            exit(0);
        }
    }
    printf("hello world\n");
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以用“`tcatch`”命令设置`catchpoint`只触发一次，以上面程序为例：

```cpp
(gdb) tcatch fork
Catchpoint 1 (fork)
(gdb) r
Starting program: /home/nan/a

Temporary catchpoint 1 (forked process 27377), 0x00000034e42acdbd in fork () from /lib64/libc.so.6
(gdb) c
Continuing.
hello world
[Inferior 1 (process 27373) exited normally]
(gdb) q 
```

可以看到当程序只在第一次调用`fork`时暂停。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 贡献者

nanxiao

# 为 fork 调用设置 catchpoint

# 为 fork 调用设置 catchpoint

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void) {
    pid_t pid;

    pid = fork();
    if (pid < 0)
    {
        exit(1);
    }
    else if (pid > 0)
    {
        exit(0);
    }
    printf("hello world\n");
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以用“`catch fork`”命令为`fork`调用设置`catchpoint`，以上面程序为例：

```cpp
(gdb) catch fork
Catchpoint 1 (fork)
(gdb) r
Starting program: /home/nan/a 

Catchpoint 1 (forked process 33499), 0x00000034e42acdbd in fork () from /lib64/libc.so.6
(gdb) bt
#0  0x00000034e42acdbd in fork () from /lib64/libc.so.6
#1  0x0000000000400561 in main () at a.c:9 
```

可以看到当`fork`调用发生后，gdb 会暂停程序的运行。
注意：目前只有 HP-UX 和 GNU/Linux 支持这个功能。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 贡献者

nanxiao

# 为 vfork 调用设置 catchpoint

# 为 vfork 调用设置 catchpoint

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

int main(void) {
    pid_t pid;

    pid = vfork();
    if (pid < 0)
    {
        exit(1);
    }
    else if (pid > 0)
    {
        exit(0);
    }
    printf("hello world\n");
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以用“`catch vfork`”命令为`vfork`调用设置`catchpoint`，以上面程序为例：

```cpp
(gdb) catch vfork
Catchpoint 1 (vfork)
(gdb) r
Starting program: /home/nan/a

Catchpoint 1 (vforked process 27312), 0x00000034e42acfc4 in vfork ()
   from /lib64/libc.so.6
(gdb) bt
#0  0x00000034e42acfc4 in vfork () from /lib64/libc.so.6
#1  0x0000000000400561 in main () at a.c:9 
```

可以看到当`vfork`调用发生后，gdb 会暂停程序的运行。
注意：目前只有 HP-UX 和 GNU/Linux 支持这个功能。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 贡献者

nanxiao

# 为 exec 调用设置 catchpoint

# 为 exec 调用设置 catchpoint

## 例子

```cpp
#include <unistd.h>

int main(void) {
    execl("/bin/ls", "ls", NULL);
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以用“`catch exec`”命令为`exec`系列系统调用设置`catchpoint`，以上面程序为例：

```cpp
(gdb) catch exec
Catchpoint 1 (exec)
(gdb) r
Starting program: /home/nan/a
process 32927 is executing new program: /bin/ls

Catchpoint 1 (exec'd /bin/ls), 0x00000034e3a00b00 in _start () from /lib64/ld-linux-x86-64.so.2
(gdb) bt
#0  0x00000034e3a00b00 in _start () from /lib64/ld-linux-x86-64.so.2
#1  0x0000000000000001 in ?? ()
#2  0x00007fffffffe73d in ?? ()
#3  0x0000000000000000 in ?? () 
```

可以看到当`execl`调用发生后，gdb 会暂停程序的运行。
注意：目前只有 HP-UX 和 GNU/Linux 支持这个功能。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 贡献者

nanxiao

# 为系统调用设置 catchpoint

# 为系统调用设置 catchpoint

## 例子

```cpp
#include <stdio.h>

int main(void)
{
    char p1[] = "Sam";
    char *p2 = "Bob";

    printf("p1 is %s, p2 is %s\n", p1, p2);
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以使用`catch syscall [name | number]`为关注的系统调用设置`catchpoint`，以上面程序为例：

```cpp
(gdb) catch syscall mmap
Catchpoint 1 (syscall 'mmap' [9])
(gdb) r
Starting program: /home/nan/a

Catchpoint 1 (call to syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2
(gdb) c
Continuing.

Catchpoint 1 (returned from syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2 
```

可以看到当`mmap`调用发生后，gdb 会暂停程序的运行。
也可以使用系统调用的编号设置`catchpoint`，仍以上面程序为例：

```cpp
(gdb) catch syscall 9
Catchpoint 1 (syscall 'mmap' [9])
(gdb) r
Starting program: /home/nan/a

Catchpoint 1 (call to syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2
(gdb) c
Continuing.

Catchpoint 1 (returned from syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2
(gdb) c
Continuing.

Catchpoint 1 (call to syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2 
```

可以看到和使用`catch syscall mmap`效果是一样的。（系统调用和编号的映射参考具体的`xml`文件，以我的系统为例，就是在`/usr/local/share/gdb/syscalls`文件夹下的`amd64-linux.xml`。）

如果不指定具体的系统调用，则会为所有的系统调用设置`catchpoint`，仍以上面程序为例：

```cpp
(gdb) catch syscall
Catchpoint 1 (any syscall)
(gdb) r
Starting program: /home/nan/a

Catchpoint 1 (call to syscall brk), 0x00000034e3a1618a in brk ()
   from /lib64/ld-linux-x86-64.so.2
(gdb) c
Continuing.

Catchpoint 1 (returned from syscall brk), 0x00000034e3a1618a in brk ()
   from /lib64/ld-linux-x86-64.so.2
(gdb)
Continuing.

Catchpoint 1 (call to syscall mmap), 0x00000034e3a16f7a in mmap64 ()
   from /lib64/ld-linux-x86-64.so.2 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Catchpoints.html).

## 贡献者

nanxiao

# 通过为 ptrace 调用设置 catchpoint 破解 anti-debugging 的程序

# 通过为 ptrace 调用设置 catchpoint 破解 anti-debugging 的程序

## 例子

```cpp
#include <sys/ptrace.h>
#include <stdio.h>

int main()                                                                      
{
        if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0 ) {
                printf("Gdb is debugging me, exit.\n");
                return 1;
        }
        printf("No debugger, continuing\n");
        return 0;
} 
```

## 技巧

有些程序不想被 gdb 调试，它们就会在程序中调用“`ptrace`”函数，一旦返回失败，就证明程序正在被 gdb 等类似的程序追踪，所以就直接退出。以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x400508: file a.c, line 6.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:6
6                       if (ptrace(PTRACE_TRACEME, 0, 0, 0) < 0 ) {
(gdb) n
7                               printf("Gdb is debugging me, exit.\n");
(gdb)
Gdb is debugging me, exit.
8                               return 1; 
```

破解这类程序的办法就是为`ptrace`调用设置`catchpoint`，通过修改`ptrace`的返回值，达到目的。仍以上面程序为例：

```cpp
(gdb) catch syscall ptrace
Catchpoint 2 (syscall 'ptrace' [101])
(gdb) r
Starting program: /data2/home/nanxiao/a

Catchpoint 2 (call to syscall ptrace), 0x00007ffff7b2be9c in ptrace () from /lib64/libc.so.6
(gdb) c
Continuing.

Catchpoint 2 (returned from syscall ptrace), 0x00007ffff7b2be9c in ptrace () from /lib64/libc.so.6
(gdb) set $rax = 0
(gdb) c
Continuing.
No debugger, continuing
[Inferior 1 (process 11491) exited normally] 
```

可以看到，通过修改`rax`寄存器的值，达到修改返回值的目的，从而让 gdb 可以继续调试程序（打印“`No debugger, continuing`”）。
详细过程，可以参见这篇文章[避開 PTRACE_TRACME 反追蹤技巧](http://blog.linux.org.tw/~jserv/archives/2011_08.html).

## 贡献者

nanxiao