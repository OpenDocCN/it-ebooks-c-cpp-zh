# [37] 类库

## FAQs in section [37]:

*   [37.1] 什么是“STL”？
*   [37.2] 哪里可以得到“STL”的拷贝？
*   [37.3] 如何才能在 Fred*的 STL 容器比如 std::vector<Fred*>中找到 Fred 对象？
*   [37.4] 哪里可以得到如何使用 STL 的帮助？
*   [37.5] 如何判断你是否有一个动态类型的 C++类库？
*   [37.6] 什么是 NIHCL？哪里可以得到它？
*   [37.7] 哪里的 FTP 下载到《数值分析》附属的代码？
*   [37.8] 为什么可执行文件这么大？
*   [37.9] 哪里可以得到更多的有关 C++类库的信息？

## 37.1 什么是“STL”？

STL 的（“标准模板库”）是一个库，主要包括（ 非常高效）容器类，以及用于容器的迭代器和算法。

技术上来说，“STL”术语不再具有任何意义，因为它所提供的 STL 类以及其他标准类比如 std::ostream 等已被完全合并到标准库，但许多人仍然使用 STL 这个术语，听起来像它是一个独立的东西，所以你不妨习惯于这种说法。

## 37.2 哪里可以得到“STL”的拷贝？

由于一部分 STL 类已经是标准类库的一部分，你的编译器应该提供这些类。如果你的编译器不包括这些标准类，要么得到一个更新版本的编译器或下载下列任何一个 STL 类的拷贝：

*   An STL site: `ftp.cs.rpi.edu/pub/stl`
*   STL HP official site: `butler.hpl.hp.com/stl/`
*   Mirror site in Europe: [`www.maths.warwick.ac.uk/ftp/mirrors/c++/stl/`](http://www.maths.warwick.ac.uk/ftp/mirrors/c++/stl/ "www.maths.warwick.ac.uk/ftp/mirrors/c++/stl/")
*   STL code alternate: `ftp.cs.rpi.edu/stl`
*   The SGI implementation: [`www.sgi.com/tech/stl/`](http://www.sgi.com/tech/stl/ "www.sgi.com/tech/stl/")
*   STLport: [`www.stlport.org`](http://www.stlport.org "www.stlport.org")

用于 GCC - 2.6.3 的 STL hacks 是 GNU libg + + 2.6.2.1 或更高版本包的一部分（也可能会在较早版本）。感谢 Mike Lindner。

另外，一些人使用“STL”来包括标准字符串头文件`<string>`，但是其他人反对这一用法。

## 37.3 如何才能在`Fred*`的 STL 容器比如`std::vector<Fred*>`中找到`Fred`对象？

如`std::find_f()`这样的 STL 函数可以帮助你找到容器内的 `T`号元素。但是，如果容器存储的是指针，比如`std::vector<Fred*>`，这些函数将找到一个匹配`Fred*`的元素，但不能找到和`Fred`匹配的元素。

解决办法是使用一个可选的参数，指定了“匹配”功能。下面的类模板可以间接地让你比较指针所指向的对象。

```cpp
 template<typename T>
 class DereferencedEqual {
 public:
   DereferencedEqual(const T* p) : p_(p) { }
   bool operator() (const T* p2) const { return *p_ == *p2; }
 private:
   const T* p_;
 }; 
```

现在你可以使用此模板来找到适合的 Fred 对象：

```cpp
 void userCode(std::vector<Fred*> v, const Fred& match)
 {
   std::find_if(v.begin(), v.end(), DereferencedEqual<Fred>(&match));
   ...
 } 
```

## 37.4 哪里可以得到如何使用 STL 的帮助？

这里有一些资源（排名不分先后）：

Rogue Wave's STL Guide: [`www.ccd.bnl.gov/bcf/cluster/pgi/pgC++_lib/stdlibug/ug1.htm`](http://www.ccd.bnl.gov/bcf/cluster/pgi/pgC++_lib/stdlibug/ug1.htm "www.ccd.bnl.gov/bcf/cluster/pgi/pgC++_lib/stdlibug/ug1.htm")

The STL FAQ: `butler.hpl.hp.com/stl/stl.faq`

Kenny Zalewski's STL guide: [`www.cs.rpi.edu/projects/STL/htdocs/stl.html`](http://www.cs.rpi.edu/projects/STL/htdocs/stl.html "www.cs.rpi.edu/projects/STL/htdocs/stl.html")

Mumit's STL Newbie's guide: [`www.xraylith.wisc.edu/~khan/software/stl/STL.newbie.html`](http://www.xraylith.wisc.edu/~khan/software/stl/STL.newbie.html "www.xraylith.wisc.edu/~khan/software/stl/STL.newbie.html")

SGI's STL Programmer's guide: [`www.sgi.com/tech/stl/`](http://www.sgi.com/tech/stl/ "www.sgi.com/tech/stl/")

还有一些有益的书籍。

## 37.5 如何判断你是否有一个动态类型的 C++类库？

*   提示#1：所有东西都派生自一个根类，通常是`object` 。
*   提示#2：容器类（ 列表 ， 堆栈 ， 集等）是非模板。
*   提示#3：容器类（ 列表 ， 堆栈 ， 集等）插入/提取的是对象指针。你可以放入`apple`到容器中，但是当你提取的时候，编译器只知道它派生自`object` ，所以你必须使用指针转换将其转换回`apple*`；你更好祈祷这的对象就是一个`apple`对象，你需要自己对自己负责。

你可以使用使用`dynamic_cast`来保证类型安全，但 它能做的也仅仅发生在运行时。这种编码风格是 C++动态类型的精华。你调用一个函数，告诉该函数：“转换`object`对象为`apple`对象，要么或返回`NULL` ，如果它不是`apple`类型”。不到运行时，你不知道会发生什么事。

当你使用模板来实现容器时，C++编译器可以静态验证 90+%的应用程序的类型信息（“90 +％”是不对的，有些人声称应该是 100％，需要持久化(persistence)的人得不到 100%），我想要强调的是：C++泛型特性是从模板获得的，而不是继承。

## 37.6 什么是 NIHCL？哪里可以得到它？

NIHCL 是“国家卫生局类库”的简写，你可以从这里取得： `128.231.128.7/pub/NIHCL/nihcl-3.0.tar.Z`

NIHCL (有人读作"N-I-H-C-L," 其他人读作"nickel") 是 Smalltalk 类库的 C++版本。NIHCL 的动态类型可以应用在一些方面（比如：对象持久化）。但是有些地方，NIHCL 的用法和 C++语言的静态类型有冲突。

## 37.7 哪里的 FTP 下载到《数值分析》附属的代码？

此软件是商业软件，因此通过网络渠道获得是非法的。然而，只有 30 元左右。

## 37.8 为什么可执行文件这么大？

许多人惊讶可执行文件怎么这么大，特别是在源代码很少的情况下。例如，一个简单的“Hello World”程序肯能生成一个大于大多数人预计（40 + K 字节）的可执行文件。

可执行文件的一个原因是部分的 C++运行时库会被静态链接到应用程序。到底有多少被静态链接，取决于是否静态或动态链接标准库的编译器选项，也取决于你对标准库的使用量，以及实现代码会怎么分开库代码。例如，`<iostream>`库非常大，包含有许多类和虚函数。对它的任何调用将会导致引入`<iostream>`的所有代码（但可能有的编译器选项，可以动态链接这些类库，在这种情况下你的程序可能较小）。

另一个可执行文件大的原因可能是如果你打开了调试功能（当然还是通过编译器选项）。如果是知名的编译器，这个选项可能会增加可执行文件的大小到 10 倍。

你需要查看编译器说明书或咨询供应商的技术支持来获得更详细的解答。

## 37.9 哪里可以得到更多的有关 C++类库的信息？

你应该检查三个地方（顺序无关）：

*   C++库常见问题，由 Nikki Lockecpplibs(AT)trmphrst(DOT)demon(DOT)co(DOT)uk "(NOSPAM)cpplibs(AT)trmphrst(DOT)demon(DOT)co(DOT)uk")维护，可以从[框架网页](http://www.trumphurst.com/cpplibs/cpplibs.phtml)和[无框架网页](http://www.trumphurst.com/cpplibs/cpplibs.phtml)得到。
*   你也应该检查[`www.mathtools.net/C_C__/`](http://www.mathtools.net/C_C__/ "www.mathtools.net/C_C__/").他们有成堆的好东西，分门别类为 60（目前）多个大类。
*   你也应该检查[`www.boost.org/`](http://www.boost.org/ "www.boost.org/").他们有一些好东西，其中一些下一次可能得到标准化的提议。

重要：这些清单可能有遗漏。如果你正在寻找一些特定的上面没有的功能，可以试试如[谷歌](http://www.google.com/)。同时，不要忘记提交“[C++类库 FAQ 的提交表格](http://www.trumphurst.com/cppsub.html)”来帮助别人 。.