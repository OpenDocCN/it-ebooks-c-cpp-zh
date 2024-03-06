# C99 功能特性

# C99 功能特性

为了与 C 语言标准保持高度的兼容性，在 C 标准委员会的协助之下，一些细小的改变被引入到 C++0x 中。

*   long long
*   扩展的整型数据类型（例如，关于可选的更长的整型数的规则）
*   关于 UCN 的改变[N2170==07-0030]: 解除了”字符常量/字面字符串中不得使用控制/基本的通用字符名”的限制

```cpp
// 译注: C++03 中允许通过\uNNNN 的形式
// 在字符/字符串中引入非 ASCII 字符(Unicode)
// 但是控制字符以及基本的英文字母、数字、符号等
// 小于 0x00A0 的字符(与 ASCII 兼容)不在此列。
// 而在 C++11 中，这一限制被进一步放开
// OK in C++03 and C++11
const char* str1 = “Hello \u3366”;
// Err in C++03, OK in C++11
const char* str2 = “Hello \u0033”; 
```

*   宽/窄字符串的连接
*   Not VLAs（变长数组）

添加了一些扩展的预处理规则

*   **func** a 该宏经过宏展开(宏替换)后即为当前函数名
*   **STDC_HOSTED**
*   _Pragma: _Pragma( X ) 扩展成#pragma X
*   支持可变长度参数的宏（通过不同数目的参数对宏进行重载）

    ```cpp
    #define report(test, …) ((test)?puts(#test):printf(_ _VA_ARGS_ _)) 
    ```

*   空的宏参数

标准库中的很多功能组件都是继承自 C99（从本质上来说，所有 C99 的库的改变都是从 C89 继承而来的）

参考:

Standard: 16.3 Macro replacement.

[N1568=04-0008] P.J. Plauger: PROPOSED ADDITIONS TO TR-1 TO IMPROVE COMPATIBILITY WITH C99.