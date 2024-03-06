# 信号

# 查看信号处理信息

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGINT, handler);
        signal(SIGALRM, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

用 gdb 调试程序时，可以用“`i signals`”命令（或者“`i handle`”命令，`i`是`info`命令缩写）查看 gdb 如何处理进程收到的信号:

```cpp
(gdb) i signals 
Signal        Stop      Print   Pass to program Description

SIGHUP        Yes       Yes     Yes             Hangup
SIGINT        Yes       Yes     No              Interrupt
SIGQUIT       Yes       Yes     Yes             Quit
......
SIGALRM       No        No      Yes             Alarm clock
...... 
```

第一项（`Signal`）：标示每个信号。
第二项（`Stop`）：表示被调试的程序有对应的信号发生时，gdb 是否会暂停程序。
第三项（`Print`）：表示被调试的程序有对应的信号发生时，gdb 是否会打印相关信息。
第四项（`Pass to program`）：gdb 是否会把这个信号发给被调试的程序。
第五项（`Description`）：信号的描述信息。

从上面的输出可以看到，当`SIGINT`信号发生时，gdb 会暂停被调试的程序，并打印相关信息，但不会把这个信号发给被调试的程序。而当`SIGALRM`信号发生时，gdb 不会暂停被调试的程序，也不打印相关信息，但会把这个信号发给被调试的程序。

启动 gdb 调试上面的程序，同时另起一个终端，先后发送`SIGINT`和`SIGALRM`信号给被调试的进程，输出如下：

```cpp
Program received signal SIGINT, Interrupt.
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) c
Continuing.
Receive signal: 14 
```

可以看到收到`SIGINT`时，程序暂停了，也输出了信号信息，但并没有把`SIGINT`信号交由进程处理（程序没有输出）。而收到`SIGALRM`信号时，程序没有暂停，也没有输出信号信息，但把`SIGALRM`信号交由进程处理了（程序打印了输出）。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signals.html).

## 贡献者

nanxiao

# 信号发生时是否暂停程序

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGHUP, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

用 gdb 调试程序时，可以用“`handle signal stop/nostop`”命令设置当信号发生时，是否暂停程序的执行，以上面程序为例:

```cpp
(gdb) i signals 
Signal        Stop      Print   Pass to program Description

SIGHUP        Yes       Yes     Yes             Hangup
......

(gdb) r
Starting program: /data1/nan/test 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]

Program received signal SIGHUP, Hangup.
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) c
Continuing.
Receive signal: 1 
```

可以看到，默认情况下，发生`SIGHUP`信号时，gdb 会暂停程序的执行，并打印收到信号的信息。此时需要执行`continue`命令继续程序的执行。

接下来用“`handle SIGHUP nostop`”命令设置当`SIGHUP`信号发生时，gdb 不暂停程序，执行如下：

```cpp
(gdb) handle SIGHUP nostop
Signal        Stop      Print   Pass to program Description
SIGHUP        No        Yes     Yes             Hangup
(gdb) c
Continuing.

Program received signal SIGHUP, Hangup.
Receive signal: 1 
```

可以看到，程序收到`SIGHUP`信号发生时，没有暂停，而是继续执行。

如果想恢复之前的行为，用“`handle SIGHUP stop`”命令即可。需要注意的是，设置`stop`的同时，默认也会设置`print`（关于`print`，请参见信号发生时是否打印信号信息）。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signals.html).

## 贡献者

nanxiao

# 信号发生时是否打印信号信息

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGHUP, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

用 gdb 调试程序时，可以用“`handle signal print/noprint`”命令设置当信号发生时，是否打印信号信息，以上面程序为例:

```cpp
(gdb) i signals 
Signal        Stop      Print   Pass to program Description

SIGHUP        Yes       Yes     Yes             Hangup
......

(gdb) r
Starting program: /data1/nan/test 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]

Program received signal SIGHUP, Hangup.
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) c
Continuing.
Receive signal: 1 
```

可以看到，默认情况下，发生`SIGHUP`信号时，gdb 会暂停程序的执行，并打印收到信号的信息。此时需要执行`continue`命令继续程序的执行。

接下来用“`handle SIGHUP noprint`”命令设置当`SIGHUP`信号发生时，gdb 不打印信号信息，执行如下：

```cpp
(gdb) handle SIGHUP noprint 
Signal        Stop      Print   Pass to program Description
SIGHUP        No        No      Yes             Hangup
(gdb) r
Starting program: /data1/nan/test 
[Thread debugging using libthread_db enabled]
Receive signal: 1 
```

需要注意的是，设置`noprint`的同时，默认也会设置`nostop`。可以看到，程序收到`SIGHUP`信号发生时，没有暂停，也没有打印信号信息。而是继续执行。

如果想恢复之前的行为，用“`handle SIGHUP print`”命令即可。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signals.html).

## 贡献者

nanxiao

# 信号发生时是否把信号丢给程序处理

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGHUP, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

用 gdb 调试程序时，可以用“`handle signal pass(noignore)/nopass(ignore)`”命令设置当信号发生时，是否把信号丢给程序处理.其中`pass`和`noignore`含义相同，`nopass`和`ignore`含义相同。以上面程序为例:

```cpp
(gdb) i signals 
Signal        Stop      Print   Pass to program Description

SIGHUP        Yes       Yes     Yes             Hangup
......

(gdb) r
Starting program: /data1/nan/test 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]

Program received signal SIGHUP, Hangup.
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) c
Continuing.
Receive signal: 1 
```

可以看到，默认情况下，发生`SIGHUP`信号时，gdb 会把信号丢给程序处理。

接下来用“`handle SIGHUP nopass`”命令设置当`SIGHUP`信号发生时，gdb 不把信号丢给程序处理，执行如下：

```cpp
(gdb) handle SIGHUP nopass
Signal        Stop      Print   Pass to program Description
SIGHUP        Yes       Yes     No              Hangup
(gdb) c
Continuing.

Program received signal SIGHUP, Hangup.
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) c
Continuing. 
```

可以看到，`SIGHUP`信号发生时，程序没有打印“Receive signal: 1”，说明 gdb 没有把信号丢给程序处理。

如果想恢复之前的行为，用“`handle SIGHUP pass`”命令即可。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signals.html).

## 贡献者

nanxiao

# 给程序发送信号

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGHUP, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

用 gdb 调试程序的过程中，当被调试程序停止后，可以用“`signal signal_name`”命令让程序继续运行，但会立即给程序发送信号。以上面程序为例:

```cpp
(gdb) r
`/data1/nan/test' has changed; re-reading symbols.
Starting program: /data1/nan/test 
[Thread debugging using libthread_db enabled]
^C[New Thread 1 (LWP 1)]

Program received signal SIGINT, Interrupt.
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) signal SIGHUP
Continuing with signal SIGHUP.
Receive signal: 1 
```

可以看到，当程序暂停后，执行`signal SIGHUP`命令，gdb 会发送信号给程序处理。

可以使用“`signal 0`”命令使程序重新运行，但不发送任何信号给进程。仍以上面程序为例：

```cpp
Program received signal SIGHUP, Hangup.
0xfeeeae55 in ___nanosleep () from /lib/libc.so.1
(gdb) signal 0
Continuing with no signal. 
```

可以看到，`SIGHUP`信号发生时，gdb 停住了程序，但是由于执行了“`signal 0`”命令，所以程序重新运行后，并没有收到`SIGHUP`信号。

使用`signal`命令和在 shell 环境使用`kill`命令给程序发送信号的区别在于：在 shell 环境使用`kill`命令给程序发送信号，gdb 会根据当前的设置决定是否把信号发送给进程，而使用`signal`命令则直接把信号发给进程。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signaling.html#Signaling).

## 贡献者

nanxiao

# 使用“$_siginfo”变量

## 例子

```cpp
#include <stdio.h>
#include <signal.h>

void handler(int sig);

void handler(int sig)
{
        signal(sig, handler);
        printf("Receive signal: %d\n", sig);
}

int main(void) {
        signal(SIGHUP, handler);

        while (1)
        {
                sleep(1);
        }
        return 0;
} 
```

## 技巧

在某些平台上（比如 Linux）使用 gdb 调试程序，当有信号发生时，gdb 在把信号丢给程序之前，可以通过`$_siginfo`变量读取一些额外的有关当前信号的信息，这些信息是`kernel`传给信号处理函数的。以上面程序为例:

```cpp
Program received signal SIGHUP, Hangup.
0x00000034e42accc0 in __nanosleep_nocancel () from /lib64/libc.so.6
Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.132.el6.x86_64
(gdb) ptype $_siginfo
type = struct {
    int si_signo;
    int si_errno;
    int si_code;
    union {
        int _pad[28];
        struct {...} _kill;
        struct {...} _timer;
        struct {...} _rt;
        struct {...} _sigchld;
        struct {...} _sigfault;
        struct {...} _sigpoll;
    } _sifields;
}
(gdb) ptype $_siginfo._sifields._sigfault
type = struct {
    void *si_addr;
}
(gdb) p $_siginfo._sifields._sigfault.si_addr
$1 = (void *) 0x850e 
```

我们可以了解`$_siginfo`变量里每个成员的类型，并且可以读到成员的值。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Signaling.html#Signaling).

## 贡献者

nanxiao