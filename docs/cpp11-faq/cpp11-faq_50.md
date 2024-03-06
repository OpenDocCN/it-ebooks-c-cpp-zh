# Lambda 表达式

# Lambda 表达式

（译注:目前支持 lambda 的 gcc 编译器版本为 4.5，其它详细的编译器对于 C++11 新特性的支持请参考[`wiki.apache.org/stdcxx/C%2B%2B0xCompilerSupport）`](http://wiki.apache.org/stdcxx/C%2B%2B0xCompilerSupport）)

Lambda 表达式是一种描述函数对象的机制，它的主要应用是描述某些具有简单行为的函数（译注：Lambda 表达式也可以称为匿名函数，具有复杂行为的函数可以采用命名函数对象，当然,何谓复杂，何谓简单，这取决于编程人员的个人选择）。例如：

```cpp
vector<int> v = {50, -10, 20, -30};
std::sort(v.begin(), v.end());    // 排序时按照默认规则
// 此时 v 中的数据应该是 { -30, -10, 20, 50 }

// 利用 Lambda 表达式，按照绝对值排序
std::sort(v.begin(), v.end(), [](int a, int b)
{ return abs(a)<abs(b); });
// 此时 v 应该是 { -10, 20, -30, 50 } 
```

参数 `[](int a, int b) { return abs(a) < abs(b); }`是一个具有如下行为的"lambda"：接受两个整数 a 和 b，然后返回对它们的绝对值进行"`<`"比较的结果。

Lambda 表达式可以访问在它被调用的作用域内的局部变量。例如：

```cpp
void f(vector<Record>& v)
{
    vector<int> indices(v.size());
    int count = 0;
    generate(indices.begin(),indices.end(),[&count]()
    { return count++; });

    // 对 indices 按照记录的名字域顺序进行排序
    std::sort(indices.begin(), indices.end(), &
    { return v[a].name<v[b].name; });
    // ...
} 
```

有人认为这“相当简洁”，也有人认为这是一种可能产生危险且晦涩的代码的方式。我的看法是，两者都正确。

[&] 是一个“捕捉列表(capture list)”，用于描述将要被 lambda 函数以引用传参方式使用的局部变量。如果我们仅想“捕捉”参数 v，则可以写为: [&v]。而如果我们想以传值方式使用参数 v，则可以写为：[=v]。如果什么都不捕捉，则为：[]。将所有的变量以引用传递方式使用时采用[&], 而相对地，使用[=] 则相应地表示以传值方式使用所有变量。（译注：“所有变量”即指 lambda 表达式在被调用处，所能见到的所有局部变量）

如果某一函数的行为既不通用也不简单，那么我建议采用命名函数对象或者函数。例如，如上示例可重写为：

```cpp
void f( vector<Record>& v)
{
    vector<int> indices(v.size() );
    int count = 0;
    fill(indices.begin(), indices.end(), [&]()
    { return ++count; };

    struct Cmp_names {
        const vector& vr;
        Cmp_names(const vector<Record>& r) : vr(r) {}
        bool operator() (Record& a, Record& b) const
                { return vr[a] < vr[b]; }
    };

    //对 indices 按照记录的名字域顺序进行排序
    std::sort(indices.begin(), indices.end(), Cmp_names(v) );
        } 
```

(译注：此处采用了函数对象 Cmp_names(v)来代替 lambda 表达式，由于 Cmp_names 具有以引用传参方式的构造函数，因此 Cmp_names(v)相当于使用了”[&v]”的 lambda 表达式)

对于简单的函数功能，比如记录名称域的比较，采用函数对象就略显冗长，尽管它与 lambda 表达式生成的代码是一致的。在 C++98 中，这样的函数对象在被用作模板参数时必须是“非本地”的（译注：即你不能在函数对象中像此处的 lambda 表达式那样使用被调用处的局部变量），然而在 C++中（译注：意指 C++0x），这不再是必须的。

为了描述一个 lambda，你必须提供：

*   它的捕捉列表：即（除了形参之外）它可以使用的变量列表（”`[&]`” 在上面的记录比较例子中意味着“所有的局部变量都将按照引用的方式进行传递”）。如果不需要捕捉任何变量，则使用 []。
*   （可选的）它的所有参数及其类型（例如： `(int a, int b)` ）。
*   组织成一个块的函数行为（例如：`{ return v[a].name < v[b].name; }`）。
*   （可选的）使用”[返回值类型后置语法](http://www.chenlq.net/books/cpp11-faq/cpp11-faq-chinese-version-series-return-type-after-syntax.html)“来指明返回类型。但典型情况下，我们仅从 return 语句中去推断返回类型，如果没有返回任何值，则推断为 void。

参考：

*   Standard 5.1.2 Lambda expressions
*   [N1968=06-0038] Jeremiah Willcock, Jaakko Jarvi, Doug Gregor, Bjarne Stroustrup, and Andrew Lumsdaine:

    [Lambda expressions and closures for C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1968.htm)

    (original proposal with a different syntax)

*   [N2550=08-0060] Jaakko Jarvi, John Freeman, and Lawrence Crowl:

    [Lambda Expressions and Closures: Wording for Monomorphic Lambdas (Revision 4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2550.pdf) (final proposal).

*   [N2859=09-0049] Daveed Vandevoorde:

    [New wording for C++0x Lambdas](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2859.pdf).