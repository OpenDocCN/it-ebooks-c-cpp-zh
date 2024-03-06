# 常见错误

# 常见错误

# error: cast from ... to ... loses precision

# error: cast from ... to ... loses precision

## 例子

```cpp
#include <iostream>

class Foo {
 public:
  void print() const {
    std::cout << (int)(this) << "\n";
  }
};

int main()
{
  class Foo foo;

  foo.print();
  return 0;
} 
```

## 技巧

在 g++编译上面的例子，会报如下错误：

```cpp
$ g++ foo.cc 
foo.cc: In member function ‘void Foo::print() const’:
foo.cc:6:28: error: cast from ‘const Foo*’ to ‘int’ loses precision [-fpermissive] 
```

这是一个强制类型转换的错误，你可以修改源代码为：

```cpp
std::cout << (int*)(this) << "\n"; 
```

即可。

如果，你不想（或不能）去修改源程序，只是应为升级了 gcc 而带来了这样的错误，那么也可以使用`-fpermissive`选项，将错误降低为警告：

```cpp
$ g++ foo.cc -fpermissive
foo.cc: In member function ‘void Foo::print() const’:
foo.cc:6:28: warning: cast from ‘const Foo*’ to ‘int’ loses precision [-fpermissive] 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/C_002b_002b-Dialect-Options.html#index-fpermissive-166)

## 贡献者

xmj

# all warnings being treated as errors

# all warnings being treated as errors

## 技巧

在 ubuntu 系统下编译一个程序包，有时会遇到这样的错误：

```cpp
$ make
...
cc1: all warnings being treated as errors 
```

这是因为缺省的 CFLAGS 里含有`-Werror`选项，将警告信息升级为错误。当然，一方面这可以让你重视这些可能会带来隐患的警告信息；但，如果你不想修改源码，也可以把这个选项关掉，通过修改 Makefile 或者使用命令行：

```cpp
$ make CFLAGS="... -Wno-error" 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

## 贡献者

xmj