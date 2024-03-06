# 算法方面的改进

标准库的算法部分进行了如下改进：新增了一些算法函数；通过新语言特性改善了一些算法实现并且更易于使用。下面分别来看一些例子：

*   新算法:

    ```cpp
    bool all_of(Iter first, Iter last, Pred pred);
    bool any_of(Iter first, Iter last, Pred pred);
    bool none_of(Iter first, Iter last, Pred pred);

    Iter find_if_not(Iter first, Iter last, Pred pred);

    OutIter copy_if(InIter first, InIter last,
            OutIter result, Pred pred);
    OutIter copy_n(InIter first, InIter::difference_type n,
            OutIter result);

    OutIter move(InIter first, InIter last, OutIter result);
    OutIter move_backward(InIter first, InIter last, OutIter result);

    pair<OutIter1, OutIter2> partition_copy(InIter first, InIter last,
            OutIter1 out_true, OutIter2 out_false, Pred pred);
    Iter partition_point(Iter first, Iter last, Pred pred);

    RAIter partial_sort_copy(InIter first, InIter last,
            RAIter result_first, RAIter result_last);
    RAIter partial_sort_copy(InIter first, InIter last,
            RAIter result_first, RAIter result_last, Compare comp);
    bool is_sorted(Iter first, Iter last);
    bool is_sorted(Iter first, Iter last, Compare comp);
    Iter is_sorted_until(Iter first, Iter last);
    Iter is_sorted_until(Iter first, Iter last, Compare comp);

    bool is_heap(Iter first, Iter last);
    bool is_heap(Iter first, Iter last, Compare comp);
    Iter is_heap_until(Iter first, Iter last);
    Iter is_heap_until(Iter first, Iter last, Compare comp);

    T min(initializer_list<T> t);
    T min(initializer_list<T> t, Compare comp);
    T max(initializer_list<T> t);
    T max(initializer_list<T> t, Compare comp);
    pair<const T&, const T&> minmax(const T& a, const T& b);
    pair<const T&, const T&> minmax(const T& a,
            const T& b,
             Compare comp);
    pair<const T&, const T&> minmax(initializer_list<T> t);
    pair<const T&, const T&> minmax(initializer_list<T> t,
             Compare comp);
    pair<Iter, Iter> minmax_element(Iter first, Iter last);
    pair<Iter, Iter> minmax_element(Iter first, Iter last, Compare comp);

    // 填充[first,last]范围内的每一个元素
    // 第一个元素为 value，第二个为++value，以此类 ui
    // 等同于
    // *(d_first)   = value;
    // *(d_first+1) = ++value;
    // *(d_first+2) = ++value;
    // *(d_first+3) = ++value; ...
    // 注意函数名，是 iota 而不是 itoa 哦
    void iota(Iter first, Iter last, T value); 
    ```

*   更有效的 move：more 操作比 copy 操作更有效率（参看 move semantics（译注：实际上是一个右值(rval)问题，核心是减少创建不必要的对象））。基于 move 的 std::sort()和 std::set::insert()要比基于 copy 的对应版本快 15 倍以上。不过它对标准库中已有操作的性能改善不多，因为它们的实现中已经使用了类似的方法进行优化了（例如 string，vector 使用了调优过的 swap 操作来代替 copy 了）。当然如果你自己的代码中包含了 move 操作的话，就能自动从新标准库中获益了。试着用 move 操作来替代下边这个 sort 函数中的智能指针（unique_ptr）吧（译注：可以通过一个 move 版 swap 来搞定，参看之前 move semantics 文章）：

    ```cpp
    template<class P> struct Cmp<P> { //比较 *P 的值
        bool operator() (P a, P b) const
                    { return *a<*b; }
    }

    vector<std::unique_ptr<Big>> vb;
    //  用指向大对象的 unique_ptr 填充 vb

    sort(vb.begin(),vb.end(),Cmp<Big>());// 不要像这样使用时用 auto_ptr 
    ```

*   对 lambda 表达式的使用：对于为标准库算法写函数/函数对象（function object，推荐）这个事儿大家已经抱怨很久了（例如 Cmp）。特别是在 C++98 标准中，这会令人更加痛苦，因为无法定义一个局部的函数对象。不过现在好多了，lambda 表达式允许用”inline”的方式来写函数了：

    ```cpp
    sort(vb.begin(),vb.end(),
         [](unique_ptr a, unique_ptr b) { return *a< *b; }); 
    ```

    我希望大家尽量多用用 lambda 表达式（它真的是一种很好很强大的机制）（译注：原文是：”I expect lambdas to be a bit overused initially (like all powerful mechanisms)”，非字面翻译）

*   对初始化列表（initializer lists）的使用：初始化表有时可以像参数那样方便的使用。看下边这个例子（x,y,z 是 string 变量，Nocase 是一个大小写不敏感的比较函数）：

    ```cpp
    auto x = max({x,y,z},Nocase()); 
    ```

参考：

*   25 Algorithms library [algorithms]
*   26.7 Generalized numeric operations [numeric.ops]
*   Howard E. Hinnant, Peter Dimov, and Dave Abrahams:

    [A Proposal to Add Move Semantics Support to the C++ Language](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1377.htm).

    N1377=02-0035.