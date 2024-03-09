# 其它

# 只做语法检查

## 例子

```cpp
$ cat foo.c
union u {
  char c;
  int i;
}
$ gcc -fsyntax-only foo.c
foo.c:4:1: error: expected identifier or ‘(’ at end of input 
```

## 技巧

如上所示，使用`-fsyntax-only`选项可以只做语法检查，不进行实际的编译输出。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-fsyntax-only-274)

## 贡献者

xmj

# 保存临时文件

## 例子

```cpp
$ gcc -save-temps a/foo.c
$ ls foo.*
foo.c  foo.i  foo.o  foo.s

$ gcc -save-temps=obj a/foo.c -o a/foo
$ ls a
foo  foo.c  foo.i  foo.o  foo.s 
```

## 技巧

如上所示，使用选项`-save-temps`可以保存 gcc 运行过程中生成的临时文件。这些中间文件的名字是基于源文件而来，并且保存在当前目录下。

如果你在不同目录下有重名的源文件，那么中间文件就会有冲突了。此时，你可以使用`-save-temps=obj`来指定中间文件名基于目标文件而定，并保存在目标文件所在目录下。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html#Debugging-Options)

## 贡献者

xmj

# 打开警告信息

## 技巧

你的程序编译通过了，但并不意味着已经万事大吉，也许还存在一些不规范的地方，或者一些错误隐患。建议，使用`-Wall`选项打开所有的警告信息，把所有的警告都处理掉。

```cpp
$ gcc -Wall ... 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

## 贡献者

xmj

# 指定语言类型

## 技巧

gcc 是通过文件名后缀来判断源代码语言类型的。

如果你从标准输入把源码传给 gcc，那么就需要通过`-x`选项显式的指定语言类型：

```cpp
$ echo "int x;" | gcc -S -x c -
$ cat ./-.s
    .file    ""
    .comm    x,4,4
    .ident    "GCC: (Ubuntu/Linaro 4.6.3-1ubuntu5) 4.6.3"
    .section    .note.GNU-stack,"",@progbits 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#Overall-Options)

## 贡献者

xmj

# 改变结构体成员的字节对齐

## 例子

```cpp
#include <stdio.h>

typedef struct
{
        char a;
        int b;
} ST_A;

int main(void)
{
        printf("sizeof(ST_A)=%ld\n",sizeof(ST_A));
} 
```

## 技巧

在上面的程序里，`ST_A`结构体的内存布局默认是这样的：

| Offset | 1byte | 1byte | 1byte | 1byte |
| 0 | a | 填充字节 | 填充字节 | 填充字节 |
| 4 | b | b | b | b |

编译执行，结果如下：

```cpp
root@ubuntu:~$ gcc -g -o a a.c
root@ubuntu:~$ ./a
sizeof(ST_A)=8 
```

使用 gcc 的"`-fpack-struct[=n]`"选项（“`n`”需要为`2`的倍数）可以改变成员的地址对齐。例如指定“`n=2`”时，将标明结构体成员的最大对齐地址为 2。这样`ST_A`结构体中的成员`b`的地址将不再按照`4`字节对齐，内存布局变为：

| Offset | 1byte | 1byte | 1byte | 1byte |
| 0 | a | 填充字节 | b | b |
| 4 | b | b |  |  |

编译执行，结果如下：

```cpp
root@ubuntu:~$ gcc -g -fpack-struct=2 -o a a.c
root@ubuntu:~$ ./a
sizeof(ST_A)=6 
```

当不指定“`n`”时，将没有填充字节，所有成员将一个挨着一个排在一起：

| Offset | 1byte | 1byte | 1byte | 1byte |
| 0 | a | b | b | b |
| 4 | b |  |  |  |

编译执行，结果如下：

```cpp
root@ubuntu:~$ gcc -g -fpack-struct -o a a.c
root@ubuntu:~$ ./a
sizeof(ST_A)=5 
```

由于这个编译选项会导致 ABI(Application Binary Interface)的改变，所以使用时一定要谨慎。 详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html)

## 贡献者

nanxiao