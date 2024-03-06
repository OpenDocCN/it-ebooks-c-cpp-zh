# 第五章 字符串处理

# 第五章 字符串处理

### 目录

*   5.1 前言
*   5.2 区域设置
*   5.3 字符串算法库 Boost.StringAlgorithms
*   5.4 正则表达式库 Boost.Regex
*   5.5 词汇分割器库 Boost.Tokenizer
*   5.6 格式化输出库 Boost.Format
*   5.7 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 5.1\. 前言

在标准 C++ 中，用于处理字符串的是 `std::string` 类，它提供很多字符串操作，包括查找指定字符或子串的函数。 尽管 `std::string` 囊括了百余函数，是标准 C++ 中最为臃肿的类之一，然而却并不能满足很多开发者在日常工作中的需要。 例如， Java 和 .Net 提供了可以将字符串转换到大写字母的函数，而 `std::string` 就没有相应的功能。 Boost C++ 库试图弥补这一缺憾。

## 5.2\. 区域设置

在进入正题之前，有必要先审视下区域设置的问题，本章中提到的很多函数都需要一个附加的区域设置参数。

区域设置在标准 C++ 中封装了文化习俗相关的内容，包括货币符号，日期时间格式， 分隔整数部分与分数部分的符号（基数符）以及多于三个数字时的分隔符（千位符）。

在字符串处理方面，区域设置和特定文化中对字符次序以及特殊字符的描述有关。 例如，字母表中是否含有变异元音字母以及其在字母表中的位置都由语言文化决定。

如果一个函数用于将字符串转换为大写形式，那么其实施步骤取决于具体的区域设置。 在德语中，字母 'ä' 显然要转换为 'Ä'， 然而在其他语言中并不一定。

使用类 `std::string` 时区域设置可以忽略， 因为它的函数均不依赖于特定语言。 然而在本章中为了使用 Boost C++ 库， 区域设置的知识是必不可少的。

C++ 标准中在 `locale` 文件中定义了类 `std::locale` 。 每个 C++ 程序自动拥有一个此类的实例， 即不能直接访问的全局区域设置。 如果要访问它，需要使用默认构造函数构造类 `std::locale` 的对象，并使用与全局区域设置相同的属性初始化。

```cpp
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale loc; 
  std::cout << loc.name() << std::endl; 
} 
```

*   下载源代码

以上程序在标准输出流输出 `C` ，这是基本区域设置的名称，它包括了 C 语言编写的程序中默认使用的描述。

这也是每个 C++ 应用的默认全局区域设置，它包括了美式文化中使用的描述。 例如，货币符号使用美元符号，基字符为英文句号，日期中的月份用英语书写。

全局区域设置可以使用类 `std::locale` 中的静态函数 `global()` 改变。

```cpp
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::locale loc; 
  std::cout << loc.name() << std::endl; 
} 
```

*   下载源代码

静态函数 `global()` 接受一个类型为 `std::locale` 的对象作为其唯一的参数。 此类的另一个版本的构造函数接受类型为 `const char*` 的字符串，可以为一个特别的文化创建区域设置对象。 然而，除了 C 区域设置相应地命名为 "C" 之外，其他区域设置的名字并没有标准化，所以这依赖于接受区域设置名字的 C++ 标准库。 在使用 Visual Studio 2008 的情况下，[语言字符串文档](http://msdn.microsoft.com/en-us/library/39cwe7zf.aspx) 指出， 可以使用语言字符串 "German" 选择定义为德国文化。

执行程序，会输出 `German_Germany.1252` 。 指定语言字符串为 "German" 等于选择了德国文化作为主要语言和子语言，这里选择了字符映射 1252。

如果想指定与德国文化不同的子语言设置，例如瑞士语，需要使用不同的语言字符串。

```cpp
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German_Switzerland")); 
  std::locale loc; 
  std::cout << loc.name() << std::endl; 
} 
```

*   下载源代码

现在程序会输出 `German_Switzerland.1252` 。

在初步理解了区域设置以及如何更改全局设置后， 下面的例子说明了区域设置如何影响字符串操作。

```cpp
#include <locale> 
#include <iostream> 
#include <cstring> 

int main() 
{ 
  std::cout << std::strcoll("ä", "z") << std::endl; 
  std::locale::global(std::locale("German")); 
  std::cout << std::strcoll("ä", "z") << std::endl; 
} 
```

*   下载源代码

本例使用了定义在文件 `cstring` 中的函数 `std::strcoll()` ，这个函数用于按照字典顺序比较第一个字符串是否小于第二个。 换言之，它会判断这两个字符串中哪一个在字典中靠前。

执行程序，得到结果为 `1` 和 `-1` 。 虽然函数的参数是一样的， 却得到了不同的结果。 原因很简单，在第一次调用函数 `std::strcoll()` 时，使用了全局 C 区域设置； 而在第二次调用时，全局区域设置更改为德国文化。 从输出中可以看出，在这两种区域设置中，字符 'ä' 和 'z' 的次序是不同的。

很多 C 函数以及 C++ 流都与区域设置有关。 尽管类 `std::string` 中的函数是与区域设置独立工作的， 但是以下各节中提到的函数并不是这样。 所以，在本章中还会多次提到区域设置的相关内容。

## 5.3\. 字符串算法库 Boost.StringAlgorithms

Boost C++ 字符串算法库 [Boost.StringAlgorithms](http://www.boost.org/doc/libs/1_36_0/doc/html/string_algo.html) 提供了很多字符串操作函数。 字符串的类型可以是 `std::string`， `std::wstring` 或任何其他模板类 `std::basic_string` 的实例。

这些函数分类别在不同的头文件定义。 例如，大小写转换函数定义在文件 `boost/algorithm/string/case_conv.hpp` 中。 因为 Boost.StringAlgorithms 类中包括超过 20 个类别和相同数目的头文件， 为了方便起见，头文件 `boost/algorithm/string.hpp` 包括了所有其他的头文件。 后面所有例子都会使用这个头文件。

正如上节提到的那样， Boost.StringAlgorithms 库中许多函数 都可以接受一个类型为 `std::locale` 的对象作为附加参数。 而此参数是可选的，如果不设置将使用默认全局区域设置。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 
#include <clocale> 

int main() 
{ 
  std::setlocale(LC_ALL, "German"); 
  std::string s = "Boris Schäling"; 
  std::cout << boost::algorithm::to_upper_copy(s) << std::endl; 
  std::cout << boost::algorithm::to_upper_copy(s, std::locale("German")) << std::endl; 
} 
```

*   下载源代码

函数 `boost::algorithm::to_upper_copy()` 用于 转换一个字符串为大写形式，自然也有提供相反功能的函数 —— `boost::algorithm::to_lower_copy()` 把字符串转换为小写形式。 这两个函数都返回转换过的字符串作为结果。 如果作为参数传入的字符串自身需要被转换为大（小）写形式，可以使用函数 `boost::algorithm::to_upper()` 或 `boost::algorithm::to_lower ()`。

上面的例子使用函数 `boost::algorithm::to_upper_copy()` 把字符串 "Boris Schäling" 转换为大写形式。 第一次调用时使用的是默认全局区域设置， 第二次调用时则明确将区域设置为德国文化。

显然后者的转换是正确的， 因为小写字母 'ä' 对应的大写形式 'Ä' 是存在的。 而在 C 区域设置中， 'ä' 是一个未知字符所以不能转换。 为了能得到正确结果，必须明确传递正确的区域设置参数或者在调用 `boost::algorithm::to_upper_copy()` 之前改变全局区域设置。

可以注意到，程序使用了定义在头文件 `clocale` 中的函数 `std::setlocale()` 为 C 函数进行区域设置， 因为 `std::cout` 使用 C 函数在屏幕上显示信息。 在设置了正确的区域后，才可以正确显示 'ä' 和 'Ä' 等元音字母。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  std::cout << boost::algorithm::to_upper_copy(s) << std::endl; 
  std::cout << boost::algorithm::to_upper_copy(s, std::locale("German")) << std::endl; 
} 
```

*   下载源代码

上述程序将全局区域设置设为德国文化，这使得对函数 `boost::algorithm::to_upper_copy()` 的调用 可以将 'ä' 转换为 'Ä' 。

注意到本例并没有调用函数 `std::setlocale()` 。 使用函数 `std::locale::global()` 设置全局区域设置后， 也自动进行了 C 区域设置。 实际上，C++ 程序几乎总是使用函数 `std::locale::global()` 进行全局区域设置， 而不是像前面的例子那样使用函数 `std::setlocale()` 。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  std::cout << boost::algorithm::erase_first_copy(s, "i") << std::endl; 
  std::cout << boost::algorithm::erase_nth_copy(s, "i", 0) << std::endl; 
  std::cout << boost::algorithm::erase_last_copy(s, "i") << std::endl; 
  std::cout << boost::algorithm::erase_all_copy(s, "i") << std::endl; 
  std::cout << boost::algorithm::erase_head_copy(s, 5) << std::endl; 
  std::cout << boost::algorithm::erase_tail_copy(s, 8) << std::endl; 
} 
```

*   下载源代码

Boost.StringAlgorithms 库提供了几个从字符串中删除单独字母的函数， 可以明确指定在哪里删除，如何删除。例如，可以使用函数 `boost::algorithm::erase_all_copy()` 从整个字符串中 删除特定的某个字符。 如果只在此字符首次出现时删除，可以使用函数 `boost::algorithm::erase_first_copy()` 。 如果要在字符串头部或尾部删除若干字符，可以使用函数 `boost::algorithm::erase_head_copy()` 和 `boost::algorithm::erase_tail_copy()` 。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::iterator_range<std::string::iterator> r = boost::algorithm::find_first(s, "Boris"); 
  std::cout << r << std::endl; 
  r = boost::algorithm::find_first(s, "xyz"); 
  std::cout << r << std::endl; 
} 
```

*   下载源代码

以下各个不同函数 `boost::algorithm::find_first()`、 `boost::algorithm::find_last()`、 `boost::algorithm::find_nth()`、 `boost::algorithm::find_head()` 以及 `boost::algorithm::find_tail()` 可以用于在字符串中查找子串。

所有这些函数的共同点是均返回类型为 `boost::iterator_range` 类 的一对迭代器。 此类起源于 Boost C++ 的 [Boost.Range](http://www.boost.org/libs/range/) 库， 它在迭代器的概念上定义了“范围”。 因为操作符 `&lt;&lt;` 由 `boost::iterator_range` 类重载而来， 单个搜索算法的结果可以直接写入标准输出流。 以上程序将 `Boris` 作为第一个结果输出而第二个结果为空字符串。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 
#include <vector> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::vector<std::string> v; 
  v.push_back("Boris"); 
  v.push_back("Schäling"); 
  std::cout << boost::algorithm::join(v, " ") << std::endl; 
} 
```

*   下载源代码

函数 `boost::algorithm::join()` 接受一个字符串的容器 作为第一个参数， 根据第二个参数将这些字符串连接起来。 相应地这个例子会输出 `Boris Schäling` 。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  std::cout << boost::algorithm::replace_first_copy(s, "B", "D") << std::endl; 
  std::cout << boost::algorithm::replace_nth_copy(s, "B", 0, "D") << std::endl; 
  std::cout << boost::algorithm::replace_last_copy(s, "B", "D") << std::endl; 
  std::cout << boost::algorithm::replace_all_copy(s, "B", "D") << std::endl; 
  std::cout << boost::algorithm::replace_head_copy(s, 5, "Doris") << std::endl; 
  std::cout << boost::algorithm::replace_tail_copy(s, 8, "Becker") << std::endl; 
} 
```

*   下载源代码

Boost.StringAlgorithms 库不但提供了查找子串或删除字母的函数， 而且提供了使用字符串替代子串的函数，包括 `boost::algorithm::replace_first_copy()`， `boost::algorithm::replace_nth_copy()`， `boost::algorithm::replace_last_copy()`， `boost::algorithm::replace_all_copy()`， `boost::algorithm::replace_head_copy()` 以及 `boost::algorithm::replace_tail_copy()` 等等。 它们的使用方法同查找和删除函数是差不多一样的，所不同的是还需要 一个替代字符串作为附加参数。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "\t Boris Schäling \t"; 
  std::cout << "." << boost::algorithm::trim_left_copy(s) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_right_copy(s) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_copy(s) << "." << std::endl; 
} 
```

*   下载源代码

可以使用修剪函数 `boost::algorithm::trim_left_copy()`， `boost::algorithm::trim_right_copy()` 以及 `boost::algorithm::trim_copy()` 等自动去除字符串中的空格或者字符串的结束符。 什么字符是空格取决于全局区域设置。

Boost.StringAlgorithms 库的函数可以接受一个附加的谓词参数，以决定函数作用于字符串的哪些字符。 谓词版本的修剪函数相应地被命名为 `boost::algorithm::trim_left_copy_if()`， `boost::algorithm::trim_right_copy_if()` 和 `boost::algorithm::trim_copy_if()` 。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "--Boris Schäling--"; 
  std::cout << "." << boost::algorithm::trim_left_copy_if(s, boost::algorithm::is_any_of("-")) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_right_copy_if(s, boost::algorithm::is_any_of("-")) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_copy_if(s, boost::algorithm::is_any_of("-")) << "." << std::endl; 
} 
```

*   下载源代码

以上程序调用了一个辅助函数 `boost::algorithm::is_any_of()` ， 它用于生成谓词以验证作为参数传入的字符是否在给定的字符串中存在。使用函数 `boost::algorithm::is_any_of 后，正如例子中做的那样，修剪字符串的字符被指定为连字符。 ()`

Boost.StringAlgorithms 类也提供了众多返回通用谓词的辅助函数。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "123456789Boris Schäling123456789"; 
  std::cout << "." << boost::algorithm::trim_left_copy_if(s, boost::algorithm::is_digit()) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_right_copy_if(s, boost::algorithm::is_digit()) << "." << std::endl; 
  std::cout << "." <<boost::algorithm::trim_copy_if(s, boost::algorithm::is_digit()) << "." << std::endl; 
} 
```

*   下载源代码

函数 `boost::algorithm::is_digit()` 返回的谓词在字符为数字时返回布尔值 `true`。 检查字符是否为大写或小写的辅助函数分别是 `boost::algorithm::is_upper()` 和 `boost::algorithm::is_lower()` 。 所有这些函数都默认使用全局区域设置，除非在参数中指定其他区域设置。

除了检验单独字符的谓词之外， Boost.StringAlgorithms 库还提供了处理字符串的函数。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  std::cout << boost::algorithm::starts_with(s, "Boris") << std::endl; 
  std::cout << boost::algorithm::ends_with(s, "Schäling") << std::endl; 
  std::cout << boost::algorithm::contains(s, "is") << std::endl; 
  std::cout << boost::algorithm::lexicographical_compare(s, "Boris") << std::endl; 
} 
```

*   下载源代码

函数 `boost::algorithm::starts_with()`、 `boost::algorithm::ends_with()`、 `boost::algorithm::contains()` 和 `boost::algorithm::lexicographical_compare()` 均可以比较两个字符串。

以下介绍一个字符串切割函数。

```cpp
#include <boost/algorithm/string.hpp> 
#include <locale> 
#include <iostream> 
#include <vector> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  std::vector<std::string> v; 
  boost::algorithm::split(v, s, boost::algorithm::is_space()); 
  std::cout << v.size() << std::endl; 
} 
```

*   下载源代码

在给定分界符后，使用函数 `boost::algorithm::split()` 可以将一个字符串拆分为一个字符串容器。 它需要给定一个谓词作为第三个参数以判断应该在字符串的哪个位置分割。 这个例子使用了辅助函数 `boost::algorithm::is_space()` 创建一个谓词，在每个空格字符处分割字符串。

本节中许多函数都有忽略字符串大小写的版本， 这些版本一般都有与原函数相似的名称，所相差的只是以 'i'.开头。 例如，与函数 `boost::algorithm::erase_all_copy()` 相对应的是函数 `boost::algorithm::ierase_all_copy()`。

最后，值得注意的是类 Boost.StringAlgorithms 中许多函数都支持正则表达式。 以下程序使用函数 `boost::algorithm::find_regex()` 搜索正则表达式。

```cpp
#include <boost/algorithm/string.hpp> 
#include <boost/algorithm/string/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::iterator_range<std::string::iterator> r = boost::algorithm::find_regex(s, boost::regex("\\w\\s\\w")); 
  std::cout << r << std::endl; 
} 
```

*   下载源代码

为了使用正则表达式，此程序使用了 Boost C++ 库中的 `boost::regex` ， 这将在下一节介绍。

## 5.4\. 正则表达式库 Boost.Regex

Boost C++ 的正则表达式库 [Boost.Regex](http://www.boost.org/libs/regex/) 可以应用正则表达式于 C++ 。 正则表达式大大减轻了搜索特定模式字符串的负担，在很多语言中都是强大的功能。 虽然现在 C++ 仍然需要以 Boost C++ 库的形式提供这一功能，但是在将来正则表达式将进入 C++ 标准库。 Boost.Regex 库有望包括在下一版的 C++ 标准中。

Boost.Regex 库中两个最重要的类是 `boost::regex` 和 `boost::smatch`， 它们都在 `boost/regex.hpp` 文件中定义。 前者用于定义一个正则表达式，而后者可以保存搜索结果。

以下将要介绍 Boost.Regex 库中提供的三个搜索正则表达式的函数。

```cpp
#include <boost/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::regex expr("\\w+\\s\\w+"); 
  std::cout << boost::regex_match(s, expr) << std::endl; 
} 
```

*   下载源代码

函数 `boost::regex_match()` 用于字符串与正则表达式的比较。 在整个字符串匹配正则表达式时其返回值为 `true` 。

函数 `boost::regex_search()` 可用于在字符串中搜索正则表达式。

```cpp
#include <boost/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::regex expr("(\\w+)\\s(\\w+)"); 
  boost::smatch what; 
  if (boost::regex_search(s, what, expr)) 
  { 
    std::cout << what[0] << std::endl; 
    std::cout << what[1] << " " << what[2] << std::endl; 
  } 
} 
```

*   下载源代码

函数 `boost::regex_search()` 可以接受一个类型为 `boost::smatch` 的引用的参数用于储存结果。 函数 `boost::regex_search()` 只用于分类的搜索， 本例实际上返回了两个结果， 它们是基于正则表达式的分组。

存储结果的类 `boost::smatch` 事实上是持有类型为 `boost::sub_match` 的元素的容器， 可以通过与类 `std::vector` 相似的界面访问。 例如， 元素可以通过操作符 `operator[]()` 访问。

另一方面，类 `boost::sub_match` 将迭代器保存在对应于正则表达式分组的位置。 因为它继承自类 `std::pair` ，迭代器引用的子串可以使用 `first` 和 `second` 访问。如果像上面的例子那样，只把子串写入标准输出流， 那么通过重载操作符 `&lt;&lt;` 就可以直接做到这一点，那么并不需要访问迭代器。

请注意结果保存在迭代器中而 `boost::sub_match` 类并不复制它们， 这说明它们只是在被迭代器引用的相关字符串存在时才可以访问。

另外，还需要注意容器 `boost::smatch` 的第一个元素存储的引用是指向匹配正则表达式的整个字符串的，匹配第一组的第一个子串由索引 1 访问。

Boost.Regex 提供的第三个函数是 `boost::regex_replace()`。

```cpp
#include <boost/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = " Boris Schäling "; 
  boost::regex expr("\\s"); 
  std::string fmt("_"); 
  std::cout << boost::regex_replace(s, expr, fmt) << std::endl; 
} 
```

*   下载源代码

除了待搜索的字符串和正则表达式之外， `boost::regex_replace()` 函数还需要一个格式参数，它决定了子串、匹配正则表达式的分组如何被替换。如果正则表达式不包含任何分组，相关子串将被用给定的格式一个个地被替换。这样上面程序输出的结果为 `_Boris_Schäling_` 。

`boost::regex_replace()` 函数总是在整个字符串中搜索正则表达式，所以这个程序实际上将三处空格都替换为下划线。

```cpp
#include <boost/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::regex expr("(\\w+)\\s(\\w+)"); 
  std::string fmt("\\2 \\1"); 
  std::cout << boost::regex_replace(s, expr, fmt) << std::endl; 
} 
```

*   下载源代码

格式参数可以访问由正则表达式分组的子串，这个例子正是使用了这项技术，交换了姓、名的位置，于是结果显示为 `Schäling Boris` 。

需要注意的是，对于正则表达式和格式有不同的标准。 这三个函数都可以接受一个额外的参数，用于选择具体的标准。 也可以指定是否以某一具体格式解释特殊字符或者替代匹配正则表达式的整个字符串。

```cpp
#include <boost/regex.hpp> 
#include <locale> 
#include <iostream> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string s = "Boris Schäling"; 
  boost::regex expr("(\\w+)\\s(\\w+)"); 
  std::string fmt("\\2 \\1"); 
  std::cout << boost::regex_replace(s, expr, fmt, boost::regex_constants::format_literal) << std::endl; 
} 
```

*   下载源代码

此程序将 `boost::regex_constants::format_literal` 标志作为第四参数传递给函数 `boost::regex_replace()` ，从而抑制了格式参数中对特殊字符的处理。 因为整个字符串匹配正则表达式，所以本例中经格式参数替换的到达的输出结果为 `\2 \1`。

正如上一节末指出的那样，正则表达式可以和 Boost.StringAlgorithms 库结合使用。它通过 Boost.Regex 库提供函数如 `boost::algorithm::find_regex()` 、 `boost::algorithm::replace_regex()` 、 `boost::algorithm::erase_regex()` 以及 `boost::algorithm::split_regex()` 等等。由于 Boost.Regex 库很有可能成为即将到来的下一版 C++ 标准的一部分，脱离 Boost.StringAlgorithms 库，熟练地使用正则表达式是个明智的选择。

## 5.5\. 词汇分割器库 Boost.Tokenizer

[Boost.Tokenizer](http://www.boost.org/libs/tokenizer/) 库可以在指定某个字符为分隔符后，遍历字符串的部分表达式。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::char_separator<char> > tokenizer; 
  std::string s = "Boost C++ libraries"; 
  tokenizer tok(s); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

Boost.Tokenizer 库在 `boost/tokenizer.hpp` 文件中定义了模板类 `boost::tokenizer` ，其模板参数为支持相关表达式的类。 上面的例子中就使用了 `boost::char_separator` 类作为模板参数，它将空格和标点符号视为分隔符。

词汇分割器必须由类型为 `std::string` 的字符串初始化。通过使用 `begin()` 和 `end()` 方法，词汇分割器可以像容器一样访问。通过使用迭代器，可以得到前述字符串的部分表达式。模板参数的类型决定了如何达到部分表达式。

因为 `boost::char_separator` 类默认将空格和标点符号视为分隔符，所以本例显示的结果为 `Boost` 、 `C` 、 `+` 、 `+` 和 `libraries` 。 为了识别这些分隔符， `boost::char_separator` 函数调用了 `std::isspace()` 函数和 `std::ispunct 函数。 ()`Boost.Tokenizer 库会区分要隐藏的分隔符和要显示的分隔符。 在默认的情况下，空格会隐藏而标点符号会显示出来，所以这个例子里显示了两个加号。

如果不需要将标点符号作为分隔符，可以在传递给词汇分割器之前相应地初始化 `boost::char_separator` 对象。 以下例子正式这样做的。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::char_separator<char> > tokenizer; 
  std::string s = "Boost C++ libraries"; 
  boost::char_separator<char> sep(" "); 
  tokenizer tok(s, sep); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

类 `boost::char_separator` 的构造函数可以接受三个参数， 只有第一个是必须的，它描述了需要隐藏的分隔符。 在本例中， 空格仍然被视为分隔符。

第二个参数指定了需要显示的分隔符。 在不提供此参数的情况下，将不显示任何分隔符。 执行程序，会显示 `Boost` 、 `C++` 和 `libraries` 。

如果将加号作为第二个参数，此例的结果将和上一个例子相同。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::char_separator<char> > tokenizer; 
  std::string s = "Boost C++ libraries"; 
  boost::char_separator<char> sep(" ", "+"); 
  tokenizer tok(s, sep); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

第三个参数决定了是否显示空的部分表达式。 如果连续找到两个分隔符，他们之间的部分表达式将为空。在默认情况下，这些空表达式是不会显示的。第三个参数可以改变默认的行为。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::char_separator<char> > tokenizer; 
  std::string s = "Boost C++ libraries"; 
  boost::char_separator<char> sep(" ", "+", boost::keep_empty_tokens); 
  tokenizer tok(s, sep); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

执行以上程序，会显示另外两个的空表达式。 其中第一个是在两个加号中间的而第二个是加号和之后的空格之间的。

词汇分割器也可用于不同的字符串类型。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::char_separator<wchar_t>, std::wstring::const_iterator, std::wstring> tokenizer; 
  std::wstring s = L"Boost C++ libraries"; 
  boost::char_separator<wchar_t> sep(L" "); 
  tokenizer tok(s, sep); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::wcout << *it << std::endl; 
} 
```

*   下载源代码

这个例子遍历了一个类型为 `std::wstring` 的字符串。 为了使用这个类型的字符串，必须使用另外的模板参数初始化词汇分割器，对 `boost::char_separator` 类也是如此，他们都需要参数 `wchar_t` 初始化。

除了 `boost::char_separator` 类之外， Boost.Tokenizer 还提供了另外两个类以识别部分表达式。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::escaped_list_separator<char> > tokenizer; 
  std::string s = "Boost,\"C++ libraries\""; 
  tokenizer tok(s); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

`boost::escaped_list_separator` 类用于读取由逗号分隔的多个值，这个格式的文件通常称为 CSV （comma separated values，逗号分隔文件），它甚至还可以处理双引号以及转义序列。所以本例的输出为 `Boost` 和 `C++ libraries` 。

另一个是 `boost::offset_separator` 类，必须用实例说明。 这个类的对象必须作为第二个参数传递给 `boost::tokenizer` 类的构造函数。

```cpp
#include <boost/tokenizer.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tokenizer<boost::offset_separator> tokenizer; 
  std::string s = "Boost C++ libraries"; 
  int offsets[] = { 5, 5, 9 }; 
  boost::offset_separator sep(offsets, offsets + 3); 
  tokenizer tok(s, sep); 
  for (tokenizer::iterator it = tok.begin(); it != tok.end(); ++it) 
    std::cout << *it << std::endl; 
} 
```

*   下载源代码

`boost::offset_separator` 指定了部分表达式应当在字符串中的哪个位置结束。 以上程序制定第一个部分表达式在 5 个字符后结束，第二个字符串在另 5 个字符后结束，第三个也就是最后一个字符串应当在之后的 9 个字符后结束。 输出的结果为 `Boost` 、 `C++` 和 `libraries` 。

## 5.6\. 格式化输出库 Boost.Format

[Boost.Format](http://www.boost.org/libs/format/) 库可以作为定义在文件 `cstdio` 中的函数 `std::printf()` 的替代。 `std::printf()` 函数最初出现在 C 标准中，提供格式化数据输出功能， 但是它既不是类型安全的有不能扩展。 因此在 C++ 应用中， Boost.Format 库通常是数据格式化输出的上佳之选。

Boost.Format 库在文件 `boost/format.hpp` 中定义了类 `boost::format` 。 与函数 `std::printf 相似的是，传递给()` `boost::format` 的构造函数的参数也是一个字符串，它由控制格式的特殊字符组成。 实际数据通过操作符 % 相连，在输出中替代特殊字符，如下例所示。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%1%.%2%.%3%") % 16 % 9 % 2008 << std::endl; 
} 
```

*   下载源代码

Boost.Format 类使用置于两个百分号之间的数字作为占位符，占位符稍后通过 % 操作符与实际数据连接。 以上程序使用数字 16、9 和 2009 组成一个日期字符串，以 `16.9.2008`的格式输出。 如果要月份出现在日期之前，即美式表示，只需交换占位符即可。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%2%/%1%/%3%") % 16 % 9 % 2008 << std::endl; 
} 
```

*   下载源代码

现在程序显示的结果变成 `9/16/2008` 。

如果要使用 C++ 操作器格式化数据，Boost.Format 库提供了函数 `boost::io::group()` 。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%1% %2% %1%") % boost::io::group(std::showpos, 99) % 100 << std::endl; 
} 
```

*   下载源代码

本例的结果显示为 `+99 100 +99` 。 因为操作器 `std::showpos()` 通过 `boost::io::group()` 与数字 99 连接，所以只要显示 99 ， 在它前面就会自动加上加号。

如果需要加号仅在 99 第一次输出时显示， 则需要改造格式化占位符。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%|1$+| %2% %1%") % 99 % 100 << std::endl; 
} 
```

为了将输出格式改为 `+99 100 99` ，不但需要将数据的引用符号由 1$ 变为 1% ，还需要在其两侧各添加一个附加的管道符号，即将占位符 %1% 替换为 %|1$+|。

请注意，虽然一般对数据的引用不是必须的，但是所有占位符一定要同时设置为指定货非指定。 以下例子在执行时会出现错误，因为它给第二个和第三个占位符设置了引用但是却忽略了第一个。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    std::cout << boost::format("%|+| %2% %1%") % 99 % 100 << std::endl; 
  } 
  catch (boost::io::format_error &ex) 
  { 
    std::cout << ex.what() << std::endl; 
  } 
} 
```

*   下载源代码

此程序抛出了类型为 `boost::io::format_error` 的异常。 严格地说，Boost.Format 库抛出的异常为 `boost::io::bad_format_string`。 但是由于所有的异常类都继承自 `boost::io::format_error` 类，捕捉此类型的异常会轻松一些。

以下例子演示了不引用数据的方法。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%|+| %|| %||") % 99 % 100 % 99 << std::endl; 
} 
```

*   下载源代码

第二、第三个占位符的管道符号可以被安全地省略，因为在这种情况下，他们并不指定格式。这样的语法看起来很像 `std::printf ()`的那种。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%+d %d %d") % 99 % 100 % 99 << std::endl; 
} 
```

*   下载源代码

虽然这看起来就像 `std::printf()` ，但是 Boost.Format 库有类型安全的优点。 格式化字符串中字母 'd' 的使用并不表示输出数字，而是表示 `boost::format` 类所使用的内部流对象上的 `std::dec()` 操作器，它可以使用某些对 `std::printf()` 函数无意义的格式字符串，如果使用 `std::printf()` 会导致程序在运行时崩溃。

```cpp
#include <boost/format.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::format("%+s %s %s") % 99 % 100 % 99 << std::endl; 
} 
```

*   下载源代码

尽管在 `std::printf()` 函数中，字母 's' 只用于表示类型为 `const char*` 的字符串，然而以上程序却能正常运行。 因为在 Boost.Format 库中，这并不代表强制为字符串，它会结合适当的操作器，调整内部流的操作模式。 所以即使在这种情况下， 在内部流中加入数字也是没问题的。

## 5.7\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  编写程序，从以下 XML 流中提取并显示数据，包括姓名、生日以及账户余额。 **`&lt;person&gt;&lt;name&gt;Karl-Heinz Huber&lt;/name&gt;&lt;dob&gt;1970-9-30&lt;/dob&gt;&lt;account&gt;2,900.64 USD&lt;/account&gt;&lt;/person&gt;`**。

    姓、名要分开显示，生日使用 “日.月.年” 的格式，账户余额忽略小数位。 使用其他 XML 流测试你的程序，如包含多余空白、其他名字、账户余额为负数等等的 XML 流。

2.  编写程序，使得格式与显示的数据记录如下：输入 **`Munich Hamburg 92.12 8:25 9:45`**， 这条记录表示从 Munich 到 Hamburg 的航班票价为 92.12 欧元，上午 8:25 起飞 9:45 到达目的地。要得到以下输出 `Munich -&gt; Hamburg 92.12 EUR (08:25-09:45)`。

    具体地说，城市名称长度为 10 并且左对齐而票价长度为 7 并且右对齐，货币在价格后显示。 起飞与降落时间一起显示在圆括号中，以连字符分隔，不留空格。对早于 10 点（上午或下午）的时间，必须在前面补 0。 用不同的数据记录测试你的程序，例如使用长度大于 10 的城市名。