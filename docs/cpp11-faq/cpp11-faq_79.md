# std::function 和 std::bind

标准库函数 bind()和 function()定义于头文件`<functional>`中（该头文件还包括许多其他函数对象），用于处理函数及函数参数。bind()接受一个函数（或者函数对象，或者任何你可以通过”(…)”符号调用的事物），生成一个其有某一个或多个函数参数被“绑定”或重新组织的函数对象。（译注：顾名思义，bind()函数的意义就像它的函数名一样，是用来绑定函数调用的某些参数的。）例如：

```cpp
int f(int, char, double);
// 绑定 f()函数调用的第二个和第三个参数，
// 返回一个新的函数对象为 ff，它只带有一个 int 类型的参数
auto ff = bind(f, _1, ‘c’, 1.2);    
int x = ff(7);                //  f(7, ‘c’, 1.2); 
```

参数的绑定通常称为”Currying”（译注：Currying—“烹制咖喱烧菜”，此处意指对函数或函数对象进行加工修饰操作）, “_1″是一个占位符对象，用于表示当函数 f 通过函数 ff 进行调用时，函数 ff 的第一个参数在函数 f 的参数列表中的位置。第一个参数称为”_1″, 第二个参数为”_2″，依此类推。例如：

```cpp
int f(int, char, double);
auto frev = bind(f, _3, _2, _1);        // 翻转参数顺序
int x = frev(1.2, ‘c’, 7);            // f(7, ‘c’, 1.2); 
```

此处，auto 关键字节约了我们去推断 bind 返回的结果类型的工作。

我们无法使用 bind()绑定一个重载函数的参数，我们必须显式地指出需要绑定的重载函数的版本：

```cpp
int g(int);
double g(double);

auto g1 = bind(g, _1);    // 错误：调用哪一个 g() ?
// 正确，但是相当丑陋
auto g2 = bind( (double(*)(double))g, _1); 
```

bind()有两种版本：一个如上所述，另一个则是“历史遗留”的版本：你可以显式地描述返回类型。例如：

```cpp
auto f2 = bind<int> (f, 7, ‘c’, _1);      // 显式返回类型
int x = f2(1.2);                    // f(7, ‘c’, 1.2); 
```

第二种形式的存在是必要的，并且因为第一个版本（(?) “and for a user simplest “，此处请参考原文)）无法在 C++98 中实现。所以第二个版本已经被广泛使用。

function 是一个拥有任何可以以”(…)”符号进行调用的值的类型。特别地，bind 的返回结果可以赋值给 function 类型。function 十分易于使用。（译注：更直观地，可以把 function 看成是一种表示函数的数据类型，就像函数对象一样。只不过普通的数据类型表示的是数据，function 表示的是函数这个抽象概念。）例如：

```cpp
// 构造一个函数对象，
// 它能表示的是一个返回值为 float，
// 两个参数为 int，int 的函数
function<float (int x, int y)> f;   

// 构造一个可以使用"()"进行调用的函数对象类型
struct int_div {   
        float operator() (int x, int y) const
             { return ((float)x)/y; };
};

f = int_div();                    // 赋值
cout<< f(5,3) <<endl;  // 通过函数对象进行调用
std::accumulate(b, e, 1, f);        // 完美传递 
```

成员函数可被看做是带有额外参数的自由函数：

```cpp
struct X {
        int foo(int);
};

// 所谓的额外参数，
// 就是成员函数默认的第一个参数，
// 也就是指向调用成员函数的对象的 this 指针
function<int (X*, int)> f;   
f = &X::foo;            // 指向成员函数

X x;
int v = f(&x, 5);  // 在对象 x 上用参数 5 调用 X::foo()
function<int (int)> ff = std::bind(f, &x, _1);    // f 的第一个参数是&x
v = ff(5);                // 调用 x.foo(5) 
```

function 对于回调函数、将操作作为参数传递等十分有用。它可以看做是 C++98 标准库中函数对象 mem_fun_t, pointer_to_unary_function 等的替代品。同样的，bind()也可以被看做是 bind1st()和 bind2nd()的替代品，当然比他们更强大更灵活。

参考：

*   Standard: 20.7.12 Function template bind, 20.7.16.2 Class template function
*   Herb Sutter:[Generalized Function Pointers](http://www.ddj.com/article/printableArticle.jhtml;jsessionid=QQIFSNAIOYXN0QSNDLPSKHSCJUNN2JVN?articleID=184403746&dept_url=/cpp/)

    .

    August 2003.

*   Douglas Gregor:

    [Boost.Function](http://www.boost.org/doc/libs/1_38_0/doc/html/function.html)

    .

*   Boost::bind

（翻译：dabaitu）