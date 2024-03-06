# 其它

# 其它

# 命令行选项的格式

# 命令行选项的格式

## 技巧

gdb 的帮助信息和在线文档对于长选项的形式使用了不同的风格。你可能有点迷惑，gdb 的长选项究竟应该是“-”，还是“--”？

是的，这两种方式都可以。例如：

```cpp
$ gdb -help
$ gdb --help

$ gdb -args ./a.out a b c
$ gdb --args ./a.out a b c 
```

好吧，使用短的。

## 贡献者

xmj

# 支持预处理器宏信息

# 支持预处理器宏信息

## 例子

```cpp
#include <stdio.h>

#define NAME "Joe"

int main()
{
  printf ("Hello %s\n", NAME);
  return 0;
} 
```

## 技巧

使用`gcc -g`编译生成的程序，是不包含预处理器宏信息的：

```cpp
(gdb) p NAME
No symbol "NAME" in current context. 
```

如果想在 gdb 中查看宏信息，可以使用`gcc -g3`进行编译：

```cpp
(gdb) p NAME
$1 = "Joe" 
```

关于预处理器宏的命令，参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Macros.html#Macros)

## 贡献者

xmj

# 使用命令的缩写形式

# 使用命令的缩写形式

## 技巧

在 gdb 中，你不用必须输入完整的命令，只需命令的（前）几个字母即可。规则是，只要这个缩写不会和其它命令有歧义（注，是否有歧义，这个规则从文档上看不出，看起来需要查看 gdb 的源代码，或者在实际使用中进行总结）。也可以使用 tab 键进行命令补全。

其中许多常用命令只使用第一个字母就可以，比如：

```cpp
b -> break
c -> continue
d -> delete
f -> frame
i -> info
j -> jump
l -> list
n -> next
p -> print
r -> run
s -> step
u -> until 
```

也有使用两个或几个字母的，比如：

```cpp
aw -> awatch
bt -> backtrace
dir -> directory
disas -> disassemble
fin -> finish
ig -> ignore
ni -> nexti
rw -> rwatch
si -> stepi
tb -> tbreak
wa -> watch
win -> winheight 
```

另外，如果直接按回车键，会重复执行上一次的命令。

## 贡献者

xmj

nanxiao

# 在 gdb 中执行 shell 命令和 make

# 在 gdb 中执行 shell 命令和 make

## 技巧

你可以不离开 gdb，直接执行 shell 命令，比如：

```cpp
(gdb) shell ls 
```

或

```cpp
(gdb) !ls 
```

这里，"!"和命令之间不需要有空格（即，有也成）。

特别是当你在构建环境(build 目录)下调试程序的时候，可以直接运行 make：

```cpp
(gdb) make CFLAGS="-g -O0" 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Shell-Commands.html#Shell-Commands)

## 贡献者

xmj

# 在 gdb 中执行 cd 和 pwd 命令

# 在 gdb 中执行 cd 和 pwd 命令

## 技巧

是的，gdb 确实支持这两个命令，虽然我没有想到它们有什么特别的用处。

也许，当你启动 gdb 之后，发现需要切换工作目录，但又不想退出 gdb 的时候：

```cpp
(gdb) pwd
Working directory /home/xmj.
(gdb) cd tmp
Working directory /home/xmj/tmp. 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Working-Directory.html#Working-Directory)

## 贡献者

xmj

# 设置命令提示符

# 设置命令提示符

## 例子

```cpp
$ gdb -q `which gdb`
Reading symbols from /home/xmj/install/binutils-gdb-git/bin/gdb...done.
(gdb) r -q
Starting program: /home/xmj/install/binutils-gdb-git/bin/gdb -q
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
(gdb) 
```

## 技巧

当你用 gdb 来调试 gdb 的时候，通过设置命令提示符，可以帮助你区分这两个 gdb：

```cpp
$ gdb -q `which gdb`
Reading symbols from /home/xmj/install/binutils-gdb-git/bin/gdb...done.
(gdb) set prompt (main gdb) 
(main gdb) r -q
Starting program: /home/xmj/install/binutils-gdb-git/bin/gdb -q
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
(gdb) 
```

注意，这里`set prompt (main gdb)`结尾处是有一个空格的。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Prompt.html#Prompt)

## 贡献者

xmj

# 设置被调试程序的参数

# 设置被调试程序的参数

## 技巧

可以在 gdb 启动时，通过选项指定被调试程序的参数，例如：

```cpp
$ gdb -args ./a.out a b c 
```

也可以在 gdb 中，通过命令来设置，例如：

```cpp
(gdb) set args a b c
(gdb) show args
Argument list to give program being debugged when it is started is "a b c". 
```

也可以在运行程序时，直接指定：

```cpp
(gdb) r a b
Starting program: /home/xmj/tmp/a.out a b
(gdb) show args
Argument list to give program being debugged when it is started is "a b".
(gdb) r
Starting program: /home/xmj/tmp/a.out a b 
```

可以看出，参数已经被保存了，下次运行时直接运行`run`命令，即可。

有意的是，如果我接下来，想让参数为空，该怎么办？是的，直接：

```cpp
(gdb) set args 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Arguments.html#Arguments)

## 贡献者

xmj

# 设置被调试程序的环境变量

# 设置被调试程序的环境变量

## 例子

```cpp
(gdb) u 309
Warning: couldn't activate thread debugging using libthread_db: Cannot find new threads: generic error
Warning: couldn't activate thread debugging using libthread_db: Cannot find new threads: generic error
warning: Unable to find libthread_db matching inferior's thread library, thread debugging will not be available. 
```

## 技巧

在 gdb 中，可以通过命令`set env varname=value`来设置被调试程序的环境变量。对于上面的例子，网上可以搜到一些解决方法，其中一种方法就是设置 LD_PRELOAD 环境变量：

```cpp
set env LD_PRELOAD=/lib/x86_64-linux-gnu/libpthread.so.0 
```

注意，这个实际路径在不同的机器环境下可能不一样。把这个命令加到~/.gdbinit 文件中，就可以了。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Environment.html#Environment)

## 贡献者

xmj

# 得到命令的帮助信息

# 得到命令的帮助信息

## 技巧

使用`help`命令可以得到 gdb 的命令帮助信息：

（1）`help`命令不加任何参数会得到命令的分类：

```cpp
(gdb) help
List of classes of commands:

aliases -- Aliases of other commands
breakpoints -- Making program stop at certain points
data -- Examining data
files -- Specifying and examining files
internals -- Maintenance commands
obscure -- Obscure features
running -- Running the program
stack -- Examining the stack
status -- Status inquiries
support -- Support facilities
tracepoints -- Tracing of program execution without stopping the program
user-defined -- User-defined commands

Type "help" followed by a class name for a list of commands in that class.
Type "help all" for the list of all commands.
Type "help" followed by command name for full documentation.
Type "apropos word" to search for commands related to "word".
Command name abbreviations are allowed if unambiguous. 
```

（2）当输入`help class`命令时，可以得到这个类别下所有命令的列表和命令功能：

```cpp
(gdb) help data
Examining data.

List of commands:

append -- Append target code/data to a local file
append binary -- Append target code/data to a raw binary file
append binary memory -- Append contents of memory to a raw binary file
append binary value -- Append the value of an expression to a raw binary file
append memory -- Append contents of memory to a raw binary file
append value -- Append the value of an expression to a raw binary file
call -- Call a function in the program
disassemble -- Disassemble a specified section of memory
display -- Print value of expression EXP each time the program stops
dump -- Dump target code/data to a local file
dump binary -- Write target code/data to a raw binary file
dump binary memory -- Write contents of memory to a raw binary file
dump binary value -- Write the value of an expression to a raw binary file
...... 
```

（3）也可以用`help command`命令得到某一个具体命令的用法：

```cpp
(gdb) help mem
Define attributes for memory region or reset memory region handling totarget-based.
Usage: mem auto
   mem <lo addr> <hi addr> [<mode> <width> <cache>],
where <mode>  may be rw (read/write), ro (read-only) or wo (write-only),
  <width> may be 8, 16, 32, or 64, and
  <cache> may be cache or nocache 
```

（4）用`apropos regexp`命令查找所有符合`regexp`正则表达式的命令信息：

```cpp
(gdb) apropos set
awatch -- Set a watchpoint for an expression
b -- Set breakpoint at specified line or function
br -- Set breakpoint at specified line or function
bre -- Set breakpoint at specified line or function
brea -- Set breakpoint at specified line or function
...... 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Help.html)

## 贡献者

nanxiao

# 记录执行 gdb 的过程

# 记录执行 gdb 的过程

## 例子

```cpp
#include <stdio.h>
#include <wchar.h>

int main(void)
{
        char str1[] = "abcd";
        wchar_t str2[] = L"abcd";

        return 0;
} 
```

## 技巧

用 gdb 调试程序时，可以使用“`set logging on`”命令把执行 gdb 的过程记录下来，方便以后自己参考或是别人帮忙分析。默认的日志文件是“`gdb.txt`”，也可以用“`set logging file file`”改成别的名字。以上面程序为例：

```cpp
(gdb) set logging file log.txt
(gdb) set logging on
Copying output to log.txt.
(gdb) start
Temporary breakpoint 1 at 0x8050abe: file a.c, line 6.
Starting program: /data1/nan/a 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Temporary breakpoint 1, main () at a.c:6
6               char str1[] = "abcd";
(gdb) n
7               wchar_t str2[] = L"abcd";
(gdb) x/s str1
0x804779f:      "abcd"
(gdb) n       
9               return 0;
(gdb) x/ws str2
0x8047788:      U"abcd"
(gdb) q
A debugging session is active.

        Inferior 1 [process 9931    ] will be killed.

Quit anyway? (y or n) y 
```

执行完后，查看 log.txt 文件：

```cpp
bash-3.2# cat log.txt 
Temporary breakpoint 1 at 0x8050abe: file a.c, line 6.
Starting program: /data1/nan/a 
[Thread debugging using libthread_db enabled]
[New Thread 1 (LWP 1)]
[Switching to Thread 1 (LWP 1)]

Temporary breakpoint 1, main () at a.c:6
6               char str1[] = "abcd";
7               wchar_t str2[] = L"abcd";
0x804779f:      "abcd"
9               return 0;
0x8047788:      U"abcd"
A debugging session is active.

        Inferior 1 [process 9931    ] will be killed.

Quit anyway? (y or n) 
```

可以看到 log.txt 详细地记录了 gdb 的执行过程。

此外“`set logging overwrite on`”命令可以让输出覆盖之前的日志文件；而 “`set logging redirect on`”命令会让 gdb 的日志不会打印在终端。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Logging-Output.html).

## 贡献者

nanxiao