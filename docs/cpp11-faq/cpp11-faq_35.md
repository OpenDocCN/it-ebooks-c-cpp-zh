# decltype – 推断表达式的数据类型

# decltype – 推断表达式的数据类型

decltype(E)是一个标识符或者表达式的推断数据类型(“declared type”)，它可以用在变量声明中作为变量的数据类型。例如：

```cpp
void f(const vector<int>& a, vector<float>& b)
{
    // 推断表达式 a[0]*b[0]的数据类型，并将其定义为 Temp 类型
    typedef decltype(a[0]*b[0]) Tmp;
    // 使用 Tmp 作为数据类型声明变量，创建对象

    for (int i=0; i < b.size(); ++i) {
      Tmp* p = new Tmp(a[i]*b[i]);
      // …
   }
   // …
} 
```

这个想法以“typeof”的形式已经在通用语言中流行很久了，但是，现在使用中的 typeof 的实现并没有完成，并有一些兼容性问题，所以新标准将其命名为 decltype。

如果你仅仅是想根据初始化值为一个变量推断合适的数据类型，那么使用 auto 是一个更加简单的选择。当你只有需要推断某个表达式的数据类型，例如某个函数调用表达式的计算结果的数据类型，而不是某个变量的数据类型时，你才真正需要 delctype。

参考

? the C++ draft 7.1.6.2 Simple type specifiers

? [Str02] Bjarne Stroustrup. Draft proposal for “typeof”. C++ reflector message c++std-ext-5364, October 2002\. (original suggestion).

? [N1478=03-0061] Jaakko Jarvi, Bjarne Stroustrup, Douglas Gregor, and Jeremy Siek: Decltype and auto (original proposal).

? [N2343=07-0203] Jaakko Jarvi, Bjarne Stroustrup, and Gabriel Dos Reis: Decltype (revision 7): proposed wording.