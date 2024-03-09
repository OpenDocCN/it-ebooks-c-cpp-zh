# 汇编

# 设置汇编指令格式

## 例子

```cpp
#include <stdio.h>
int global_var;

void change_var(){
    global_var=100;
}

int main(void){
    change_var();
    return 0;
} 
```

## 技巧

在 Intel x86 处理器上，gdb 默认显示汇编指令格式是 AT&T 格式。例如：

```cpp
(gdb) disassemble main
Dump of assembler code for function main:
   0x08050c0f <+0>:     push   %ebp
   0x08050c10 <+1>:     mov    %esp,%ebp
   0x08050c12 <+3>:     call   0x8050c00 <change_var>
   0x08050c17 <+8>:     mov    $0x0,%eax
   0x08050c1c <+13>:    pop    %ebp
   0x08050c1d <+14>:    ret
End of assembler dump. 
```

可以用“set disassembly-flavor”命令将格式改为 intel 格式：

```cpp
(gdb) set disassembly-flavor intel
(gdb) disassemble main
Dump of assembler code for function main:
   0x08050c0f <+0>:     push   ebp
   0x08050c10 <+1>:     mov    ebp,esp
   0x08050c12 <+3>:     call   0x8050c00 <change_var>
   0x08050c17 <+8>:     mov    eax,0x0
   0x08050c1c <+13>:    pop    ebp
   0x08050c1d <+14>:    ret
End of assembler dump. 
```

目前“set disassembly-flavor”命令只能用在 Intel x86 处理器上，并且取值只有“intel”和“att”。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Machine-Code.html)

## 贡献者

nanxiao

# 在函数的第一条汇编指令打断点

## 例子

```cpp
#include <stdio.h>
int global_var;

void change_var(){
    global_var=100;
}

int main(void){
    change_var();
    return 0;
} 
```

## 技巧

通常给函数打断点的命令：“b func”（b 是 break 命令的缩写），不会把断点设置在汇编指令层次函数的开头，例如：

```cpp
(gdb) b main
Breakpoint 1 at 0x8050c12: file a.c, line 9.
(gdb) r
Starting program: /data1/nan/a
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Breakpoint 1, main () at a.c:9
9           change_var();
(gdb) disassemble
Dump of assembler code for function main:
   0x08050c0f <+0>:     push   %ebp
   0x08050c10 <+1>:     mov    %esp,%ebp
=> 0x08050c12 <+3>:     call   0x8050c00 <change_var>
   0x08050c17 <+8>:     mov    $0x0,%eax
   0x08050c1c <+13>:    pop    %ebp
   0x08050c1d <+14>:    ret
End of assembler dump. 
```

可以看到程序停在了第三条汇编指令（箭头所指位置）。如果要把断点设置在汇编指令层次函数的开头，要使用如下命令：“b *func”，例如：

```cpp
(gdb) b *main
Breakpoint 1 at 0x8050c0f: file a.c, line 8.
(gdb) r
Starting program: /data1/nan/a
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Breakpoint 1, main () at a.c:8
8       int main(void){
(gdb) disassemble
Dump of assembler code for function main:
=> 0x08050c0f <+0>:     push   %ebp
   0x08050c10 <+1>:     mov    %esp,%ebp
   0x08050c12 <+3>:     call   0x8050c00 <change_var>
   0x08050c17 <+8>:     mov    $0x0,%eax
   0x08050c1c <+13>:    pop    %ebp
   0x08050c1d <+14>:    ret
End of assembler dump. 
```

可以看到程序停在了第一条汇编指令（箭头所指位置）。

## 贡献者

nanxiao

# 自动反汇编后面要执行的代码

## 例子

```cpp
(gdb) set disassemble-next-line on
(gdb) start 
The program being debugged has been started already.
Start it from the beginning? (y or n) y
Temporary breakpoint 3 at 0x400543: file 1.c, line 14.
Starting program: /home/teawater/tmp/a.out 

Temporary breakpoint 3, main (argc=1, argv=0x7fffffffdf38, envp=0x7fffffffdf48) at 1.c:14
14      printf("1\n");
=> 0x0000000000400543 <main+19>:    bf f0 05 40 00  mov    $0x4005f0,%edi
   0x0000000000400548 <main+24>:    e8 c3 fe ff ff  callq  0x400410 <puts@plt>
(gdb) si
0x0000000000400548  14      printf("1\n");
0x0000000000400543 <main+19>:    bf f0 05 40 00  mov    $0x4005f0,%edi
=> 0x0000000000400548 <main+24>:    e8 c3 fe ff ff  callq  0x400410 <puts@plt>
(gdb) 
0x0000000000400410 in puts@plt ()
=> 0x0000000000400410 <puts@plt+0>: ff 25 02 0c 20 00   jmpq   *0x200c02(%rip)        # 0x601018 <puts@got.plt>

(gdb) set disassemble-next-line auto 
(gdb) start 
Temporary breakpoint 1 at 0x400543: file 1.c, line 14.
Starting program: /home/teawater/tmp/a.out 

Temporary breakpoint 1, main (argc=1, argv=0x7fffffffdf38, envp=0x7fffffffdf48) at 1.c:14
14      printf("1\n");
(gdb) si
0x0000000000400548  14      printf("1\n");
(gdb) 
0x0000000000400410 in puts@plt ()
=> 0x0000000000400410 <puts@plt+0>: ff 25 02 0c 20 00   jmpq   *0x200c02(%rip)        # 0x601018 <puts@got.plt>
(gdb) 
0x0000000000400416 in puts@plt ()
=> 0x0000000000400416 <puts@plt+6>: 68 00 00 00 00  pushq  $0x0 
```

## 技巧

如果要在任意情况下反汇编后面要执行的代码：

```cpp
(gdb) set disassemble-next-line on 
```

如果要在后面的代码没有源码的情况下才反汇编后面要执行的代码：

```cpp
(gdb) set disassemble-next-line auto 
```

关闭这个功能：

```cpp
(gdb) set disassemble-next-line off 
```

## 贡献者

teawater

# 将源程序和汇编指令映射起来

## 例子

```cpp
#include <stdio.h>

typedef struct
{
        int a;
        int b;
        int c;
        int d;
}ex_st;

int main(void) {
        ex_st st = {1, 2, 3, 4};
        printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
        return 0;
} 
```

## 技巧一

可以用“disas /m fun”（disas 是 disassemble 命令缩写）命令将函数代码和汇编指令映射起来，以上面代码为例：

```cpp
(gdb) disas /m main
Dump of assembler code for function main:
11      int main(void) {
   0x00000000004004c4 <+0>:     push   %rbp
   0x00000000004004c5 <+1>:     mov    %rsp,%rbp
   0x00000000004004c8 <+4>:     push   %rbx
   0x00000000004004c9 <+5>:     sub    $0x18,%rsp

12              ex_st st = {1, 2, 3, 4};
   0x00000000004004cd <+9>:     movl   $0x1,-0x20(%rbp)
   0x00000000004004d4 <+16>:    movl   $0x2,-0x1c(%rbp)
   0x00000000004004db <+23>:    movl   $0x3,-0x18(%rbp)
   0x00000000004004e2 <+30>:    movl   $0x4,-0x14(%rbp)

13              printf("%d,%d,%d,%d\n", st.a, st.b, st.c, st.d);
   0x00000000004004e9 <+37>:    mov    -0x14(%rbp),%esi
   0x00000000004004ec <+40>:    mov    -0x18(%rbp),%ecx
   0x00000000004004ef <+43>:    mov    -0x1c(%rbp),%edx
   0x00000000004004f2 <+46>:    mov    -0x20(%rbp),%ebx
   0x00000000004004f5 <+49>:    mov    $0x400618,%eax
   0x00000000004004fa <+54>:    mov    %esi,%r8d
   0x00000000004004fd <+57>:    mov    %ebx,%esi
   0x00000000004004ff <+59>:    mov    %rax,%rdi
   0x0000000000400502 <+62>:    mov    $0x0,%eax
   0x0000000000400507 <+67>:    callq  0x4003b8 <printf@plt>

14              return 0;
   0x000000000040050c <+72>:    mov    $0x0,%eax

15      }
   0x0000000000400511 <+77>:    add    $0x18,%rsp
   0x0000000000400515 <+81>:    pop    %rbx
   0x0000000000400516 <+82>:    leaveq
   0x0000000000400517 <+83>:    retq

End of assembler dump. 
```

可以看到每一条 C 语句下面是对应的汇编代码。

## 技巧二

如果只想查看某一行所对应的地址范围，可以：

```cpp
(gdb) i line 13
Line 13 of "foo.c" starts at address 0x4004e9 <main+37> and ends at 0x40050c <main+72>. 
```

如果只想查看这一条语句对应的汇编代码，可以使用“`disassemble [Start],[End]`”命令：

```cpp
(gdb) disassemble 0x4004e9, 0x40050c
Dump of assembler code from 0x4004e9 to 0x40050c:
   0x00000000004004e9 <main+37>:        mov    -0x14(%rbp),%esi
   0x00000000004004ec <main+40>:        mov    -0x18(%rbp),%ecx
   0x00000000004004ef <main+43>:        mov    -0x1c(%rbp),%edx
   0x00000000004004f2 <main+46>:        mov    -0x20(%rbp),%ebx
   0x00000000004004f5 <main+49>:        mov    $0x400618,%eax
   0x00000000004004fa <main+54>:        mov    %esi,%r8d
   0x00000000004004fd <main+57>:        mov    %ebx,%esi
   0x00000000004004ff <main+59>:        mov    %rax,%rdi
   0x0000000000400502 <main+62>:        mov    $0x0,%eax
   0x0000000000400507 <main+67>:        callq  0x4003b8 <printf@plt>
End of assembler dump. 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Machine-Code.html)

## 贡献者

nanxiao

xmj

# 显示将要执行的汇编指令

## 例子

```cpp
#include <stdio.h>
int global_var;

void change_var(){
    global_var=100;
}

int main(void){
    change_var();
    return 0;
} 
```

## 技巧

使用 gdb 调试汇编程序时，可以用“`display /i $pc`”命令显示当程序停止时，将要执行的汇编指令。以上面程序为例：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x400488: file a.c, line 9.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:9
9           change_var();
(gdb) display /i $pc
1: x/i $pc
=> 0x400488 <main+4>:   mov    $0x0,%eax
(gdb) si
0x000000000040048d      9           change_var();
1: x/i $pc
=> 0x40048d <main+9>:   callq  0x400474 <change_var>
(gdb)
change_var () at a.c:4
4       void change_var(){
1: x/i $pc
=> 0x400474 <change_var>:       push   %rbp 
```

可以看到打印出了将要执行的汇编指令。此外也可以一次显示多条指令：

```cpp
(gdb) display /3i $pc
2: x/3i $pc
=> 0x400474 <change_var>:       push   %rbp
   0x400475 <change_var+1>:     mov    %rsp,%rbp
   0x400478 <change_var+4>:     movl   $0x64,0x2003de(%rip)        # 0x600860 <global_var> 
```

可以看到一次显示了`3`条指令。

取消显示可以用`undisplay`命令。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Auto-Display.html)

## 贡献者

nanxiao

# 打印寄存器的值

## 技巧

用 gdb 调试程序时，如果想查看寄存器的值，可以使用“i registers”命令（i 是 info 命令缩写），例如:

```cpp
(gdb) i registers
rax            0x7ffff7dd9f60   140737351884640
rbx            0x0      0
rcx            0x0      0
rdx            0x7fffffffe608   140737488348680
rsi            0x7fffffffe5f8   140737488348664
rdi            0x1      1
rbp            0x7fffffffe510   0x7fffffffe510
rsp            0x7fffffffe4c0   0x7fffffffe4c0
r8             0x7ffff7dd8300   140737351877376
r9             0x7ffff7deb9e0   140737351956960
r10            0x7fffffffe360   140737488348000
r11            0x7ffff7a68be0   140737348275168
r12            0x4003e0 4195296
r13            0x7fffffffe5f0   140737488348656
r14            0x0      0
r15            0x0      0
rip            0x4004cd 0x4004cd <main+9>
eflags         0x206    [ PF IF ]
cs             0x33     51
ss             0x2b     43
ds             0x0      0
es             0x0      0
fs             0x0      0
gs             0x0      0 
```

以上输出不包括浮点寄存器和向量寄存器的内容。使用“i all-registers”命令，可以输出所有寄存器的内容：

```cpp
(gdb) i all-registers
    rax            0x7ffff7dd9f60   140737351884640
    rbx            0x0      0
    rcx            0x0      0
    rdx            0x7fffffffe608   140737488348680
    rsi            0x7fffffffe5f8   140737488348664
    rdi            0x1      1
    rbp            0x7fffffffe510   0x7fffffffe510
    rsp            0x7fffffffe4c0   0x7fffffffe4c0
    r8             0x7ffff7dd8300   140737351877376
    r9             0x7ffff7deb9e0   140737351956960
    r10            0x7fffffffe360   140737488348000
    r11            0x7ffff7a68be0   140737348275168
    r12            0x4003e0 4195296
    r13            0x7fffffffe5f0   140737488348656
    r14            0x0      0
    r15            0x0      0
    rip            0x4004cd 0x4004cd <main+9>
    eflags         0x206    [ PF IF ]
    cs             0x33     51
    ss             0x2b     43
    ds             0x0      0
    es             0x0      0
    fs             0x0      0
    gs             0x0      0
    st0            0        (raw 0x00000000000000000000)
    st1            0        (raw 0x00000000000000000000)
    st2            0        (raw 0x00000000000000000000)
    st3            0        (raw 0x00000000000000000000)
    st4            0        (raw 0x00000000000000000000)
    st5            0        (raw 0x00000000000000000000)
    st6            0        (raw 0x00000000000000000000)
    st7            0        (raw 0x00000000000000000000)
    ...... 
```

要打印单个寄存器的值，可以使用“i registers regname”或者“p $regname”，例如：

```cpp
(gdb) i registers eax
eax            0xf7dd9f60       -136470688
(gdb) p $eax
$1 = -136470688 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Registers.html).

## 贡献者

nanxiao

# 显示程序原始机器码

## 例子

```cpp
#include <stdio.h>

int main(void)
{
        printf("Hello, world\n");
        return 0;
} 
```

## 技巧

使用“disassemble /r”命令可以用 16 进制形式显示程序的原始机器码。以上面程序为例：

```cpp
(gdb) disassemble /r main
Dump of assembler code for function main:
   0x0000000000400530 <+0>:     55      push   %rbp
   0x0000000000400531 <+1>:     48 89 e5        mov    %rsp,%rbp
   0x0000000000400534 <+4>:     bf e0 05 40 00  mov    $0x4005e0,%edi
   0x0000000000400539 <+9>:     e8 d2 fe ff ff  callq  0x400410 <puts@plt>
   0x000000000040053e <+14>:    b8 00 00 00 00  mov    $0x0,%eax
   0x0000000000400543 <+19>:    5d      pop    %rbp
   0x0000000000400544 <+20>:    c3      retq
End of assembler dump.
(gdb) disassemble /r 0x0000000000400534,+4
Dump of assembler code from 0x400534 to 0x400538:
   0x0000000000400534 <main+4>: bf e0 05 40 00  mov    $0x4005e0,%edi
End of assembler dump. 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Machine-Code.html)

## 贡献者

nanxiao