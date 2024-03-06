# 具有作用域的内存分配器

为了使容器对象小巧和简单起见，C++98 没有要求容器支持具有状态的内存分配器，即不用把分配器对象存储在容器对象中。在 C++11 中，这仍然默认做法。但是，在 C++0x 中，也可以使用具有状态的内存分配器：这种内存分配器拥有一个指向分配区域的指针。例如：

```cpp
template<class T> class Simple_alloc {   // C++98 style
   // no data
   // usual allocator stuff
};

class Arena {
   void* p;
   int s;
public:
   Arena(void* pp, int ss);
   // allocate from p[0..ss-1]
};

template<class T> struct My_alloc {
   Arena& a;
   My_alloc(Arena& aa) : a(aa) { }
   // usual allocator stuff
};

Arena my_arena1(new char[100000],100000);
Arena my_arena2(new char[1000000],1000000);

vector<int> v0;   // allocate using default allocator

vector<int,My_alloc<int>> v1(My_alloc<int>{my_arena1});   // allocate from my_arena1

vector<int,My_alloc<int>> v2(My_alloc<int>{my_arena2});   // allocate from my_arena2

vector<int,Simple_alloc<int>> v3;   // allocate using Simple_alloc 
```

通常我们可以使用 typedef 来简化上述例子中繁冗的表达。

虽然并不能保证默认内存分配器和 Simple_alloc 不在一个 vector 对象中占用空间，但是在实现库的时候可以使用一些模板的元程序设计方法来做到这点。因此，如果对象拥有状态的话（比如 My_alloc），内存分配器将会带来一定的空间开销。

在同时使用容器和用户自定义内存分配器时存在一个更为隐蔽的问题：一个内存分配器成员是否应该和它的容器处于相同的分配区域中？例如，你使用

Your_allocator 来给 Your_string 的成员分配内存，而我用 My_allocator 来给 My_vector 的成员分配内存。在这种

情况下，My_vector 会使用哪个分配器。要解决这问题的话就需要告诉容器使用什么分配器来传递成员。下面的例子将给出实现这一点的方法。在这个例子

中我将使用分配器 My_alloc 来分配 vector 成员和 string 成员。首先，我必须要有一个能够接受 My_alloc 对象的 string 版本：

```cpp
//使用 My_alloc 的一个 string 类型
using xstring = basic_string<char, char_traits<char>, My_alloc<char>>; 
```

然后，我要有一个能够接受这种 string 类型以及 My_alloc 对象的 vector 版本。同时，该 vector 版本需要能够把 My_alloc 对象传递给 string。

```cpp
using svec =  vector<xstring,scoped_allocator_adaptor<My_alloc<xstring>>>; 
```

最后，我们可以按照下面的示例实现 My_alloc 类型的分配器：

```cpp
svec v(svec::allocator_type(My_alloc{my_arena1})); 
```

在此，svec 是一个成员为 string 类型的 vector，并且该 vector 使用 My_alloc 来为其 string 类型的成员分配内存。另外，标准

库中新提供了”adaptor” (“wrapper type”)

scoped_allocator_adaptor。它可以用来指明使用 My_alloc 来为 string 类型变量分配内存。需要注意的

是，scoped_allocator_adaptor 可以将 My_alloc 转换为 My_alloc。而这正是 xstring 所需要提供的功能。

于是，我们又有了 4 种可用于替换的方法：

```cpp
// vector and string use their own (the default) allocator:
using svec0 = vector<string>;
svec0 v0;

// vector (only) uses My_alloc and string uses its own (the default) allocator:
using svec1 = vector<string,My_alloc<string>>;
svec1 v1(My_alloc<string>{my_arena1});

// vector and string use My_alloc (as above):
using xstring = basic_string<char, char_traits<char>, My_alloc<char>>;
using svec2 = vector<xstring,scoped_allocator_adaptor<My_alloc<xstring>>>;
svec2 v2(scoped_allocator_adaptor<My_alloc<xstring>>{my_arena1});

// vector uses My_alloc and string uses My_string_alloc:
using xstring2 = basic_string<char, char_traits<char>, My_string_alloc<char>>;
using svec3 = vector<xstring2,scoped_allocator_adaptor<My_alloc<xstring>, My_string_alloc<char>>>;  
svec3 v3(scoped_allocator_adaptor<My_alloc<xstring2>, My_string_alloc<char>>{my_arena1,my_string_arena}); 
```

很明显，第一种方法 svec0 是最常用的。但是在内存会严重影响性能的系统中，其它版本（特别是 svec2）将会显得尤为重要。当然了，我们可以使用一些

typedef 来增强代码的可读性。不过还好，毕竟你不用每天都写这些。另外，我们提供了 scoped_allocator_adaptor2 的一个变

种：scoped_allocator_adaptor2。它用于两个非默认内存分配器不一样的情况。

同时可参考：

*   Standard: 20.8.5 Scoped allocator adaptor [allocator.adaptor]
*   Pablo Halpern:

    [The Scoped Allocator Model (Rev 2)](http://www.open-std.org/JTC1/SC22/WG21/docs/papers/2008/n2554.pdf).

    N2554=08-0064.

（翻译：Yibo Zhu）