# std::future 和 std::promise

并行开发挺复杂的，特别是在试图用好线程和锁的过程中。如果要用到条件变量或 std-atomics（一种无锁开发方式），那就更复杂了。C++0x 提供了 future 和 promise 来简化任务线程间的返回值操作；同时为启动任务线程提供了 packaged_task 以方便操作。其中的关键点是允许 2 个任务间使用无（显式）锁的方式进行值传递；标准库帮你高效的做好这些了。基本思路很简单：当一个任务需要向父线程（启动它的线程）返回值时，它把这个值放到 promise 中。之后，这个返回值会出现在和此 promise 关联的 future 中。于是父线程就能读到返回值。更简单点的方法，参看 async()。

标准库中提供了 3 种 future：普通 future 和为复杂场合使用的 shared_future 和 atomic_future。在本主题中，只展示了普通 future，它已经完全够用了。如果我们有一个 future

f，通过 get()可以获得它的值：

```cpp
X v = f.get();  // if necessary wait for the value to get computed 
```

如果它的返回值还没有到达，调用线程会进行阻塞等待。要是等啊等啊，等到花儿也谢了的话，get()会抛出异常的（从标准库或等待的线程那个线程中抛出）。

如果我们不需要等待返回值（非阻塞方式），可以简单询问一下 future，看返回值是否已经到达：

```cpp
if (f.wait_for(0))
{   
    // there is a value to get()                
    // do something        
}        
else
{                
    // do something else       
} 
```

但是，future 最主要的目的还是提供一个简单的获取返回值的方法：get()。

promise 的主要目的是提供一个”put”（或”get”，随你）操作，以和 future 的 get()对应。future 和 promise 的名字是有历史来历的，是一个双关语。感觉有点别扭？请别怪我。

promise 为 future 传递的结果类型有 2 种：传一个普通值或者抛出一个异常

```cpp
try {
        X res;
        // compute a value for res
        p.set_value(res);
}
catch (…) {   // oops: couldn’t compute res
        p.set_exception(std::current_exception());
} 
```

到目前为止还不错，不过我们如何匹配 future/promise 对呢？一个在我的线程，另一个在别的啥线程中吗？是这样：既然 future 和 promise 可以被到处移动（不是拷贝），那么可能性就挺多的。最普遍的情况是父子线程配对形式，父线程用 future 获取子线程 promise 返回的值。在这种情况下，使用 async()是很优雅的方法。

packaged_task 提供了启动任务线程的简单方法。特别是它处理好了 future 和 promise 的关联关系，同时提供了包装代码以保证返回值/异常可以放到 promise 中，示例代码：

```cpp
void comp(vector& v)
{
        // package the tasks:
        // (the task here is the standard
        //  accumulate() for an array of doubles):
        packaged_task pt0{std::accumulate};
        packaged_task pt1{std::accumulate};

        auto f0 = pt0.get_future();     // get hold of the futures
        auto f1 = pt1.get_future();

        pt0(&v[0],&v[v.size()/2],0);    // start the threads
        pt1(&[v.size()/2],&v[size()],0);

        return f0.get()+f1.get();       // get the results
} 
```

参看：

*   Standard: 30.6 Futures [futures]
*   Anthony Williams:

    [Moving Futures – Proposed Wording for UK comments 335, 336, 337 and 338](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2888.html).

    N2888==09-0078.

*   Detlef Vollmann, Howard Hinnant, and Anthony Williams

    [An Asynchronous Future Value (revised)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2627.html)

    N2627=08-0137.

*   Howard E. Hinnant:

    [Multithreading API for C++0X – A Layered Approach](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2094.html).

    N2094=06-0164\. The original proposal for a complete threading package..

（翻译：interma）