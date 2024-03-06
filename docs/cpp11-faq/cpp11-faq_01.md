# C++11 FAQ 中文版 - C++11 FAQ

更新至英文版 January 3, 2012。

译者前言：

经过 C++标准委员会的不懈努力，最新的 ISO C++标准 C++11，也即是原来的 C++0x，已经正式发布了。让我们欢迎 C++11！

今天获得[Stroustrup](http://www2.research.att.com/%7Ebs/) 先生的许可，开始翻译由他撰写和维护的[C++11 FAQ](http://www2.research.att.com/%7Ebs/C++0xFAQ.html) 。我

觉得这是一件伟大而光荣的事情，但是我又觉得压力很大，因为我的英语水平很差劲，同时自己的 C++水平也很有限，很害怕在翻译过程中出现什么错误，贻笑大方不要紧，而误人子弟就罪过大了。所以，我这里的翻译只

能算是抛砖引玉，如果你的英文很好，你可以直接阅读[他的原文](http://www2.research.att.com/%7Ebs/C++0xFAQ.html) 。或者，你也可以参照两者进行阅读，我想一定会有更多的收获。

当然，我也非常欢迎大家指出翻译中的错误，或者是加入进来和我一起翻译这份文档，共同为 C++11 在中国的推广做一点事情。你可以通过 chenlq at live.com 联系到我。

对自己的翻译做一点说明：

*   在翻译的过程中，尽量遵照原文含义，可能有时候也会自己根据自己的理解加一点批注，希望可以帮助大家理解。
*   另外，虽然 C++11 刚刚公布，但是现在已经有很多编译器支持 C++11 中一些相对比较独立的特性，比如 gcc 以及它在 Windows 下的 MinGW，Visual C++ 2010 也部分支持，大家可以使用这三款编译器尝试这个文档中的部分例子。
*   在下面的目录中，已经翻译的问题链接到相应的中文文档，未翻译的问题则链接到英文原文。

感谢所有参与翻译的志愿者：interma，Chilli，张潇，dabaidu，Yibo Zhu，lianggang jiang，nivo，陈良乔

友情提示：因为网站编辑器的原因，部分示例代码中模板类的模板参数会发生丢失，请读者阅读时注意参考原文的代码，由此给大家造成的不便，深表歉意。

在[这里](http://www.royaloo.com/bjarne/interviews/BS_Codeguru_2011.pdf) 有一份 Stroustrup 先生关于 C++11 的访谈，可以帮助你从更高地角度把握整个 C++11 新标准，你应该[阅读](http://www.royaloo.com/bjarne/interviews/BS_Codeguru_2011.pdf) 一下。

最后，祝大家阅读愉快:)

—————————————————————————

**目录**

*   C++11 FAQ 中文版 - C++11 FAQ
*   Stroustrup 先生关于中文版的授权许可邮件
*   Stroustrup 先生关于 C++11 FAQ 的一些说明
*   关于 C++11 的一般性的问题
    *   您是如何看待 C++11 的?
    *   什么时候 C++0x 会成为一部正式的标准呢？
    *   编译器何时将会实现 C++11 标准呢？
    *   我们何时可以用到新的标准库文件？
    *   C++0x 将提供何种新的语言特性呢？
    *   C++11 会提供哪些新的标准库文件呢？
    *   C++0x 努力要达到的目标有哪些？
    *   指导标准委员会的具体设计目标是什么？
    *   在哪里可以找到标准委员会的报告？
    *   从哪里可以获得有关 C++11 的学术性和技术性的参考资料？
    *   还有哪些地方我可以读到关于 C++0x 的资料？
    *   有关于 C++11 的视频吗？
    *   C++0x 难学吗?
    *   标准委员会是如何运行的？
    *   谁在标准委员会里？
    *   实现者应以什么顺序提供 C++11 特性？
    *   将会是 C++1x 吗？
    *   标准中的"concepts"怎么了?
    *   有你不喜欢的 C++特性吗？
*   关于独立的语言特性的问题
    *   __cplusplus 宏
    *   alignment(对齐方式)
    *   属性（Attributes）
    *   atomic_operations
    *   auto – 从初始化中推断数据类型
    *   C99 功能特性
    *   枚举类——具有类域和强类型的枚举
    *   carries_dependency
    *   复制和重新抛出异常
    *   常量表达式（constexpr）
    *   decltype – 推断表达式的数据类型
    *   控制默认函数——默认或者禁用
    *   控制默认函数——移动(move)或者复制(copy)
    *   委托构造函数（Delegating constructors）
    *   并发性动态初始化和析构
    *   noexcept – 阻止异常的传播与扩散
    *   显式转换操作符
    *   扩展整型
    *   外部模板声明
    *   序列 for 循环语句
    *   返回值类型后置语法
    *   类成员的内部初始化
    *   继承的构造函数
    *   初始化列表
    *   内联命名空间
    *   Lambda 表达式
    *   用作模板参数的局部类型
    *   long long（长长整数类型）
    *   内存模型
    *   预防窄转换
    *   nullptr——空指针标识
    *   对重载(override)的控制: override
    *   对重载(override)的控制：final
    *   POD
    *   原生字符串标识
    *   右角括号
    *   右值引用
    *   Simple SFINAE rule
    *   静态（编译期）断言 — static_assert
    *   模板别名（正式的名称为"template typedef"）
    *   线程本地化存储 (thread_local)
    *   unicode 字符
    *   统一初始化的语法和语义
    *   （广义的）联合体
    *   用户定义数据标识（User-defined literals）
    *   可变参数模板（Variadic Templates）
*   关于标准库的问题
    *   abandoning_a_process
    *   算法方面的改进
    *   array
    *   async()
    *   atomic_operations
    *   条件变量（Condition variables）
    *   标准库中容器方面的改进
    *   std::function 和 std::bind
    *   std::forward_list
    *   std::future 和 std::promise
    *   垃圾回收（应用程序二进制接口）
    *   无序容器（unordered containers）
    *   锁（locks）
    *   metaprogramming（元编程）and type traits
    *   互斥
    *   随机数的产生
    *   正则表达式（regular expressions）
    *   具有作用域的内存分配器
    *   共享资源的智能指针——shared_ptr
    *   smart pointers
    *   线程（thread）
    *   时间工具程序
    *   标准库中的元组（std::tuple）
    *   unique_ptr
    *   weak_ptr
    *   system error