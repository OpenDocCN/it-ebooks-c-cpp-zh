# 预处理

# 生成没有行号标记的预处理文件

## 技巧

有时编译程序会遇到如下类似的错误，

```cpp
In file included from foo.c:15,
from a.h:45,
b.h:53: error: ... ... 
```

如果错误是由于你所定义的一个很复杂的宏所引起的，你可能会需要先手动编译生成相应的预处理文件，查看下预处理文件中的宏扩展代码。比如，先运行

```cpp
gcc -E foo.c -o foo.i 
```

来生成 foo.i 预处理文件。然后，还可以尝试手动修改、编译这个预处理文件。

但是，由于生成的预处理文件中含有行号标记（linemarker），所以，运行

```cpp
gcc -c foo.i -o foo.o 
```

所得到的错误行号信息还是跟最初的一样，如果可以将预处理文件中的行号标记都去掉，似乎会有些帮助。

幸好，gcc 提供了这个选项：

> -P Inhibit generation of linemarkers in the output from the preprocessor. This might be useful when running the preprocessor on something that is not C code, and will be sent to a program which might be confused by the linemarkers.

运行

```cpp
gcc -E -P foo.c -o foo.i 
```

即可。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

## 贡献者

xmj

# 在命令行中预定义宏

## 例子

```cpp
#include <stdio.h>

int main (void)
{
  int i, sum;

  for (i = 1, sum = 0; i <= 10; i++)
    {
      sum += i;
    #ifdef DEBUG
      printf ("sum += %d is %d\n", i, sum);
    #endif
    }
  printf ("total sum is %d\n", sum);

  return 0;
} 
```

## 技巧

使用`-D`选项可以在命令行中预定义一个宏，比如：

```cpp
$ gcc -D DEBUG macro.c 
```

中间可以没有空格：

```cpp
$ gcc -DDEBUG macro.c 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

## 贡献者

xmj

# 在命令行中取消宏定义

## 技巧

类似于`-D`选项，你可以使用`-U`选项在命令行中取消一个宏的定义，比如：

```cpp
$ gcc -U DEBUG macro.c 
```

中间可以没有空格：

```cpp
$ gcc -UDEBUG macro.c 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

## 贡献者

xmj