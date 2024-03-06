# 多进程/线程

# 多进程/线程

# 调试已经运行的进程

# 调试已经运行的进程

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>
void *thread_func(void *p_arg)
{
        while (1)
        {
                printf("%s\n", (char*)p_arg);
                sleep(10);
        }
}
int main(void)
{
        pthread_t t1, t2;

        pthread_create(&t1, NULL, thread_func, "Thread 1");
        pthread_create(&t2, NULL, thread_func, "Thread 2");

        sleep(1000);
        return;
} 
```

## 技巧

调试已经运行的进程有两种方法：一种是 gdb 启动时，指定进程的 ID：gdb program processID（也可以用-p 或者--pid 指定进程 ID，例如：gdb program -p=10210）。以上面代码为例，用“ps”命令已经获得进程 ID 为 10210：

```cpp
bash-3.2# gdb -q a 10210
Reading symbols from /data/nan/a...done.
Attaching to program `/data/nan/a', process 10210
[New process 10210]
Retry #1:
Retry #2:
Retry #3:
Retry #4:
Reading symbols from /usr/lib/libc.so.1...(no debugging symbols found)...done.
[Thread debugging using libthread_db enabled]
[New LWP    3        ]
[New LWP    2        ]
[New Thread 1 (LWP 1)]
[New Thread 2 (LWP 2)]
[New Thread 3 (LWP 3)]
Loaded symbols for /usr/lib/libc.so.1
Reading symbols from /lib/ld.so.1...(no debugging symbols found)...done.
Loaded symbols for /lib/ld.so.1
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /usr/lib/libc.so.1
(gdb) bt
#0  0xfeeeae55 in ___nanosleep () from /usr/lib/libc.so.1
#1  0xfeedcae4 in sleep () from /usr/lib/libc.so.1
#2  0x080509ef in main () at a.c:17 
```

如果嫌每次 ps 查看进程号比较麻烦，请尝试如下脚本

```cpp
# 保存为 xgdb.sh（添加可执行权限）
# 用法 xgdb.sh a 
prog_bin=$1
running_name=$(basename $prog_bin)
pid=$(/sbin/pidof $running_name)
gdb attach $pid 
```

另一种是先启动 gdb，然后用“attach”命令“附着”在进程上：

```cpp
bash-3.2# gdb -q a
Reading symbols from /data/nan/a...done.
(gdb) attach 10210
Attaching to program `/data/nan/a', process 10210
[New process 10210]
Retry #1:
Retry #2:
Retry #3:
Retry #4:
Reading symbols from /usr/lib/libc.so.1...(no debugging symbols found)...done.
[Thread debugging using libthread_db enabled]
[New LWP    3        ]
[New LWP    2        ]
[New Thread 1 (LWP 1)]
[New Thread 2 (LWP 2)]
[New Thread 3 (LWP 3)]
Loaded symbols for /usr/lib/libc.so.1
Reading symbols from /lib/ld.so.1...(no debugging symbols found)...done.
Loaded symbols for /lib/ld.so.1
[Switching to Thread 1 (LWP 1)]
0xfeeeae55 in ___nanosleep () from /usr/lib/libc.so.1
(gdb) bt
#0  0xfeeeae55 in ___nanosleep () from /usr/lib/libc.so.1
#1  0xfeedcae4 in sleep () from /usr/lib/libc.so.1
#2  0x080509ef in main () at a.c:17 
```

如果不想继续调试了，可以用“detach”命令“脱离”进程：

```cpp
(gdb) detach
Detaching from program: /data/nan/a, process 10210
(gdb) bt
No stack. 
```

详情参见[gdb 手册](https://sourceware.org/gdb/current/onlinedocs/gdb/Attach.html#index-attach)

## 贡献者

nanxiao

# 调试子进程

# 调试子进程

## 例子

```cpp
#include <stdio.h>
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

在调试多进程程序时，gdb 默认会追踪父进程。例如：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x40055c: file a.c, line 8.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:8
8               pid = fork();
(gdb) n
9               if (pid < 0)
(gdb) hello world

13              else if (pid > 0)
(gdb)
15                      exit(0);
(gdb)
[Inferior 1 (process 12786) exited normally] 
```

可以看到程序执行到第 15 行：父进程退出。

如果要调试子进程，要使用如下命令：“set follow-fork-mode child”，例如：

```cpp
(gdb) set follow-fork-mode child
(gdb) start
Temporary breakpoint 1 at 0x40055c: file a.c, line 8.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:8
8               pid = fork();
(gdb) n
[New process 12241]
[Switching to process 12241]
9               if (pid < 0)
(gdb)
13              else if (pid > 0)
(gdb)
17              printf("hello world\n");
(gdb)
hello world
18              return 0; 
```

可以看到程序执行到第 17 行：子进程打印“hello world”。

这个命令目前 Linux 支持，其它很多操作系统都不支持，使用时请注意。参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Forks.html)

## 贡献者

nanxiao

# 同时调试父进程和子进程

# 同时调试父进程和子进程

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    pid_t pid;

    pid = fork();
    if (pid < 0)
    {
        exit(1);
    }
    else if (pid > 0)
    {
        printf("Parent\n");
        exit(0);
    }
    printf("Child\n");
    return 0;
} 
```

## 技巧

在调试多进程程序时，gdb 默认只会追踪父进程的运行，而子进程会独立运行，gdb 不会控制。以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x40055c: file a.c, line 7.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:7
7           pid = fork();
(gdb) n
8           if (pid < 0)
(gdb) Child

12          else if (pid > 0)
(gdb)
14              printf("Parent\n");
(gdb)
Parent
15              exit(0); 
```

可以看到当单步执行到第 8 行时，程序打印出“Child” ，证明子进程已经开始独立运行。

如果要同时调试父进程和子进程，可以使用“`set detach-on-fork off`”（默认`detach-on-fork`是`on`）命令，这样 gdb 就能同时调试父子进程，并且在调试一个进程时，另外一个进程处于挂起状态。仍以上面程序为例：

```cpp
(gdb) set detach-on-fork off
(gdb) start
Temporary breakpoint 1 at 0x40055c: file a.c, line 7.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:7
7           pid = fork();
(gdb) n
[New process 1050]
8           if (pid < 0)
(gdb)
12          else if (pid > 0)
(gdb) i inferior
  Num  Description       Executable
  2    process 1050      /data2/home/nanxiao/a
* 1    process 1046      /data2/home/nanxiao/a
(gdb) n
14              printf("Parent\n");
(gdb) n
Parent
15              exit(0);
(gdb)
[Inferior 1 (process 1046) exited normally]
(gdb)
The program is not being run.
(gdb) i inferiors
  Num  Description       Executable
  2    process 1050      /data2/home/nanxiao/a
* 1    <null>            /data2/home/nanxiao/a
(gdb) inferior 2
[Switching to inferior 2 [process 1050] (/data2/home/nanxiao/a)]
[Switching to thread 2 (process 1050)]
#0  0x00007ffff7af6cad in fork () from /lib64/libc.so.6
(gdb) bt
#0  0x00007ffff7af6cad in fork () from /lib64/libc.so.6
#1  0x0000000000400561 in main () at a.c:7
(gdb) n
Single stepping until exit from function fork,
which has no line number information.
main () at a.c:8
8           if (pid < 0)
(gdb)
12          else if (pid > 0)
(gdb)
17          printf("Child\n");
(gdb)
Child
18          return 0;
(gdb) 
```

在使用“`set detach-on-fork off`”命令后，用“`i inferiors`”（`i`是`info`命令缩写）查看进程状态，可以看到父子进程都在被 gdb 调试的状态，前面显示“*”是正在调试的进程。当父进程退出后，用“`inferior infno`”切换到子进程去调试。

这个命令目前 Linux 支持，其它很多操作系统都不支持，使用时请注意。参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Forks.html)

此外，如果想让父子进程都同时运行，可以使用“`set schedule-multiple on`”（默认`schedule-multiple`是`off`）命令，仍以上述代码为例：

```cpp
(gdb) set detach-on-fork off
(gdb) set schedule-multiple on
(gdb) start
Temporary breakpoint 1 at 0x40059c: file a.c, line 7.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:7
7           pid = fork();
(gdb) n
[New process 26597]
Child 
```

可以看到打印出了“Child”，证明子进程也在运行了。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/All_002dStop-Mode.html#All_002dStop-Mode)

## 贡献者

nanxiao

# 查看线程信息

# 查看线程信息

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>
void *thread_func(void *p_arg)
{
        while (1)
        {
                printf("%s\n", (char*)p_arg);
                sleep(10);
        }
}
int main(void)
{
        pthread_t t1, t2;

        pthread_create(&t1, NULL, thread_func, "Thread 1");
        pthread_create(&t2, NULL, thread_func, "Thread 2");

        sleep(1000);
        return;
} 
```

## 技巧

用 gdb 调试多线程程序，可以用“i threads”命令（i 是 info 命令缩写）查看所有线程的信息，以上面程序为例（运行平台为 Linux，CPU 为 X86_64）:

```cpp
 (gdb) i threads
  Id   Target Id         Frame
  3    Thread 0x7ffff6e2b700 (LWP 31773) 0x00007ffff7915911 in clone () from /lib64/libc.so.6
  2    Thread 0x7ffff782c700 (LWP 31744) 0x00007ffff78d9bcd in nanosleep () from /lib64/libc.so.6
* 1    Thread 0x7ffff7fe9700 (LWP 31738) main () at a.c:18 
```

第一项（Id）：是 gdb 标示每个线程的唯一 ID：1，2 等等。
第二项（Target Id）：是具体系统平台用来标示每个线程的 ID，不同平台信息可能会不同。 像当前 Linux 平台显示的就是： Thread 0x7ffff6e2b700 (LWP 31773)。
第三项（Frame）：显示的是线程执行到哪个函数。
前面带“*”表示的是“current thread”，可以理解为 gdb 调试多线程程序时，选择的一个“默认线程”。

再以 Solaris 平台（CPU 为 X86_64）为例，可以看到显示信息会略有不同：

```cpp
(gdb) i threads
[New Thread 2 (LWP 2)]
[New Thread 3 (LWP 3)]
  Id   Target Id         Frame
  6    Thread 3 (LWP 3)  0xfeec870d in _thr_setup () from /usr/lib/libc.so.1
  5    Thread 2 (LWP 2)  0xfefc9661 in elf_find_sym () from /usr/lib/ld.so.1
  4    LWP    3          0xfeec870d in _thr_setup () from /usr/lib/libc.so.1
  3    LWP    2          0xfefc9661 in elf_find_sym () from /usr/lib/ld.so.1
* 2    Thread 1 (LWP 1)  main () at a.c:18
  1    LWP    1          main () at a.c:18 
```

也可以用“i threads [Id...]”指定打印某些线程的信息，例如：

```cpp
 (gdb) i threads 1 2
  Id   Target Id         Frame
  2    Thread 0x7ffff782c700 (LWP 12248) 0x00007ffff78d9bcd in nanosleep () from /lib64/libc.so.6
* 1    Thread 0x7ffff7fe9700 (LWP 12244) main () at a.c:18 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Threads.html).

## 贡献者

nanxiao

# 在 Solaris 上使用 maintenance 命令查看线程信息

# 在 Solaris 上使用 maintenance 命令查看线程信息

## 技巧

用 gdb 调试多线程程序时，如果想查看线程信息，可以使用“i threads”命令（i 是 info 命令缩写），例如:

```cpp
(gdb) i threads
106 process 2689429      0xff04af84 in __lwp_park () from /lib/libc.so.1
105 process 2623893      0xff04af84 in __lwp_park () from /lib/libc.so.1
104 process 2558357      0xff04af84 in __lwp_park () from /lib/libc.so.1
103 process 2492821      0xff04af84 in __lwp_park () from /lib/libc.so.1 
```

在 Solaris 操作系统上，gdb 为 Solaris 量身定做了一个查看线程信息的命令：“maint info sol-threads”（maint 是 maintenance 命令缩写），例如:

```cpp
(gdb) maint info sol-threads
user   thread #1, lwp 1, (active)
user   thread #2, lwp 2, (active)    startfunc: monitor_thread
user   thread #3, lwp 3, (asleep)    startfunc: mem_db_thread
- Sleep func: 0x000aa32c 
```

可以看到相比于 info 命令，maintenance 命令显示了更多信息。例如线程当前状态（active，asleep），入口函数（startfunc）等。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Threads.html)

## 贡献者

nanxiao

# 不显示线程启动和退出信息

# 不显示线程启动和退出信息

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>

void *thread_func(void *p_arg)
{
       sleep(10);
}

int main(void)
{
        pthread_t t1, t2;

        pthread_create(&t1, NULL, thread_func, "Thread 1");
        pthread_create(&t2, NULL, thread_func, "Thread 2");

        sleep(1000);
        return;
} 
```

## 技巧

默认情况下，gdb 检测到有线程产生和退出时，会打印提示信息，以上面程序为例:

```cpp
(gdb) r
Starting program: /data/nan/a
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[New LWP    2        ]
[New LWP    3        ]
[LWP    2         exited]
[New Thread 2        ]
[LWP    3         exited]
[New Thread 3        ] 
```

如果不想显示这些信息，可以使用“`set print thread-events off`”命令，这样当有线程产生和退出时，就不会打印提示信息：

```cpp
(gdb) set print thread-events off
(gdb) r
Starting program: /data/nan/a
[Thread debugging using libthread_db enabled] 
```

可以看到不再打印相关信息。

这个命令有些平台不支持，使用时需注意。参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Threads.html).

## 贡献者

nanxiao

# 只允许一个线程运行

# 只允许一个线程运行

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>
int a = 0;
int b = 0;
void *thread1_func(void *p_arg)
{
        while (1)
        {
                a++;
                sleep(1);
        }
}

void *thread2_func(void *p_arg)
{
        while (1)
        {
                b++;
                sleep(1);
        }
}

int main(void)
{
        pthread_t t1, t2;

        pthread_create(&t1, NULL, thread1_func, "Thread 1");
        pthread_create(&t2, NULL, thread2_func, "Thread 2");

        sleep(1000);
        return;
} 
```

## 技巧

用 gdb 调试多线程程序时，一旦程序断住，所有的线程都处于暂停状态。此时当你调试其中一个线程时（比如执行“`step`”，“`next`”命令），所有的线程都会同时执行。以上面程序为例:

```cpp
(gdb) b a.c:9
Breakpoint 1 at 0x400580: file a.c, line 9.
(gdb) r
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff782c700 (LWP 17368)]
[Switching to Thread 0x7ffff782c700 (LWP 17368)]

Breakpoint 1, thread1_func (p_arg=0x400718) at a.c:9
9                       a++;
(gdb) p b
$1 = 0
(gdb) s
10                      sleep(1);
(gdb) s
[New Thread 0x7ffff6e2b700 (LWP 17369)]
11              }
(gdb)

Breakpoint 1, thread1_func (p_arg=0x400718) at a.c:9
9                       a++;
(gdb)
10                      sleep(1);
(gdb) p b
$2 = 3 
```

`thread1_func`更新全局变量`a`的值，`thread2_func`更新全局变量`b`的值。我在`thread1_func`里`a++`语句打上断点，当断点第一次命中时，打印`b`的值是`0`，在单步调试`thread1_func`几次后，`b`的值变成`3`，证明在单步调试`thread1_func`时，`thread2_func`也在执行。
如果想在调试一个线程时，让其它线程暂停执行，可以使用“`set scheduler-locking on`”命令：

```cpp
(gdb) b a.c:9
Breakpoint 1 at 0x400580: file a.c, line 9.
(gdb) r
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff782c700 (LWP 19783)]
[Switching to Thread 0x7ffff782c700 (LWP 19783)]

Breakpoint 1, thread1_func (p_arg=0x400718) at a.c:9
9                       a++;
(gdb) set scheduler-locking on
(gdb) p b
$1 = 0
(gdb) s
10                      sleep(1);
(gdb)
11              }
(gdb)

Breakpoint 1, thread1_func (p_arg=0x400718) at a.c:9
9                       a++;
(gdb)
10                      sleep(1);
(gdb)
11              }
(gdb) p b
$2 = 0 
```

可以看到在单步调试`thread1_func`几次后，`b`的值仍然为`0`，证明在在单步调试`thread1_func`时，`thread2_func`没有执行。

此外，“`set scheduler-locking`”命令除了支持`off`和`on`模式外（默认是`off`），还有一个`step`模式。含义是：当用"`step`"命令调试线程时，其它线程不会执行，但是用其它命令（比如"`next`"）调试线程时，其它线程也许会执行。

这个命令依赖于具体操作系统的调度策略，使用时需注意。参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/All_002dStop-Mode.html#All_002dStop-Mode).

## 贡献者

nanxiao

# 使用“$_thread”变量

# 使用“$_thread”变量

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>

int a = 0;

void *thread1_func(void *p_arg)
{
        while (1)
        {
                a++;
                sleep(10);
        }
}

void *thread2_func(void *p_arg)
{
        while (1)
        {
                a++;
                sleep(10);
        }
}

int main(void)
{
        pthread_t t1, t2;

        pthread_create(&t1, NULL, thread1_func, "Thread 1");
        pthread_create(&t2, NULL, thread2_func, "Thread 2");

        sleep(1000);
        return;
} 
```

## 技巧

gdb 从 7.2 版本引入了`$_thread`这个“`convenience variable`”，用来保存当前正在调试的线程号。这个变量在写断点命令或是命令脚本时会很有用。以上面程序为例:

```cpp
(gdb) wa a
Hardware watchpoint 2: a
(gdb) command 2
Type commands for breakpoint(s) 2, one per line.
End with a line saying just "end".
>printf "thread id=%d\n", $_thread
>end 
```

首先设置了观察点：“wa a”（`wa`是`watch`命令缩写），也就是当`a`的值发生变化时，程序会暂停，接下来在`commands`语句中打印线程号。
然后继续执行程序：

```cpp
(gdb) c
Continuing.
[New Thread 0x7ffff782c700 (LWP 20928)]
[Switching to Thread 0x7ffff782c700 (LWP 20928)]
Hardware watchpoint 2: a

Old value = 0
New value = 1
thread1_func (p_arg=0x400718) at a.c:11
11                      sleep(10);
thread id=2
(gdb) c
Continuing.
[New Thread 0x7ffff6e2b700 (LWP 20929)]
[Switching to Thread 0x7ffff6e2b700 (LWP 20929)]
Hardware watchpoint 2: a

Old value = 1
New value = 2
thread2_func (p_arg=0x400721) at a.c:20
20                      sleep(10);
thread id=3 
```

可以看到程序暂停时，会打印线程号：“`thread id=2`”或者“`thread id=3`”。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Threads.html).

## 贡献者

nanxiao

# 一个 gdb 会话中同时调试多个程序

# 一个 gdb 会话中同时调试多个程序

## 例子

```cpp
a.c:
#include <stdio.h>
int func(int a, int b)
{
        int c = a * b;
        printf("c is %d\n", c);
}

int main(void)
{
        func(1, 2);
        return 0;
}

b.c:
#include <stdio.h>

int func1(int a)
{
        return 2 * a;
}

int func2(int a)
{
        int c = 0;
        c = 2 * func1(a);
        return c;
}

int func3(int a)
{
        int c = 0;
        c = 2 * func2(a);
        return c;
}

int main(void)
{
        printf("%d\n", func3(10));
        return 0;
} 
```

## 技巧

gdb 支持在一个会话中同时调试多个程序。以上面程序为例，首先调试`a`程序：

```cpp
root@bash:~$ gdb a
GNU gdb (Ubuntu 7.7-0ubuntu3) 7.7
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x400568: file a.c, line 10.
Starting program: /home/nanxiao/a 
```

接着使用“`add-inferior [ -copies n ] [ -exec executable ]`”命令加载可执行文件`b`。其中`n`默认为 1：

```cpp
(gdb) add-inferior -copies 2 -exec b
Added inferior 2
Reading symbols from b...done.
Added inferior 3
Reading symbols from b...done.
(gdb) i inferiors
  Num  Description       Executable
  3    <null>            /home/nanxiao/b
  2    <null>            /home/nanxiao/b
* 1    process 1586      /home/nanxiao/a
(gdb) inferior 2
[Switching to inferior 2 [<null>] (/home/nanxiao/b)]
(gdb) start
Temporary breakpoint 2 at 0x400568: main. (3 locations)
Starting program: /home/nanxiao/b

Temporary breakpoint 2, main () at b.c:24
24              printf("%d\n", func3(10));
(gdb) i inferiors
  Num  Description       Executable
  3    <null>            /home/nanxiao/b
* 2    process 1590      /home/nanxiao/b
  1    process 1586      /home/nanxiao/a 
```

可以看到可以调试`b`程序了。

另外也可用“`clone-inferior [ -copies n ] [ infno ]`”克隆现有的`inferior`，其中`n`默认为 1，`infno`默认为当前的`inferior`：

```cpp
(gdb) i inferiors
  Num  Description       Executable
  3    <null>            /home/nanxiao/b
* 2    process 1590      /home/nanxiao/b
  1    process 1586      /home/nanxiao/a
(gdb) clone-inferior -copies 1
Added inferior 4.
(gdb) i inferiors
  Num  Description       Executable
  4    <null>            /home/nanxiao/b
  3    <null>            /home/nanxiao/b
* 2    process 1590      /home/nanxiao/b
  1    process 1586      /home/nanxiao/a 
```

可以看到又多了一个`b`程序。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Inferiors-and-Programs.html).

## 贡献者

nanxiao

# 打印程序进程空间信息

# 打印程序进程空间信息

## 例子

```cpp
a.c:
#include <stdio.h>
int func(int a, int b)
{
        int c = a * b;
        printf("c is %d\n", c);
}

int main(void)
{
        func(1, 2);
        return 0;
}

b.c:
#include <stdio.h>

int func1(int a)
{
        return 2 * a;
}

int func2(int a)
{
        int c = 0;
        c = 2 * func1(a);
        return c;
}

int func3(int a)
{
        int c = 0;
        c = 2 * func2(a);
        return c;
}

int main(void)
{
        printf("%d\n", func3(10));
        return 0;
} 
```

## 技巧

使用 gdb 调试多个进程时，可以使用“`maint info program-spaces`”打印当前所有被调试的进程信息。以上面程序为例：

```cpp
[root@localhost nan]# gdb a
GNU gdb (GDB) 7.8.1
......
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x4004f9: file a.c, line 10.
Starting program: /home/nan/a 

Temporary breakpoint 1, main () at a.c:10
10              func(1, 2);
(gdb) add-inferior -exec b
Added inferior 2
Reading symbols from b...done.
(gdb) i inferiors b
Args must be numbers or '$' variables.
(gdb) i inferiors
  Num  Description       Executable        
  2    <null>            /home/nan/b       
* 1    process 15753     /home/nan/a       
(gdb) inferior 2
[Switching to inferior 2 [<null>] (/home/nan/b)]
(gdb) start
Temporary breakpoint 2 at 0x4004f9: main. (2 locations)
Starting program: /home/nan/b 

Temporary breakpoint 2, main () at b.c:24
24              printf("%d\n", func3(10));
(gdb) i inferiors
  Num  Description       Executable        
* 2    process 15902     /home/nan/b       
  1    process 15753     /home/nan/a       
(gdb) clone-inferior -copies 2
Added inferior 3.
Added inferior 4.
(gdb) i inferiors
  Num  Description       Executable        
  4    <null>            /home/nan/b       
  3    <null>            /home/nan/b       
* 2    process 15902     /home/nan/b       
  1    process 15753     /home/nan/a       
(gdb) maint info program-spaces
  Id   Executable        
  4    /home/nan/b       
        Bound inferiors: ID 4 (process 0)
  3    /home/nan/b       
        Bound inferiors: ID 3 (process 0)
* 2    /home/nan/b       
        Bound inferiors: ID 2 (process 15902)
  1    /home/nan/a       
        Bound inferiors: ID 1 (process 15753) 
```

可以看到执行“`maint info program-spaces`”命令后，打印出当前有 4 个`program-spaces`（编号从 1 到 4）。另外还有每个`program-spaces`对应的程序，`inferior`编号及进程号。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Inferiors-and-Programs.html).

## 贡献者

nanxiao

# 使用“$_exitcode”变量

# 使用“$_exitcode”变量

## 例子

```cpp
int main(void)
{
    return 0;
} 
```

## 技巧

当被调试的程序正常退出时，gdb 会使用`$_exitcode`这个“`convenience variable`”记录程序退出时的“`exit code`”。以调试上面程序为例:

```cpp
[root@localhost nan]# gdb -q a
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x400478: file a.c, line 3.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:3
3               return 0;
(gdb) n
4       }
(gdb)
0x00000034e421ed1d in __libc_start_main () from /lib64/libc.so.6
(gdb)
Single stepping until exit from function __libc_start_main,
which has no line number information.
[Inferior 1 (process 1185) exited normally]
(gdb) p $_exitcode
$1 = 0 
```

可以看到打印的`$_exitcode`的值为`0`。
改变程序，返回值改为`1`：

```cpp
int main(void)
{
    return 0;
} 
```

接着调试：

```cpp
[root@localhost nan]# gdb -q a
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x400478: file a.c, line 3.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:3
3               return 1;
(gdb)
(gdb) n
4       }
(gdb)
0x00000034e421ed1d in __libc_start_main () from /lib64/libc.so.6
(gdb)
Single stepping until exit from function __libc_start_main,
which has no line number information.
[Inferior 1 (process 2603) exited with code 01]
(gdb) p $_exitcode
$1 = 1 
```

可以看到打印的`$_exitcode`的值变为`1`。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Convenience-Vars.html).

## 贡献者

nanxiao