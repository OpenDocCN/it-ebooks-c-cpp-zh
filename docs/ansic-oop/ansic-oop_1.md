# 序言

> 来源：[`blog.csdn.net/besidemyself/article/details/6376405`](http://blog.csdn.net/besidemyself/article/details/6376405)
> 
> 译者：[besidemyself](http://my.csdn.net/besidemyself)

没有能解决所有问题的编程技术。

没有只产生正确结果的编程语言。

没有从头开始每个项目的程序员。

面向对象程序设计几乎是当今包治百病的——虽然它已经发展了超过 10 年之久。作为一种核心语言，一些技术专家对它的研究已经付出很多，从而形成了很好的编程规则，这些规则我们一直引以为鉴了长达 20 年之久。C++（Eiffel，Oberon-2，Smalltalk… 由你选择）是一种新的编程语言，因为它是面向对象的——尽管你不需要使用它或许你不想用（或不知道怎么用），但是你却能够使用平凡的 ANSI-C （标准化）像使用这些面向对象语言一样的方便。在项目当中，仅仅面向对象语言允许代码重用——虽然一些子程序的思想就像计算机的发展历史一样久远，好的程序员却总是会利用工具箱或库来使用这些子程序。

这本书的本意并不是赞扬面向对象程序或批判老的编程规则。我们只是简单的使用 ANSI-C 来发掘面向对象语言是怎么实现的，技巧方法是什么，为什么它能够帮助我们解决比较大的问题，和我们怎么能利用普遍性方法和编程来更早的捕获错误。沿着这么一个宗旨，我们会邂逅一些行话—类，继承，实例，链接，方法，对象，多态等等—不过我们剥去其神奇的外衣，看如何转化为一种我们一直都知道和做过的事情。

我很乐意的发掘 ANSI-C（标准化）是一种全面的面向对象语言。为了能够分享这种乐趣，在你开始之前，你需要适当的对 ANSI-C 有一定的流利程度—熟悉结构体，指针，原型，并且掌握函数指针是必须的。贯穿全文，你会遇见所有的新说法 — 依照 Orwell 和 Webster 的话：“设计是为了缩小思想的广度”—并且我会尽力证明它是怎样的结合所有的这些你一直想连续使用的好的方法，结果，你很可能成为一个精通 ANSI-C 的程序员。

前六章建立 ANSI-C 面向对象程序设计的基础。我们将以一个隐藏抽象数据结构的很精确的信息开始，然后通过结构体的扩展，基于动态链接和代码继承的方式来增加功能属性。最终把他们放到一起构成一个可继承的类使得代码更加容易维护。

编程需要规则。好的程序遵循很多规则，大量的规则，标准，自我防御的方式往往能使事情做对做好。程序员要学会使用工具。优秀的程序员编写工具去处理日常一些编程例程。ANSI-C 面向对象程序要求有一定数量固定的代码 — 即名称改变，数据结构没有变。因此，在第七章我们建立了一个小的预处理器创建一个必须的样板，它更像另外一种新的面向对象语言（也许是 yanoodl）但是它并不被如此看待，OOC（面向对象程序设计 ANSI-C）拓新而出，更让我们专注于在解决问题上使用创造性方面的一种新技巧。OOC 有很强的可塑性。我们设计出它，理解它并且可改变它，能够依照我们的意愿去写 ANSI-C 代码。

接下来的章节中将精炼我们的技术。在第八章节，我们增加了动态类型检查预先的捕获异常。第九章中我们编排了自动初始化机制防止其它缺陷类的产生。第十章，我们介绍了多态以及怎样相互协作达到简化的目的。例如：产生标准主程序的日常事务。更多的章节将与使用类的方法，存储，和加载结构体数据相关联，这些结构体数据遵循一致性策略。并且通过嵌套异常处理器系统来做到一致性错误恢复自愈。

最终，在最后一章节中，我们避开 ANSI-C 的局限性并且实现了一个鼠标操作的计算器，首先对于 curses 终端，接下来应用于 X Window 系统（如果你想了解什么事 curses 和 x window 查阅相关资料）。 这个例子清晰的证明了我们使用类和对象所做的设计和实例的优美性，虽然我们必须处理外在的库和类层次特性。

在每个章节的前面都有一个概要说明，在这个概要中我会给出一个纲要，来介绍章节中的主要内容和接下来该做什么，这对于略读此章的读者很有帮助。大部分章节建议做一下练习；但是这并不意味着很正规，因为我们是从零开始建立这样的技术。我已经避免制造和使用庞大的类库，即使一些例子这样做似乎很有利。如果你想更好的理解面向对象程序设计，掌握这项技术和考虑代码设计显得尤为重要；依靠其他人的类库来做开发是稍后小菜一碟的事。

这本书的一个重要的组成部分是附上了代码软盘---- 使用 DOS 文件系统，包含了单独的 SHELL 命了来创建所有章节中的代码。软盘上有一个 ReadMe 文件----再你生成代码之前先来阅读一下它。编程，就像使用如 diff 的程序，跟踪基类的演变对你会很有好处的。OOC 在接下来的章节中会有很好的体现。

这些技术的描述出自我对 C++的觉悟，当我需要使用面向对象技术去实现交互式编程语言时。并且意识到在 C++中不能铸造一个便携式的实例。我转向我所了解的，即 ANSI-C，我能够完全做我必须做的。我已经在教学过程中把这项技术分享给很多人，包括我的工作场所。而且其他人已经使用这些方法很好了完成了他们的工作。这本书就驻留于此，因为对我的脚注却显得尤为暗淡。Brian Kernighan ，我的出版社，Hans-Joachim Niclas 和 John 等等，并没有鼓励我出版这些笔记（有机会，在适当的时候会从新组织一次），我感谢他们以及帮助我去继续这本书的人，最后，感谢我的家人——并且，不，面向对象机制不会取代“切片面包”。

Hollage, October 1993

Axel-Tobias Schreiner