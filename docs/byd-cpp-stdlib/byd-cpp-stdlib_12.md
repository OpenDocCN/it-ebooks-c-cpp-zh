# Library 5\. Regex

# Regex 库如何改进你的程序？

## Regex 库如何改进你的程序？

*   为 C++带来了对正则表达式的支持

*   改进有效输入的健壮性

在文本处理中常常会用到正则表达式。例如，有很多验证有效性的工作适合使用正则表达式。考虑一个应用 程序，它要求输入只由数字组成。而另一个程序可能要求一种特殊的格式，如三个数字，后跟一个字母，再后跟两个数字。你可能要验证邮政编码、信用卡号码、社 会保险号码，或者其它东西；使用正则表达式来做这些验证是很简单的。另外一个可以使用正则表达式的地方是文本替换，即用某些文本替换掉另一些文本。假如你 需要在很多文档中将单词 colour 转换为 color。正则表达式提供了最好的方法来做这个工作，包括同时记住替换 Colour 和 COLOUR, 以及复数形式的 colours, 动词 colourize, 等等。还有一个可以使用正则表达式的地方就是，格式化文本。

许多流行的编程语言，Perl 是其中一例，都内建了对正则表达式的支持，但在 C++里没有。C++标 准在提到正则表达式时也保持了沉默。Boost.Regex 是一个非常完善并有效的库，它将正则表达式并入了 C++程序，并且它包含了 Perl, grep 和 Emacs 等常见工具所使用的几种不同语法。它是最著名的 C++正则表达式库之一，既容易使用又异常强大。

# Regex 如何适用于标准库？

## Regex 如何适用于标准库？

目前的 C++标准库是不支持正则表达式的。这是令人遗憾的，有那么多对正则表达式的需要，有时用户为 了编写需要支持正则表达式的程序而不得不放弃使用 C++。Boost.Regex 填补了标准在这方面的空白，并且它已经被提议加入到下一个版本的 C++标 准中。Boost.Regex 已经被即将发布的 Library Technical Report 所接受。

# Regex

## Regex

### 头文件: `"boost/regex.hpp"`

正则表达式被封装为一个类型 `basic_regex`的对象。我们将在下一节更深入地讨论正则表达式如何被编译和分析，这里我们首先粗略地看看 `basic_regex` ，以及这个库中三个最重要的算法。

```cpp
namespace boost {
  template <class charT,
            class traits=regex_traits<charT> >
  class basic_regex {
  public:
    explicit basic_regex(
      const charT* p, 
      flag_type f=regex_constants::normal);

    bool empty() const; 

    unsigned mark_count() const; 

    flag_type flags() const;
  };

  typedef basic_regex<char> regex;
  typedef basic_regex<wchar_t> wregex;
} 
```

### 成员函数

```cpp
explicit basic_regex (
  const charT* p, 
  flag_type f=regex_constants::normal); 
```

这个构造函数接受一个包含正则表达式的字符序列，还有一个参数用于指定使用正则表达式时的选项，例如是否忽略大小写。如果`p`中的正则表达式无效，则抛出一个 `bad_expression` 或 `regex_error` 的异常。注意这两个异常其实是同一个东西；在写这本书之时，尚未改变当前使用的名字 `bad_expression` ，但下一个版本的 Boost.Regex 将会使用 `regex_error`.

```cpp
bool empty() const; 
```

这个成员函数是一个谓词，当`basic_regex`实例没有包含一个有效的正则表达式时返回 `true` ，即它被赋予一个空的字符序列时。

```cpp
unsigned mark_count() const; 
```

`mark_count` 返回`regex`中带标记子表达式的数量。带标记子表达式是指正则表达式中用圆括号括起来的部分。匹配这个子表达式的文本可以通过调用某个正则表达式算法而获得。

```cpp
flag_type flags() const; 
```

返回一个位掩码，其中包含这个`basic_regex`所设置的选项标志。例如标志 `icase`, 表示正则表达式忽略大小写，标志 `JavaScript`, 表示 regex 使用 JavaScript 的语法。

```cpp
typedef basic_regex<char> regex;
typedef basic_regex<wchar_t> wregex; 
```

不要使用类型 `basic_regex`来定义变量，你应该使用这两个`typedef`中的一个。这两个类型，`regex` 和 `wregex`, 是两种字符类型的缩写，就如 `string` 和 `wstring` 是 `basic_string&lt;char&gt;` 和 `basic_string&lt;wchar_t&gt;`的缩写一样。这种相似性是不一样的，某种程度上，`regex` 是一个特定类型的字符串的容器。

### 普通函数

```cpp
template <class charT,class Allocator,class traits >
  bool regex_match(
    const charT* str, 
    match_results<const charT*,Allocator>& m,
    const basic_regex<charT,traits >& e,
    match_flag_type flags = match_default); 
```

`regex_match` 判断一个正则表达式(参数 `e`)是否匹配整个字符序列 `str`. 它主要用于验证文本。注意，这个正则表达式必须匹配被分析串的全部，否则函数返回 `false`. 如果整个序列被成功匹配，`regex_match` 返回 `True`.

```cpp
template <class charT,class Allocator, class traits> 
  bool regex_search(
    const charT* str,
    match_results<const charT*,Allocator>& m,
    const basic_regex<charT,traits >& e,
    match_flag_type flags = match_default); 
```

`regex_search` 类似于 `regex_match`, 但它不要求整个字符序列完全匹配。你可以用 `regex_search` 来查找输入中的一个子序列，该子序列匹配正则表达式 `e`.

```cpp
template <class traits,class charT>
  basic_string<charT> regex_replace(
    const basic_string<charT>& s,
    const basic_regex<charT,traits >& e,
    const basic_string<charT>& fmt,
    match_flag_type flags = match_default); 
```

`regex_replace` 在整个字符序列中查找正则表达式`e`的所有匹配。这个算法每次成功匹配后，就根据参数`fmt`对匹配字符串进行格式化。缺省情况下，不匹配的文本不会被修改，即文本会被输出但没有改变。

这三个算法都有几个不同的重载形式：一个接受 `const charT*` (`charT` 为字符类型), 另一个接受 `const basic_string&lt;charT&gt;&`, 还有一个重载接受两个双向迭代器作为输入参数。

# 用法

## 用法

要使用 Boost.Regex, 你需要包含头文件`"boost/regex.hpp"`. Regex 是本书中两个需要独立编译的库之一(另一个是 Boost.Signals)。你会很高兴获知如果你已经构建了 Boost— —那只需在命令提示符下打一行命令——就可以自动链接了(对于 Windows 下的编译器)，所以你不需要为指出那些 库文件要用而费心。

你要做的第一件事就是声明一个类型 `basic_regex` 的变量。这是该库的核心类之一，也是存放正则表达式的地方。创建这样一个变量很简单；只要将一个含有你要用的正则表达式的字符串传递给构造函数就行了。

```cpp
boost::regex reg("(A.*)"); 
```

这个正则表达式具有三个有趣的特性。第一个是，用圆括号把一个子表达式括起来，这样可以稍后在同一个正则表达式中引用它，或者取出匹配它的文本。我们稍后会详细讨论它，所以如果你还不知道它有什么用也不必担心。第二个是，通配符(wildcard)字符，点。这个通配符在正则表达式中有非常特殊的意义；这可以匹配任意字符。最后一个是，这个表达式用到了一个重复符，`*`, 称为 Kleene star, 表示它前面的表达式可以被匹配零次或多次。这个正则表达式已可以用于某个算法了，如下：

```cpp
bool b=boost::regex_match(
  "This expression could match from A and beyond.",
  reg); 
```

如你所见，你把正则表达式和要分析的字符串传递给算法 `regex_match`. 如果的确存在与正则表达式的匹配，则该函数调用返回结果 `true` ；否则，返回 `false`. 在这个例子中，结果是 `false`, 因为 `regex_match` 仅当整个输入数据被正则表达式成功匹配时才返回 `true` 。你知道为什么是这样吗？再看一下那个正则表达式。第一个字符是大写的 `A`, 很明显能够匹配这个表达式的第一个字符在哪。所以，输入的一部分`"A and beyond."`可以匹配这个表达式，但这不是整个输入。让我们试一下另一个输入字符串。

```cpp
bool b=boost::regex_match(
  "As this string starts with A, does it match? ",
  reg); 
```

这一次，`regex_match` 返回 `true`. 当正则表达式引擎匹配了 `A`, 它接着看后续有什么。在我们的 regex 变量中，`A` 后跟一个通配符和一个 Kleene star, 这意味着任意字符可以被匹配任意次。因而，分析过程开始扔掉输入字符串的剩余部分，即匹配了输入的所有部分。

接下来，我们看看如何使用 regexes 和 `regex_match` 来进行数据验证。

### 验证输入

正则表达式常用于对输入数据的格式进行验证。应用软件通常要求输入符合某种结构。考虑一个应用软件，它要求输入一定要符合如下格式，"3 个数字, 一个单词, 任意字符, 2 个数字或字符串"N/A," 一个空格, 然后重复第一个单词." 手工编写代码来验证这个输入既沉闷又容易出错，而且这些格式还很可能会改变；在你弄明白之前，可能就需要支持其它的格式，你精心编写的分析器可能就需要修 改并重新调试。让我们写出一个可以验证这个输入的正则表达式。首先，我们需要一个匹配 3 个数字的表达式。对于数字，我们应该使用一个特别的缩写，`\d`。要表示它被重复 3 次，需要一个称为 bounds operator 的特定重复，它用花括号括起来。把这两个合起来，就是我们的正则表达式的开始部分了。

```cpp
boost::regex reg("\\d{3}"); 
```

注意，我们需要在转义字符()之前加一个转义字符，即在我们的字符串中，缩写 `\d` 变成了 `\\d` 。这是因为编译器会把第一个\当成转义字符扔掉；我们需要对\进行转义，这样\才可以出现在我们的正则表达式中。

接下来，我们需要定义一个单词的方法，即定义一个字符序列，该序列结束于一个非字母字符。有不只一种方法可以实现它，我们将使用字符类别(也称为字符集)和范围这两个正则表达式的特性来做。字符类别即一个用方括号括起来的表达式。例如，一个匹配字符`a`, `b`, 和 `c`中任一个的字符类别表示为：`[abc]`. 如果用范围来表示同样的东西，我们要写：`[a-c]`. 要写一个包含所有字母的字符类型，我们可能会有点发疯，如果要把它写成： `[abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ]`, 但不用这样；我们可以用范围来表示：`[a-zA-Z]`. 要注意的是，象这样使用范围要依赖于当前所用的 locale，如果正则表达式的 `basic_regex::collate` 标志被打开。使用以上工具以及重复符 `+`, 它表示前面的表达式可以重复，但至少重复一次，我们现在可以表示一个单词了。

```cpp
boost::regex reg("[a-zA-Z]+"); 
```

以上正则表达式可以工作，但由于经常要表示一个单词，所以有一个更简单的方法：`\w`. 这个符号匹配所有单词，不仅是 ASCII 的单词，因此它不仅更短，而且也更适用于国际化的环境。接下来的字符是一个任意字符，我们已经知道要用点来表示。

```cpp
boost::regex reg("."); 
```

再接下来是 2 个数字或字符串 "N/A." 为了匹配它，我们需要用到一个称为选择的特性。选择即是匹配两个或更多子表达式中的任意一个，每种选择之间用 `|` 分隔开。就象这样：

```cpp
boost::regex reg("(\\d{2}|N/A)"); 
```

注意，这个表达式被圆括号括了起来，以确保整个表达式被看作为两个选择。在正则表达式中增加一个空格是很简单的；用缩写`\s`. 把以上每一样东西合并起来，就得到了以下表达式：

```cpp
boost::regex reg("\\d{3}[a-zA-Z]+.(\\d{2}|N/A)\\s"); 
```

现在事情变得有点复杂了。我们需要某种方法，来验证接下来的输入数据中的单词是否匹配第一个单词(即那个我们用表达式`[a-zA-Z]+`所捕获的单词)。关键是要使用后向引用(back reference)，即对前面的子表达式的引用。为了可以引用表达式 `[a-zA-Z]+`, 我们必须先把它用圆括号括起来。这使得表达式`([a-zA-Z]+)`成为我们的正则表达式中的第一个子表达式，我们就可以用索引 1 来建立一个后向引用了。

这样，我们就得到了整个正则表达式，用于表示"3 个数字, 一个单词, 任意字符, 2 个数字或字符串"N/A," 一个空格, 然后重复第一个单词."：

```cpp
boost::regex reg("\\d{3}([a-zA-Z]+).(\\d{2}|N/A)\\s\\1"); 
```

干的好！下面是一个简单的程序，把这个表达式用于算法 `regex_match`, 验证两个输入字符串。

```cpp
#include <iostream>
#include <cassert>
#include <string>
#include "boost/regex.hpp"

int main() {
  // 3 digits, a word, any character, 2 digits or "N/A", 
  // a space, then the first word again
  boost::regex reg("\\d{3}([a-zA-Z]+).(\\d{2}|N/A)\\s\\1");

  std::string correct="123Hello N/A Hello";
  std::string incorrect="123Hello 12 hello";

  assert(boost::regex_match(correct,reg)==true);
  assert(boost::regex_match(incorrect,reg)==false);
} 
```

第一个字符串，`123Hello N/A Hello`, 是正确的；`123` 是 3 个数字，`Hello` 是一个后跟任意字符(一个空格)的单词， 然后是 N/A 和另一个空格，最后重复单词`Hello` 。第二个字符串是不正确的，因为单词 `Hello` 没有被严格重复。缺省情况下，正则表达式是大小写敏感的，因而反向引用不能匹配。

写出正则表达式的一个关键是成功地分解问题。看一下你刚才建立的最终的那个表达式，对于未经过训练的人来说它的确很难懂。但是，如果把这个表达式分解成小的部分，它就不太复杂了。

### 查找

现在我们来看一下另一个 Boost.Regex 算法, `regex_search`. 与 `regex_match` 不同的是，`regex_search` 不要求整个输入数据完全匹配，则仅要求部分数据可以匹配。作为说明，考虑一个程序员的问题，他可能在他的程序中有一至两次忘记了调用 `delete` 。虽然他知道这个简单的测试可能没什么意义，他还是决定计算一下 new 和 delete 出现的次数，看看数字是否符合。这个正则表达式很简单；我们有两个选择，new 和 delete.

```cpp
boost::regex reg("(new)|(delete)"); 
```

有两个原因我们要把子表达式用括号括起来：一个是为了表明我们的选择是两个组。另一个原因是我们想在调用`regex_search`时引用这些子表达式，这样我们就可以判断是哪一个选择被匹配了。我们使用`regex_search`的一个重载，它接受一个`match_results`类型的参数。当 `regex_search` 执行匹配时，它通过一个`match_results`类型的对象报告匹配的子表达式。类模板 `match_results` 使用一个输入序列所用的迭代器类型来参数化。

```cpp
template <class Iterator,
  class Allocator=std::allocator<sub_match<Iterator> >
    class match_results;

typedef match_results<const char*> cmatch;
typedef match_results<const wchar_t> wcmatch;
typedef match_results<std::string::const_iterator> smatch;
typedef match_results<std::wstring::const_iterator> wsmatch; 
```

我们将使用 `std::string`, 所以要留意 `typedef smatch`, 它是 `match_results&lt;std::string::const_iterator&gt;`的缩写。如果 `regex_search` 返回 `true`, 传递给该函数的 `match_results` 引用将包含匹配的子表达式结果。在 `match_results`里，用已索引的`sub_match`来表示正则表达式中的每个子表达式。我们来看一下我们如何帮助这位困惑的程序员来计算对`new` 和 `delete`的调用。

```cpp
boost::regex reg("(new)|(delete)");
boost::smatch m;
std::string s=
  "Calls to new must be followed by delete. \
  Calling simply new results in a leak!";

if (boost::regex_search(s,m,reg)) {
  // Did new match?
  if (m[1].matched)
    std::cout << "The expression (new) matched!\n";
  if (m[2].matched)
    std::cout << "The expression (delete) matched!\n";
} 
```

以上程序在输入字符串中查找 `new` 或 `delete`, 并报告哪一个先被找到。通过传递一个类型 `smatch` 的对象给 `regex_search`, 我们可以得知算法如何执行成功的细节。我们的表达式中有两个子表达式，因此我们可以通过`match_results`的索引 1 得到子表达式 `new` . 这样我们得到一个 `sub_match`实例，它有一个 Boolean 成员，`matched`, 告诉我们这个子表达式是否参与了匹配。因此，对于上例的输入，运行结果将输出"The expression (new) matched!\n". 现在，你还有一些工作要做。你需要继续把正则表达式应用于输入的剩余部分，为此，你要使用另外一个 `regex_search`的重载，它接受两个迭代器，指示出要查找的字符序列。因为 `std::string` 是一个容器，它提供了迭代器。现在，在每一次匹配时，你必须把指示范围起始点的迭代器更新为上一次匹配的结束点。最后，增加两个变量来记录 `new` 和 `delete`的次数。以下是完整的程序：

```cpp
#include <iostream>
#include <string>
#include "boost/regex.hpp"

int main() {
  // "new" and "delete" 出现的次数是否一样？
  boost::regex reg("(new)|(delete)");
  boost::smatch m;
  std::string s=
    "Calls to new must be followed by delete. \
     Calling simply new results in a leak!";
  int new_counter=0;
  int delete_counter=0;
  std::string::const_iterator it=s.begin();
  std::string::const_iterator end=s.end();

  while (boost::regex_search(it,end,m,reg)) {
    // 是 new 还是 delete?
    m[1].matched ? ++new_counter : ++delete_counter;
    it=m[0].second;
  }

  if (new_counter!=delete_counter)
    std::cout << "Leak detected!\n";
  else
    std::cout << "Seems ok...\n";
} 
```

注意，这个程序总是把迭代器 `it` 设置为 `m[0].second`。`match_results[0]` 返回对匹配整个正则表达式的子匹配的引用，因此我们可以确认这个匹配的结束点就是下次运行`regex_search`的起始点。运行这个程序将输出"Leak detected!", 因为这里有两次 `new`, 而只有一次 `delete`. 当然，一个变量也可能在两个地方删除，还有可能调用 `new[]` 和 `delete[]`, 等等。

现在，你应该已经对子表达式如何分组使用有了较好的了解。现在是时候进入到最后一个 Boost.Regex 算法，该算法用于执行替换工作。

### 替换

Regex 算法家族中的第三个算法是 `regex_replace`. 顾名思义，它是用于执行文本替换的。它在整个输入数据中进行搜索，查找正则表达式的所有匹配。对于表达式的每一个匹配，该算法调用 `match_results::format` 并输入结果到一个传入函数的输出迭代器。

在本章的介绍部分，我给出了一个例子，将英式拼法的 colour 替换为美式拼法 color. 不使用正则表达式来进行这个拼写更改会非常乏味，也很容易出错。问题是可能存在不同的大小写，而且会有很多单词被影响，如 colourize. 要正确地解决这个问题，我们需要把正则表达式分为三个子表达式。

```cpp
boost::regex reg("(Colo)(u)(r)",
  boost::regex::icase|boost::regex::perl); 
```

我们将要去掉的字母 u 独立开，为了在所有匹配中可以很容易地删掉它。另外，注意到这个正则表达式是大小写无关的，我们要把格式标志 `boost::regex::icase` 传给 `regex` 的构造函数。你还要传递你想要设置的其它标志。设置标志时一个常见的错误就是忽略了`regex`缺省打开的那些标志，如果你没有设置这些标志，它们不会打开，你必须设置所有你要打开的标志。

调用 `regex_replace`时，我们要以参数方式提供一个格式化字符串。该格式化字符串决定如何进行替换。在这个格式化字符串中，你可以引用匹配的子表达式，这正是我们想要的。你想保留第一个和第三个匹配的子表达式，而去掉第二个`(u)`。表达式 `$N`表示匹配的子表达式, `N` 为子表达式索引。因此我们的格式化串应该是 `"$1$3"`, 表示替换文本为第一个和第三个子表达式。通过引用匹配的子表达式，我们可以保留匹配文本中的所有大小写，而如果我们用字符串来作替换文本则不能做到这一点。以下是解决这个问题的完整程序。

```cpp
#include <iostream>
#include <string>
#include "boost/regex.hpp"

int main() {
  boost::regex reg("(Colo)(u)(r)",
    boost::regex::icase|boost::regex::perl);

  std::string s="Colour, colours, color, colourize";

  s=boost::regex_replace(s,reg,"$1$3");
  std::cout << s;
} 
```

程序的输出是 `"Color, colors, color, colorize"`. `regex_replace` 对于这样的文本替换非常有用。

### 用户常见的误解

我所见到的与 Boost.Regex 相关的最常见的问题与`regex_match`的语义有关。人们很容易忘记必须使`regex_match`的所有输入匹配给定的正则表达式。因此，用户常以为以下代码会返回 `true`.

```cpp
boost::regex reg("\\d*");
bool b=boost::regex_match("17 is prime",reg); 
```

无疑这个调用永远不会得到成功的匹配。只有所有输入被 `regex_match` 匹配才可以返回 `true`！几乎所有的用户都会问为什么 `regex_search` 不是这样而 `regex_match` 是。

```cpp
boost::regex reg("\\d*");
bool b=boost::regex_search("17 is prime",reg); 
```

这次肯定返回 `true`. 值得注意的是，你可以用一些特定的缓冲操作符来让 `regex_search` 象 `regex_match` 那样运行。`\A` 匹配缓冲的起始点，而 `\Z` 匹配缓冲的结束点，因此如果你把 `\A` 放在正则表达式的开始，把 `\Z` 放在最后，你就可以让 `regex_search` 象 `regex_match` 那样使用，即必须匹配所有输入。以下正则表达式要求所有输入被匹配掉，而不管你使用的是 `regex_match` 或是 `regex_search`.

```cpp
boost::regex reg("\\A\\d*\\Z"); 
```

请记住，这并不表示可以无需使用 `regex_match`；相反，它可以清晰地表明我们刚才说到的语义，即必须匹配所有输入。

### 关于重复和贪婪

另一个容易混淆的地方是关于重复的贪婪。有些重复，如 `+` 和 `*`，是贪婪的。即是说，它们会消耗掉尽可能多的输入。以下正则表达式并不罕见，它用于在一个贪婪的重复后捕获两个数字。

```cpp
boost::regex reg("(.*)(\\d{2})"); 
```

这个正则表达式是对的，但它可能不能匹配你想要的子表达式！表达式 `.*` 会吞掉所有东西而后续的子表达式将不能匹配。以下是示范这个行为的一个例子：

```cpp
int main() {
  boost::regex reg("(.*)(\\d{2})");
  boost::cmatch m;
  const char* text = "Note that I'm 31 years old, not 32.";
  if(boost::regex_search(text,m, reg)) {
    if (m[1].matched)
      std::cout << "(.*) matched: " << m[1].str() << '\n';
    if (m[2].matched)
      std::cout << "Found the age: " << m[2] << '\n';
  }
} 
```

在这个程序中，我们使用了`match_results`的另一个特化版本，即类型 `cmatch`. 它就是 `match_results&lt;const char*&gt;` 的`typedef`, 之所以我们必须用它而不是用之前用过的 `smatch`，是因为我们现在是用一个字符序列调用 `regex_search` 而不是用类型 `std::string` 来调用。你期望这个程序的运行结果是什么？通常，一个刚开始使用正则表达式的用户会首先想到 `m[1].matched` 和 `m[2].matched` 都为 `true`, 且第二个子表达式的结果会是 "`31`". 接着在认识到贪婪的重复所带来的效果后，即重复会尽可能消耗输入，用户会想到只有第一个子表达式是 `true`，即 `.*` 成功地吞掉了所有的输入。最后，新用户得到了下结论，两个子表达式都被匹配，但第二个表达式匹配的是最后一个可能的序列。即第一个子表达式匹配的是 "`Note that I'm 31 years old, not`" 而第二个匹配 "`32`".

那么，如果你想使用重复并匹配另一个子表达式的第一次出现，该怎么办？要使用非贪婪的重复。在重复符后加一个 `?` ，重复就变为非贪婪的了。这意味着该表达式会尝试发现最短的匹配可能而不再阻止表达式的剩余部分进行匹配。因此，要让前面的正则表达式正确工作，我们需要把它改为这样。

```cpp
boost::regex reg("(.*?)(\\d{2})"); 
```

如果我们用这个正则表达式来修改程序，那么 `m[1].matched` 和 `m[2].matched` 都会为 `true`. 表达式 `.*?` 只消耗最少可能的输入，即它将在第一个字符 `3`处停止，因为这就是表达式要成功匹配所需要的。因此，第一个子表达式会匹配 "`Note that I'm`" 而第二个匹配 "`31`".

### 看一下 regex_iterator

我们已经看过如何用几次 `regex_search` 调用来处理所有输入，但另一方面，更为优雅的方法是使用 `regex_iterator`. 这个迭代器类型用一个序列来列举正则表达式的所有匹配。解引用一个 `regex_iterator` 会产生对一个 `match_results` 实例的引用。构造一个 `regex_iterator` 时，你要把指示输入序列的迭代器传给它，并提供相应的正则表达式。我们来看一个例子，输入数据是一组由逗号分隔的整数。相应的正则表达式很简单。

```cpp
boost::regex reg("(\\d+),?"); 
```

在正则表达式的最后加一个 `?` (匹配零次或一次) 确保最后一个数字可以被成功分析，即使输入序列不是以逗号结束。另外，我们还使用了另一个重复符 `+`. 这个重复符表示匹配一次或多次。现在，不需要多次调用 `regex_search`, 我们创建一个 `regex_iterator`, 并调用算法 `for_each`, 传给它一个函数对象，该函数对象以迭代器的解引用进行调用。下面是一个接受任意形式的`match_results`的函数对象，它有一个泛型的调用操作符。它所执行的就是把当前匹配的值加到一个总和中(在我们的正则表达式中，第一个子表达式是我们要用的)。

```cpp
class regex_callback {
  int sum_;
public:
  regex_callback() : sum_(0) {}

  template <typename T> void operator()(const T& what) {
    sum_+=atoi(what[1].str().c_str());
  }

  int sum() const {
    return sum_;
  }
}; 
```

现在把这个函数对象的一个实例传递给 `std::for_each`, 结果是对每一个迭代器`it`的解引用调用该函数对象，即对每一次匹配的子表达式进行调用。

```cpp
int main() {
  boost::regex reg("(\\d+),?");
  std::string s="1,1,2,3,5,8,13,21";

  boost::sregex_iterator it(s.begin(),s.end(),reg);
  boost::sregex_iterator end;

  regex_callback c;
  int sum=for_each(it,end,c).sum();
} 
```

如你所见，传递给`for_each`的 end 迭代器是 `regex_iterator` 一个缺省构造实例。`it` 和 `end` 的类型均为 `boost::sregex_iterator`, 即为 `regex_iterator&lt;std::string::const_iterator&gt;`的`typedef`. 这种使用 `regex_iterator` 的方法要比我们前面试过的多次匹配的方法更清晰，在多次匹配的方法中我们不得不在一个循环中让起始迭代器不断地前进并调用 `regex_search` 。

### 用 regex_token_iterator 分割字符串

另一个迭代器类型，或者说得更准确些，迭代器适配器，就是 `boost::regex_token_iterator`. 它与 `regex_iterator` 很类似，但却是用于列举不匹配某个正则表达式的每一个字符序列，这对于分割字符串很有用。它也可以用于选择对哪一个子表达式感兴趣，当解引用 `regex_token_iterator`时，只有预订的那个子表达式被返回。考虑这样一个应用程序，它接受一些用斜线号分隔的数据项作为输入。两个斜线号之间的数据组成应用程序要处理的项。使用 `regex_token_iterator`来分割这个字符串很容易。该正则表达式很简单。

```cpp
boost::regex reg("/"); 
```

这个 regex 匹配各项间的分割符。要用它来分割输入，只需简单地把指定的索引 `1` 传递给 `regex_token_iterator` 的构造函数。以下是完整的程序：

```cpp
int main() {
  boost::regex reg("/");
  std::string s="Split/Values/Separated/By/Slashes,";
  std::vector<std::string> vec;
  boost::sregex_token_iterator it(s.begin(),s.end(),reg,-1);
  boost::sregex_token_iterator end;
  while (it!=end) 
    vec.push_back(*it++);

  assert(vec.size()==std::count(s.begin(),s.end(),'/')+1);
  assert(vec[0]=="Split");
} 
```

就象 `regex_iterator` 一样，`regex_token_iterator` 是一个模板类，它使用所包装的序列的迭代器类型来进行特化。这里，我们用的是 `sregex_token_iterator`, 它是 `regex_token_iterator&lt;std::string::const_iterator&gt;` 的 `typedef` 。每一次解引用这个迭代器`it`，它返回当前的 `sub_match`, 当这个迭代器前进时，它尝试再次匹配该正则表达式。这两个迭代器类型，`regex_iterator` 和 `regex_token_iterator`, 都非常有用；你应该明白，当你考虑反复调用`regex_search`时，就该用它们了。

### 更多的正则表达式

你已经看到了不少正则表达式的语法，但还有更多的要了解。这一节简单地示范一些你每天都会使用的正则表达式的其它功能。作为开始，我们先看一下一组完整的重复符；我们之前已经看到了 `*`, `+`, 以及使用 `{}` 进行限定重复。还有一个重复符，即是 `?`. 你可能已经留意到它也可以用于声明非贪婪的重复，但对于它本身而言，它是表示一个表达式必须出现零次或一次。还有一点值得提及的是，限定重复符可以很灵活；下面是三种不同的用法：

```cpp
boost::regex reg1("\\d{5}");
boost::regex reg2("\\d{2,4}");
boost::regex reg3("\\d{2,}"); 
```

第一个正则表达式匹配 5 个数字。第二个匹配 2 个, 3 个, 或者 4 个数字。第三个匹配 2 个或更多个数字，没有上限。

另一种重要的正则表达式特性是使用元字符 `^` 表示非字符类别。用它来表示一个匹配任意不在给定字符类别中的字符；即你所列字符类别的补集。例如，看如下正则表达式。

```cpp
boost::regex reg("[¹³⁵⁷⁹]"); 
```

它包含一个非字符类别，匹配任意不是奇数数字的字符。看一下以下这个小程序，试着给出程序的输出。

```cpp
int main() {
  boost::regex reg4("[¹³⁵⁷⁹]");
  std::string s="0123456789";
  boost::sregex_iterator it(s.begin(),s.end(),reg4);
  boost::sregex_iterator end;

  while (it!=end) 
    std::cout << *it++;
} 
```

你给出答案了吗？输出是 "`02468`"，即所有偶数数字。注意，这个字符类别不仅匹配偶数数字，如果输入字符串是 "`AlfaBetaGamma`"，那么也会全部匹配。

我们看到的这个元字符, `^`, 还有另一个意思。它可以用来表示一行的开始。而元字符 `$` 则表示一行的结束。

### 错的正则表达式

一个错的正则表达式就是一个不遵守规则的正则表达式。例如，你可能忘了一个右括号，这样正则表达式引擎将无法成功编译这个正则表达式。这时，将抛出一个 `bad_expression` 类型的异常。正如我前面提到的，这个异常的名字将会在下一版本的 Boost.Regex 中被修改，还有在即将加入 Library Technical Report 的版本中也是。异常类型 `bad_expression` 将被更名为 `regex_error`.

如果你的应用程序中的正则表达式全都是硬编码的，你可能不用处理错误表达式，但如果你是接受了用户的 输入来作为正则表达式，你就必须准备进行错误处理。这里有一个程序，提示用户输入一个正则表达式，接着输入一个用来对正则表达式进行匹配的字符串。由用户 进行输入时，总是有可能会导致无效的输入。

```cpp
int main() {  
  std::cout << "Enter a regular expression:\n";
  std::string s;
  std::getline(std::cin, s);
  try {
    boost::regex reg(s);
    std::cout << "Enter a string to be matched:\n";

    std::getline(std::cin,s);

    if (boost::regex_match(s,reg))
      std::cout << "That's right!\n";
    else
      std::cout << "No, sorry, that doesn't match.\n";
  }
  catch(const boost::bad_expression& e) {
    std::cout << 
      "That's not a valid regular expression! (Error: " << 
      e.what() << ") Exiting...\n";
  }
} 
```

为了保护应用程序和用户，一个 `try`/`catch` 块用于处理构造时抛出 `boost::regex` 的情形，这时会打印一个提示信息，而程序会温和地退出。用这个程序来测试，我们开始时输入一些合理的数据。

```cpp
Enter a regular expression:
\d{5}
Enter a string to be matched:
12345
That's right! 
```

现在，给一些错误的数据，试着输入一个错误的正则表达式。

```cpp
Enter a regular expression:
(\w*))
That's not a valid regular expression! (Error: Unmatched ( or \() Exiting... 
```

在`regex reg`构造时，就会抛出一个异常，因为这个正则表达式不能被编译。因此，进入到 `catch` 的处理例程中，程序将打印一个错误信息并退出。你只需知道有三个可能会发生异常的地方。一个是在构造一个正则表达式时，就象你刚刚看到的那样；另一个是使用成员函数 `assign` 把正则表达式赋给一个 `regex` 时。最后一个是，regex 迭代器和算法也可能抛出异常，如果内存不够或者匹配的复杂度过快增长的话。

# Regex 总结

## Regex 总结

无可争议，正则表达式是非常有用和重要的，而本库给 C++带来了强大的正则表达式功能。传统上，用户 除了使用 POSIX C API 来实现正则表达式功能以外，别无选择。对于文本处理的验证工作，正则表达式比手工编写分析代码要灵活和可靠得多。对于查找和替换，使用正则表达式可 以优美地解决很多相关问题，而不用它们则根本无法解决。

Boost.Regex 是一个强大的库，因此不可能在这一章中完全覆盖它所有的内容。同样，正则表达 式的完美表现和广泛的应用范围意味着本章也不仅仅是简单地介绍一下它们。这个主题可以写成一本单独的书。要知道更多，可以学习 Boost.Regex 的在 线文档，并且找一本关于正则表达式的书(考虑一下参考书目中的建议)。不论 Boost.Regex 有多强大，正则表达式有多广多深，初学者还是可以有效地 使用本库中的正则表达式。对于那些由于 C++不支持正则表达式而选择了其它语言的程序员，欢迎你们回家。

Boost.Regex 并不是 C++程序员唯一可以使用的正则表达式库，但它的确是最好的一个。它易于使用，并且在匹配你的正则表达式时快如闪电。你应该尽可能去用它。

Boost.Regex 的作者是 Dr. John Maddock.