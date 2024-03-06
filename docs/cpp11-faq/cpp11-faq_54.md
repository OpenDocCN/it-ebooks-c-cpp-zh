# 预防窄转换

# 预防窄转换

> （译注：“窄转换”是我见到过的一个翻译术语，但我忘记是在那本书上看到的。此处也可译为“预防类型截断”或者“预防类型切割”。）

问题现象：C 和 C++会进行隐式的（类型）截断

```cpp
int x = 7.3;    // 啊哦!
void f(int);
f(7.3);        // 啊哦! 
```

但是，在 C++11 中，使用{}进行初始化不会发生这种窄转换（译注：也就是使用{}对变量进行初始化时，不会进行隐式的类型截断，编译器会产生一个编译错误，防止隐式的类型截断的发生。）：

```cpp
int x0 {7.3};    // 编译错误: 窄转换
int x1 = {7.3};    // 编译错误：窄转换
double d = 7;
int x2{d};    // 编译错误：窄转换（double 类型转化为 int 类型）
char x3{7};  // OK：虽然 7 是一个 int 类型，但这不是窄转换
vector vi = {1, 2.3, 4, 5.6};  //错误：double 至 int 到窄转换 
```

C++11 避免许多不兼容性的方法是在进行窄转换时尽可能地依赖于用于初始化的实际值（如上例中的 7），而非仅仅依赖于变量类型作判断。如果一个值可以无损地用目标类型来存放，那么就不存在窄转换。

```cpp
// OK: 7 是一个 int 类型的数据，但是它可以被无损地表达为 char 类型数据
char c1{7};
// error: 发生了窄转换，初始值超出了 char 类型的范围
char c2{77777}; 
```

请注意，double 至 int 类型的转换通常都会被认为是窄转换，即使从 7.0 转换至 7。

> （评注：“{}初始化”对于类型转换的处理增强了 C++静态类型系统的安全性。传统的 C/C++中依赖于编程人员的初始化类型安全检查在 C++11 中通过“{}初始化”由编译器实施。）

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