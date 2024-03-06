# async()

# async()

async()函数是一个简单任务的”启动”（launcher）函数，它是本 FAQ 中唯一一个尚未在标准草案中投票通过的特性。我希望它能在调和两个略微不同的意见之后最终于 10 月份获得通过（记得随时骚扰你那边的投票委员，一定要为它投票啊?）。

下边是一种优于传统的线程+锁的并发编程方法示例(译注：山寨 map-reduce 哦)：

```cpp
template<class T,class V> struct Accum  { // 简单的积函数对象
    T* b;
    T* e;
    V val;
    Accum(T* bb, T* ee, const V& v) : b{bb}, e{ee}, val{vv} {}
    V operator() () 
    { return std::accumulate(b,e,val); }
};

void comp(vector<double>& v)
    // 如果 v 够大，则产生很多任务        {
    if (v.size()<10000) 
        return std::accumulate(v.begin(),v.end(),0.0);

    auto f0 {async(Accum{&v[0],&v[v.size()/4],0.0})};
    auto f1 {async(Accum{&v[v.size()/4],&v[v.size()/2],0.0})};
    auto f2 {async(Accum{&v[v.size()/2],&v[v.size()*3/4],0.0})};
    auto f3 {async(Accum{&v[v.size()*3/4],&v[v.size()],0.0})};

    return f0.get()+f1.get()+f2.get()+f3.get();
} 
```

尽管这只是一个简单的并发编程示例（留意其中的”magic number“），不过我们可没有使用线程，锁，缓冲区等概念。f*变量的类型（即 async()的返回值）是”std::future”类型。future.get()表示如果有必要的话则等待相应的线程(std::thread)运行结束。async 的工作是根据需要来启动新线程，而 future 的工作则是等待新线程运行结束。”简单性”是 async/future 设计中最重视的一个方面；future 一般也可以和线程一起使用，不过不要使用 async()来启动类似 I/O 操作，操作互斥体（mutex），多任务交互操作等复杂任务。async()背后的理念和 range-for statement 很类似：简单事儿简单做，把复杂的事情留给一般的通用机制来搞定吧。

async()可以启动一个新线程或者复用一个它认为合适的已有线程（非调用线程即可）(译注：语义上并发即可，不关心具体的调度策略。和 go 语义中的 goroutines 有点像)。后者从用户视角看更有效一些（只对简单任务而言）。

参考：

*   Standard: ???
*   Lawrence Crowl:

    [An Asynchronous Call for C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2889.html).

    N2889 = 09-0079.

*   Herb Sutter :

    [A simple async()](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2901.pdf)

    N2901 = 09-0091 .

（翻译：interma）