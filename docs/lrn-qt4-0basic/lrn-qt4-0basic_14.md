# 附录 A qmake 使用指南

# 附录 A qmake 使用指南

# A.1 qmake 简介

## A.1 qmake 简介

简而言之，qmake 是用来编译 Qt 应用程序的。

编译工具的使用很大程度上的简化了 Qt 应用程序的编译工作。我们可以使用三种方法 来编译 Qt 应用程序：第一种方法是使用 Qt 提供的 qmake 工具，第二种方法是使用第三方 的编译工具，而第三种方法是使用集成开发环境（ IDE）。

qmake 工具可以使用与平台无关的.pro 文件生成与平台相关的 makefile。该工具包含 了调用 Qt 内置代码生成工具（moc、uic、和 rcc）的必要的逻辑规则。在本书的所有例子 中都使用了 qmake，在多数情况下会使用相对简单的.pro 文件。实际上，qmake 提供了很多 的特性，包括创建可以递归调用其他 makefile 的 makefile，以及根据目前平台切换相关特 性的开关状态等。

# A.2 使用 qmake

## A.2 使用 qmake

qmake 工具是与 Qt 一起提供的。它用来编译 Qt 本身，并且生成 Qt 自带的工具和例 子。贯穿整书，我们一直使用 qmake 工程文件（.pro 文件）编译示例应用程序。本附录将 系统（但非全面）的学习.pro 文件的语法，并且会介绍几个 qmake 的基本概念。要想全面 了解它们，请参阅 qmake 指南的在线帮助文档。

### A.2.1 .pro 文件语法

.pro 文件的目的是列举工程中包含的源文件。由于 qmake 用于编译 Qt 及其相关工具， 所以它很熟悉 Qt，并且能够生成一些触发 moc、uic、和 rcc 的规则。因此，qmake 的语法 很简明，而且很容易学习。

工具文件主要分为三种：app（单独的应用程序）、lib（静态和动态库）和 subfirs（递归编译）。工程文件的类型可以使用 TEMPLATE 变量指定如下：

```cpp
TEMPLATE = lib 
```

subdirs 模板可以用来编译子目录里的目标文件。在这种情况下，除 TEMPLATE = subdirs 外，还需要指定 SUBDIRS 变量。在每个子目录中，qmake 会搜寻以目录名命名 的.pro 文件，并且会编译该工程。如果没有 TEMPLATE 这一项，那么默认工程是 app。对于

app 或者 lib 工程，最常用使用的变量是下面这些：

*   HEADERS 指定工程的 C++头文件（.h）。
*   SOURCES 指定工程的Ｃ++实现文件（.cpp）。
*   FORMS 指定需要 uic 处理的由 Qt 设计师生成的.ui 文件。
*   RESOURCES 指定需要 rcc 处理的.qrc 文件。
*   DEFINES 指定预定义的 C++预处理符号。
*   INCLUDEPATH 指定 C++编译器搜索全局头文件的路径。
*   LIBS 指定工程要链接的库。库既可以通过绝对路径指定，也可以使用源自 Unix 的- L 和-l 标识符来指定（例如，-L/usr/local/lib 和-ldb_cxx）。
*   CONFIG 指定各种用于工程配置和编译的参数。
*   QT 指定所要使用的 Qt 模块（默认是 core gui，对应于 QtCore 和 QtGui 模块）。
*   VERSION 指定目标库的版本号。
*   TARGET 指定可执行文件或库的基本文件名，其中不包含任何的扩展、前缀或版本 号（默认的是当前的目录名）。
*   DESTDIR 指定可执行文件放置的目录（默认值是平台相关的。例如，在 Linux 上，指当前目录；在 Windows 上，则是指 debug 或 release 子目 录）。
*   DLLDESTDIR 指定目标库文件放置的目录（默认路径与 DESTDIR 相同）。
*   CONFIG 变量用来控制编译过程中的各个方面。它支持下面这些参数：
    *   debug 是指具有调试信息的可执行文件或者库，链接 Qt 库的调试版。
    *   release 是指编译不具有调试信息的可执行文件或者库，链接发行版的 Qt 库。如果 同时指定 debug 和 release，则 debug 有效。
    *   warn_off 会关闭大量的警告。默认情况下，警告的状态是打开的。
    *   qt 是指应用程序或者库使用 Qt。这一选项是默认包括的。
    *   dll 是指动态编译库。
    *   staticlib 是指静态编译库。
    *   plugin 是指编译一个插件。插件总是动态库，因此这一参数暗含 dll 参数。
    *   console 是指应用程序需要写控制台（使用 cout、cerr、qWarning()，等等）。
    *   app_bundle 只适用于 Mac OS X 编译，是指可执行文件被放到束中，这是 Mac OS X 的默认情况。
    *   lib_bundle 只适用于 Mac OS X 编译，指库被放到框架中。

要生成工程文件 hello.pro 的 makefile，可以输入：

```cpp
qmake hello.pro 
```

在这之后，可以调用 make 或 nmake 编译工程。通过键入以下命令，也可以使用 qmake 生成一个 Microsoft Visual Studio 工程（.dsp 或.vproj）文件：

```cpp
qmake -tp vd hello.pro 
```

在 Mac OS X 系统上，可以创建一个 Xcode 工程文件：

```cpp
qmake -spec macx-xcode hello.pro 
```

要创建 makefile，可以输入：

```cpp
qmake -spec macx-g++ hello.pro 
```

这里的-spec 命令行参数可以用来指定平台/编译器的组合。通常，qmake 可以正确的检测到所在的平台，但在某些情况下则由必要显式的指定平台的情况。例如，在 Linux 上 以 64 位模式调用 Intel C++编译器（ICC）生成 makefile，应当输入：

```cpp
qmake -spec linux-icc-64 hello.pro 
```

那些可用的规则在 Qt 的 mkspecs 目录中。

尽管 qmake 的主要用途是生成.pro 文件的 makefile，但也可以使用-project 参数在当 前目录下使用 qmake 生成.pro 文件，例如：

```cpp
qmake -project 
```

在这种模式下，qmake 将搜索当前目录下已知扩展名（.h、.cpp、.ui 等等）的文件，生成一个列举这些文件的.pro 文件。

本附录的余下部分将更详细的介绍.pro 文件的语法。一个.pro 文件中的条目的语法通 常具有如下形式：

```cpp
variable = values 
```

values 是字符串的列表。注释以井号（#）开头，在行尾处结束。例如：

```cpp
CONFIG = qt release warn_off # I know what I'm doing 
```

将列表["qt"，"release"，"warn_off"]赋给 CONFIG 变量，它会覆盖以前的各个值。

额外的操作符作为=操作符的补充。+=操作符可以用来扩展变量的值。因此：

```cpp
CONFIG = qt
CONFIG += release
CONFIG += warn_off 
```

这几行会和前面的例子一样，可以有效的把列表 ["qt"，"release"，"warn_off"] 赋值给 CONFIG 变量。-=操作符从当前的变量中移除所有出现的指定的值。因此：

```cpp
CONFIG = qt realease warn_off
CONFIG -= qt 
```

会使 CONFIG 的值变成["release"，"warn_off"]。*=操作在一个变量上添加一个值，但要求被添加的值不在变量的列表上；否则，就什么都不做。例如：

```cpp
SOURCE *= main.cpp 
```

这一行将把 main.cpp 实现文件添加到工程中，只有当还没有被添加的情况下才添加它。最后，=操作符使用指定的值替换符合正则表达式的值，这是一种类似于 sed（UNIX 流 编辑器）的语法。

例如：

```cpp
SOURCE ~= s/\.cpp\b/.cxx/ 
```

使用.cxx 替换 SOURCES 变量中所有.cpp 文件的扩展名。

在值的列表中，qmake 提供了访问其他 qmake 变量、环境变量和配置参数的方法。表附 录 A-1 列举了这些语法。

表附录 A-1 qmake 变量语法

| 存取函数 | 说明 |
| --- | --- |
| $$var/Name 或者$${varName} | .pro 文件中 qmake 变量在那一时刻的值 |
| $$(varName) | 当 qmake 运行时环境变量的值 |
| $(varName) | 当处理 makefile 时环境变量的值 |
| $$[varName] | Qt 的配置参数值 |

### A.2.2 qmake 的存取函数

前面的例子中使用的总是一些标准变量，例如 SOURCES 和 CONFIG，然而，我们也可以 设置任意变量的值，并且可以使用 $$varName 或者$${varName}语法引用它。例如：

```cpp
MY_VRESION = 1.2
SOURCE_BASIC = alphadialog.cpp \
main.cpp \
windowpanel.cpp
SOURCE_EXTRA = bezierextension.cpp \
xplot.cpp
SOURCES = $$SOURCES_BASIC \
$$SOURCES_EXTRA
TARGET = imgpro_$${MY_VERSION} 
```

接下来的例子组合了前面介绍的几种语法，使用内置函数 $$lower()把字符串转换为小写：

```cpp
# List of classes in the project
MY_CLASS = Annotation \
CityBlock \
CityScape \
CityView
# Append .cpp extension to lowercased class names,and add main.cpp
SOURCES = $$lower($$MY_CLASSES)
SOURCES ~= s/([a-z0-9_]+)/\1.cpp/
SOURCES += main.cpp
# Append .h extension to lowercased class names
HEADERS = $$lower($$MY_CLASSES)
HEADERS ~= s/([a-z0-9_]+)/\1.h/ 
```

有时可能需要在.pro 文件中指定包含空格的文件名。在这种情况下，只需简单的把文件名用引号括起来即可。

当在不同的平台上编译工程时，可能有必要基于平台指定不同的文件或者不同的参 数。qmake 的条件语法是：

```cpp
condition
{
    then-case
}
else
{
    else-case
} 
```

condition 部分可以是平台名字（例如，win32、unix、或者 macx），或者更复杂的断言。then-case 和 else-case 部分使用标准语法为变量赋值。例如：

```cpp
win32
{
    SOURCES += serial_win.cpp
}
else
{
    SOURCES += serial_unix.cpp
} 
```

else 分支是可选的。为了方便，当 then-case 部分仅有一条变量赋值，而且在没有 else-case 分支时，qmake 也支持单行形式的语法：

```cpp
condition:then-case 
```

例如：

```cpp
macx:SOURCE += serial_mac.cpp 
```

如果有几个工程文件需要共享相同的项，则可以把相同的项提取到单独的文件中，在各自的.pro 文件中使用 include()语句包含它们：

```cpp
include(../common.pri)
HEADERS += window.h
SOURCES += main.cpp \
window.cpp 
```

通常，打算被别的工程文件所包含的工程文件会带有 .pri（工程包含）的扩展名。

在前面的例子中，我们了解了$$lower()内置函数，它可以返回参数的小写版本。另外 一个有用的函数是$$system()，它允许我们从外部应用程序中产生字符串。例如，如果想要 确认当前使用的 UNIX 版本，可以这样写：

```cpp
OS_VERSION = $$ system(uname -r) 
```

然后，可以在条件中使用结果变量，并与 contains()合用：

```cpp
contains(OS_VERSION,SunOS):SOURCES += mythread_sun.c 
```

本附录只讲了一些表面的东西。qmake 工具提供了许多参数和特性，远多于这里所讲到的这些，包括对预编译头文件的支持、对 Mac OS X 通用二进制库的支持以及对用户定义的 编译器或者其他工具的支持等。对于这方面更多信息的了解，可以参阅 qmake 指南在线帮助文档。