# long long（长长整数类型）

这是一个至少为 64 bit 的整数类型（译注：实际宽度依赖于具体的实现平台），例如：

```cpp
long long x = 9223372036854775807LL; 
```

不过，不要想当然地认为存在 long long long 或者将 long 拼写为 short long long。

（译注：如同 J. Stephen Adamczyk 在参考文献中所言，”long long”是一个晦涩的拼写 64-bit 整数类型的方式，也不是一个可以解决不断增长的数据类型宽度的有效方法，目前，它仅仅是一个用于表达 64bit 整形数的标准。）

参考：

*   the C++ draft ???.
*   [05-0071==N1811] J. Stephen Adamczyk:

    [Adding the long long type to C++ (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1811.pdf).