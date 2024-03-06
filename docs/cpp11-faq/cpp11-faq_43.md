# 外部模板声明

# 外部模板声明

模板特化可以被显式声明，这可以作为消除多重实例化的一种方式。例如：

```cpp
#include "MyVector.h"
extern template class MyVector<int>; // 消除下面的隐式实例化
// MyVector 类将在“其他地方”被程序员显式实例化
void foo(MyVector<int>& v)
{
    //在这个地方使用 vector 类型
} 
```

下列代码就是上例中的”其他地方”：

```cpp
#include "MyVector.h"
// 使 MyVector 类对客户端（clients）可用（例如，共享库）
template class MyVector<int>; 
```

这种方法的主要目的是为避免编译器和链接器的大量“去除冗余的实例化”工作。

参见：

*   Standard 14.7.2 Explicit instantiation
*   [N1448==03-0031] Mat Marcus and Gabriel Dos Reis: [Controlling Implicit Template Instantiation](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1448.pdf)

    .

（翻译：lianggang jiang）