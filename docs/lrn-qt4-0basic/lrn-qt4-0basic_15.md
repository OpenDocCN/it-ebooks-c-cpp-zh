# 附录 B make 命令

# B.1 命令解释

## B.1 命令解释

make 命令本身可带有四种参数：标志、宏定义、描述文件名和目标文件名。其标准 形式为：

```cpp
make [flags] [macro definitions] [targets] 
```

Unix 系统下标志位 flags 选项及其含义为：

*   -f file 指定 file 文件为描述文件，如果 file 参数为"-"符，那么描述文件指向标 准输入。如果没有"-f"参数，则系统将默认当前目录下名为 makefile 或者名为 Makefile 的文件为描述文件。在 Linux 中， GNU make 工具在当前工作目录中按照 GNUmakefile、 makefile、Makefile 的顺序搜索 makefile 文件。
*   -i 忽略命令执行返回的出错信息。
*   -s 沉默模式，在执行之前不输出相应的命令行信息。
*   -r 禁止使用 build-in 规则。
*   -n 非执行模式，输出所有执行命令，但并不执行。
*   -t 更新目标文件。
*   -q make 操作将根据目标文件是否已经更新返回"0"或非"0"的状态信息。
*   -p 输出所有宏定义和目标文件描述。
*   -d Debug 模式，输出有关文件和检测时间的详细信息。

Linux 下 make 标志位的常用选项与 Unix 系统中稍有不同，下面我们只列出了不同部 分：

*   -c dir 在读取 makefile 之前改变到指定的目录 dir。
*   -I dir 当包含其他 makefile 文件时，利用该选项指定搜索目录。
*   -h help 文挡，显示所有的 make 选项。
*   -w 在处理 makefile 之前和之后，都显示工作目录。

# B.2 使用 make 自动构建

## B.2 使用 make 自动构建

make 实用程序是一个工具，用于控制构建以及重构软件的过程。make 将构建什么软 件、如何构建以及何时构建这些过程自动化了，使程序员能够专注于编写代码。因为它包 含的逻辑可调用适于 GCC 编译器的选项和参数，所以节省了很多输入操作。另外，它还可 以帮助您在构建应用程序时，不会在输入所有复杂命令时出现错误；相反，只需输入一个 或两个 make 命令即可。通过本节的学习可以熟悉 makefile 的外观和布局。

除去最简单的软件项目外，对于其他软件项目而言 make 是根本。首先，由多个源文 件组成的项目要求长且复杂的编译器调用。make 简化了这项工作，方法是在 makefile 中存储这些难记的命令，makefile 是一个文本文件，包含构建该软件项目所需的所有命 令。

make 对想要构建程序的开发者和用户而言都很方便。当开发者修改某个程序后，无 论是增加新功能还是合并 bug 修复，make 允许使用单条短的命令即可重构该程序。make 对用户也是很方便的，因为他们不必阅读大量详细解释如何构建程序的文档。相反，他们 只需输入 make，然后是 make test，最后是 make install。多数用户都喜欢这种简单构 建指令的便利。

最后，make 加快了编辑－编译－调试过程。它最大限度的降低了重构时间，因为它 的智能可以确定哪个文件被修改了，并且只重新编译被修改过的文件。