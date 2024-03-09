# 序

C++社区正在发生着好事。C++依旧是最广泛使用的编程语言，它正在变 得更强大和更易用。你不信？听我细说。

标准 C++的当前版本发布于 1998，它为传统的面向过程编程、面向对象 编程和泛型编程提供了坚实的 支持。正如旧 C++ (1998 之前的) 独力承担了把面向对象普及到日常的软件开发中一样，C++98 在为泛型编程做着同样的事情。九十年代中期标准模板库(STL)与标准 C++的集成已经引起 了另一次编程范式的转变，就象八十年代的时候 Bjarne Stroustrup 把类引入到 C 一样。现在大多数的 C++开发者都熟悉 STL 的概念，这再次提升了整体的水平。

C++能力的应用仍旧被不断发现。今天许多的 C++库，包括特殊的数学 库，都大量利用了模板元编译的技术，它是设计 C++模板的时候没有预测到的幸运结果。随着 C++社区里的高级工具和技术不断涌现，开发复杂应用软件正变得 更简单、更令人愉快。

很难描述 Boost 对于 C++世界的重要性。自从 C++98 发布后，除了 ISO 的标准 C++委员会， 没有一个团体对于 C++的发展方向有比 Boost 更大的影响(许多 Boost 的成员本身就是 WG21 的重要成员，包括它的创始人，我的朋友 Beman Dawes)。成千上万个杰出的 Boost 志愿者无私地，以对等审查方式开发了许多 C++98 没有提供的很有用的库。这些库中的十个已被接受将加入到即将 到来的 C++0x 的库中，更多的库正被考虑接受。Where a library approach has been shown to be wanting, the wisdom gained from the cross-pollination of Boost and WG21 has suggested a few modest language enhancements, which are now being entertained.

你不太可能没有听说过 Boost，我来问一下你… 你需要在文本和数字之间进行转换，或 在任意的可流处理的类之间进行转换？没有问题，用 Boost.lexical_cast。噢，你有更多复杂的文本处理要求？那么你可以用 Boost.Tokenizer 或 Boost.Regex, 或 Boost.Spirit, 如果你需要完整的语法分析。Boost.Bind 的函数反射和组合能力会让你吃惊。要用函数对象来编程，有 Boost.Lambda。静态断言？用 MPL。如果你是用数学库，记住：你有 Boost.Math, Graph, Quaternion, Octonion, MultiArray, Random, 和 Rational。如果你刚好喜欢 Python，有了 Boost.Python 的帮助，你可以和 C++一起用它了。并且你可以从以上所有东西中挑选出你要 的。

Björn Karlsson 是一名 Boost 狂热者以及 C++社区的热心的支持者。你曾经在 C/C++ Users Journal 上出版了很多有用的好文章，最近还为 C++ Source, 一个新的 C++社区的在线杂志(见 [www.artima.com/cppsource](http://www.artima.com/cppsource)) 写文章。他在本书中讲述了关键的 Boost 组件并给出示例，展示了如何使用它们扩充 C++标准库。本书不仅是 Boost 的深入教程，也是标准 C++的未来 版本的预示。希望你喜欢！

Chuck Allison, Editor, The C++ Source