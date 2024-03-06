# array

# array

std::array 是一个支持随机访问且大小（size）固定的容器（译注：可以认为是一个紧缩版的 vector 吧）。它有如下特点：

*   不预留多余空间，只分配必须空间（译注：size() == capacity()）。
*   可以使用初始化表（initializer list）的方式进行初始化。
*   保存了自己的 size 信息。
*   不支持隐式指针类型转换。

换句话说，可以认为它是一个很不错的内建数组类型。一些代码片段：

```cpp
array<int,6> a = { 1, 2, 3 };
a[3]=4;
int x = a[5];    // array 的默认数据元素为 0，所以 x 的值变成 0 
int* p1 = a; // 错误: std::array 不能隐式地转换为指针
int* p2 = a.data(); // 正确，data()得到指向第一个元素的指针 
```

不过要注意：是可以定义一个长度为 0 的 array 的；但是无法从初始化表中推导出 size 信息：

```cpp
array<int> a3 = { 1, 2, 3 };  //错误：没有 size 信息
array<int,0> a0;   // 正确: 没有任何元素
int* p = a0.data();  // 为定义行为，不要这样做 
```

array 非常适合在嵌入式系统（和有类似限制/性能敏感/安全关键系统等）中使用。它提供了序列型容器该有的大部分通用函数（和 vector 很像）：

```cpp
template<class C> C::value_type sum(const C& a)
{
    return accumulate(a.begin(),a.end(),0);
}

array<int,10> a10;
array<double,1000> a1000;
vector<int> v;
// …
int x1 = sum(a10);
int x2 = sum(a1000);
int x3 = sum(v); 
```

但是，它是不支持由子类到基类的自动类型转换的（注意这个潜在陷阱）：

```cpp
struct Apple : Fruit { /* … */ };
struct Pear : Fruit { /* … */ };

void nasty(array<fruit *,10>& f)
{
        f[7] = new Pear();
};

array<apple ,10> apples;
// …
nasty(apples);  // 错误: 不能将 array 转换为 array; 
```

如果支持这种转换的话，apple array 中就能放 pear 啦。

参考：

*   Standard: 23.3.1 Class template array

（翻译：interma）