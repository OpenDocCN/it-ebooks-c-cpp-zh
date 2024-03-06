# 静态（编译期）断言 — static_assert

静态（编译期）断言由一个常量表达式及一个字符串文本构成：

```cpp
static_assert(expression, string); 
```

**expression**在编译期进行求值，当结果为 false（即：断言失败）时，将**string**作为错误消息输出。例如：

```cpp
static_assert(sizeof(long) >= 8,
   “64-bit code generation required for this library.”);
struct S { X m1; Y m2; };
static_assert(sizeof(S)==sizeof(X)+sizeof(Y),
    ”unexpected padding in S”); 
```

static_assert 在判断代码的编译环境方面（译注：比如判断当前编译环境是否 64 位）十分有用。但需要注意的是，由于 static_assert 在编译期进行求值，它不能对那些依赖于运行期计算的值的进行检验。例如：

```cpp
int f(int* p, int n)
{
      //错误：表达式“p == 0”不是一个常量表达式
      static_assert(p == 0,
          “p is not null”);
} 
```

（正确的做法是在运行期进行判断，假如条件不成立则抛出异常）

参考：

*   the C++ draft 7 [4].
*   [N1381==02-0039] Robert Klarer and John Maddock: [Proposal to Add Static Assertions to the Core Language](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2002/n1381.htm) .
*   [N1720==04-0160] Robert Klarer, John Maddock, Beman Dawes, Howard Hinnant: [Proposal to Add Static Assertions to the Core Language (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2004/n1720.html) .

（翻译：张潇，dabaitu）