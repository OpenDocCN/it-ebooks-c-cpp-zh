# 标准库中的元组（std::tuple）

# 标准库中的元组（std::tuple）

在标准库中，tuple（一个 N 元组：N-tuple）被定义为 N 个值的有序序列。在这里，N 可以是从 0 到文件中所定义的最大值中的任何一个常数。你可以认为 tuple 是一个未命名的结构体，该结构体包含了特定的 tuple 元素类型的数据成员。特别需要指出的是，tuple 中元素是被紧密地存储的（位于连续的内存区域），而不是链式结构。

可以显式地声明 tuple 的元素类型，也可以通过 make_tuple()来推断出元素类型。另外，可以使用 get()来通过索引（和 C++的数组索引一样，从 0 而不是从 1 开始）来访问 tuple 中的元素。

```cpp
tuple<string,int> t2(“Kylling”,123);

// t 的类型被推断为 tuple
auto t = make_tuple(string(“Herring”),10, 1.23);    
// 获取元组中的分量
string s = get<0>(t);
int x = get<1>(t);
double d = get<2>(t); 
```

有时候，我们需要一个编译时异构元素列表（a heterogeneous list of elements at compile time），但又不想定义一个有名字的结构来保存。这种情况下，我们就可以使用 tuple（直接地或间接地）. 例如，我们在 std::function and std::bind 中使用 tuple 来保存参数。

最常用的 tuple 是 2-tuple(二元组)，也就是一个 pair。但是标准库已经定义了 pair：std::pair (20.3.3 Pairs)。 我们可以使用 pair 来初始化一个 tuple，然而反之则不可。

另外，需要为 tuple 中的元素类型定义比较操作（`==`, `!=`, `<` , `<=`, `>`, 和 `>=`）.

参考：

*   Standard: 20.5.2 Class template tuple
*   Variadic template paper
*   Boost::tuple