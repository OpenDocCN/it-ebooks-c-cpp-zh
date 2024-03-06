# 信息显示

# 显示 gdb 版本信息

## 技巧

使用 gdb 时，如果想查看 gdb 版本信息，可以使用“`show version`”命令:

```cpp
(gdb) show version
GNU gdb (GDB) 7.7.1
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-pc-solaris2.10".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word". 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Help.html#index-GDB-version-number)。

## 贡献者

nanxiao

# 显示 gdb 版权相关信息

## 技巧

使用 gdb 时，如果想查看 gdb 版权相关信息，可以使用“`show copying`”命令:

```cpp
(gdb) show copying
                GNU GENERAL PUBLIC LICENSE
                   Version 3, 29 June 2007

 Copyright (C) 2007 Free Software Foundation, Inc. <http://fsf.org/>
 Everyone is permitted to copy and distribute verbatim copies
 of this license document, but changing it is not allowed.

                            Preamble

  The GNU General Public License is a free, copyleft license for
software and other kinds of works.

  The licenses for most software and other practical works are designed
to take away your freedom to share and change the works.  By contrast,
the GNU General Public License is intended to guarantee your freedom to
share and change all versions of a program--to make sure it remains free
software for all its use
...... 
```

或者“`show warranty`”命令：

```cpp
(gdb) show warranty
  15\. Disclaimer of Warranty.

  THERE IS NO WARRANTY FOR THE PROGRAM, TO THE EXTENT PERMITTED BY
APPLICABLE LAW.  EXCEPT WHEN OTHERWISE STATED IN WRITING THE COPYRIGHT
HOLDERS AND/OR OTHER PARTIES PROVIDE THE PROGRAM "AS IS" WITHOUT WARRANTY
OF ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING, BUT NOT LIMITED TO,
THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE.  THE ENTIRE RISK AS TO THE QUALITY AND PERFORMANCE OF THE PROGRAM
IS WITH YOU.  SHOULD THE PROGRAM PROVE DEFECTIVE, YOU ASSUME THE COST OF
ALL NECESSARY SERVICING, REPAIR OR CORRECTION.

  16\. Limitation of Liability.

  IN NO EVENT UNLESS REQUIRED BY APPLICABLE LAW OR AGREED TO IN WRITING
WILL ANY COPYRIGHT HOLDER, OR ANY OTHER PARTY WHO MODIFIES AND/OR CONVEYS
THE PROGRAM AS PERMITTED ABOVE, BE LIABLE TO YOU FOR DAMAGES, INCLUDING ANY
GENERAL, SPECIAL, INCIDENTAL OR CONSEQUENTIAL DAMAGES ARISING OUT OF THE
USE OR INABILITY TO USE THE PROGRAM (INCLUDING BUT NOT LIMITED TO LOSS OF
DATA OR DATA BEING RENDERED INACCURATE OR LOSSES SUSTAINED BY YOU OR THIRD
PARTIES OR A FAILURE OF THE PROGRAM TO OPERATE WITH ANY OTHER PROGRAMS),
EVEN IF SUCH HOLDER OR OTHER PARTY HAS BEEN ADVISED OF THE POSSIBILITY OF
SUCH DAMAGES.

  17\. Interpretation of Sections 15 and 16.

  If the disclaimer of warranty and limitation of liability provided
above cannot be given local legal effect according to their terms,
reviewing courts shall apply local law that most closely approximates
an absolute waiver of all civil liability in connection with the
Program, unless a warranty or assumption of liability accompanies a
copy of the Program in return for a fee. 
```

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Help.html#index-GDB-version-number)。

## 贡献者

nanxiao

# 启动时不显示提示信息

## 例子

```cpp
$ gdb
GNU gdb (GDB) 7.7.50.20140228-cvs
Copyright (C) 2014 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word". 
```

## 技巧

gdb 在启动时会显示如上类似的提示信息。

如果不想显示这个信息，则可以使用`-q`选项把提示信息关掉:

```cpp
$ gdb -q
(gdb) 
```

你可以在~/.bashrc 中，为 gdb 设置一个别名：

```cpp
alias gdb="gdb -q" 
```

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Invoking-GDB.html#Invoking-GDB)

## 贡献者

xmj

# 退出时不显示提示信息

# gdb 退出时不显示提示信息

## 技巧

gdb 在退出时会提示:

```cpp
A debugging session is active.

    Inferior 1 [process 29686    ] will be killed.

Quit anyway? (y or n) n 
```

如果不想显示这个信息，则可以在 gdb 中使用如下命令把提示信息关掉:

```cpp
(gdb) set confirm off 
```

也可以把这个命令加到.gdbinit 文件里。

## 贡献者

nanxiao

# 输出信息多时不会暂停输出

## 技巧

有时当 gdb 输出信息较多时，gdb 会暂停输出，并会打印“`---Type <return> to continue, or q <return> to quit---`”这样的提示信息，如下面所示：

```cpp
 81 process 2639102      0xff04af84 in __lwp_park () from /usr/lib/libc.so.1
 80 process 2573566      0xff04af84 in __lwp_park () from /usr/lib/libc.so.1
---Type <return> to continue, or q <return> to quit---Quit 
```

解决办法是使用“`set pagination off`”或者“`set height 0`”命令。这样 gdb 就会全部输出，中间不会暂停。
参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Screen-Size.html).

## 贡献者

nanxiao