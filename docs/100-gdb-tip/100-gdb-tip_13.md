# 脚本

# 脚本

# 配置 gdb init 文件

# 配置 gdb init 文件

## 技巧

当 gdb 启动时，会读取 HOME 目录和当前目录下的的配置文件，执行里面的命令。这个文件通常为“.gdbinit”。

这里给出了本文档中介绍过的，可以放在“.gdbinit”中的一些配置：

```cpp
# 打印 STL 容器中的内容
python
import sys
sys.path.insert(0, "/home/xmj/project/gcc-trunk/libstdc++-v3/python")
from libstdcxx.v6.printers import register_libstdcxx_printers
register_libstdcxx_printers (None)
end

# 保存历史命令
set history filename ~/.gdb_history
set history save on

# 退出时不显示提示信息
set confirm off

# 按照派生类型打印对象
set print object on

# 打印数组的索引下标
set print array-indexes on

# 每行打印一个结构体成员
set print pretty on 
```

欢迎补充。

## 贡献者

xmj

# 按何种方式解析脚本文件

# 按何种方式解析脚本文件

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

## 技巧

gdb 支持的脚本文件分为两种：一种是只包含 gdb 自身命令的脚本，例如“.gdbinit”文件，当 gdb 在启动时，就会执行“.gdbinit”文件中的命令；此外，gdb 还支持其它一些语言写的脚本文件（比如 python）。
gdb 用“`set script-extension`”命令来决定按何种格式来解析脚本文件。它可以取 3 个值：
a）`off`：所有的脚本文件都解析成 gdb 的命令脚本；
b）`soft`：根据脚本文件扩展名决定如何解析脚本。如果 gdb 支持解析这种脚本语言（比如 python），就按这种语言解析，否则就按命令脚本解析；
c）`strict`：根据脚本文件扩展名决定如何解析脚本。如果 gdb 支持解析这种脚本语言（比如 python），就按这种语言解析，否则不解析；
以上面程序为例，进行调试：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4004cd: file a.c, line 12.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:12
12              ex_st st = {1, 2, 3, 4};
(gdb) q
A debugging session is active.

        Inferior 1 [process 24249] will be killed.

Quit anyway? (y or n) y 
```

可以看到 gdb 退出时，默认行为会提示用户是否退出。

下面写一个脚本文件（gdb.py），但内容是一个 gdb 命令，不是真正的 python 脚本。用途是退出 gdb 时不提示：

```cpp
set confirm off 
```

再次开始调试：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4004cd: file a.c, line 12.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:12
12              ex_st st = {1, 2, 3, 4};
(gdb) show script-extension
Script filename extension recognition is "soft".
(gdb) source gdb.py
  File "gdb.py", line 1
    set confirm off
              ^
SyntaxError: invalid syntax 
```

可以看到“`script-extension`”默认值是`soft`，接下来执行“`source gdb.py`”,会按照 pyhton 语言解析 gdb.py 文件，但是由于这个文件实质上是一个 gdb 命令脚本，所以解析出错。
再执行一次：

```cpp
(gdb) start
Temporary breakpoint 1 at 0x4004cd: file a.c, line 12.
Starting program: /data2/home/nanxiao/a

Temporary breakpoint 1, main () at a.c:12
12              ex_st st = {1, 2, 3, 4};
(gdb) set script-extension off
(gdb) source gdb.py
(gdb) q
[root@linux:~]$ 
```

这次把“`script-extension`”值改为`off`，所以脚本会按 gdb 命令脚本去解析，可以看到这次脚本命令生效了。

参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Extending-GDB.html)

## 贡献者

nanxiao

# 保存历史命令

# 保存历史命令

## 技巧

在 gdb 中，缺省是不保存历史命令的。你可以通过如下命令来设置成保存历史命令：

```cpp
(gdb) set history save on 
```

但是，历史命令是缺省保存在了当前目录下的.gdb_history 文件中。可以通过如下命令来设置要保存的文件名和路径：

```cpp
(gdb) set history filename fname 
```

现在，我们把这两个命令放到$HOME/.gdbinit 文件中：

```cpp
set history filename ~/.gdb_history
set history save on 
```

好了，下次启动 gdb 时，你就可以直接查找使用之前的历史命令了。

详情参见[gdb 手册](https://sourceware.org/gdb/onlinedocs/gdb/Command-History.html#Command-History)

## 贡献者

xmj