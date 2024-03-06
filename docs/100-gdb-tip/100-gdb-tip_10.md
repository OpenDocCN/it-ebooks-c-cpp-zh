# 改变程序的执行

# 改变字符串的值

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

使用 gdb 调试程序时，可以用“`set`”命令改变字符串的值，以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x8050af0: file a.c, line 5.
Starting program: /data1/nan/a 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Temporary breakpoint 1, main () at a.c:5
5               char p1[] = "Sam";
(gdb) n
6               char *p2 = "Bob";
(gdb) 
8               printf("p1 is %s, p2 is %s\n", p1, p2);
(gdb) set main::p1="Jil"
(gdb) set main::p2="Bill"
(gdb) n
p1 is Jil, p2 is Bill
9               return 0; 
```

可以看到执行`p1`和`p2`的字符串都发生了变化。也可以通过访问内存地址的方法改变字符串的值：

```cpp
Starting program: /data1/nan/a 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Temporary breakpoint 2, main () at a.c:5
5               char p1[] = "Sam";
(gdb) n
6               char *p2 = "Bob";
(gdb) p p1
$1 = "Sam"
(gdb) p &p1
$2 = (char (*)[4]) 0x80477a4
(gdb) set {char [4]} 0x80477a4 = "Ace"
(gdb) n
8               printf("p1 is %s, p2 is %s\n", p1, p2);
(gdb) 
p1 is Ace, p2 is Bob
9               return 0; 
```

在改变字符串的值时候，一定要注意内存越界的问题。
参见[stackoverflow](http://stackoverflow.com/questions/19503057/in-gdb-how-can-i-write-a-string-to-memory).

## 贡献者

nanxiao

# 设置变量的值

## 例子

```cpp
#include <stdio.h>

int func(void)
{
    int i = 2;

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

在 gdb 中，可以用“`set var variable=expr`”命令设置变量的值，以上面代码为例：

```cpp
Breakpoint 2, func () at a.c:5
5                   int i = 2;
(gdb) n
7                   return i;
(gdb) set var i = 8
(gdb) p i
$4 = 8 
```

可以看到在`func`函数里用`set`命令把`i`的值修改成为`8`。

也可以用“`set {type}address=expr`”的方式，含义是给存储地址在`address`，变量类型为`type`的变量赋值，仍以上面代码为例：

```cpp
Breakpoint 2, func () at a.c:5
5                   int i = 2;
(gdb) n
7                   return i;
(gdb) p &i
$5 = (int *) 0x8047a54
(gdb) set {int}0x8047a54 = 8
(gdb) p i
$6 = 8 
```

可以看到`i`的值被修改成为`8`。

另外寄存器也可以作为变量，因此同样可以修改寄存器的值：

```cpp
Breakpoint 2, func () at a.c:5
5                   int i = 2;
(gdb)
(gdb) n
7                   return i;
(gdb)
8               }
(gdb) set var $eax = 8
(gdb) n
main () at a.c:15
15                  printf("%d\n", a);
(gdb)
8
16                  return 0; 
```

可以看到因为 eax 寄存器存储着函数的返回值，所以当把 eax 寄存器的值改为`8`后，函数的返回值也变成了`8`。

详情参见[gdb 手册](https://sourceware.org/gdb/current/onlinedocs/gdb/Assignment.html#Assignment)

## 贡献者

nanxiao

# 修改 PC 寄存器的值

## 例子

```cpp
#include <stdio.h>
int main(void)
{       
        int a =0;               

        a++;    
        a++;    
        printf("%d\n", a);      
        return 0;
} 
```

## 技巧

PC 寄存器会存储程序下一条要执行的指令，通过修改这个寄存器的值，可以达到改变程序执行流程的目的。
上面的程序会输出“`a=2`”，下面介绍一下如何通过修改 PC 寄存器的值，改变程序执行流程。

```cpp
4               int a =0;
(gdb) disassemble main
Dump of assembler code for function main:
0x08050921 <main+0>:    push   %ebp
0x08050922 <main+1>:    mov    %esp,%ebp
0x08050924 <main+3>:    sub    $0x8,%esp
0x08050927 <main+6>:    and    $0xfffffff0,%esp
0x0805092a <main+9>:    mov    $0x0,%eax
0x0805092f <main+14>:   add    $0xf,%eax
0x08050932 <main+17>:   add    $0xf,%eax
0x08050935 <main+20>:   shr    $0x4,%eax
0x08050938 <main+23>:   shl    $0x4,%eax
0x0805093b <main+26>:   sub    %eax,%esp
0x0805093d <main+28>:   movl   $0x0,-0x4(%ebp)
0x08050944 <main+35>:   lea    -0x4(%ebp),%eax
0x08050947 <main+38>:   incl   (%eax)
0x08050949 <main+40>:   lea    -0x4(%ebp),%eax
0x0805094c <main+43>:   incl   (%eax)
0x0805094e <main+45>:   sub    $0x8,%esp
0x08050951 <main+48>:   pushl  -0x4(%ebp)
0x08050954 <main+51>:   push   $0x80509b4
0x08050959 <main+56>:   call   0x80507cc <printf@plt>
0x0805095e <main+61>:   add    $0x10,%esp
0x08050961 <main+64>:   mov    $0x0,%eax
0x08050966 <main+69>:   leave
0x08050967 <main+70>:   ret
End of assembler dump.
(gdb) info line 6
Line 6 of "a.c" starts at address 0x8050944 <main+35> and ends at 0x8050949 <main+40>.
(gdb) info line 7
Line 7 of "a.c" starts at address 0x8050949 <main+40> and ends at 0x805094e <main+45>. 
```

通过“`info line 6`”和“`info line 7`”命令可以知道两条“`a++;`”语句的汇编指令起始地址分别是`0x8050944`和`0x8050949`。

```cpp
(gdb) n
6               a++;
(gdb) p $pc
$3 = (void (*)()) 0x8050944 <main+35>
(gdb) set var $pc=0x08050949 
```

当程序要执行第一条“`a++;`”语句时，打印`pc`寄存器的值，看到`pc`寄存器的值为`0x8050944`，与“`info line 6`”命令得到的一致。接下来，把`pc`寄存器的值改为`0x8050949`，也就是通过“`info line 7`”命令得到的第二条“`a++;`”语句的起始地址。

```cpp
(gdb) n
8               printf("a=%d\n", a);
(gdb)
a=1
9               return 0; 
```

接下来执行，可以看到程序输出“`a=1`”，也就是跳过了第一条“`a++;`”语句。

## 贡献者

nanxiao

# 跳转到指定位置执行

## 例子

```cpp
#include <stdio.h>

void fun (int x)
{
  if (x < 0)
    puts ("error");
}

int main (void)
{
  int i = 1;

  fun (i--);
  fun (i--);
  fun (i--);

  return 0;
} 
```

## 技巧

当调试程序时，你可能不小心走过了出错的地方：

```cpp
(gdb) n
13      fun (i--);
(gdb) 
14      fun (i--);
(gdb) 
15      fun (i--);
(gdb) 
error
17      return 0; 
```

看起来是在 15 行，调用 fun 的时候出错了。常见的办法是在 15 行设置个断点，然后从头`run`一次。

如果你的环境支持反向执行，那么更好了。

如果不支持，你也可以直接`jump`到 15 行，再执行一次：

```cpp
(gdb) b 15
Breakpoint 2 at 0x40056a: file jump.c, line 15.
(gdb) j 15
Continuing at 0x40056a.

Breakpoint 2, main () at jump.c:15
15      fun (i--);
(gdb) s
fun (x=-2) at jump.c:5
5      if (x < 0)
(gdb) n
6        puts ("error"); 
```

需要注意的是：

1.  `jump`命令只改变 pc 的值，所以改变程序执行可能会出现不同的结果，比如变量 i 的值
2.  通过（临时）断点的配合，可以让你的程序跳到指定的位置，并停下来

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Jumping.html#Jumping)

## 贡献者

xmj

# 使用断点命令改变程序的执行

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>

void drawing (int n)
{
  if (n != 0)
    puts ("Try again?\nAll you need is a dollar, and a dream.");
  else
    puts ("You win $3000!");
}

int main (void)
{
  int n;

  srand (time (0));
  n = rand () % 10;
  printf ("Your number is %d\n", n);
  drawing (n);

  return 0;
} 
```

## 技巧

这个例子程序可能不太好，只是可以用来演示下断点命令的用法：

```cpp
(gdb) b drawing
Breakpoint 1 at 0x40064d: file win.c, line 6.
(gdb) command 1
Type commands for breakpoint(s) 1, one per line.
End with a line saying just "end".
>silent
>set variable n = 0
>continue
>end
(gdb) r
Starting program: /home/xmj/tmp/a.out 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
Your number is 6
You win $3000!
[Inferior 1 (process 4134) exited normally] 
```

可以看到，当程序运行到断点处，会自动把变量 n 的值修改为 0，然后继续执行。

如果你在调试一个大程序，重新编译一次会花费很长时间，比如调试编译器的 bug，那么你可以用这种方式在 gdb 中先实验性的修改下试试，而不需要修改源码，重新编译。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Break-Commands.html#Break-Commands)

## 贡献者

xmj

# 修改被调试程序的二进制文件

## 例子

```cpp
#include <stdio.h>
#include <stdlib.h>

void drawing (int n)
{
  if (n != 0)
    puts ("Try again?\nAll you need is a dollar, and a dream.");
  else
    puts ("You win $3000!");
}

int main (void)
{
  int n;

  srand (time (0));
  n = rand () % 10;
  printf ("Your number is %d\n", n);
  drawing (n);

  return 0;
} 
```

## 技巧

gdb 不仅可以用来调试程序，还可以修改程序的二进制代码。

缺省情况下，gdb 是以只读方式加载程序的。可以通过命令行选项指定为可写：

```cpp
$ gcc -write ./a.out
(gdb) show write
Writing into executable and core files is on. 
```

也可以在 gdb 中，使用命令设置并重新加载程序：

```cpp
(gdb) set write on
(gdb) file ./a.out 
```

接下来，查看反汇编：

```cpp
(gdb) disassemble /mr drawing 
Dump of assembler code for function drawing:
5    {
   0x0000000000400642 <+0>:    55    push   %rbp
   0x0000000000400643 <+1>:    48 89 e5    mov    %rsp,%rbp
   0x0000000000400646 <+4>:    48 83 ec 10    sub    $0x10,%rsp
   0x000000000040064a <+8>:    89 7d fc    mov    %edi,-0x4(%rbp)

6      if (n != 0)
   0x000000000040064d <+11>:    83 7d fc 00    cmpl   $0x0,-0x4(%rbp)
   0x0000000000400651 <+15>:    74 0c    je     0x40065f <drawing+29>

7        puts ("Try again?\nAll you need is a dollar, and a dream.");
   0x0000000000400653 <+17>:    bf e0 07 40 00    mov    $0x4007e0,%edi
   0x0000000000400658 <+22>:    e8 b3 fe ff ff    callq  0x400510 <puts@plt>
   0x000000000040065d <+27>:    eb 0a    jmp    0x400669 <drawing+39>

8      else
9        puts ("You win $3000!");
   0x000000000040065f <+29>:    bf 12 08 40 00    mov    $0x400812,%edi
   0x0000000000400664 <+34>:    e8 a7 fe ff ff    callq  0x400510 <puts@plt>

10    }
   0x0000000000400669 <+39>:    c9    leaveq 
   0x000000000040066a <+40>:    c3    retq   

End of assembler dump. 
```

修改二进制代码（注意大小端和指令长度）：

```cpp
(gdb) set variable *(short*)0x400651=0x0ceb
(gdb) disassemble /mr drawing 
Dump of assembler code for function drawing:
5    {
   0x0000000000400642 <+0>:    55    push   %rbp
   0x0000000000400643 <+1>:    48 89 e5    mov    %rsp,%rbp
   0x0000000000400646 <+4>:    48 83 ec 10    sub    $0x10,%rsp
   0x000000000040064a <+8>:    89 7d fc    mov    %edi,-0x4(%rbp)

6      if (n != 0)
   0x000000000040064d <+11>:    83 7d fc 00    cmpl   $0x0,-0x4(%rbp)
   0x0000000000400651 <+15>:    eb 0c    jmp    0x40065f <drawing+29>

7        puts ("Try again?\nAll you need is a dollar, and a dream.");
   0x0000000000400653 <+17>:    bf e0 07 40 00    mov    $0x4007e0,%edi
   0x0000000000400658 <+22>:    e8 b3 fe ff ff    callq  0x400510 <puts@plt>
   0x000000000040065d <+27>:    eb 0a    jmp    0x400669 <drawing+39>

8      else
9        puts ("You win $3000!");
   0x000000000040065f <+29>:    bf 12 08 40 00    mov    $0x400812,%edi
   0x0000000000400664 <+34>:    e8 a7 fe ff ff    callq  0x400510 <puts@plt>

10    }
   0x0000000000400669 <+39>:    c9    leaveq 
   0x000000000040066a <+40>:    c3    retq   

End of assembler dump. 
```

可以看到，条件跳转指令“je”已经被改为无条件跳转“jmp”了。

退出，运行一下：

```cpp
$ ./a.out 
Your number is 2
You win $3000! 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Patching.html#Patching)

## 贡献者

xmj