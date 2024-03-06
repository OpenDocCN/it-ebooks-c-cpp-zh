# weak_ptr

# weak_ptr

弱指针（weak pointer）经常被解释为用来打破使用 shared_ptr 管理的数据结构中循环(?)。但是我认为，将 weak_ptr 看成是指向具有下列特征的对象的指针更好一些。

*   只有当对象存在的时候，你才需要对其进行访问
*   并且它可能被其他人删除释放
*   并且在最后一次使用之后调用其析构函数（通常用于释放那些不具名的内存(anon-memory)资源

（译注：weak_ptr 可以保存一个“弱引用”，指向一个已经用 shared_ptr 进行管理的对象。为了访问这个对象，一个 weak_ptr 可以通过 shared_ptr 的构造函数或者是 weak_ptr 的成员函数 lock()转化为一个 shared_ptr。当最后一个指向这个对象的 shared_ptr 退出其生命周期并且这个对象被释放之后，将无法从指向这个对象的 weak_ptr 获得一个 shared_ptr 指针，shared_ptr 的构造函数会抛出异常，而 weak_ptr::lock 也会返回一个空指针。）

我们来考虑一下如何实现一个老式的“星盘棋”（ asteroid game）游戏，所有星星（asteroid）都属于游戏（the game），但是所有星星都必须与它周围的 星星保持联系，并且与之保持相反的状态。要维持一个相反的状态，通常会消去一个或者多个星星，也就是会调用其析构函数。每个星星必须有一个列表来保存记录它周围的星星。这里需要注意的是，在这样一个相邻星星列表中的星星不应该是具有完整生命的(?)，所以 shared_ptr 在这种情况下并不适合。另外一方面，当另外一个星星正看着某个星星时，（例如，依赖于这个星星计算其相反状态），这个星星就不能被析构。 当然，星星的析构函数必须被调用以释放其占用的资源（比如与图形系统的连接）。我们所需要的是一个在任何时间都应该保持完整无缺，并且随时都可以从中获取一个星星的星星列表(?)。weak_ptr 可以帮我们做到这一切：

```cpp
void owner()
{
    // …
    vector<shared_ptr<Asteroid>> va(100);
    for (int i=0; i<va.size(); ++i) { // 访问相邻的星星，计算相反状态
        va[i].reset(new Asteroid(weak_ptr(va[neighbor]));
        launch(i);
    }
    // …
} 
```

reset() 可以让一个 shared_ptr 指向另外一个新的对象。

当然，我对 ower 类作了相当大的简化，并且只给了每个星星一个邻居。这里的关键是，我们使用了 weak_ptr 指向其邻居星星。在计算相反状态的时候，ower 类则使用 shared_ptr 来代表星星与 owner 之间的所属关系(?)。一个星星的相反关系的计算应该是这样的：

```cpp
void collision(weak_ptr<Asteroid> p)
{
    // p.lock 返回一个指向 p 所指对象的 shared_ptr
    if (auto q = p.lock()) {    
        // … p 以及 q 指向的星星对象依然存在:进行计算…
    }
    else {
        // … oops: 星星对象已经被析构，我们可以忘掉它了(?)…
    }
} 
```

注意，即使 owner 决定关闭整个游戏并释放所有的星星对象（通过删除代表所属关系的多个 shared_ptr），每一个正在计算过程中的星星对象仍然可以正确地结束，因为 p.lock()将维持一个 shared_ptr，直到计算过程结束。（译注，也即是说，如果正在计算过程中关闭游戏并通过 shared_ptr 释放对象，那么 p.lock()会维持一个 shared_ptr，这样可以使得 shared_ptr 不会变成 0，在计算过程中，星星对象也就不会被错误地释放。当整个计算过程结束后，shared_ptr 的引用计数变为 0，星星对象被正确释放。）

我期望看到 weak_ptr 比简单的 shared_ptr 更少地被用到，并且我希望 unique_ptr 可以比 shared_ptr 更加流行，因为 unique_ptr 的所属关系更简单一些（译注：只能有一个 unique_ptr 指针指向某个对象，不向 shared_ptr,可以同时有多个 shared_ptr 指向同一个对象）并且性能更高，因而可以让局部的代码更容易理解。

参考

*   the C++ draft: Weak_ptr (20.7.13.3)