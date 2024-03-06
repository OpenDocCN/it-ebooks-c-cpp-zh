# 观察点

# 观察点

# 设置观察点

# 设置观察点

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>
int a = 0;

void *thread1_func(void *p_arg)
{
        while (1)
        {
                a++;
                sleep(10);
        }
}

int main(int argc, char* argv[])
{
        pthread_t t1;

        pthread_create(&t1, NULL, thread1_func, "Thread 1");

        sleep(1000);
        return 0;
} 
```

## 技巧

gdb 可以使用“`watch`”命令设置观察点，也就是当一个变量值发生变化时，程序会停下来。以上面程序为例:

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4005a8: file a.c, line 19.
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Temporary breakpoint 1, main () at a.c:19
19              pthread_create(&t1, NULL, thread1_func, "Thread 1");
(gdb) watch a
Hardware watchpoint 2: a
(gdb) r
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff782c700 (LWP 8813)]
[Switching to Thread 0x7ffff782c700 (LWP 8813)]
Hardware watchpoint 2: a

Old value = 0
New value = 1
thread1_func (p_arg=0x4006d8) at a.c:11
11                      sleep(10);
(gdb) c
Continuing.
Hardware watchpoint 2: a

Old value = 1
New value = 2
thread1_func (p_arg=0x4006d8) at a.c:11
11                      sleep(10); 
```

可以看到，使用“`watch a`”命令以后，当`a`的值变化：由`0`变成`1`，由`1`变成`2`，程序都会停下来。
此外也可以使用“`watch *(data type*)address`”这样的命令，仍以上面程序为例:

```cpp
(gdb) p &a
$1 = (int *) 0x6009c8 <a>
(gdb) watch *(int*)0x6009c8
Hardware watchpoint 2: *(int*)0x6009c8
(gdb) r
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff782c700 (LWP 15431)]
[Switching to Thread 0x7ffff782c700 (LWP 15431)]
Hardware watchpoint 2: *(int*)0x6009c8

Old value = 0
New value = 1
thread1_func (p_arg=0x4006d8) at a.c:11
11                      sleep(10);
(gdb) c
Continuing.
Hardware watchpoint 2: *(int*)0x6009c8

Old value = 1
New value = 2
thread1_func (p_arg=0x4006d8) at a.c:11
11                      sleep(10); 
```

先得到`a`的地址：`0x6009c8`，接着用“`watch *(int*)0x6009c8`”设置观察点，可以看到同“`watch a`”命令效果一样。
观察点可以通过软件或硬件的方式实现，取决于具体的系统。但是软件实现的观察点会导致程序运行很慢，使用时需注意。参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html).

如果系统支持硬件观测的话，当设置观测点是会打印如下信息： Hardware watchpoint num: expr

如果不想用硬件观测点的话可如下设置： set can-use-hw-watchpoints

## 查看断点

列出当前所设置了的所有观察点：
info watchpoints

watch 所设置的断点也可以用控制断点的命令来控制。如 disable、enable、delete 等

## 贡献者

nanxiao

# 设置观察点只针对特定线程生效

# 设置观察点只针对特定线程生效

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

gdb 可以使用“`watch expr thread threadnum`”命令设置观察点只针对特定线程生效，也就是只有编号为`threadnum`的线程改变了变量的值，程序才会停下来，其它编号线程改变变量的值不会让程序停住。以上面程序为例:

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4005d4: file a.c, line 28.
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Temporary breakpoint 1, main () at a.c:28
28              pthread_create(&t1, NULL, thread1_func, "Thread 1");
(gdb) n
[New Thread 0x7ffff782c700 (LWP 25443)]
29              pthread_create(&t2, NULL, thread2_func, "Thread 2");
(gdb)
[New Thread 0x7ffff6e2b700 (LWP 25444)]
31              sleep(1000);
(gdb) i threads
  Id   Target Id         Frame
  3    Thread 0x7ffff6e2b700 (LWP 25444) 0x00007ffff7915911 in clone () from /lib64/libc.so.6
  2    Thread 0x7ffff782c700 (LWP 25443) 0x00007ffff78d9bcd in nanosleep () from /lib64/libc.so.6
* 1    Thread 0x7ffff7fe9700 (LWP 25413) main () at a.c:31
(gdb) wa a thread 2
Hardware watchpoint 2: a
(gdb) c
Continuing.
[Switching to Thread 0x7ffff782c700 (LWP 25443)]
Hardware watchpoint 2: a

Old value = 1
New value = 3
thread1_func (p_arg=0x400718) at a.c:11
11                      sleep(10);
(gdb) c
Continuing.
Hardware watchpoint 2: a

Old value = 3
New value = 5
thread1_func (p_arg=0x400718) at a.c:11
11                      sleep(10);
(gdb) c
Continuing.
Hardware watchpoint 2: a

Old value = 5
New value = 7
thread1_func (p_arg=0x400718) at a.c:11
11                      sleep(10); 
```

可以看到，使用“`wa a thread 2`”命令（`wa`是`watch`命令的缩写）以后，只有`thread1_func`改变`a`的值才会让程序停下来。
需要注意的是这种针对特定线程设置观察点方式只对硬件观察点才生效，参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html).

## 贡献者

nanxiao

# 设置读观察点

# 设置读观察点

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>

int a = 0;

void *thread1_func(void *p_arg)
{
        while (1)
        {
                printf("%d\n", a);
                sleep(10);
        }
}

int main(void)
{
        pthread_t t1;

        pthread_create(&t1, NULL, thread1_func, "Thread 1");

        sleep(1000);
        return;
} 
```

## 技巧

gdb 可以使用“`rwatch`”命令设置读观察点，也就是当发生读取变量行为时，程序就会暂停住。以上面程序为例:

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4005f3: file a.c, line 19.
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".

Temporary breakpoint 1, main () at a.c:19
19              pthread_create(&t1, NULL, thread1_func, "Thread 1");
(gdb) rw a
Hardware read watchpoint 2: a
(gdb) c
Continuing.
[New Thread 0x7ffff782c700 (LWP 5540)]
[Switching to Thread 0x7ffff782c700 (LWP 5540)]
Hardware read watchpoint 2: a

Value = 0
0x00000000004005c6 in thread1_func (p_arg=0x40071c) at a.c:10
10                      printf("%d\n", a);
(gdb) c
Continuing.
0
Hardware read watchpoint 2: a

Value = 0
0x00000000004005c6 in thread1_func (p_arg=0x40071c) at a.c:10
10                      printf("%d\n", a);
(gdb) c
Continuing.
0
Hardware read watchpoint 2: a

Value = 0
0x00000000004005c6 in thread1_func (p_arg=0x40071c) at a.c:10
10                      printf("%d\n", a); 
```

可以看到，使用“`rw a`”命令（`rw`是`rwatch`命令的缩写）以后，每次访问`a`的值都会让程序停下来。
需要注意的是`rwatch`命令只对硬件观察点才生效，参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html).

## 贡献者

nanxiao

# 设置读写观察点

# 设置读写观察点

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
                printf("%d\n", a);;
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

gdb 可以使用“`awatch`”命令设置读写观察点，也就是当发生读取变量或改变变量值的行为时，程序就会暂停住。以上面程序为例:

```cpp
(gdb) aw a
Hardware access (read/write) watchpoint 1: a
(gdb) r
Starting program: /data2/home/nanxiao/a
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib64/libthread_db.so.1".
[New Thread 0x7ffff782c700 (LWP 16938)]
[Switching to Thread 0x7ffff782c700 (LWP 16938)]
Hardware access (read/write) watchpoint 1: a

Value = 0
0x00000000004005c6 in thread1_func (p_arg=0x40076c) at a.c:10
10                      a++;
(gdb) c
Continuing.
Hardware access (read/write) watchpoint 1: a

Old value = 0
New value = 1
thread1_func (p_arg=0x40076c) at a.c:11
11                      sleep(10);
(gdb) c
Continuing.
[New Thread 0x7ffff6e2b700 (LWP 16939)]
[Switching to Thread 0x7ffff6e2b700 (LWP 16939)]
Hardware access (read/write) watchpoint 1: a

Value = 1
0x00000000004005f2 in thread2_func (p_arg=0x400775) at a.c:19
19                      printf("%d\n", a);;
(gdb) c
Continuing.
1
[Switching to Thread 0x7ffff782c700 (LWP 16938)]
Hardware access (read/write) watchpoint 1: a

Value = 1
0x00000000004005c6 in thread1_func (p_arg=0x40076c) at a.c:10
10                      a++; 
```

可以看到，使用“`aw a`”命令（`aw`是`awatch`命令的缩写）以后，每次读取或改变`a`的值都会让程序停下来。
需要注意的是`awatch`命令只对硬件观察点才生效，参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Watchpoints.html).

## 贡献者

nanxiao