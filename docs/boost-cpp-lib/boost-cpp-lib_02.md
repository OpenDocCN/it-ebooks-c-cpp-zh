# 第一章 简介

### 目录

*   1.1 C++ 与 Boost
*   1.2 开发过程
*   1.3 安装
*   1.4 概述

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 1.1\. C++ 与 Boost

[Boost C++ 库](http://www.boost.org/doc/libs/) 是一组基于 C++标准的现代库。 其源码按 [Boost Software License](http://www.boost.org/LICENSE_1_0.txt) 来发布，允许任何人自由地使用、修改和分发。 这些库是平台独立的，且支持大多数知名和不那么知名的编译器。

Boost 社区负责开发和发布 Boost C++ 库。 社区由一个很大的 C++开发人员群组组成，这些开发人员来自于全球，他们通过网站 [www.boost.org](http://www.boost.org/) 以及几个邮件列表相互协调。 社区的使命是开发和收集高质量的库，作为 C++标准的补充。 那些被证实有价值且对于 C++应用开发非常重要的库，将会有很大机会在某天被纳入 C++标准中。

Boost 社区在 1998 年左右出现，当时刚刚发布了 C++标准的第一个版本。 从那时起，社区就不断地扩大，现在已成为 C++标准化工作中的一个重要角色。 虽然 Boost 社区与标准化委员会之间没有直接的关系，但有部分开发者同时活跃于两方。 下一个版本的 C++标准很大可能在 2011 年通过，其中将扩展一批库，这些库均起源于 Boost 社区。

要增强 C++项目的生产力，除了 C++标准以外，Boost C++ 库是一个不错的选择。 由于当前版本的 C++标准在 2003 年修订之后，C++又有了新的发展，所以 Boost C++ 库提供了许多新的特性。 由于有了 Boost C++ 库，我们无需等待下一个版本的 C++标准，就可以立即享用 C++演化中取得的最新进展。

Boost C++ 库具有良好的声誉，这基于它们的使用已被证实是非常有价值的。 在面试中询问关于 Boost C++ 库的知识是不常见的，因为知道这些库的开发人员通常也清楚 C++的最新创新，并且能够编写和理解现代的 C++代码。

## 1.2\. 开发过程

正是因为大量的独立开发者和组织的支持和参与，才使用 Boost C++ 库的开发成为可能。 由于 Boost 只接受满足以下条件的库：解决了真实存在的问题、表现出令人信服的设计、使用现代 C++来开发且以可理解的方式提供文档，所以每一个 Boost C++ 库的背后都有大量的工作。

C++ 开发者都可以加入到 Boost 社区中，并提出自己的新库。 但是，要将一个想法变成一个 Boost C++ 库，需要投入大量的时间和努力。 其中最重要的是在 Boost 邮件列表中与其他开发者及潜在用户讨论需求和可能的解决方案。

除了这些好象不知从何处冒出来的新库以外，也可以提名一些已有的 C++ 库进入 Boost。 不过，由于对这些库的要求是与专门为 Boost 开发的库一样的，所以可能需要对它们进行大量的修改。

一个库是否被接纳入 Boost，取决于评审过程的结果。 库的开发者可以申请评审，这通常需要 10 天的时间。 在这段时间内，其他开发者被邀请对这个库进行评分。 基于正面和负面评价的数量，评审经理将决定该库是否被接纳进入 Boost。 由于有些开发者是在评审阶段才首次公开库的代码，所以在评审期间被要求对库进行修改并不罕见。

如果一个库是因为技术原因被拒绝，那么它还有可能在修改之后对更新后的版本申请新的评审。 但是，如果一个库是因为不能解决实际问题或未能提供令人信服的解决方案而被拒绝，那么再一次评审也很可能会被拒绝。

因为可能随时接纳新的库，所以 Boost C++ 库会每三个月发布一次新版本。本书所涉及的库均基于 2010 年 2 月发布的 1.42.0 版本。

请注意，另外还有一些库已被接纳，但尚未成为 Boost C++ 库发布版的一部分。在被包含进发布版之前，它们必须手工安装。

## 1.3\. 安装

Boost C++ 库均带有源代码。其中大多数库只包含头文件，可以直接使用，但也有一些库需要编译。 为了尽可能容易安装，可以使用 Boost Jam 进行自动安装。 无需逐个库进行检查和编译，Boost Jam 自动安装整个库集。 它支持许多操作系统和编译器，并且知道如何基于适当的配置文件来编译单个库。

为了在 Boost Jam 的帮助下自动安装，要使用一个名为 **bjam** 的应用程序，它也带有源代码。 对于某些操作系统，包括 Windows 和 Linux，也有预编译好的 **bjam** 二进制文件。

为了编译 **bjam** 本身，要执行一个名为 **build** 的简单脚本，它也为不同的操作系统提供了源代码。 对于 Windows，它是批处理文件 `build.bat`。 对于 Linux，文件名为 `build.sh`。

如果执行 **build** 时不带任何命令行选项，则该脚本尝试找到一个合适的编译器来生成 **bjam**。 通过使用命令行开关，称为 toolset，可以选择特定的编译器。 对于 Windows，**build** 支持 toolsets `vc7`, `vc8` 和 `vc9`，可以选择不同版本的 Microsoft C++ 编译器。 要从 Visual Studio 2008 的 C++编译器编译 **bjam**，需要指定命令 **build vc9**。对于 Linux，支持 toolsets `gcc` 和 `intel-linux`，分别选定 GCC 和 Intel 的 C++编译器。

应用程序 **bjam** 必须复制到本地的 Boost 目录 - 不论它是编译出来的还是下载的预编译二进制文件。 然后就可以不带任何命令行选项地执行 **bjam**，编译并安装 Boost C++ 库。 由于缺省选项 - 在这种情况下所使用的 - 并不一定是最好的选择，所以以下列出最重要的几个选项供参考：

*   声明 `stage` 或 `install` 可以指定 Boost C++ 库是安装在一个名为 `stage` 的子目录下，还是在系统范围内安装。 "系统范围"的意义取决于操作系统。 在 Windows 中，目标目录是 `C:\Boost`；而在 Linux 中则是 `/usr/local`。 目标目录也可以用 `--prefix` 选项明确指出。

*   如果不带任何命令行选项执行 **bjam**，则它会自己搜索一个合适的 C++编译器。 可以通过 `--toolset` 选项来指定一个特定的编译器。 要在 Windows 中指定 Visual Studio 2008 的 Microsoft C++ 编译器，**bjam** 执行时要带上 `--toolset=msvc-9.0` 选项。 要在 Linux 中指定 GCC 编译器，则要给出 `--toolset=gcc` 选项。

*   命令行选项 `--build-type` 决定了创建何种方式的库。 缺省情况下，该选项设为 `minimal`，即只创建发布版。 对于那些想用 Visual Studio 或 GCC 构建他们项目的调试版的开发者来说，可能是一个问题。 因为这些编译器会自动尝试链接调试版的 Boost C++ 库，这样就会给出一个错误信息。 在这种情况下，应将 `--build-type` 选项设为 `complete`，以同时生成 Boost C++ 库的调试版和发布版，当然，所需时间也会更长一些。

要用 Visual Studio 2008 的 C++编译器同时生成 Boost C++ 库的调试版和发布版，并将它们安装在目录 `D:\Boost` 中，应执行的命令是 **bjam --toolset=msvc-9.0 --build-type=complete --prefix=D:\Boost install**. 要在 Linux 中使用缺省目录创建它们，则要执行的命令是 **bjam --toolset=gcc --build-type=complete install**.

其它多个命令行选项可用于指定如何编译 Boost C++ 库的一些细节设定。 我通常在 Windows 下使用以下命令：**bjam --toolset=msvc-9.0 debug release link=static runtime-link=shared install**. `debug` 和 `release` 使得调试版和发布版均被生成。 `link=static` 则只创建静态库。 `runtime-link=shared` 则是指定 C++ 运行时库是动态链接的，这是在 Visual Studio 2008 中对 C++项目的缺省设置。

## 1.4\. 概述

Boost C++ 库的 1.42.0 版本包含了超过 90 个库，本书只详细讨论了以下各库：

表 1.1\. 讨论到的库

| Boost C++ 库 | C++ 标准 | 简要说明 |
| --- | --- | --- |
| Boost.Any |  | Boost.Any 提供了一个名为 `boost::any` 的数据类型，可以存放任意的类型。 例如，一个类型为 `boost::any` 的变量可以先存放一个 `int` 类型的值，然后替换为一个 `std::string` 类型的字符串。 |
| Boost.Array | TR1 | Boost.Array 可以把 C++ 数组视同 C++ 标准的容器。 |
| Boost.Asio | TR2 | Boost.Asio 可用于开发异步处理数据的应用，如网络应用。 |
| Boost.Bimap |  | Boost.Bimap 提供了一个名为 `boost::bimap` 的类，它类似于 `std::map`. 主要的差别在于 `boost::bimap` 可以同时从键和值进行搜索。 |
| Boost.Bind | TR1 | Boost.Bind 是一种适配器，可以将函数作为模板参数，即使该函数的签名与模板参数不兼容。 |
| Boost.Conversion |  | Boost.Conversion 提供了三个转型操作符，分别执行向下转型、交叉转型，以及不同数字类型间的值转换。 |
| Boost.DateTime |  | Boost.DateTime 可用于以灵活的格式处理、读入和写出日期及时间值。 |
| Boost.Exception |  | Boost.Exception 可以在抛出的异常中加入额外的数据，以便在 `catch` 处理中提供更多的信息。 这有助于更容易地调试，以及对异常情况更好地作出反应。 |
| Boost.Filesystem | TR2 | Boost.Filesystem 提供了一个类来处理路径信息，还包含了几个访问文件和目录的函数。 |
| Boost.Format |  | Boost.Format 以一个类型安全且可扩展的 `boost::format` 类替代了 `std::printf()` 函数。 |
| Boost.Function | TR1 | Boost.Function 简化了函数指针的定义。 |
| Boost.Interprocess |  | Boost.Interprocess 允许多个应用通过共享内存以快速、高效的方式进行通信。 |
| Boost.Lambda |  | Boost.Lambda 可以定义匿名的函数。 代码被内联地声明和执行，避免了单独的函数调用。 |
| Boost.Multiindex |  | Boost.Multiindex 定义了一些新的容器，它们可以同时支持多个接口，如 `std::vector` 和 `std::map` 的接口。 |
| Boost.NumericConversion |  | Boost.NumericConversion 提供了一个转型操作符，可以安全地在不同的数字类型间进行值转换，不会生成上溢出或下溢出的条件。 |
| Boost.PointerContainer |  | Boost.PointerContainer 提供了专门为动态分配对象进行优化的容器。 |
| Boost.Ref | TR1 | Boost.Ref 的适配器可以将不可复制对象的引用传给需要复制的函数。 |
| Boost.Regex | TR1 | Boost.Regex 提供了通过正则表达式进行文本搜索的函数。 |
| Boost.Serialization |  | 通过 Boost.Serialization，对象可以被序列化，如保存在文件中，并在以后重新导入。 |
| Boost.Signals |  | Boost.Signal 是一个事件处理的框架，基于所谓的 signal/slot 概念。 函数与信号相关联并在信号被触发时自动被调用。 |
| Boost.SmartPoiners | TR1 | Boost.SmartPoiners 提供了多个智能指针，简化了动态分配对象的管理。 |
| Boost.Spirit |  | Boost.Spirit 可以用类似于 EBNF (扩展巴科斯范式)的语法生成词法分析器。 |
| Boost.StringAlgorithms |  | Boost.StringAlgorithms 提供了多个独立的函数，以方便处理字符串。 |
| Boost.System | TR2 | Boost.System 提供了一个处理系统相关或应用相关错误代码的框架。 |
| Boost.Thread | C++0x | Boost.Thread 可用于开发多线程应用。 |
| Boost.Tokenizer |  | Boost.Tokenizer 可以对一个字符串的各个组件进行迭代。 |
| Boost.Tuple | TR1 | Boost.Tuple 提供了泛化版的 `std::pair`，可以将任意数量的数据组在一起。 |
| Boost.Unordered | TR1 | Boost.Unordered 扩展了 C++ 标准的容器，增加了`boost::unordered_set` 和 `boost::unordered_map`. |
| Boost.Variant |  | Boost.Variant 可以定义多个数据类型，类似于 `union`, 将多个数据类型组在一起。 Boost.Variant 比 `union` 优胜的地方在于它可以使用类。 |

Technical Report 1 是在 2003 年发布的，有关 C++0x 标准和 Technical Report 2 的一些细节才能反映当前的状态。由于无论是下一个版本的 C++ 标准，还是 Technical Report 2 都尚未被批准，所以在往后的时间里，它们仍然可能会有改变。