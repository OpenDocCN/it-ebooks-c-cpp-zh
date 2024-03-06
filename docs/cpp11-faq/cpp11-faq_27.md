# 属性（Attributes）

# 属性（Attributes）

“属性”是 C11 标准中的新语法，用于让程序员在代码中提供额外信息。相较于风格各异的传统方式(**attribute**, __declspec, #pragma 等)，“属性”语法致力于将各种“方言”进行统一。

与传统语法不同的是，“属性”语法相当灵活，可以随处添加，且总是作用于与之相邻的语法实体。

```cpp
void f [[ noreturn ]] () // f() 永不返回
{
  throw "error"; // 虽然不得返回，但可以抛出异常
}

struct foo* f [[carries_dependency]] (int i); // 编译优化指示
int* g(int* x, int* y [[carries_dependency]]); 
```

正如你看到的那样，属性被放置在两个双重中括号“[[…]]”之间。目前，noreturn 和 carries_dependency 是 C++11 标准中仅有的两个通用属性。

我们有理由担心属性的大量使用会引起 C++语言的混乱，很可能将产生很多 C++语言的“方言”。

所以，我们推荐仅在不影响源代码的业务逻辑的前提下，才使用属性来帮助编译器作更好的错误检查（例如，[[noreturn]]，或者是帮助代码优化（例如， [[carries_dependency]]）。

在未来的计划中，属性的一个重要用途是为 OpenMP 提供更好的辅助信息。例如：

```cpp
// 使用[[omp::parallel()]]属性告诉编译器，这个 for 循环可以并行执行
for [[omp::parallel()]] (int i=0; i<v.size(); ++i) {
// ... 
```

就像上面的代码展示的那样，通过指定 for 循环的[[omp::parallel()]]属性，编译器将使用 OpenMP 对这个 for 循环进行并行化处理，从而这个 for 循环将并行执行。

参考文献:

Standard: 7.6.1 Attribute syntax and semantics, 7.6.3-4 noreturn, carries_dependency 8 Declarators, 9 Classes, 10 Derived classes, 12.3.2 Conversion functions

[N2418=07-027] Jens Maurer, Michael Wong: Towards support for attributes in C++ (Revision 3)