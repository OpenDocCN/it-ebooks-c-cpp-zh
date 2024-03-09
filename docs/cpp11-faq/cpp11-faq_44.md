# 序列 for 循环语句

C++11 中引入了序列 for 循环以实现区间遍历的简化。这里所谓的区间可以是任一种能用迭代器遍历的区间，例如 STL 中由 begin()和 end()定义的序列。所有的标准容器，例如 std::string、 初始化列表、数组，甚至是 istream，只要定义了 begin()和 end()就行。

这里是一个序列 for 循环语句的例子：

```cpp
void f(const vector& v)
{
    for (auto x : v) cout << x << ‘n’;
    for (auto& x : v) ++x;    // 使用引用，方便我们修改容器中的数据
} 
```

可以这样理解这里的序列 for 循环语句，“对于 v 中的所有数据元素 x”，循环由 v.begin()开始，循环到 v.end()结束。又如：

```cpp
for (const auto x : { 1,2,3,5,8,13,21,34 }) 
   cout << x << ‘n’; 
```

begin()函数（包括 end()函数）可以是成员函数通过 x.begin()方式调用，或者是独立函数通过 begin(x)方式调用。

（译注：好像 C#中早就有这种形式的 for 循环语句，用于遍历一个容器中的所有数据很方便，难道 C++是从 C#中借用过来的？）

或参见：

*   the C++ draft section 6.5.4 (note: changed not to use concepts)
*   [N2243==07-0103] Thorsten Ottosen:

    [Wording for range-based for-loop (revision 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2243.html).

*   [N3257=11-0027 ] Jonathan Wakely and Bjarne Stroustrup: [Range-based for statements and ADL](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2011/n3257.pdf) (Option 5 was chosen).