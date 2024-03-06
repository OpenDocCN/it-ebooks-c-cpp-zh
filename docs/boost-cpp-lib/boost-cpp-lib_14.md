# 第十三章 容器

# 第十三章 容器

### 目录

*   13.1 概述
*   13.2 Boost.Array
*   13.3 Boost.Unordered
*   13.4 Boost.MultiIndex
*   13.5 Boost.Bimap
*   13.6 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 13.1\. 概述

这一章将会介绍许多我们在 C++ 标准中已经很熟悉的容器的 Boost 版本。 在这一章里， 我们会对 Boost.Unordered 的用法有一定的了解 （这个容器已经在 TR1 里被加入到了 C++ 标准）； 我们将会学习如何定义一个 Boost.MultiIndex； 我们还会了解何时应该使用 MuitiIndex 的一个特殊的扩展 —— Boost.Bimap。 接下来， 我们会向你介绍第一个容器 —— Boost.Array， 通过使用它， 你可以把 C++ 标准里普通的数组以容器的形式实现。

## 13.2\. Boost.Array

库 [Boost.Array](http://www.boost.org/libs/array/) 在 `boost/array.hpp` 中定义了一个模板类 `boost::array` 。 通过使用这个类， 你可以创建一个跟 C++ 里传统的数组有着相同属性的容器。 而且， `boost::array` 还满足了 C++ 中容器的一切需求， 因此， 你可以像操作容器一样方便的操作这个 array。 基本上， 你可以把 `boost::array` 当成 `std::vector` 来使用， 只不过 `boost::array` 是定长的。

```cpp
#include <boost/array.hpp> 
#include <iostream> 
#include <string> 
#include <algorithm> 

int main() 
{ 
  typedef boost::array<std::string, 3> array; 
  array a; 

  a[0] = "Boris"; 
  a.at(1) = "Anton"; 
  *a.rbegin() = "Caesar"; 

  std::sort(a.begin(), a.end()); 

  for (array::const_iterator it = a.begin(); it != a.end(); ++it) 
    std::cout << *it << std::endl; 

  std::cout << a.size() << std::endl; 
  std::cout << a.max_size() << std::endl; 
} 
```

*   下载源代码

就像我们在上面的例子看到的那样， `boost::array` 简直就是简单的不需要任何多余的解释， 因为所有操作都跟 `std::vector` 是一样的。

在下面的例子里， 我们会见识到 Boost.Array 的一个特性。

```cpp
#include <boost/array.hpp> 
#include <string> 

int main() 
{ 
  typedef boost::array<std::string, 3> array; 
  array a = { "Boris", "Anton", "Caesar" }; 
} 
```

*   下载源代码

一个 `boost::array` 类型的数组可以使用传统 C++ 数组的初始化方式来初始化。

既然这个容器已经在 TR1 中加入到了 C++ 标准， 它同样可以通过 `std::array` 来访问到。 他被定义在头文件 `array` 中， 使用它的前提是你正在使用一个支持 TR1 的 C++ 标准的库。

## 13.3\. Boost.Unordered

[Boost.Unordered](http://www.boost.org/libs/unordered/) 在 C++ 标准容器 `std::set`， `std::multiset`， `std::map` 和 `std::multimap` 的基础上多实现了四个容器： `boost::unordered_set`， `boost::unordered_multiset`， `boost::unordered_map` 和 `boost::unordered_multimap`。 那些名字很相似的容器之间并没有什么不同， 甚至还提供了相同的接口。 在很多情况下， 替换这两种容器 (std 和 boost) 对你的应用不会造成任何影响。

Boost.Unordered 和 C++ 标准里的容器的不同之处在于—— Boost.Unordered 不要求其中的元素是可排序的， 因为它不会做出排序操作。 在排序操作无足轻重时（或是根本不需要）， Boost.Unordered 就很合适了。

为了能够快速的查找元素， 我们需要使用 Hash 值。 Hash 值是一些可以唯一标识容器中元素的数字， 它在比较时比起类似 String 的数据类型会更加有效率。 为了计算 Hash 值， 容器中的所有元素都必须支持对他们自己唯一 ID 的计算。 比如 `std::set` 要求其中的元素都要是可比较的， 而 `boost::unordered_set` 要求其中的元素都要可计算 Hash 值。 尽管如此， 在对排序没有需求时， 你还是应该倾向使用 Boost.Unordered 。

下面的例子展示了 `boost::unordered_set` 的用法。

```cpp
#include <boost/unordered_set.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::unordered_set<std::string> unordered_set; 
  unordered_set set; 

  set.insert("Boris"); 
  set.insert("Anton"); 
  set.insert("Caesar"); 

  for (unordered_set::iterator it = set.begin(); it != set.end(); ++it) 
    std::cout << *it << std::endl; 

  std::cout << set.size() << std::endl; 
  std::cout << set.max_size() << std::endl; 

  std::cout << (set.find("David") != set.end()) << std::endl; 
  std::cout << set.count("Boris") << std::endl; 
} 
```

*   下载源代码

`boost::unordered_set` 提供了与 `std::set` 相似的函数。 当然， 这个例子不需要多大改进就可以用 `std::set` 来实现。

下面的例子展示了如何用 `boost::unordered_map` 来存储每一个的 person 的 name 和 age。

```cpp
#include <boost/unordered_map.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::unordered_map<std::string, int> unordered_map; 
  unordered_map map; 

  map.insert(unordered_map::value_type("Boris", 31)); 
  map.insert(unordered_map::value_type("Anton", 35)); 
  map.insert(unordered_map::value_type("Caesar", 25)); 

  for (unordered_map::iterator it = map.begin(); it != map.end(); ++it) 
    std::cout << it->first << ", " << it->second << std::endl; 

  std::cout << map.size() << std::endl; 
  std::cout << map.max_size() << std::endl; 

  std::cout << (map.find("David") != map.end()) << std::endl; 
  std::cout << map.count("Boris") << std::endl; 
} 
```

*   下载源代码

就像我们看到的， `boost::unordered_map` 和 `std::map` 之间并没多大区别。 同样地， 你可以很方便的用 `std::map` 来重新实现这个例子。

就像上面提到过的， Boost.Unordered 需要其中的元素可计算 Hash 值。 一些类似于 `std::string` 的数据类型“天生”就支持 Hash 值的计算。 对于那些自定义的类型， 你需要手动的定义 Hash 函数。

```cpp
#include <boost/unordered_set.hpp> 
#include <string> 

struct person 
{ 
  std::string name; 
  int age; 

  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 

  bool operator==(const person &p) const 
  { 
    return name == p.name && age == p.age; 
  } 
}; 

std::size_t hash_value(person const &p) 
{ 
  std::size_t seed = 0; 
  boost::hash_combine(seed, p.name); 
  boost::hash_combine(seed, p.age); 
  return seed; 
} 

int main() 
{ 
  typedef boost::unordered_set<person> unordered_set; 
  unordered_set set; 

  set.insert(person("Boris", 31)); 
  set.insert(person("Anton", 35)); 
  set.insert(person("Caesar", 25)); 
} 
```

*   下载源代码

在代码中， `person` 类型的元素被存到了 `boost::unordered_set` 中。 因为 `boost::unordered_set` 中的 Hash 函数不能识别 `person` 类型， Hash 值就变得无法计算了。 若果没有定义另一个 Hash 函数， 你的代码将不会通过编译。

Hash 函数的签名必须是： `hash_value()`。 它接受唯一的一个参数来指明需要计算 Hash 值的对象的类型。 因为 Hash 值是单纯的数字， 所以函数的返回值为： `std::size_t`。

每当一个对象需要计算它的 Hash 值时， `hash_value()` 都会自动被调用。 Boost C++ 库已经为一些数据类型定义好了 Hash 函数， 比如： `std::string`。 但对于像 `person` 这样的自定义类型， 你就需要自己手工定义了。

`hash_value()` 的实现往往都很简单： 你只需要按顺序对其中的每个属性都调用 Boost 在 `boost/functional/hash.hpp` 中提供的 `boost::hash_combine()` 函数就行了。 当你使用 Boost.Unordered 时， 这个头文件已经自动被包含了。

除了自定义 `hash_value()` 函数， 自定义的类型还需要支持通过 `==` 运算符的比较操作。 因此， `person` 就重载了相应的 `operator==()` 操作符。

## 13.4\. Boost.MultiIndex

[Boost.MultiIndex](http://www.boost.org/libs/multi_index/) 比我们之前介绍的任何库都要复杂。 不像 Boost.Array 和 Boost.Unordered 为我们提供了可以直接使用的容器， Boost.MultiIndex 让我们可以自定义新的容器。 跟 C++ 标准中的容器不同的是： 一个用户自定义的容器可以对其中的数据提供多组访问接口。 举例来说， 你可以定义一个类似于 `std::map` 的容器， 但它可以通过 value 值来查询。 如果不用 Boost.MultiIndex， 你就需要自己整合两个 `std::map` 类型的容器， 还要自己处理一些同步操作来确保数据的完整性。

下面这个例子就用 Boost.MultiIndex 定义了一个新容器来存储每个人的 name 和 age， 不像 `std::map`， 这个容器可以分别通过 name 和 age 来查询（std::map 只能用一个值）。

```cpp
#include <boost/multi_index_container.hpp> 
#include <boost/multi_index/hashed_index.hpp> 
#include <boost/multi_index/member.hpp> 
#include <iostream> 
#include <string> 

struct person 
{ 
  std::string name; 
  int age; 

  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 
}; 

typedef boost::multi_index::multi_index_container< 
  person, 
  boost::multi_index::indexed_by< 
    boost::multi_index::hashed_non_unique< 
      boost::multi_index::member< 
        person, std::string, &person::name 
      > 
    >, 
    boost::multi_index::hashed_non_unique< 
      boost::multi_index::member< 
        person, int, &person::age 
      > 
    > 
  > 
> person_multi; 

int main() 
{ 
  person_multi persons; 

  persons.insert(person("Boris", 31)); 
  persons.insert(person("Anton", 35)); 
  persons.insert(person("Caesar", 25)); 

  std::cout << persons.count("Boris") << std::endl; 

  const person_multi::nth_index<1>::type &age_index = persons.get<1>(); 
  std::cout << age_index.count(25) << std::endl; 
} 
```

*   下载源代码

就像上面提到的， Boost.MultiIndex 并没有提供任何特定的容器而是一些类来方便我们定义新的容器。 典型的做法是： 你需要用到 `typedef` 来为你的新容器提供对 Boost.MultiIndex 中类的方便的访问。

每个容器定义都需要的类 `boost::multi_index::multi_index_container` 被定义在了 `boost/multi_index_container.hpp` 里。 因为他是一个模板类， 你需要为它传递两个模板参数。 第一个参数是容器中储存的元素类型， 在例子中是 `person`； 而第二个参数指明了容器所提供的所有索引类型。

基于 Boost.MultiIndex 的容器最大的优势在于： 他对一组同样的数据提供了多组访问接口。 访问接口的具体细节都可以在定义容器时被指定。 因为例子中的 person 为 age 和 name 都提供了查询功能， 我们必须要定义两组接口。

接口的定义必须借由模板类 `boost::multi_index::indexed_by` 来实现。 每一个接口都作为参数传递给它。 例子中定义了两个 `boost::multi_index::hashed_non_unique` 类型的接口，（定义在头文件 `boost/multi_index/hashed_index.hpp` 中） 如果你希望容器像 Boost.Unordered 一样存储一些可以计算 Hash 值的元素， 你就可以使用这个接口。

`boost::multi_index::hashed_non_unique` 是一个模板类， 他需要一个可计算 Hash 值的类型作为它的参数。 因为接口需要访问 person 中的 name 和 age， 所以 name 和 age 都要是可计算 Hash 值的。

Boost.MultiIndex 提供了一个辅助模板类： `boost::multi_index::member` （定义在 `boost/multi_index/member.hpp` 中） 来访问类中的属性。 就像我们在例子中所看到的， 我们指定了好几个参数来让 `boost::multi_index::member` 明白可以访问 `person` 中的哪些属性以及这些属性的类型。

不得不说 `person_multi` 的定义第一眼看起来相当复杂， 但这个类本身跟 Boost.Unordered 中的 `boost::unordered_map` 并没有什么不同， 他也可以分别通过其中的两个属性 name 和 age 来查询容器。

为了访问 MultiIndex 容器， 你必须要定义至少一个接口。 如果用 `insert()` 或者 `count()` 来直接访问 `persons` 对象， 第一个接口会被隐式的调用 —— 在例子中是 `name` 属性的 Hash 容器。 如果你想用其他接口， 你必须要显示的指定它。

接口都是用从 0 开始的索引值来编号的。 想要访问第二个接口， 你需要调用 `get()` 函数并且传入想要访问的接口的索引值。

函数 `get()` 的返回值看起来也有点复杂： 他是一个用来访问 MultiIndex 容器的类 `nth_index` ， 同样的， 你也需要指定需要访问的接口的索引值。 当然， 这个值肯定跟 `get()` 函数指定的模板参数是一样的。 最后一步： 用 `::` 来得到 `nth_index` 的 `type`， 也就是接口的真正的`type`。

虽然我们并不知道细节就用 `nth_index` 和 `type` 得到了接口， 我们还是需要明白这到底是什么接口。 通过传给 `get()` 和 `nth_index` 的索引值， 我们就可以很容易得知所使用的哪一个接口了。 例子中的 `age_index` 就是一个通过 age 来访问的 Hash 容器。

既然 MultiIndex 容器中的 name 和 key 作为了接口访问的键值， 他们都不能再被更改了。 比如一个 person 的 age 在通过 name 搜索以后被改变了， 使用 age 作为键值的接口却意识不到这种更改， 因此， 你需要重新计算 Hash 值才行。

就像 `std::map` 一样， MultiIndex 容器中的值也不允许被修改。 严格的说， 所有存储在 MultiIndex 中的元素都该是常量。 为了避免删除或修改其中元素真正的值， Boost.MultiIndex 提供了一些常用函数来操作其中的元素。 使用这些函数来操作 MultiIndex 容器中的值并不会引起那些元素所指向的真正的对象改变， 所以更新动作是安全的。 而且所有接口都会被通知这种改变， 然后去重新计算新的 Hash 值等。

```cpp
#include <boost/multi_index_container.hpp> 
#include <boost/multi_index/hashed_index.hpp> 
#include <boost/multi_index/member.hpp> 
#include <iostream> 
#include <string> 

struct person 
{ 
  std::string name; 
  int age; 

  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 
}; 

typedef boost::multi_index::multi_index_container< 
  person, 
  boost::multi_index::indexed_by< 
    boost::multi_index::hashed_non_unique< 
      boost::multi_index::member< 
        person, std::string, &person::name 
      > 
    >, 
    boost::multi_index::hashed_non_unique< 
      boost::multi_index::member< 
        person, int, &person::age 
      > 
    > 
  > 
> person_multi; 

void set_age(person &p) 
{ 
  p.age = 32; 
} 

int main() 
{ 
  person_multi persons; 

  persons.insert(person("Boris", 31)); 
  persons.insert(person("Anton", 35)); 
  persons.insert(person("Caesar", 25)); 

  person_multi::iterator it = persons.find("Boris"); 
  persons.modify(it, set_age); 

  const person_multi::nth_index<1>::type &age_index = persons.get<1>(); 
  std::cout << age_index.count(32) << std::endl; 
} 
```

*   下载源代码

每个 Boost.MultiIndex 中的接口都支持 `modify()` 函数来提供直接对容器本身的操作。 它的第一个参数是一个需要更改对象的迭代器； 第二参数则是一个对该对象进行操作的函数。 在例子中， 对应的两个参数则是： `person` 和 `set_age()` 。

至此， 我们都还只介绍了一个接口： `boost::multi_index::hashed_non_unique` ， 他会计算其中元素的 Hash 值， 但并不要求是唯一的。 为了确保容器中存储的值是唯一的， 你可以使用 `boost::multi_index::hashed_unique` 接口。 请注意： 所有要被存入容器中的值都必须满足它的接口的限定。 只要一个接口限定了容器中的值必须是唯一的， 那其他接口都不会对该限定造成影响。

```cpp
#include <boost/multi_index_container.hpp> 
#include <boost/multi_index/hashed_index.hpp> 
#include <boost/multi_index/member.hpp> 
#include <iostream> 
#include <string> 

struct person 
{ 
  std::string name; 
  int age; 

  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 
}; 

typedef boost::multi_index::multi_index_container< 
  person, 
  boost::multi_index::indexed_by< 
    boost::multi_index::hashed_non_unique< 
      boost::multi_index::member< 
        person, std::string, &person::name 
      > 
    >, 
    boost::multi_index::hashed_unique< 
      boost::multi_index::member< 
        person, int, &person::age 
      > 
    > 
  > 
> person_multi; 

int main() 
{ 
  person_multi persons; 

  persons.insert(person("Boris", 31)); 
  persons.insert(person("Anton", 31)); 
  persons.insert(person("Caesar", 25)); 

  const person_multi::nth_index<1>::type &age_index = persons.get<1>(); 
  std::cout << age_index.count(31) << std::endl; 
} 
```

*   下载源代码

上例中的容器现在使用了 `boost::multi_index::hashed_unique` 来作为他的第二个接口， 因此他不允许其中有两个同 age 的 person 存在。

上面的代码尝试存储一个与 Boris 同 age 的 Anton， 因为这个动作违反了容器第二个接口的限定， 它（Anton）将不会被存入到容器中。 因此， 程序将会输出： `1` 而不是 2。

接下来的例子向我们展示了 Boost.MultiIndex 中剩下的三个接口： `boost::multi_index::sequenced`， `boost::multi_index::ordered_non_unique` 和 `boost::multi_index::random_access`。

```cpp
#include <boost/multi_index_container.hpp> 
#include <boost/multi_index/sequenced_index.hpp> 
#include <boost/multi_index/ordered_index.hpp> 
#include <boost/multi_index/random_access_index.hpp> 
#include <boost/multi_index/member.hpp> 
#include <iostream> 
#include <string> 

struct person 
{ 
  std::string name; 
  int age; 

  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 
}; 

typedef boost::multi_index::multi_index_container< 
  person, 
  boost::multi_index::indexed_by< 
    boost::multi_index::sequenced<>, 
    boost::multi_index::ordered_non_unique< 
      boost::multi_index::member< 
        person, int, &person::age 
      > 
    >, 
    boost::multi_index::random_access<> 
  > 
> person_multi; 

int main() 
{ 
  person_multi persons; 

  persons.push_back(person("Boris", 31)); 
  persons.push_back(person("Anton", 31)); 
  persons.push_back(person("Caesar", 25)); 

  const person_multi::nth_index<1>::type &ordered_index = persons.get<1>(); 
  person_multi::nth_index<1>::type::iterator lower = ordered_index.lower_bound(30); 
  person_multi::nth_index<1>::type::iterator upper = ordered_index.upper_bound(40); 
  for (; lower != upper; ++lower) 
    std::cout << lower->name << std::endl; 

  const person_multi::nth_index<2>::type &random_access_index = persons.get<2>(); 
  std::cout << random_access_index[2].name << std::endl; 
} 
```

*   下载源代码

`boost::multi_index::sequenced` 接口让我们可以像使用 `std::list` 一样的使用 MultiIndex。 这个接口定义起来十分容易： 你不用为它传递任何模板参数。 `person` 类型的对象在容器中就是像 list 一样按照加入的顺序来排列的。

而通过使用 `boost::multi_index::ordered_non_unique` 接口， 容器中的对象会自动的排序。 你在定义容器时就必须指定接口的排序规则。 示例中的对象 `person` 就是以 age 来排序的， 它借助了辅助类 `boost::multi_index::member` 来实现这一功能。

`boost::multi_index::ordered_non_unique` 为我们提供了一些特别的函数来查找特定范围的数据。 通过使用 `lower_bound()` 和 `upper_bound()`， 示例实现了对所有 30 岁至 40 岁的 person 的查询。 要注意因为容器中的数据是有序的， 所以才提供了这些函数， 其他接口中并不提供这些函数。

最后一个接口是： `boost::multi_index::random_access`， 他让我们可以像使用 `std::vector` 一样使用 MultiIndex 容器。 你又可以使用你熟悉的 `operator[]()` 和 `at()` 操作了。

请注意 `boost::multi_index::random_access` 已经被完整的包含在了 `boost::multi_index::sequenced` 接口中。 所以当你使用 `boost::multi_index::random_access` 的时候， 你也可以使用 `boost::multi_index::sequenced` 接口中的所有函数。

在介绍完 Boost.MultiIndex 剩下的 4 个接口后， 本章剩下的部分将向你介绍所谓的“键值提取器”（key extractors）。 目前为止， 我们已经见过一个在 `boost/multi_index/member.hpp` 定义的键值提取器了—— `boost::multi_index::member` 。 这个辅助函数的得名源自它可以显示的声明类中的哪些属性会作为接口中的键值使用。

接下来的例子介绍了另外两个键值提取器。

```cpp
#include <boost/multi_index_container.hpp> 
#include <boost/multi_index/ordered_index.hpp> 
#include <boost/multi_index/hashed_index.hpp> 
#include <boost/multi_index/identity.hpp> 
#include <boost/multi_index/mem_fun.hpp> 
#include <iostream> 
#include <string> 

class person 
{ 
public: 
  person(const std::string &n, int a) 
    : name(n), age(a) 
  { 
  } 

  bool operator<(const person &p) const 
  { 
    return age < p.age; 
  } 

  std::string get_name() const 
  { 
    return name; 
  } 

private: 
  std::string name; 
  int age; 
}; 

typedef boost::multi_index::multi_index_container< 
  person, 
  boost::multi_index::indexed_by< 
    boost::multi_index::ordered_unique< 
      boost::multi_index::identity<person> 
    >, 
    boost::multi_index::hashed_unique< 
      boost::multi_index::const_mem_fun< 
        person, std::string, &person::get_name 
      > 
    > 
  > 
> person_multi; 

int main() 
{ 
  person_multi persons; 

  persons.insert(person("Boris", 31)); 
  persons.insert(person("Anton", 31)); 
  persons.insert(person("Caesar", 25)); 

  std::cout << persons.begin()->get_name() << std::endl; 

  const person_multi::nth_index<1>::type &hashed_index = persons.get<1>(); 
  std::cout << hashed_index.count("Boris") << std::endl; 
} 
```

*   下载源代码

键值提取器`boost::multi_index::identity`（定义在 `boost/multi_index/identity.hpp` 中） 可以使用容器中的数据类型作为键值。 示例中， 就需要 `person` 类是可排序的， 因为它已经作为了接口 `boost::multi_index::ordered_unique` 的键值。 在示例里， 它是通过重载 `operator&lt;()` 操作符来实现的。

头文件 `boost/multi_index/mem_fun.hpp` 定义了两个可以把函数返回值作为键值的键值提取器： `boost::multi_index::const_mem_fun` 和 `boost::multi_index::mem_fun` 。 在示例程序中， 就是用到了 `get_name()` 的返回值作为键值。 显而易见的， `boost::multi_index::const_mem_fun` 适用于返回常量的函数， 而 `boost::multi_index::mem_fun` 适用于返回非常量的函数。

Boost.MultiIndex 还提供了两个键值提取器： `boost::multi_index::global_fun` 和 `boost::multi_index::composite_key`。 前一个适用于独立的函数或者静态函数， 后一个允许你将几个键值提取器组合成一个新的的键值提取器。

## 13.5\. Boost.Bimap

[Boost.Bimap](http://www.boost.org/libs/bimap/) 库提供了一个建立在 Boost.MultiIndex 之上但不需要预先定义就可以使用的容器。 这个容器十分类似于 `std::map`， 但他不仅可以通过 key 搜索， 还可以用 value 来搜索。

```cpp
#include <boost/bimap.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::bimap<std::string, int> bimap; 
  bimap persons; 

  persons.insert(bimap::value_type("Boris", 31)); 
  persons.insert(bimap::value_type("Anton", 31)); 
  persons.insert(bimap::value_type("Caesar", 25)); 

  std::cout << persons.left.count("Boris") << std::endl; 
  std::cout << persons.right.count(31) << std::endl; 
} 
```

*   下载源代码

在 `boost/bimap.hpp` 中定义的 `boost::bimap` 为我们提供了两个属性： `left` 和 `right` 来访问在 `boost::bimap` 统一的两个 `std::map` 类型的容器。 在例子中， `left` 用 `std::string` 类型的 key 来访问容器， 而 `right` 用到了 `int` 类型的 key。

除了支持用 left 和 right 对容器中的记录进行单独的访问， `boost::bimap` 还允许像下面的例子一样展示记录间的关联关系。

```cpp
#include <boost/bimap.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::bimap<std::string, int> bimap; 
  bimap persons; 

  persons.insert(bimap::value_type("Boris", 31)); 
  persons.insert(bimap::value_type("Anton", 31)); 
  persons.insert(bimap::value_type("Caesar", 25)); 

  for (bimap::iterator it = persons.begin(); it != persons.end(); ++it) 
    std::cout << it->left << " is " << it->right << " years old." << std::endl; 
} 
```

*   下载源代码

对一个记录访问时， `left` 和 `right` 并不是必须的。 你也可以使用迭代器来访问每个记录中的 left 和 right 容器。

`std::map` 和 `std::multimap` 组合让你觉得似乎可以存储多个具有相同 key 值的记录， 但 `boost::bimap` 并没有这样做。 但这并不代表在 `boost::bimap` 存储两个具有相同 key 值的记录是不可能的。 严格来说， 那两个模板参数并不会对 `left` 和 `right` 的容器类型做出具体的规定。 如果像例子中那样并没有指定容器类型时， `boost::bimaps::set_of` 类型会缺省的使用。 跟 `std::map` 一样， 它要求记录有唯一的 key 值。

第一个 `boost::bimap` 例子也可以像下面这样写。

```cpp
#include <boost/bimap.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::bimap<boost::bimaps::set_of<std::string>, boost::bimaps::set_of<int>> bimap; 
  bimap persons; 

  persons.insert(bimap::value_type("Boris", 31)); 
  persons.insert(bimap::value_type("Anton", 31)); 
  persons.insert(bimap::value_type("Caesar", 25)); 

  std::cout << persons.left.count("Boris") << std::endl; 
  std::cout << persons.right.count(31) << std::endl; 
} 
```

*   下载源代码

除了 `boost::bimaps::set_of`， 你还可以用一些其他的容器类型来定制你的 `boost::bimap`。

```cpp
#include <boost/bimap.hpp> 
#include <boost/bimap/multiset_of.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::bimap<boost::bimaps::set_of<std::string>, boost::bimaps::multiset_of<int>> bimap; 
  bimap persons; 

  persons.insert(bimap::value_type("Boris", 31)); 
  persons.insert(bimap::value_type("Anton", 31)); 
  persons.insert(bimap::value_type("Caesar", 25)); 

  std::cout << persons.left.count("Boris") << std::endl; 
  std::cout << persons.right.count(31) << std::endl; 
} 
```

*   下载源代码

代码中的容器使用了定义在 `boost/bimap/multiset_of.hpp` 中的 `boost::bimaps::multiset_of`。 这个容器的操作和 `boost::bimaps::set_of` 差不了多少， 只是它不再要求 key 值是唯一的。 因此， 上面的例子将会在计算 age 为 31 的 person 数时输出： `2`。

既然 `boost::bimaps::set_of` 会在定义 `boost::bimap` 被缺省的使用， 你没必要再显示的包含头文件： `boost/bimap/set_of.hpp`。 但在使用其它类型的容器时， 你就必须要显示的包含一些相应的头文件了。

Boost.Bimap 还提供了类： `boost::bimaps::unordered_set_of`， `boost::bimaps::unordered_multiset_of`， `boost::bimaps::list_of`，`boost::bimaps::vector_of` 和 `boost::bimaps::unconstrainted_set_of` 以供使用。 除了 `boost::bimaps::unconstrainted_set_of`， 剩下的所有容器类型的使用方法都和他们在 C++ 标准里的版本一样。

```cpp
#include <boost/bimap.hpp> 
#include <boost/bimap/unconstrained_set_of.hpp> 
#include <boost/bimap/support/lambda.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  typedef boost::bimap<std::string, boost::bimaps::unconstrained_set_of<int>> bimap; 
  bimap persons; 

  persons.insert(bimap::value_type("Boris", 31)); 
  persons.insert(bimap::value_type("Anton", 31)); 
  persons.insert(bimap::value_type("Caesar", 25)); 

  bimap::left_map::iterator it = persons.left.find("Boris"); 
  persons.left.modify_key(it, boost::bimaps::_key = "Doris"); 

  std::cout << it->first << std::endl; 
} 
```

*   下载源代码

`boost::bimaps::unconstrainted_set_of` 可以使 `boost::bimap` 的 `right` （也就是 age）值无法用来查找 person。 在这种特定的情况下， `boost::bimap` 可以被视为是一个 `std::map` 类型的容器。

虽然如此， 例子还是向我们展示了 `boost::bimap` 对于 `std::map` 的优越性。 因为 Boost.Bimap 是基于 Boost.MultiIndex 的， 你当然可以使用 Boost.MultiIndex 提供的所有函数。 例子中就用 `modify_key()` 修改了 key 值， 这在 `std::map` 中是不可能的。

请注意修改 key 值的以下细节： key 通过 `boost::bimaps::_key` 函数赋予了新值， 而 `boost::bimaps::_key` 是一个定义在 `boost/bimap/support/lambda.hpp` 中的 lambda 函数。 有关 lambda 函数， 详见：第三章 *函数对象*。

`boost/bimap/support/lambda.hpp` 还定义了 `boost::bimaps::_data`。 函数 `modify_data()` 可以用来修改 `boost::bimap` 中的 value 值。

## 13.6\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  创建你的应用： 它支持指派员工到一家公司不同的部门。 存入一些示例记录， 然后通过指定员工来得到他所在的部门， 再通过指定部门来得到该部门的员工数。

2.  扩展你的应用： 加入员工的 ID 号。 ID 号必须是唯一的， 保证在员工同名的情况下依然可以唯一的标识员工。 通过指定 ID 号来得到某个员工的信息并把它输出以验证正确性。