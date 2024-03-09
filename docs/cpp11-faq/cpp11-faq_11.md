# C++0x 努力要达到的目标有哪些？

C++是一种偏向于系统编程的通用编程语言，所以它应该：

*   支持数据抽象
*   支持面向对象的编程
*   支持泛型编程

C++0x 努力的总体目标是为了加强：

*   使 C++成为一种适用于系统编程和创建程序库的更好的语言——也就是直接利用 C++进行编程，而不是为某个特定的领域提供专门的开发语言。（例如，专门为数值计算或 Windows 的应用程序开发提供支持）。
*   使 C++更容易教和学——增加一致性，加强安全性，为初学者提供相关的配套组件（让 C++更容易学习和使用）（初学者总是比专家多的）。（译注：C++0x 现在真的是更好用了）

自然，这是在非常严格的兼容性约束下完成的。虽然我们在 C++0x 中引入了很多新的关键词（例如：static assert，nullptr，还有 constexpr），但是标准委员会也很少会破环标准库中的已经让人非常满意的代码。(?)

你可以通过下面这些参考获得更多详细的信息：

*   B. Stroustrup: [What is C++0x?](http://www2.research.att.com/%7Ebs/what-is-2009.pdf) . CVu. Vol 21, Issues 4 and 5\. 2009.
*   B. Stroustrup: [Evolving a language in and for the real world: C++ 1991-2006](http://www.research.att.com/%7Ebs/hopl-almost-final.pdf) . ACM HOPL-III. June 2007.
*   B. Stroustrup: [A History of C++: 1979-1991](http://www.research.att.com/%7Ebs/hopl2.pdf) . Proc ACM History of Programming Languages conference (HOPL-2). March 1993.
*   B. Stroustrup: [C and C++: Siblings](http://www.research.att.com/%7Ebs/siblings_short.pdf) . The C/C++ Users Journal. July 2002.

（翻译：Chilli）