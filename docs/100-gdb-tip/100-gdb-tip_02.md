# 函数

# 函数

# 列出函数的名字

# 列出函数的名字

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>
void *thread_func(void *p_arg)
{
        while (1)
        {
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

使用 gdb 调试时，使用“`info functions`”命令可以列出可执行文件的所有函数名称。以上面代码为例：

```cpp
(gdb) info functions
All defined functions:

File a.c:
int main(void);
void *thread_func(void *);

Non-debugging symbols:
0x0805079c  _PROCEDURE_LINKAGE_TABLE_
0x080507ac  _cleanup@plt
0x080507bc  atexit
0x080507bc  atexit@plt
0x080507cc  __fpstart
0x080507cc  __fpstart@plt
0x080507dc  exit@plt
0x080507ec  __deregister_frame_info_bases@plt
0x080507fc  __register_frame_info_bases@plt
0x0805080c  _Jv_RegisterClasses@plt
0x0805081c  sleep
0x0805081c  sleep@plt
0x0805082c  pthread_create@plt
0x0805083c  _start
0x080508b4  _mcount
0x080508b8  __do_global_dtors_aux
0x08050914  frame_dummy
0x080509f4  __do_global_ctors_aux
0x08050a24  _init
0x08050a31  _fini 
```

可以看到会列出函数原型以及不带调试信息的函数。

另外这个命令也支持正则表达式：“`info functions regex`”，这样只会列出符合正则表达式的函数名称，例如：

```cpp
(gdb) info functions thre*
All functions matching regular expression "thre*":

File a.c:
void *thread_func(void *);

Non-debugging symbols:
0x0805082c  pthread_create@plt 
```

可以看到 gdb 只会列出名字里包含“`thre`”的函数。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Symbols.html)

## 贡献者

nanxiao

# 是否进入带调试信息的函数

# 是否进入带调试信息的函数

## 例子

```cpp
#include <stdio.h>

int func(void)
{
    return 3;
}

int main(void)
{
    int a = 0;

    a = func();
    printf("%d\n", a);
    return 0;
} 
```

## 技巧

使用 gdb 调试遇到函数时，使用 step 命令（缩写为 s）可以进入函数（函数必须有调试信息）。以上面代码为例：

```cpp
(gdb) n
12              a = func();
(gdb) s
func () at a.c:5
5               return 3;
(gdb) n
6       }
(gdb)
main () at a.c:13
13              printf("%d\n", a); 
```

可以看到 gdb 进入了 func 函数。

可以使用 next 命令（缩写为 n）不进入函数，gdb 会等函数执行完，再显示下一行要执行的程序代码：

```cpp
(gdb) n
12              a = func();
(gdb) n
13              printf("%d\n", a);
(gdb) n
3
14              return 0; 
```

可以看到 gdb 没有进入 func 函数。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Continuing-and-Stepping.html)

## 贡献者

nanxiao

# 进入不带调试信息的函数

# 进入不带调试信息的函数

## 例子

```cpp
#include <stdio.h>
#include <pthread.h>

typedef struct
{
        int a;
        int b;
        int c;
        int d;
        pthread_mutex_t mutex;
}ex_st;

int main(void) {
        ex_st st = {1, 2, 3, 4, PTHREAD_MUTEX_INITIALIZER};
        printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
        return 0;
} 
```

## 技巧

默认情况下，gdb 不会进入不带调试信息的函数。以上面代码为例：

```cpp
(gdb) n
15              printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
(gdb) s
1,2,3,4
16              return 0; 
```

可以看到由于 printf 函数不带调试信息，所以“s”命令（s 是“step”缩写）无法进入 printf 函数。

可以执行“set step-mode on”命令，这样 gdb 就不会跳过没有调试信息的函数：

```cpp
(gdb) set step-mode on
(gdb) n
15              printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
(gdb) s
0x00007ffff7a993b0 in printf () from /lib64/libc.so.6
(gdb) s
0x00007ffff7a993b7 in printf () from /lib64/libc.so.6 
```

可以看到 gdb 进入了 printf 函数，接下来可以使用调试汇编程序的办法去调试函数。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Continuing-and-Stepping.html)

## 贡献者

nanxiao

# 退出正在调试的函数

# 退出正在调试的函数

## 例子

```cpp
#include <stdio.h>

int func(void)
{
    int i = 0;

    i += 2;
    i *= 10;

    return i;
}

int main(void)
{
    int a = 0;

    a = func();
    printf("%d\n", a);
    return 0;
} 
```

## 技巧

当单步调试一个函数时，如果不想继续跟踪下去了，可以有两种方式退出。

第一种用“`finish`”命令，这样函数会继续执行完，并且打印返回值，然后等待输入接下来的命令。以上面代码为例：

```cpp
(gdb) n
17          a = func();
(gdb) s
func () at a.c:5
5               int i = 0;
(gdb) n
7               i += 2;
(gdb) fin
find    finish
(gdb) finish
Run till exit from #0  func () at a.c:7
0x08050978 in main () at a.c:17
17          a = func();
Value returned is $1 = 20 
```

可以看到当不想再继续跟踪`func`函数时，执行完“`finish`”命令，gdb 会打印结果：“`20`”，然后停在那里。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Continuing-and-Stepping.html)

第二种用“`return`”命令，这样函数不会继续执行下面的语句，而是直接返回。也可以用“`return expression`”命令指定函数的返回值。仍以上面代码为例：

```cpp
(gdb) n
17          a = func();
(gdb) s
func () at a.c:5
5               int i = 0;
(gdb) n
7               i += 2;
(gdb) n
8               i *= 10;
(gdb) re
record              remove-inferiors    return              reverse-next        reverse-step
refresh             remove-symbol-file  reverse-continue    reverse-nexti       reverse-stepi
remote              restore             reverse-finish      reverse-search
(gdb) return 40
Make func return now? (y or n) y
#0  0x08050978 in main () at a.c:17
17          a = func();
(gdb) n
18          printf("%d\n", a);
(gdb)
40
19          return 0; 
```

可以看到“`return`”命令退出了函数并且修改了函数的返回值。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Returning.html#Returning)

## 贡献者

nanxiao

# 直接执行函数

# 直接执行函数

## 例子

```cpp
#include <stdio.h>

int global = 1;

int func(void) 
{
    return (++global);
}

int main(void)
{
    printf("%d\n", global);
    return 0;
} 
```

## 技巧

使用 gdb 调试程序时，可以使用“`call`”或“`print`”命令直接调用函数执行。以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4004e3: file a.c, line 12.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:12
12              printf("%d\n", global);
(gdb) call func()
$1 = 2
(gdb) print func()
$2 = 3
(gdb) n
3
13              return 0; 
```

可以看到执行两次`func`函数后，`global`的值变成`3`。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Calling.html).

## 贡献者

nanxiao

# 打印函数堆栈帧信息

# 打印函数堆栈帧信息

## 例子

```cpp
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
```

## 技巧

使用 gdb 调试程序时，可以使用“`i frame`”命令（`i`是`info`命令缩写）显示函数堆栈帧信息。以上面程序为例：

```cpp
Breakpoint 1, func (a=1, b=2) at a.c:5
5               printf("c is %d\n", c);
(gdb) i frame
Stack level 0, frame at 0x7fffffffe590:
 rip = 0x40054e in func (a.c:5); saved rip = 0x400577
 called by frame at 0x7fffffffe5a0
 source language c.
 Arglist at 0x7fffffffe580, args: a=1, b=2
 Locals at 0x7fffffffe580, Previous frame's sp is 0x7fffffffe590
 Saved registers:
  rbp at 0x7fffffffe580, rip at 0x7fffffffe588
(gdb) i registers
rax            0x2      2
rbx            0x0      0
rcx            0x0      0
rdx            0x7fffffffe688   140737488348808
rsi            0x2      2
rdi            0x1      1
rbp            0x7fffffffe580   0x7fffffffe580
rsp            0x7fffffffe560   0x7fffffffe560
r8             0x7ffff7dd4e80   140737351863936
r9             0x7ffff7dea560   140737351951712
r10            0x7fffffffe420   140737488348192
r11            0x7ffff7a35dd0   140737348066768
r12            0x400440 4195392
r13            0x7fffffffe670   140737488348784
r14            0x0      0
r15            0x0      0
rip            0x40054e 0x40054e <func+24>
eflags         0x202    [ IF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0
(gdb) disassemble func
Dump of assembler code for function func:
   0x0000000000400536 <+0>:     push   %rbp
   0x0000000000400537 <+1>:     mov    %rsp,%rbp
   0x000000000040053a <+4>:     sub    $0x20,%rsp
   0x000000000040053e <+8>:     mov    %edi,-0x14(%rbp)
   0x0000000000400541 <+11>:    mov    %esi,-0x18(%rbp)
   0x0000000000400544 <+14>:    mov    -0x14(%rbp),%eax
   0x0000000000400547 <+17>:    imul   -0x18(%rbp),%eax
   0x000000000040054b <+21>:    mov    %eax,-0x4(%rbp)
=> 0x000000000040054e <+24>:    mov    -0x4(%rbp),%eax
   0x0000000000400551 <+27>:    mov    %eax,%esi
   0x0000000000400553 <+29>:    mov    $0x400604,%edi
   0x0000000000400558 <+34>:    mov    $0x0,%eax
   0x000000000040055d <+39>:    callq  0x400410 <printf@plt>
   0x0000000000400562 <+44>:    leaveq
   0x0000000000400563 <+45>:    retq
End of assembler dump. 
```

可以看到执行“`i frame`”命令后，输出了当前函数堆栈帧的地址，指令寄存器的值，局部变量地址及值等信息，可以对照当前寄存器的值和函数的汇编指令看一下。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Frame-Info.html).

## 贡献者

nanxiao

# 打印尾调用堆栈帧信息

# 打印尾调用堆栈帧信息

## 例子

```cpp
#include<stdio.h>
void a(void)
{
        printf("Tail call frame\n");
}

void b(void)
{
        a();
}

void c(void)
{
        b();
}

int main(void)
{
        c();
        return 0;
} 
```

## 技巧

当一个函数最后一条指令是调用另外一个函数时，开启优化选项的编译器常常以最后被调用的函数返回值作为调用者的返回值，这称之为“尾调用（Tail call）”。以上面程序为例，编译程序（使用‘-O’）：

```cpp
gcc -g -O -o test test.c 
```

查看`main`函数汇编代码：

```cpp
(gdb) disassemble main
Dump of assembler code for function main:
0x0000000000400565 <+0>:     sub    $0x8,%rsp
0x0000000000400569 <+4>:     callq  0x400536 <a>
0x000000000040056e <+9>:     mov    $0x0,%eax
0x0000000000400573 <+14>:    add    $0x8,%rsp
0x0000000000400577 <+18>:    retq 
```

可以看到`main`函数直接调用了函数`a`，根本看不到函数`b`和函数`c`的影子。

在函数`a`入口处打上断点，程序停止后，打印堆栈帧信息：

```cpp
(gdb) i frame
Stack level 0, frame at 0x7fffffffe590:
 rip = 0x400536 in a (test.c:4); saved rip = 0x40056e
 called by frame at 0x7fffffffe5a0
 source language c.
 Arglist at 0x7fffffffe580, args:
 Locals at 0x7fffffffe580, Previous frame's sp is 0x7fffffffe590
 Saved registers:
  rip at 0x7fffffffe588 
```

看不到尾调用的相关信息。

可以设置“`debug entry-values`”选项为非 0 的值，这样除了输出正常的函数堆栈帧信息以外，还可以输出尾调用的相关信息：

```cpp
(gdb) set debug entry-values 1
(gdb) b test.c:4
Breakpoint 1 at 0x400536: file test.c, line 4.
(gdb) r
Starting program: /home/nanxiao/test

Breakpoint 1, a () at test.c:4
4       {
(gdb) i frame
tailcall: initial:
Stack level 0, frame at 0x7fffffffe590:
 rip = 0x400536 in a (test.c:4); saved rip = 0x40056e
 called by frame at 0x7fffffffe5a0
 source language c.
 Arglist at 0x7fffffffe580, args:
 Locals at 0x7fffffffe580, Previous frame's sp is 0x7fffffffe590
 Saved registers:
  rip at 0x7fffffffe588 
```

可以看到输出了“`tailcall: initial:`”信息。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Tail-Call-Frames.html).

## 贡献者

nanxiao

# 选择函数堆栈帧

# 选择函数堆栈帧

## 例子

```cpp
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

用 gdb 调试程序时，当程序暂停后，可以用“`frame n`”命令选择函数堆栈帧，其中`n`是层数。以上面程序为例：

```cpp
(gdb) b test.c:5
Breakpoint 1 at 0x40053d: file test.c, line 5.
(gdb) r
Starting program: /home/nanxiao/test

Breakpoint 1, func1 (a=10) at test.c:5
5               return 2 * a;
(gdb) bt
#0  func1 (a=10) at test.c:5
#1  0x0000000000400560 in func2 (a=10) at test.c:11
#2  0x0000000000400586 in func3 (a=10) at test.c:18
#3  0x000000000040059e in main () at test.c:24
(gdb) frame 2
#2  0x0000000000400586 in func3 (a=10) at test.c:18
18              c = 2 * func2(a); 
```

可以看到程序断住后，最内层的函数帧为第`0`帧。执行`frame 2`命令后，当前的堆栈帧变成了`fun3`的函数帧。

也可以用“`frame addr`”命令选择函数堆栈帧，其中`addr`是堆栈地址。仍以上面程序为例：

```cpp
(gdb) frame 2
#2  0x0000000000400586 in func3 (a=10) at test.c:18
18              c = 2 * func2(a);
(gdb) i frame
Stack level 2, frame at 0x7fffffffe590:
 rip = 0x400586 in func3 (test.c:18); saved rip = 0x40059e
 called by frame at 0x7fffffffe5a0, caller of frame at 0x7fffffffe568
 source language c.
 Arglist at 0x7fffffffe580, args: a=10
 Locals at 0x7fffffffe580, Previous frame's sp is 0x7fffffffe590
 Saved registers:
  rbp at 0x7fffffffe580, rip at 0x7fffffffe588
(gdb) frame 0x7fffffffe568
#1  0x0000000000400560 in func2 (a=10) at test.c:11
11              c = 2 * func1(a); 
```

使用“`i frame`”命令可以知道`0x7fffffffe568`是`func2`的函数堆栈帧地址，使用“`frame 0x7fffffffe568`”可以切换到`func2`的函数堆栈帧。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Selection.html#Selection).

## 贡献者

nanxiao

# 向上或向下切换函数堆栈帧

# 向上或向下切换函数堆栈帧

## 例子

```cpp
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

用 gdb 调试程序时，当程序暂停后，可以用“`up n`”或“`down n`”命令向上或向下选择函数堆栈帧，其中`n`是层数。以上面程序为例：

```cpp
(gdb) b test.c:5
Breakpoint 1 at 0x40053d: file test.c, line 5.
(gdb) r
Starting program: /home/nanxiao/test

Breakpoint 1, func1 (a=10) at test.c:5
5               return 2 * a;
(gdb) bt
#0  func1 (a=10) at test.c:5
#1  0x0000000000400560 in func2 (a=10) at test.c:11
#2  0x0000000000400586 in func3 (a=10) at test.c:18
#3  0x000000000040059e in main () at test.c:24
(gdb) frame 2
#2  0x0000000000400586 in func3 (a=10) at test.c:18
18              c = 2 * func2(a);
(gdb) up 1
#3  0x000000000040059e in main () at test.c:24
24              printf("%d\n", func3(10));
(gdb) down 2
#1  0x0000000000400560 in func2 (a=10) at test.c:11
11              c = 2 * func1(a); 
```

可以看到程序断住后，先执行“`frame 2`”命令，切换到`fun3`函数。接着执行“`up 1`”命令，此时会切换到`main`函数，也就是会往外层的堆栈帧移动一层。反之，当执行“`down 2`”命令后，又会向内层堆栈帧移动二层。如果不指定`n`，则`n`默认为`1`.

还有“`up-silently n`”和“`down-silently n`”这两个命令，与“`up n`”和“`down n`”命令区别在于，切换堆栈帧后，不会打印信息，仍以上面程序为例：

```cpp
(gdb) up
#2  0x0000000000400586 in func3 (a=10) at test.c:18
18              c = 2 * func2(a);
(gdb) bt
#0  func1 (a=10) at test.c:5
#1  0x0000000000400560 in func2 (a=10) at test.c:11
#2  0x0000000000400586 in func3 (a=10) at test.c:18
#3  0x000000000040059e in main () at test.c:24
(gdb) up-silently
(gdb) i frame
Stack level 3, frame at 0x7fffffffe5a0:
 rip = 0x40059e in main (test.c:24); saved rip = 0x7ffff7a35ec5
 caller of frame at 0x7fffffffe590
 source language c.
 Arglist at 0x7fffffffe590, args:
 Locals at 0x7fffffffe590, Previous frame's sp is 0x7fffffffe5a0
 Saved registers:
  rbp at 0x7fffffffe590, rip at 0x7fffffffe598 
```

可以看到从`func3`切换到`main`函数堆栈帧时，并没有打印出相关信息。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Selection.html#Selection).

## 贡献者

nanxiao