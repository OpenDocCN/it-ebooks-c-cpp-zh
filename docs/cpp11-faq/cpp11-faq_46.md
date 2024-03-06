# 类成员的内部初始化

# 类成员的内部初始化

在 C++98 标准里，只有 static const 声明的整型成员能在类内部初始化，并且初始化值必须是常量表达式。这些限制确保了初始化操作可以在编译时期进行。例如：

```cpp
int var = 7;
class X {
    static const int m1 = 7;   // 正确
    const int m2 = 7;    // 错误：无 static
    static int m3 = 7;              // 错误：无 const
    static const int m4 = var;  // 错误：初始化值不是常量表达式
    static const string m5 = “odd”; //错误：非整型
    // …
}; 
```

C++11 的基本思想是，允许非静态（non-static）数据成员在其声明处（在其所属类内部）进行初始化。这样，在运行时，需要初始值时构造函数可以使用这个初始值。考虑下面的代码：

```cpp
class A {
public:
    int a = 7;
}; 
```

这等同于：

```cpp
class A {
public:
    int a;
    A() : a(7) {}
}; 
```

单纯从代码来看，这样只是省去了一些文字输入，其实它的真正永无之地在于拥有多个构造函数的类。因为大多情况下，对于同一个成员，多个构造函数应使用相同的值去初始化。例如：

```cpp
class A {
public:
     A(): a(7), b(5), hash_algorithm(“MD5″),
       s(“Constructor run”) {}
    A(int a_val) :
      a(a_val), b(5), hash_algorithm(“MD5″),
      s(“Constructor run”)
      {}
    A(D d) : a(7), b(g(d)),
        hash_algorithm(“MD5″), s(“Constructor run”)
        {}
    int a, b;
private:
    // 哈希加密函数可应用于类 A 的所有实例
    HashingFunction hash_algorithm;
    std::string s;  // 用以指明对象正处于生命周期内何种状态的字符串
}; 
```

对于每一个构造函数，程序员必须使用完全一样的字面值来来初始化 hash_algorithm 和 s 这两个成员。但是并不是所有人都记得严格遵守这条规则，一旦出现纰漏，程序将难以维护。C++11 给出了解决之道：可在成员声明的地方直接赋以初值：

```cpp
class A {
public:
    A(): a(7), b(5) {}
    A(int a_val) : a(a_val), b(5) {}
    A(D d) : a(7), b(g(d)) {}
    int a, b;
private:
    //哈希加密函数可应用于类 A 的所有实例
    HashingFunction hash_algorithm{“MD5″};
    //用以指明对象正处于生命周期内何种状态的字符串
    std::string s{“Constructor run”};
}; 
```

如果一个成员同时在类内部初始化时和构造函数内被初始化，则只有构造函数的初始化有效（这个初始化值“优先于”默认值）（译注：可以认为，类内部初始化先于构造函数初始化进行，如果是对同一个变量进行初始化，构造函数初始化会覆盖类内部初始化）。因此，我们可以进一步简化：

```cpp
class A {
public:
    A() {}
    A(int a_val) : a(a_val) {}
    A(D d) : b(g(d)) {}
    int a = 7;
    int b = 5;
private:
    //哈希加密函数可应用于类 A 的所有实例
    HashingFunction hash_algorithm{“MD5″};
    //用以指明对象正处于生命周期内何种状态的字符串
    std::string s{“Constructor run”};
}; 
```

参考文献：

*   the C++ draft section “one or two words all over the place”; see proposal.
*   [N2628=08-0138] Michael Spertus and Bill Seymour:

    [Non-static data member initializers](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2628.html).

（翻译：lianggang jiang）