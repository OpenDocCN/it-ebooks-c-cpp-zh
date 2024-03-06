# 图形化界面

# 进入和退出图形化调试界面

## 例子

```cpp
#include <stdio.h>

void fun1(void)
{
        int i = 0;

        i++;
        i = i * 2;
        printf("%d\n", i);
}

void fun2(void)
{
        int j = 0;

        fun1();
        j++;
        j = j * 2;
        printf("%d\n", j);
}

int main(void)
{
        fun2();
        return 0;
} 
```

## 技巧

启动 gdb 时指定“`-tui`”参数（例如：`gdb -tui program`），或者运行 gdb 过程中使用“`Crtl+X+A`”组合键，都可以进入图形化调试界面。以调试上面程序为例：

```cpp
 ┌──a.c──────────────────────────────────────────────────────────────────────────────────────────┐
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
B+>│24              fun2();                                                                        │
   │25              return 0;                                                                      │
   │26      }                                                                                      │
   │27                                                                                             │
   │28                                                                                             │
   │29                                                                                             │
   │30                                                                                             │
   │31                                                                                             │
   │32                                                                                             │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 22141 In: main                                               Line: 24   PC: 0x40052b
Type "apropos word" to search for commands related to "word"...
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24
(gdb) 
```

可以看到，显示了当前的程序的进程号，将要执行的代码行号，`PC`寄存器的值。
退出图形化调试界面也是用“`Crtl+X+A`”组合键。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/TUI.html).

## 贡献者

nanxiao

# 显示汇编代码窗口

## 例子

```cpp
#include <stdio.h>

void fun1(void)
{
        int i = 0;

        i++;
        i = i * 2;
        printf("%d\n", i);
}

void fun2(void)
{
        int j = 0;

        fun1();
        j++;
        j = j * 2;
        printf("%d\n", j);
}

int main(void)
{
        fun2();
        return 0;
} 
```

## 技巧

使用 gdb 图形化调试界面时，可以使用“`layout asm`”命令显示汇编代码窗口。以调试上面程序为例：

```cpp
 ┌───────────────────────────────────────────────────────────────────────────────────────────────┐
  >│0x40052b <main+4>               callq  0x4004f3 <fun2>                                         │
   │0x400530 <main+9>               mov    $0x0,%eax                                               │
   │0x400535 <main+14>              leaveq                                                         │
   │0x400536 <main+15>              retq                                                           │
   │0x400537                        nop                                                            │
   │0x400538                        nop                                                            │
   │0x400539                        nop                                                            │
   │0x40053a                        nop                                                            │
   │0x40053b                        nop                                                            │
   │0x40053c                        nop                                                            │
   │0x40053d                        nop                                                            │
   │0x40053e                        nop                                                            │
   │0x40053f                        nop                                                            │
   │0x400540 <__libc_csu_fini>      repz retq                                                      │
   │0x400542                        data16 data16 data16 data16 nopw %cs:0x0(%rax,%rax,1)          │
   │0x400550 <__libc_csu_init>      mov    %rbp,-0x28(%rsp)                                        │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 44658 In: main                                               Line: 24   PC: 0x40052b

(gdb) start
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24
(gdb) 
```

可以看到，显示了当前的程序的汇编代码。
如果既想显示源代码，又想显示汇编代码，可以使用“`layout split`”命令：

```cpp
 ┌──a.c──────────────────────────────────────────────────────────────────────────────────────────┐
  >│24              fun2();                                                                        │
   │25              return 0;                                                                      │
   │26      }                                                                                      │
   │27                                                                                             │
   │28                                                                                             │
   │29                                                                                             │
   │30                                                                                             │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
  >│0x40052b <main+4>       callq  0x4004f3 <fun2>                                                 │
   │0x400530 <main+9>       mov    $0x0,%eax                                                       │
   │0x400535 <main+14>      leaveq                                                                 │
   │0x400536 <main+15>      retq                                                                   │
   │0x400537                nop                                                                    │
   │0x400538                nop                                                                    │
   │0x400539                nop                                                                    │
   │0x40053a                nop                                                                    │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 44658 In: main                                               Line: 24   PC: 0x40052b

(gdb) start
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24
(gdb) 
```

可以看到上面显示的是源代码，下面显示的是汇编代码。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Commands.html).

## 贡献者

nanxiao

# 显示寄存器窗口

## 例子

```cpp
#include <stdio.h>

void fun1(void)
{
        int i = 0;

        i++;
        i = i * 2;
        printf("%d\n", i);
}

void fun2(void)
{
        int j = 0;

        fun1();
        j++;
        j = j * 2;
        printf("%d\n", j);
}

int main(void)
{
        fun2();
        return 0;
} 
```

## 技巧

使用 gdb 图形化调试界面时，可以使用“`layout regs`”命令显示寄存器窗口。以调试上面程序为例：

```cpp
┌──Register group: general─────────────────────────────────────────────────────────────────────────┐
│rax            0x34e4590f60     227169341280     rbx            0x0      0                        │
│rcx            0x0      0                        rdx            0x7fffffffe4b8   140737488348344  │
│rsi            0x7fffffffe4a8   140737488348328  rdi            0x1      1                        │
│rbp            0x7fffffffe3c0   0x7fffffffe3c0   rsp            0x7fffffffe3c0   0x7fffffffe3c0   │
│r8             0x34e458f300     227169334016     r9             0x34e3a0e9f0     227157273072     │
│r10            0x7fffffffe210   140737488347664  r11            0x34e421ec20     227165727776     │
│r12            0x4003e0 4195296                  r13            0x7fffffffe4a0   140737488348320  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
  >│24              fun2();                                                                        │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 12552 In: main                                               Line: 24   PC: 0x40052b
Reading symbols from a...done.
(gdb) start
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24
(gdb) 
```

可以看到，显示了通用寄存器的内容。
如果想查看浮点寄存器，可以使用“`tui reg float`”命令：

```cpp
┌──Register group: float───────────────────────────────────────────────────────────────────────────┐
│st0            0        (raw 0x00000000000000000000)                                              │
│st1            0        (raw 0x00000000000000000000)                                              │
│st2            0        (raw 0x00000000000000000000)                                              │
│st3            0        (raw 0x00000000000000000000)                                              │
│st4            0        (raw 0x00000000000000000000)                                              │
│st5            0        (raw 0x00000000000000000000)                                              │
│st6            0        (raw 0x00000000000000000000)                                              │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
   │16              fun1();                                                                        │
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 12552 In: main                                               Line: 24   PC: 0x40052b
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24
(gdb) tui reg float 
```

“`tui reg system`”命令显示系统寄存器：

```cpp
┌──Register group: system──────────────────────────────────────────────────────────────────────────┐
│orig_rax       0xffffffffffffffff       -1                                                        │
│                                                                                                  │
│                                                                                                  │
│                                                                                                  │
│                                                                                                  │
│                                                                                                  │
│                                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
   │16              fun1();                                                                        │
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 12552 In: main                                               Line: 24   PC: 0x40052b

Temporary breakpoint 1, main () at a.c:24
(gdb) tui reg system
(gdb) 
```

想切换回显示通用寄存器内容，可以使用“`tui reg general`”命令：

```cpp
┌──Register group: general─────────────────────────────────────────────────────────────────────────┐
│rax            0x34e4590f60     227169341280     rbx            0x0      0                        │
│rcx            0x0      0                        rdx            0x7fffffffe4b8   140737488348344  │
│rsi            0x7fffffffe4a8   140737488348328  rdi            0x1      1                        │
│rbp            0x7fffffffe3c0   0x7fffffffe3c0   rsp            0x7fffffffe3c0   0x7fffffffe3c0   │
│r8             0x34e458f300     227169334016     r9             0x34e3a0e9f0     227157273072     │
│r10            0x7fffffffe210   140737488347664  r11            0x34e421ec20     227165727776     │
│r12            0x4003e0 4195296                  r13            0x7fffffffe4a0   140737488348320  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
   │16              fun1();                                                                        │
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 12552 In: main                                               Line: 24   PC: 0x40052b
(gdb) tui reg general
(gdb) 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Commands.html).

## 贡献者

nanxiao

# 调整窗口大小

## 例子

```cpp
#include <stdio.h>

void fun1(void)
{
        int i = 0;

        i++;
        i = i * 2;
        printf("%d\n", i);
}

void fun2(void)
{
        int j = 0;

        fun1();
        j++;
        j = j * 2;
        printf("%d\n", j);
}

int main(void)
{
        fun2();
        return 0;
} 
```

## 技巧

使用 gdb 图形化调试界面时，可以使用“`winheight <win_name> [+ | -]count`”命令调整窗口大小（`winheight`缩写为`win`。`win_name`可以是`src`、`cmd`、`asm`和`regs`）。以调试上面程序为例，这是原始的`src`窗口大小：

```cpp
 ┌──a.c──────────────────────────────────────────────────────────────────────────────────────────┐
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21      int main(void)                                                                        22
   │23      {                                                                                      │
   │24              fun2();                                                                        │
B+>│25                                                                                             │
   │                return 0;                                                                      │
   │26      }                                                                                      │
   │27                                                                                            32
   │                                                                                               │
   │                                                                                               │
   │                                                                                               │
   │                                                                                               │
   │                                                                                               │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 9667 In: main                                                Line: 24   PC: 0x40052b
Usage: winheight <win_name> [+ | -] <#lines>
(gdb) start
Temporary breakpoint 1 at 0x40052b: file a.c, line 24.
Starting program: /home/nan/a

Temporary breakpoint 1, main () at a.c:24 
```

执行“`winheight src -5`”命令后：

```cpp
 ┌──a.c──────────────────────────────────────────────────────────────────────────────────────────┐
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
  >│24              fun2();                                                                        │
   │25              return 0;                                                                      │
   │26      }                                                                                      │
   │27                                                                                             │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 9667 In: main                                               Line: 24   PC: 0x40052b
Usage: winheight <win_name> [+ | -] <#lines>
(gdb) 
```

可以看到窗口变小了。
接着执行“`winheight src +5`”命令：

```cpp
 ┌──a.c──────────────────────────────────────────────────────────────────────────────────────────┐
   │17              j++;                                                                           │
   │18              j = j * 2;                                                                     │
   │19              printf("%d\n", j);                                                             │
   │20      }                                                                                      │
   │21                                                                                             │
   │22      int main(void)                                                                         │
   │23      {                                                                                      │
  >│24              fun2();                                                                        │
   │25              return 0;                                                                      │
   │26      }                                                                                      │
   │27                                                                                             │
   │28                                                                                             │
   │29                                                                                             │
   │30                                                                                             │
   │31                                                                                             │
   │32                                                                                             │
   └───────────────────────────────────────────────────────────────────────────────────────────────┘
native process 9667 In: main                                               Line: 24   PC: 0x40052b
Usage: winheight <win_name> [+ | -] <#lines>
(gdb) 
```

可以看到窗口恢复了原样。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Commands.html).

## 贡献者

nanxiao