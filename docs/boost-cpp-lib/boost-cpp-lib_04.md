# 第三章 函数对象

### 目录

*   3.1 概述
*   3.2 Boost.Bind
*   3.3 Boost.Ref
*   3.4 Boost.Function
*   3.5 Boost.Lambda
*   3.6 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 3.1\. 概述

本章介绍的是函数对象，可能称为'高阶函数'更为适合。 它实际上是指那些可以被传入到其它函数或是从其它函数返回的一类函数。 在 C++中高阶函数是被实现为函数对象的，所以这个标题还是有意义的。

在这整一章中，将会介绍几个用于处理函数对象的 Boost C++ 库。 其中，[Boost.Bind](http://www.boost.org/libs/bind/) 可替换来自 C++标准的著名的 `std::bind1st()` 和 `std::bind2nd()` 函数，而 [Boost.Function](http://www.boost.org/libs/function/) 则提供了一个用于封装函数指针的类。 最后，[Boost.Lambda](http://www.boost.org/libs/lambda/) 则引入了一种创建匿名函数的方法。

## 3.2\. Boost.Bind

Boost.Bind 是这样的一个库，它简化了由 C++标准中的 `std::bind1st()` 和 `std::bind2nd()` 模板函数所提供的一个机制：将这些函数与几乎不限数量的参数一起使用，就可以得到指定签名的函数。 这种情形的一个最好的例子就是在 C++标准中定义的多个不同算法。

```cpp
#include <iostream> 
#include <vector> 
#include <algorithm> 

void print(int i) 
{ 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), print); 
} 
```

*   下载源代码

算法 `std::for_each()` 要求它的第三个参数是一个仅接受正好一个参数的函数或函数对象。 如果 `std::for_each()` 被执行，指定容器中的所有元素 - 在上例中，这些元素的类型为 `int` - 将按顺序被传入至 `print()` 函数。 但是，如果要使用一个具有不同签名的函数的话，事情就复杂了。 例如，如果要传入的是以下函数 `add()`，它要将一个常数值加至容器中的每个元素上，并显示结果。

```cpp
void add(int i, int j) 
{ 
  std::cout << i + j << std::endl; 
} 
```

由于 `std::for_each()` 要求的是仅接受一个参数的函数，所以不能直接传入 `add()` 函数。 源代码必须要修改。

```cpp
#include <iostream> 
#include <vector> 
#include <algorithm> 
#include <functional> 

class add 
  : public std::binary_function<int, int, void> 
{ 
public: 
  void operator()(int i, int j) const 
  { 
    std::cout << i + j << std::endl; 
  } 
}; 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), std::bind1st(add(), 10)); 
} 
```

*   下载源代码

以上程序将值 10 加至容器 `v` 的每个元素之上，并使用标准输出流显示结果。 源代码必须作出大幅的修改，以实现此功能：`add()` 函数已被转换为一个派生自 `std::binary_function` 的函数对象。

Boost.Bind 简化了不同函数之间的绑定。 它只包含一个 `boost::bind()` 模板函数，定义于 `boost/bind.hpp` 中。 使用这个函数，可以如下实现以上例子：

```cpp
#include <boost/bind.hpp> 
#include <iostream> 
#include <vector> 
#include <algorithm> 

void add(int i, int j) 
{ 
  std::cout << i + j << std::endl; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), boost::bind(add, 10, _1)); 
} 
```

*   下载源代码

象 `add()` 这样的函数不再需要为了要用于 `std::for_each()` 而转换为函数对象。 使用 `boost::bind()`，这个函数可以忽略其第一个参数而使用。

因为 `add()` 函数要求两个参数，两个参数都必须传递给 `boost::bind()`。 第一个参数是常数值 10，而第二个参数则是一个怪异的 `_1`。

`_1` 被称为占位符(placeholder)，定义于 Boost.Bind。 除了 `_1`，Boost.Bind 还定义了 `_2` 和 `_3`。 通过使用这些占位符，`boost::bind()` 可以变为一元、二元或三元的函数。 对于 `_1`, `boost::bind()` 变成了一个一元函数 - 即只要求一个参数的函数。 这是必需的，因为 `std::for_each()` 正是要求一个一元函数作为其第三个参数。

当这个程序执行时，`std::for_each()` 对容器 `v` 中的第一个元素调用该一元函数。 元素的值通过占位符 `_1` 传入到一元函数中。 这个占位符和常数值被进一步传递到 `add()` 函数。 通过使用这种机制，`std::for_each()` 只看到了由 `boost::bind()` 所定义的一元函数。 而 `boost::bind()` 本身则只是调用了另一个函数，并将常数值或占位符作为参数传入给它。

下面这个例子通过 `boost::bind()` 定义了一个二元函数，用于 `std::sort()` 算法，该算法要求一个二元函数作为其第三个参数。

```cpp
#include <boost/bind.hpp> 
#include <vector> 
#include <algorithm> 

bool compare(int i, int j) 
{ 
  return i > j; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::sort(v.begin(), v.end(), boost::bind(compare, _1, _2)); 
} 
```

*   下载源代码

因为使用了两个占位符 `_1` 和 `_2`，所以 `boost::bind()` 定义了一个二元函数。 `std::sort()` 算法以容器 `v` 的两个元素来调用该函数，并根据返回值来对容器进行排序。 基于 `compare()` 函数的定义，容器将被按降序排列。

但是，由于 `compare()` 本身就是一个二元函数，所以使用 `boost::bind()` 确是多余的。

```cpp
#include <boost/bind.hpp> 
#include <vector> 
#include <algorithm> 

bool compare(int i, int j) 
{ 
  return i > j; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::sort(v.begin(), v.end(), compare); 
} 
```

*   下载源代码

不过使用 `boost::bind()` 还是有意义的。例如，如果容器要按升序排列而又不能修改 `compare()` 函数的定义。

```cpp
#include <boost/bind.hpp> 
#include <vector> 
#include <algorithm> 

bool compare(int i, int j) 
{ 
  return i > j; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::sort(v.begin(), v.end(), boost::bind(compare, _2, _1)); 
} 
```

*   下载源代码

该例子仅改变了占位符的顺序：`_2` 被作为第一参数传递，而 `_1` 则被作为第二参数传递至 `compare()`，这样即可改变排序的顺序。

## 3.3\. Boost.Ref

本库 [Boost.Ref](http://www.boost.org/doc/html/ref.html) 通常与 Boost.Bind 一起使用，所以我把它们挨着写。 它提供了两个函数 - `boost::ref()` 和 `boost::cref()` - 定义于 `boost/ref.hpp`.

当要用于 `boost::bind()` 的函数带有至少一个引用参数时，Boost.Ref 就很重要了。 由于 `boost::bind()` 会复制它的参数，所以引用必须特别处理。

```cpp
#include <boost/bind.hpp> 
#include <iostream> 
#include <vector> 
#include <algorithm> 

void add(int i, int j, std::ostream &os) 
{ 
  os << i + j << std::endl; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), boost::bind(add, 10, _1, boost::ref(std::cout))); 
} 
```

*   下载源代码

以上例子使用了上一节中的 `add()` 函数。 不过这一次该函数需要一个流对象的引用来打印信息。 因为传给 `boost::bind()` 的参数是以值方式传递的，所以 `std::cout` 不能直接使用，否则该函数会试图创建它的一份拷贝。

通过使用模板函数 `boost::ref()`，象 `std::cout` 这样的流就可以被以引用方式传递，也就可以成功编译上面这个例子了。

要以引用方式传递常量对象，可以使用模板函数 `boost::cref()`。

## 3.4\. Boost.Function

为了封装函数指针，[Boost.Function](http://www.boost.org/libs/function/) 提供了一个名为 `boost::function` 的类。 它定义于 `boost/function.hpp`，用法如下：

```cpp
#include <boost/function.hpp> 
#include <iostream> 
#include <cstdlib> 
#include <cstring> 

int main() 
{ 
  boost::function<int (const char*)> f = std::atoi; 
  std::cout << f("1609") << std::endl; 
  f = std::strlen; 
  std::cout << f("1609") << std::endl; 
} 
```

*   下载源代码

`boost::function` 可以定义一个指针，指向具有特定签名的函数。 以上例子定义了一个指针 `f`，它可以指向某个接受一个类型为 `const char*` 的参数且返回一个类型为 `int` 的值的函数。 定义完成后，匹配此签名的函数均可赋值给这个指针。 这个例程就是先将 `std::atoi()` 赋值给 `f`，然后再将它重赋值为 `std::strlen()`。

注意，给定的数据类型并不需要精确匹配：虽然 `std::strlen()` 是以 `std::size_t` 作为返回类型的，但是它也可以被赋值给 `f`。

因为 `f` 是一个函数指针，所以被赋值的函数可以通过重载的 `operator()()` 操作符来调用。 取决于当前被赋值的是哪一个函数，在以上例子中将调用 `std::atoi()` 或 `std::strlen()`。

如果 `f` 未赋予一个函数而被调用，则会抛出一个 `boost::bad_function_call` 异常。

```cpp
#include <boost/function.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    boost::function<int (const char*)> f; 
    f(""); 
  } 
  catch (boost::bad_function_call &ex) 
  { 
    std::cout << ex.what() << std::endl; 
  } 
} 
```

*   下载源代码

注意，将值 0 赋给一个 `boost::function` 类型的函数指针，将会释放当前所赋的函数。 释放之后再调用它也会导致 `boost::bad_function_call` 异常被抛出。 要检查一个函数指针是否被赋值某个函数，可以使用 `empty()` 函数或 `operator bool()` 操作符。

通过使用 Boost.Function，类成员函数也可以被赋值给类型为 `boost::function` 的对象。

```cpp
#include <boost/function.hpp> 
#include <iostream> 

struct world 
{ 
  void hello(std::ostream &os) 
  { 
    os << "Hello, world!" << std::endl; 
  } 
}; 

int main() 
{ 
  boost::function<void (world*, std::ostream&)> f = &world::hello; 
  world w; 
  f(&w, boost::ref(std::cout)); 
} 
```

*   下载源代码

在调用这样的一个函数时，传入的第一个参数表示了该函数被调用的那个特定对象。 因此，在模板定义中的左括号后的第一个参数必须是该特定类的指针。 接下来的参数才是表示相应的成员函数的签名。

这个程序还使用了来自 Boost.Ref 库的 `boost::ref()`，它提供了一个方便的机制向 Boost.Function 传递引用。

## 3.5\. Boost.Lambda

匿名函数 - 又称为 lambda 函数 - 已经在多种编程语言中存在，但 C++ 除外。 不过在 [Boost.Lambda](http://www.boost.org/libs/lambda/) 库的帮助下，现在在 C++ 应用中也可以使用它们了。

lambda 函数的目标是令源代码更为紧凑，从而也更容易理解。 以本章第一节中的代码例子为例。

```cpp
#include <iostream> 
#include <vector> 
#include <algorithm> 

void print(int i) 
{ 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), print); 
} 
```

*   下载源代码

这段程序接受容器 `v` 中的元素并使用 `print()` 函数将它们写出到标准输出流。 由于 `print()` 只是写出一个简单的 `int`，所以该函数的实现相当简单。 严格来说，它是如此地简单，以致于如果可以在 `std::for_each()` 算法里面直接定义它的话，会更为方便； 从而省去增加一个函数的需要。 另外一个好处是代码更为紧凑，使得算法与负责数据输出的函数不是局部性分离的。 Boost.Lambda 正好使之成为现实。

```cpp
#include <boost/lambda/lambda.hpp> 
#include <iostream> 
#include <vector> 
#include <algorithm> 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), std::cout << boost::lambda::_1 << "\n"); 
} 
```

*   下载源代码

Boost.Lambda 提供了几个结构来定义匿名函数。 代码就被置于执行的地方，从而省去将它包装为一个函数再进行相应的函数调用的这些开销。 与原来的例子一样，这个程序将容器 `v` 的所有元素写出至标准输出流。

与 Boost.Bind 相类似，Boost.Lambda 也定义了三个占位符，名为 `_1`, `_2` 和 `_3`。 但与 Boost.Bind 不同的是，这些占位符是定义在单独的名字空间的。 因此，该例中的第一个占位符是通过 `boost::lambda::_1` 来引用的。 为了满足编译器的要求，必须包含相应的头文件 `boost/lambda/lambda.hpp`。

虽然代码的位置位于 `std::for_each()` 的第三个参数处，看起来很怪异，但 Boost.Lambda 可以写出正常的 C++ 代码。 通过使用占位符，容器 `v` 的元素可以通过 `&lt;&lt;` 传给 `std::cout` 以将它们写出到标准输出流。

虽然 Boost.Lambda 非常强大，但也有一些缺点。 要在以上例子中插入换行的话，必须用 "\n" 来替代 `std::endl` 才能成功编译。 因为一元 `std::endl` 模板函数所要求的类型不同于 lambda 函数 `std::cout &lt;&lt; boost::lambda::_1` 的函数，所以在此不能使用它。

下一个版本的 C++ 标准很可能会将 lambda 函数作为 C++ 语言本身的组成部分加入，从而消除对单独的库的需要。 但是在下一个版本到来并被不同的编译器厂商所采用可能还需要好几年。 在此之前，Boost.Lambda 被证明是一个完美的替代品，从以下例子可以看出，这个例子只将大于 1 的元素写出到标准输出流。

```cpp
#include <boost/lambda/lambda.hpp> 
#include <boost/lambda/if.hpp> 
#include <iostream> 
#include <vector> 
#include <algorithm> 

int main() 
{ 
  std::vector<int> v; 
  v.push_back(1); 
  v.push_back(3); 
  v.push_back(2); 

  std::for_each(v.begin(), v.end(), 
    boost::lambda::if_then(boost::lambda::_1 > 1, 
    std::cout << boost::lambda::_1 << "\n")); 
} 
```

*   下载源代码

头文件 `boost/lambda/if.hpp` 定义了几个结构，允许在 lambda 函数内部使用 `if` 语句。 最基本的结构是 `boost::lambda::if_then()` 模板函数，它要求两个参数：第一个参数对条件求值 - 如果为真，则执行第二个参数。 如例中所示，每个参数本身都可以是 lambda 函数。

除了 `boost::lambda::if_then()`, Boost.Lambda 还提供了 `boost::lambda::if_then_else()` 和 `boost::lambda::if_then_else_return()` 模板函数 - 它们都要求三个参数。 另外还提供了用于实现循环、转型操作符，甚至是 `throw` - 允许 lambda 函数抛出异常 - 的模板函数。

虽然可以用这些模板函数在 C++ 中构造出复杂的 lambda 函数，但是你必须要考虑其它方面，如可读性和可维护性。 因为别人需要学习并理解额外的函数，如用 `boost::lambda::if_then()` 来替代已知的 C++ 关键字 `if` 和 `else`，lambda 函数的好处通常随着它的复杂性而降低。 多数情况下，更为合理的方法是用熟悉的 C++ 结构定义一个单独的函数。

## 3.6\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  简化以下程序，将函数对象 `divide_by` 转换为一个函数，并将 `for` 循环替换为用一个标准的 C++ 算法来输出数据：

    ```cpp
    #include &lt;algorithm&gt; 
    #include &lt;functional&gt; 
    #include &lt;vector&gt; 
    #include &lt;iostream&gt; 

    class divide_by 
      : public std::binary_function&lt;int, int, int&gt; 
    { 
    public: 
      int operator()(int n, int div) const 
      { 
        return n / div; 
      } 
    }; 

    int main() 
    { 
      std::vector&lt;int&gt; numbers; 
      numbers.push_back(10); 
      numbers.push_back(20); 
      numbers.push_back(30); 

      std::transform(numbers.begin(), numbers.end(), numbers.begin(), std::bind2nd(divide_by(), 2)); 

      for (std::vector&lt;int&gt;::iterator it = numbers.begin(); it != numbers.end(); ++it) 
        std::cout &lt;&lt; *it &lt;&lt; std::endl; 
    } 
    ```

    *   下载源代码
2.  简化以下程序，将两个 `for` 循环都替换为标准的 C++ 算法：

    ```cpp
    #include &lt;string&gt; 
    #include &lt;vector&gt; 
    #include &lt;iostream&gt; 

    int main() 
    { 
      std::vector&lt;std::string&gt; strings; 
      strings.push_back("Boost"); 
      strings.push_back("C++"); 
      strings.push_back("Libraries"); 

      std::vector&lt;int&gt; sizes; 

      for (std::vector&lt;std::string&gt;::iterator it = strings.begin(); it != strings.end(); ++it) 
        sizes.push_back(it-&gt;size()); 

      for (std::vector&lt;int&gt;::iterator it = sizes.begin(); it != sizes.end(); ++it) 
        std::cout &lt;&lt; *it &lt;&lt; std::endl; 
    } 
    ```

    *   下载源代码
3.  简化以下程序，修改变量 `processors` 的类型，并将 `for` 循环替换为标准的 C++ 算法：

    ```cpp
    #include &lt;vector&gt; 
    #include &lt;iostream&gt; 
    #include &lt;cstdlib&gt; 
    #include &lt;cstring&gt; 

    int main() 
    { 
      std::vector&lt;int(*)(const char*)&gt; processors; 
      processors.push_back(std::atoi); 
      processors.push_back(reinterpret_cast&lt;int(*)(const char*)&gt;(std::strlen)); 

      const char data[] = "1.23"; 

      for (std::vector&lt;int(*)(const char*)&gt;::iterator it = processors.begin(); it != processors.end(); ++it) 
        std::cout &lt;&lt; (*it)(data) &lt;&lt; std::endl; 
    } 
    ```

    *   下载源代码