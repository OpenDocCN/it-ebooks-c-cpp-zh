# 标准库中容器方面的改进

新的语言特性和近 10 年来的经验会给标准库中的容器带来啥改进呢？首先，新容器类型：array(大小固定容器)，forward_list

（单向链表），unordered containers（哈希表，无序容器）。其次，新特性：initializer lists（初始化列表），rvalue

references（右值引用），variadic templates（可变参数模板），constexpr（常量表达式）。下边均以 vector 为例加以介绍：

*   初始化列表（Initializer lists）：最显著的改进是容器的构造函数可以接受初始化列表来作为参数：

    ```cpp
    vector<string> vs = { "Hello", ", ", "World!", "\n" };
    for (auto s : vs ) cout << s; 
    ```

*   move 操作：容器新增了 move 版的构造和赋值函数（作为传统 copy 操作的补充）。它最重要的内涵就是允许我们高效的从函数中返回一个容器：

    ```cpp
    vector<int&gt; make_random(int n)
    {
        vector<int&gt; ref(n);
        // 产生 0-255 之间的随机数
            for(auto x& : ref) x = rand_int(0,255);

            return ref;
    }

    vector<int> v = make_random(10000);
    for (auto x : make_random(1000000)) cout << x << '\n'; 
    ```

    上边代码的关键点是 vector 没有被拷贝操作(译注：vector ref 的内存空间不是应该在函数返回时被 stack 自动回收吗？move assignment 通过右值引用精巧的搞定了这个问题)。对比我们现在的两种惯用法：在自由存储区来分配 vector 的空间，我们得负担上内存管理的问题了；通过参数传进已经分配好空间的 vector，我们得要写不太美观的代码了（同时也增加了出错的可能）。

*   改进的 push 操作：作为我最喜爱的容器操作函数，push_back()允许我们优雅的增大容器：

    ```cpp
    vector<pair<string,int>> vp;
    string s;
    int i;
    while(cin&gt;&gt;s&gt;&gt;i) vp.push_back({s,i}); 
    ```

    如上代码通过 r 和 i 构造了一个 pair 对象，然后将它 move 到 vp 中。注意这里是”move”而不是”copy”。这个 push_back 版本接受了一个右值引用参数，因此我们可以从 string 的移动构造函数（move

    constructor）（译注：直接由拷贝构造函数（copy ctor）对应而来）中获益。同时使用了[统一初始化语法]（unified initializer syntax）来避免哆嗦。

*   原地安置操作（Emplace operations）：在大多数情况下，push_back()使用移动构造函数（而不是拷贝构造函数）来保证它更有效率，不过在极端情况下我们可以走的更远。为何一定要进行拷贝/移动操作？为什么不能在 vector 中分配好空间，然后直接在这个空间上构造我们需要的对象呢？做这种事儿的操作被叫做”原地安置”（emplace，含义是：putting in place）。举一个 emplace_back()的例子：

    ```cpp
    vector<pair<string,int>> vp;
    string s;
    int i;
    while(cin&gt;&gt;s&gt;&gt;i) vp.emplace_back(s,i); 
    ```

    emplace_back()接受了可变参数模板变量并通过它来构造所需类型。至于 emplace_back()是否比 push_back()更有效率，取决于它和可变参数模板的具体实现。如果你认为这是一个重要的问题，那就实际测试一下。否则，就从美感上来选择它们吧。到目前为止，我更喜欢 push_back()，不过只是目前哦。

*   Scoped allocators：现在容器中可以持有”拥有状态的空间分配对象(allocationobjects)”了，并通过它来进行”nested/scoped”方式的空间分配（译注：原文：use those to control nested/scoped allocation ）（举例：为容器中的元素分配空间）。

显然，容器不是唯一从新语言特性中获益的标准库部分：

*   编译期计算（Compile-time evaluation）：常量表达式

    为 bitset, duration, char_traits, array, atomic types,

    random numbers,

    complex 等类型引入了编译期计算。对于某些情况，这意味着性能上的改善；而对于其它情况（无法编译期优化的情况下），则意味着可以减少晦涩代码和宏的使用了。

*   元组（Tuples）：如果没有可变参数模板

    ，它就不存在了。

（翻译：interma）