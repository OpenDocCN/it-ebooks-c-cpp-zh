# 初始化列表

# 初始化列表

考虑如下代码：

```cpp
vector<double> v = { 1, 2, 3.456, 99.99 };
list<pair<string,string>> languages = {
    {"Nygaard","Simula"}, {"Richards","BCPL"}, {"Ritchie","C"}
};
map<vector<string>,vector<int>> years = {
    { {"Maurice","Vincent", "Wilkes"},
              {1913, 1945, 1951, 1967, 2000} },
    { {"Martin", "Ritchards"},
              {1982, 2003, 2007} },
    { {"David", "John", "Wheeler"},
              {1927, 1947, 1951, 2004} }
}; 
```

现在，初始化列表不再仅限于数组。可以接受一个“{}列表”对变量进行初始化的机制实际上是通过一个可以接受参数类型为 std::initializer_list 的函数（通常为构造函数）来实现的。例如：

```cpp
void f(initializer_list<int>);
f({1,2});
f({23,345,4567,56789});
f({});  // 以空列表为参数调用 f()
f{1,2}; // 错误：缺少函数调用符号( )
years.insert({{"Bjarne","Stroustrup"},{1950, 1975, 1985}}); 
```

初始化列表可以是任意长度，但必须是同质的（所有的元素必须属于某一模板类型 T, 或可转化至 T 类型的)。

容器可以用如下方式来实现“初始化列表构造函数”：

```cpp
template<class E> class vector {
public:
    // 初始化列表构造函数
            vector (std::initializer_list<E> s)
    {
           // 预留出合适的容量
              reserve(s.size());    //
        // 初始化所有元素
            uninitialized_copy(s.begin(), s.end(), elem);
     sz = s.size(); // 设置容器的 size
    }
    // ... 其他部分保持不变 ...
}; 
```

使用“{}初始化”时，直接构造与拷贝构造之间仍有细微差异，但不再像以前那样明显。例如，std::vector 拥有一个参数类型为 int 的显式构造函数及一个带有初始化列表的构造函数：

```cpp
vector<double> v1(7); // OK: v1 有 7 个元素<br />
v1 = 9;         // Err: 无法将 int 转换为 vector
vector<double> v2 = 9;    // Err: 无法将 int 转换为 vector

void f(const vector<double>&);
f(9);               // Err: 无法将 int 转换为 vector

vector<double> v1{7};     // OK: v1 有一个元素，其值为 7.0
v1 = {9};           // OK: v1 有一个元素，其值为 9.0
vector<double> v2 = {9};  // OK: v2 有一个元素，其值为 9.0
f({9});  // OK: f 函数将以列表{9}为参数被调用

vector<vector<double>> vs = {
    vector<double>(10), // OK, 显式构造(10 个元素，都是默认值 0.0)
    vector<double>{10}, // OK：显式构造(1 个元素，值为 10.0)
        10          // Err ：vector 的构造函数是显式的
}; 
```

函数可以将 initializer_list 作为一个不可变的序列进行读取。例如：

```cpp
void f(initializer_list<int> args)
{
    for (auto p=args.begin(); p!=args.end(); ++p)
        cout << *p << "\n";
} 
```

仅具有一个 std::initializer_list 的单参数构造函数被称为初始化列表构造函数。

标准库容器，string 类型及正则表达式均具有初始化列表构造函数，以及（初始化列表）赋值函数等。初始化列表亦可作为一种“序列”以供“序列化 for 语句”使用。（译注：参见“序列 for 循环语句”章节）

初始化列表同时也是“统一初始化”方案的一部分。（译注：参见“统一初始化的语法和语义”章节）

参考：

*   the C++ draft 8.5.4 List-initialization [dcl.init.list]
*   [N1890=05-0150 ] Bjarne Stroustrup and Gabriel Dos Reis:

    [Initialization and initializers](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1890.pdf)

    (an overview of initialization-related problems with suggested solutions).

*   [N1919=05-0179] Bjarne Stroustrup and Gabriel Dos Reis:

    [Initializer lists](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1919.pdf).

*   [N2215=07-0075] Bjarne Stroustrup and Gabriel Dos Reis :

    [Initializer lists (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2215.pdf) .

*   [N2640=08-0150] Jason Merrill and Daveed Vandevoorde:

    [Initializer Lists — Alternative Mechanism and Rationale (v. 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2640.pdf) (final proposal).

（翻译：dabaitu）