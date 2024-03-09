# 原生字符串标识

比如，你用标准 regex 库来写一个正则表达式，但正则表达式中的反斜杠’\’其实却是一个“转义(escape)”操作符(用于特殊字符)，这相当令人讨厌。考虑如何去写*“由反斜杠隔开的两个词语”*这样一个模式(\w\\w):

```cpp
string s = "\\w\\\\\\w";  // 希望它是对的（译注：不直观、不美观，且容易出错） 
```

请注意，在正则表达式和普通 C++字符串中，各自都需要使用连续两个反斜杠来表示反斜杠本身。然而，假如使用 C++11 的原生字符串，***反斜杠本身***仅需一个反斜杠就可以表示。因而，上述的例子简化为：

```cpp
string s = R"(\w\\\w)";  // 这次百分百正确 
```

引发原生字符串标识提议的是这样一个“惊天地泣鬼神”的例子：

```cpp
"('(?:[^\\\\']|\\\\.)*'|\"(?:[^\\\\\"]|\\\\.)*\")|"  // 这五个反斜杠是否正确?
      // 即使是专家，也很容易被这么多反斜杠搞得晕头转向 
```

**R”(…)”**记法相比于”…”会有一点点的冗长，但为了不必使用烦琐的“转义(escape)”符号，“多一点”是必要的。

那么，如何将双引号**‘”‘**本身放到原生字符串里呢？只要它不是正好跟在***右括弧’)’***之后，那么非常简单：

```cpp
R"("quoted string")"   // 这个字符串是 “quoted string” 
```

但是，假如我们偏要在原生字符串中表达***右括弧后跟双引号 )”*** 这样一个奇葩组合呢？首先，幸运地是，这种情况一般很少碰到；其次，”(…)”分隔法只不过是默认的分隔语法罢了。通过在**“(…)”**的**(…)**前后添加显式的自定义分隔号(译注:例如下面例子中的三个星号***)，我们还可以创造出任何我们想要的分隔语法。

```cpp
// 字符串为："quoted string containing the usual terminator (")"
R"***("quoted string containing the usual terminator (")")***" 
```

在右括弧之后的字符序列(即:自定义分隔号)必须与左括弧之前的字符序列相同。通过这种方式，我们几乎可以处理任意复杂的模式。

参考：

*   Standard 2.13.4
*   [N2053=06-0123] Beman Dawes: [Raw string literals](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2053.html) . (original proposal)
*   [N2442=07-0312] Lawrence Crowl and Beman Dawes: [Raw and Unicode String Literals; Unified Proposal (Rev. 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2442.htm) . (final proposal combined with the [User-defined literals](http://www2.research.att.com/%7Ebs/C++0xFAQ.html#UD-literals) proposal).

（翻译：张潇，dabaitu）