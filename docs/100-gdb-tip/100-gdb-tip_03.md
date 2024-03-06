# 断点

# 断点

# 在匿名空间设置断点

# 在匿名空间设置断点

## 例子

```cpp
namespace Foo
{
  void foo()
  {
  }
}

namespace
{
  void bar()
  {
  }
} 
```

## 技巧

在 gdb 中，如果要对 namespace Foo 中的 foo 函数设置断点，可以使用如下命令：

```cpp
(gdb) b Foo::foo 
```

如果要对匿名空间中的 bar 函数设置断点，可以使用如下命令：

```cpp
(gdb) b (anonymous namespace)::bar 
```

## 贡献者

xmj

# 在程序地址上打断点

# 在程序地址上打断点

## 例子

```cpp
0000000000400522 <main>:
  400522:       55                      push   %rbp
  400523:       48 89 e5                mov    %rsp,%rbp
  400526:       8b 05 00 1b 00 00       mov    0x1b00(%rip),%eax        # 40202c <he+0xc>
  40052c:       85 c0                   test   %eax,%eax
  40052e:       75 07                   jne    400537 <main+0x15>
  400530:       b8 7c 06 40 00          mov    $0x40067c,%eax
  400535:       eb 05                   jmp    40053c <main+0x1a> 
```

## 技巧

当调试汇编程序，或者没有调试信息的程序时，经常需要在程序地址上打断点，方法为`b *address`。例如：

```cpp
(gdb) b *0x400522 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Specify-Location.html#Specify-Location)

## 贡献者

xmj

# 在程序入口处打断点

# 在程序入口处打断点

## 获取程序入口

### 方法一：

```cpp
$ strip a.out
$ readelf -h a.out 
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x400440
  Start of program headers:          64 (bytes into file)
  Start of section headers:          4496 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         29
  Section header string table index: 28 
```

### 方法二：

```cpp
$ gdb a.out 
>>> info files
Symbols from "/home/me/a.out".
Local exec file:
    `/home/me/a.out', file type elf64-x86-64.
    Entry point: 0x400440
    0x0000000000400238 - 0x0000000000400254 is .interp
    0x0000000000400254 - 0x0000000000400274 is .note.ABI-tag
    0x0000000000400274 - 0x0000000000400298 is .note.gnu.build-id
    0x0000000000400298 - 0x00000000004002b4 is .gnu.hash
    0x00000000004002b8 - 0x0000000000400318 is .dynsym
    0x0000000000400318 - 0x0000000000400355 is .dynstr
    0x0000000000400356 - 0x000000000040035e is .gnu.version
    0x0000000000400360 - 0x0000000000400380 is .gnu.version_r
    0x0000000000400380 - 0x0000000000400398 is .rela.dyn
    0x0000000000400398 - 0x00000000004003e0 is .rela.plt
    0x00000000004003e0 - 0x00000000004003fa is .init
    0x0000000000400400 - 0x0000000000400440 is .plt
    0x0000000000400440 - 0x00000000004005c2 is .text
    0x00000000004005c4 - 0x00000000004005cd is .fini
    0x00000000004005d0 - 0x00000000004005e0 is .rodata
    0x00000000004005e0 - 0x0000000000400614 is .eh_frame_hdr
    0x0000000000400618 - 0x000000000040070c is .eh_frame
    0x0000000000600e10 - 0x0000000000600e18 is .init_array
    0x0000000000600e18 - 0x0000000000600e20 is .fini_array
    0x0000000000600e20 - 0x0000000000600e28 is .jcr
    0x0000000000600e28 - 0x0000000000600ff8 is .dynamic
    0x0000000000600ff8 - 0x0000000000601000 is .got
    0x0000000000601000 - 0x0000000000601030 is .got.plt
    0x0000000000601030 - 0x0000000000601040 is .data
    0x0000000000601040 - 0x0000000000601048 is .bss 
```

## 技巧

当调试没有调试信息的程序时，直接运行`start`命令是没有效果的：

```cpp
(gdb) start
Function "main" not defined. 
```

如果不知道 main 在何处，那么可以在程序入口处打断点。先通过`readelf`或者进入 gdb，执行`info files`获得入口地址，然后：

```cpp
(gdb) b *0x400440
(gdb) r 
```

## 贡献者

*   xmj
*   [weekface](https://github.com/weekface)

# 在文件行号上打断点

# 在文件行号上打断点

## 例子

```cpp
/* a/file.c */
#include <stdio.h>

void print_a (void)
{
  puts ("a");
}

/* b/file.c */
#include <stdio.h>

void print_b (void)
{
  puts ("b");
}

/* main.c */
extern void print_a(void);
extern void print_b(void);

int main(void)
{
  print_a();
  print_b();
  return 0;
} 
```

## 技巧

这个比较简单，如果要在当前文件中的某一行打断点，直接`b linenum`即可，例如：

```cpp
(gdb) b 7 
```

也可以显式指定文件，`b file:linenum`例如：

```cpp
(gdb) b file.c:6
Breakpoint 1 at 0x40053b: file.c:6\. (2 locations)
(gdb) i breakpoints 
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   <MULTIPLE>         
1.1                         y     0x000000000040053b in print_a at a/file.c:6
1.2                         y     0x000000000040054b in print_b at b/file.c:6 
```

可以看出，gdb 会对所有匹配的文件设置断点。你可以通过指定（部分）路径，来区分相同的文件名：

```cpp
(gdb) b a/file.c:6 
```

注意：通过行号进行设置断点的一个弊端是，如果你更改了源程序，那么之前设置的断点就可能不是你想要的了。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Specify-Location.html#Specify-Location)

## 贡献者

xmj

# 保存已经设置的断点

# 保存已经设置的断点

## 例子

```cpp
$ gdb -q `which gdb`
Reading symbols from /home/xmj/install/binutils-trunk/bin/gdb...done.
(gdb) b gdb_main
Breakpoint 1 at 0x5a7af0: file /home/xmj/project/binutils-trunk/gdb/main.c, line 1061.
(gdb) b captured_main
Breakpoint 2 at 0x5a6bd0: file /home/xmj/project/binutils-trunk/gdb/main.c, line 310.
(gdb) b captured_command_loop
Breakpoint 3 at 0x5a68b0: file /home/xmj/project/binutils-trunk/gdb/main.c, line 261. 
```

## 技巧

在 gdb 中，可以使用如下命令将设置的断点保存下来：

```cpp
(gdb) save breakpoints file-name-to-save
```

下此调试时，可以使用如下命令批量设置保存的断点：

```cpp
(gdb) source file-name-to-save
```

```cpp
(gdb) info breakpoints 
Num     Type           Disp Enb Address            What
1       breakpoint     keep y   0x00000000005a7af0 in gdb_main at /home/xmj/project/binutils-trunk/gdb/main.c:1061
2       breakpoint     keep y   0x00000000005a6bd0 in captured_main at /home/xmj/project/binutils-trunk/gdb/main.c:310
3       breakpoint     keep y   0x00000000005a68b0 in captured_command_loop at /home/xmj/project/binutils-trunk/gdb/main.c:261 
```

详情参见[gdb 手册](https://sourceware.org/gdb/download/onlinedocs/gdb/Save-Breakpoints.html#Save-Breakpoints)

## 贡献者

xmj

# 设置临时断点

# 设置临时断点

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

在使用 gdb 时，如果想让断点只生效一次，可以使用“tbreak”命令（缩写为：tb）。以上面程序为例：

```cpp
(gdb) tb a.c:15
Temporary breakpoint 1 at 0x400500: file a.c, line 15.
(gdb) i b
Num     Type           Disp Enb Address            What
1       breakpoint     del  y   0x0000000000400500 in main at a.c:15
(gdb) r
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:15
15              printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
(gdb) i b
No breakpoints or watchpoints. 
```

首先在文件的第 15 行设置临时断点，当程序断住后，用“i b”（"info breakpoints"缩写）命令查看断点，发现断点没有了。也就是断点命中一次后，就被删掉了。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Breaks.html)

## 贡献者

nanxiao

# 设置条件断点

# 设置条件断点

## 例子

```cpp
#include <stdio.h>

int main(void)
{
        int i = 0;
        int sum = 0;

        for (i = 1; i <= 200; i++)
        {
            sum += i;
        }

        printf("%d\n", sum);
        return 0;
} 
```

## 技巧

gdb 可以设置条件断点，也就是只有在条件满足时，断点才会被触发，命令是“`break … if cond`”。以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4004cc: file a.c, line 5.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:5
5                       int i = 0;
(gdb) b 10 if i==101
Breakpoint 2 at 0x4004e3: file a.c, line 10.
(gdb) r
Starting program: /data2/home/nanxiao/a

Breakpoint 2, main () at a.c:10
10                                      sum += i;
(gdb) p sum
$1 = 5050 
```

可以看到设定断点只在`i`的值为`101`时触发，此时打印`sum`的值为`5050`。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Set-Breaks.html)

## 贡献者

nanxiao

# 忽略断点

# 忽略断点

## 例子

```cpp
#include <stdio.h>

int main(void)
{
        int i = 0;
        int sum = 0;

        for (i = 1; i <= 200; i++)
        {
            sum += i;
        }

        printf("%d\n", sum);
        return 0;
} 
```

## 技巧

在设置断点以后，可以忽略断点，命令是“`ignore bnum count`”：意思是接下来`count`次编号为`bnum`的断点触发都不会让程序中断，只有第`count + 1`次断点触发才会让程序中断。以上面程序为例：

```cpp
(gdb) b 10
Breakpoint 1 at 0x4004e3: file a.c, line 10.
(gdb) ignore 1 5
Will ignore next 5 crossings of breakpoint 1.
(gdb) r
Starting program: /data2/home/nanxiao/a

Breakpoint 1, main () at a.c:10
10                                      sum += i;
(gdb) p i
$1 = 6 
```

可以看到设定忽略断点前`5`次触发后，第一次断点断住时，打印`i`的值是`6`。如果想让断点下次就生效，可以将`count`置为`0`：“`ignore 1 0`”。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Conditions.html)

## 贡献者

nanxiao