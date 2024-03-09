# 右值引用

左值（赋值操作符“=”的左侧，通常是一个变量）与右值（赋值操作符“=”的右侧，通常是一个常数、表达式、函数调用）之间的差别可以追溯到 Christopher Strachey （C++的祖先语言 CPL 之父，指称语义学之父）时代。在 C++中，左值可被绑定到非 const 引用，左值或者右值则可被绑定到 const 引用。但是却没有什么可以绑定到非 const 的右值（译注：即右值无法被非 const 的引用绑定），这是为了防止人们修改临时变量的值，这些临时变量在被赋予新的值之前，都会被销毁。例如：

```cpp
void incr(int& a) { ++a; }
int i = 0;
incr(i);    // i 变为 1
//错误：0 不是一个左值
incr(0);
//（译注：0 不是左值，无法直接绑定到非 const 引用：int&。
// 假如可行，那么在调用时，将会产生一个值为 0 的临时变量，
// 用于绑定到 int&中，但这个临时变量将在函数返回时被销毁，
// 因而，对于它的任何更改都是没有意义的，
// 所以编译器拒绝将临时变量绑定到非 const 引用，但对于 const 的引用，
// 则是可行的） 
```

假设 incr(0)合法，那么要么产生一个程序员不可见的临时变量并进行无意义的递增操作，要么发生更悲剧的情况：把字面常量“0”的实际值会变成 1。尽管后者听起来是天方夜谭，但是对于早期 Frotran 等这一类把字面常量“0”也存到内存里的某个位置的编译器来说，这真的会变成一个 bug。（译注：指的是假如把用于存储字面常量 0 的内存单元上的值从 0 修改为 1 以后，后续所有使用字面常数 0 的地方实际上都在使用“1”）。

到目前为止，一切都很美好。但考虑如下函数：

```cpp
template<class T> swap(T& a, T& b) // 老式的 swap 函数
{
    T tmp(a);// 现在有两份"a"
    a = b;        // 现在有两份"b"
    b = tmp;    // 现在有两份 tmp(值同 a)
} 
```

如果 T 是一个拷贝代价相当高昂的类型，例如 string 和 vector，那么上述 swap()操作也将煞费气力（不过对于标准库来说，我们已经针对 string 和 vector 的 swap()进行了特化来处理这个问题）。注意这个奇怪的现象，我们的初衷其实并不是为了把这些变量拷来拷去，我是仅仅是想将变量 a,b,tmp 的值做一个“移动”（译注：即通过 tmp 来交换 a,b 的值）。

在 C++11 中，我们可以定义“移动构造函数(move constructors)”和“移动赋值操作符(move assignments”来“移动”而非复制它们的参数：

```cpp
template<class T> class vector {
        // …
        vector(const vector&);  // 拷贝构造函数
        vector(vector&&);   // 移动构造函数
        vector& operator= (const vector&); // 拷贝赋值函数
        vector& operator =(vector&&);  // 移动赋值函数
}; //注意：移动构造函数和移动赋值操作符接受
// 非 const 的右值引用参数，而且通常会对传入的右值引用参数作修改 
```

”&&”表示“右值引用”。右值引用可以绑定到右值（但不能绑定到左值）：

```cpp
X a;
X f();
X& r1 = a;        // 将 r1 绑定到 a(一个左值)
X& r2 = f();    // 错误：f()的返回值是右值，无法绑定
X&& rr1 = f();    // OK：将 rr1 绑定到临时变量
X&& rr2 = a;    // 错误：不能将右值引用 rr2 绑定到左值 a 
```

移动赋值操作背后的思想是，“赋值”不一定要通过“拷贝”来做，还可以通过把源对象简单地“偷换”给目标对象来实现。例如对于表达式 s1=s2，我们可以不从 s2 逐字拷贝，而是直接让 s1“侵占”s2 内部的数据存储(译注：比如 char* p)，并以某种方式“删除”s1 中原有的数据存储（或者干脆把它扔给 s2，因为大多情况下 s2 随后就会被析构）。（译注：仔细体会 copy 与 move 的区别。）

我们如何知道源对象能否“移动”呢？我们可以这样告诉编译器：（译注：通过 move()操作符）

```cpp
template <class T>
void swap(T& a, T& b)  //“完美 swap”（大多数情况下）
{
      T tmp = move(a);  // 变量 a 现在失效（译注：内部数据被 move 到 tmp 中了）
      a = move(b);      // 变量 b 现在失效（译注：内部数据被 move 到 a 中了，变量 a 现在“满血复活”了）
      b = move(tmp);    // 变量 tmp 现在失效（译注：内部数据被 move 到 b 中了，变量 b 现在“满血复活”了）
} 
```

move(x) 意味着“你可以把 x 当做一个右值”，把 move()改名为 rval()或许会更好，但是事到如今，move()已经使用很多年了。在 C++11 中，move()模板函数（参考“brief introduction”），以及右值引用被正式引入。

右值引用同时也可以用作完美转发(perfect forwarding)。（译注：比如某个接口函数什么也不做，只是将工作“委派”给其他工作函数）

在 C++11 的标准库中，所有的容器都提供了移动构造函数和移动赋值操作符，那些插入新元素的操作，如 insert()和 push_back(), 也都有了可以接受右值引用的版本。最终的结果是，在没有用户干预的情况下，标准容器和算法的性能都提升了，而这些都应归功于拷贝操作的减少。

参考：

*   the C++ draft section ???
*   N1385 N1690 N1770 N1855 N1952
*   [N2027==06-0097] Howard Hinnant, Bjarne Stroustrup, and Bronek Kozicki:

    [A brief introduction to rvalue references](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2027.html)

*   [N1377=02-0035] Howard E. Hinnant, Peter Dimov, and Dave Abrahams:

    [A Proposal to Add Move Semantics Support to the C++ Language](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1377.htm) (original proposal).

*   [N2118=06-0188] Howard Hinnant:

    [A Proposal to Add an Rvalue Reference to the C++ Language Proposed Wording (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2118.html) (final proposal).

（翻译：dabaitu，感谢：dave）