# 垃圾回收（应用程序二进制接口）

在 C++中，垃圾回收机制（自动回收没有被引用的内存区域）是可选的；也就是说在编译器中并不是一定要实现垃圾回收器。尽管如此，C++0x 还是定义了垃圾回收器的功能。与此同时，C++0x 还提供了应用程序二进制接口（ABI: Application Binary Interface）来辅助控制垃圾回收器的行为。

我们用“safely derived pointer”（3.7.3.3）（译注：我搜索后发现，是在 3.7.3.3 而不是 3.7.4.3 讲了 safely derived pointer。这里可能是原文作者的笔误）来表示指针和生存时间的规则；粗略地说就是 “指向由 new 分配的对象或者指向相应的子对象的指针”。 下面是一些关于“not safely derived pointers”又名“disguised pointers”，或者说是为便于正常人理解和认可你的程序从而你应该注意的一些问题。

*   将指针暂时指到别处：

    ```cpp
    int* p = new int;
    p+=10;
    //…垃圾回收器可能在这里运行…
    p-=10;
    //在此，我们是否可以肯定开始为 p 分配的那块 int 内存还存在？
    *p = 10; 
    ```

*   将指针隐藏到一个 int 变量中：

    ```cpp
    int* p = new int;
    int x = reinterpret_cast(p); // non-portable (不可移植)
    p=0;
    //…垃圾回收器可能在这里运行…
    p = reinterpret_cast(x);
    //在此，我们是否可以肯定开始为 p 分配的那块 int 内存还存在？
    *p = 10; 
    ```

*   还有更多甚至更危险的陷阱。比如 I/O，以及将存储不同的数据位打散，..

虽然我们也有一些合理的理由来伪装指针（例如：xor 有可能在 exceptionally memory-constrained applications 中导致错误），但是理由并不像一些程序员所认为的那么多。

程序员可以在代码中声明哪些地方不会有指针被发现（比如在一个图像中），也可以声明哪些内存区域不能被回收，即使垃圾回收器发现没有任何指针指向这块内存区域。以下是相关的例子：

void declare_reachable(void* p); //以 p 起始的内存区域

//（用能够记住所分配内存区域大小

//的内存分配操作符分配）不能被回收

```cpp
template<class T> T* undeclared_reachable(T* p);

 void declare_no_pointers(char* p, size_t n);        //p[0..n] 中没有指针
 void undeclared_no_poitners(char* p, size_t n); 
```

程序员要能够查询到关于指针安全和回收的规则是否是强制性的：

```cpp
enum class pointer_saftety {relaxed, preferred, strict};

pointer_safety get_pointer_safety(); 
```

3.7.4.3[4]：满足以下三个条件的行为没有被定义：如果一个非”safely-derived pointer”值被释放，并且它所引用的对象是动态存储期（由 new 操作符动态创建并由 delete 销毁的对象拥有动态存储期），与此同时这个指针之前也没被声明为可到达的（20.7.13.7）。

*   relaxed: safely-derived pointer 和 not safely-derived pointer 被认为是等同的。这和 C 以及 C++98 中是一样的。但这并不是我的初衷。我的想法是用户如果没有使用有效的指针指向一个对象则应启用垃圾回收。
*   Preferred：和 relaxed 类型一样。只不过垃圾回收器可能被用作内存泄露检测以及（或者）检测对象是否被一个错误的指针解引用。
*   strict: safely-derived 和 not safely-derive 这两种指针可能不再被等同。也就是说，垃圾回收器可能被启用而且将会忽略那些 not safely derived pointer。

并不存在任何标准化的方法以供你选择任何一种。这是一个实现的质量（quality of implementation）和编程环境（programming environment）的问题。

另外，可以参考以下文献：

*   the C++ draft 3.7.4.3
*   the C++ draft 20.7.13.7
*   Hans Boehm’s

    [GC page](http://www.hpl.hp.com/personal/Hans_Boehm/gc/)

*   Hans Boehm’s

    [Discussion of Conservative GC](http://www.hpl.hp.com/personal/Hans_Boehm/gc/issues.html)

*   [final proposal](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2527.pdf)

*   Michael Spertus and Hans J. Boehm:

    [The Status of Garbage Collection in C++0X](http://portal.acm.org/citation.cfm?doid=1542431.1542437)

    .

    ACM ISMM’09.

（翻译：Yibo Zhu）