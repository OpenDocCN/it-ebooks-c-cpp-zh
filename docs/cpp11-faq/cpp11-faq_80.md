# std::forward_list

# std::forward_list

std::forward_list 是一个基本的单向链表。它只提供了前向迭代器（forward

iteration）；并在执行插入/删除操作后，其他节点也不会受到影响（译注：其它迭代器不失效）。它尽可能减少所占用空间的大小（空链表很可能只占用一个 word2

（译注：2Byte））且不提供 size()操作（所以也没有存储 size 的数据成员），简略原型如下：

```cpp
template <ValueType T, Allocator Alloc = allocator<T> >
    requires NothrowDestructible<T>
class forward_list {
public:
    // the usual container stuff
    // no size()
    // no reverse iteration
    // no back() or push_back()
}; 
```

参看:

*   Standard: 23.3.3 Class template forward_list

（翻译：interma）