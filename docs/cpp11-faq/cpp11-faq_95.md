# unique_ptr

unique_ptr（定义在中）提供了一种严格的语义上的所有权

*   拥有它所指向的对象
*   无法进行复制构造，也无法进行复制赋值操作（译注：也就是对其无法进行复制，我们无法得到指向同一个对象的两个 unique_ptr），但是可以进行移动构造和移动赋值操作
*   保存指向某个对象的指针，当它本身被删除释放的时候（例如，离开某个作用域），会使用给定的删除器（deleter）删除释放它指向的对象，

unique_ptr 的使用能够包括：

*   为动态申请的内存提供异常安全
*   将动态申请内存的所有权传递给某个函数
*   从某个函数返回动态申请内存的所有权
*   在容器中保存指针

“所有 auto_ptr 应该已经具有的（但是我们无法在 C++98 中实现的）功能”

unique_ptr 十分依赖于右值引用和移动语义。

下面是一段传统的会产生不安全的异常的代码：

```cpp
X* f()
{
    X* p = new X;
    // 做一些事情 – 可能会抛出某个异常
    return p;
} 
```

解决方法是，用一个 unique_ptr 来管理这个对象的所有权，由其进行这个对象的删除释放工作：

```cpp
X* f()
{
    unique_ptr p(new X); // 或者使用{new X}，但是不能 = new X
    // 做一些事情 – 可能会抛出异常
    return p.release();
} 
```

现在，如果程序执行过程中抛出了异常，unique_ptr 就会（毫无疑问地）删除释放它所指向的对象，这是最基本的 RAII。但是，除非我们真的需要返回一个内建的指针，我们可以返回一个 unique_ptr，让事情变得更好。

```cpp
unique_ptr f()
{
    unique_ptr p(new X); // 或者使用{new X}，但是不能 = new X
    //做一些事情 – 可能会抛出异常
    return p; // 对象的所有权被传递出 f()
} 
```

现在我们可以这样使用函数 f()：

```cpp
void g()
{
    unique_ptr q = f(); // 使用移动构造函数（move constructor）
    q->memfct(2); // 使用 q
    X x = *q; // 复制指针 q 所指向的对象
    // …
} // 在函数退出的时候，q 以及它所指向的对象都被删除释放 
```

unique_ptr 拥有“移动意义（move semantics）”，所以我们可以使用函数 f() 返回的右值对 q 进行初始化，这样就简单地将所有权传递给了 q。

在那些要不是为了避免不安全的异常问题（以及为了保证指针所指向的对象都被正确地删除释放），我们不可以使用内建指针的情况下，我们可以在容器中保存 unique_ptr 以代替内建指针：

```cpp
vector<unique_ptr<string>> vs { new string{“Doug”},
                                new string{“Adams”} }; 
```

unique_ptr 可以通过一个简单的内建指针构造完成，并且与内建指针相比，两者在使用上的差别很小。特殊情况下，unique_ptr 并不提供任何形式的动态检查(?)。

参考：

*   the C++ draft section 20.7.10
*   Howard E. Hinnant: unique_ptr Emulation for C++03 Compilers
*   final proposal.