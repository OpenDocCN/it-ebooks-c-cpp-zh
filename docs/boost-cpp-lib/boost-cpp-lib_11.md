# 第十章 日期与时间

# 第十章 日期与时间

### 目录

*   10.1 概述
*   10.2 历法日期
*   10.3 位置无关的时间
*   10.4 位置相关的时间
*   10.5 格式化输入输出
*   10.6 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 10.1\. 概述

库 [Boost.DateTime](http://www.boost.org/libs/date_time/) 可用于处理时间数据，如历法日期和时间。 另外，Boost.DateTime 还提供了扩展来处理时区的问题，且支持历法日期和时间的格式化输入与输出。 本章将覆盖 Boost.DateTime 的各个部分。

## 10.2\. 历法日期

Boost.DateTime 只支持基于格里历的历法日期，这通常不成问题，因为这是最广泛使用的历法。 如果你与其它国家的某人有个会议，时间在 2010 年 1 月 5 日，你可以期望无需与对方确认这个日期是否基于格里历。

格里历是教皇 Gregory XIII 在 1582 年颁发的。 严格来说，Boost.DateTime 支持由 1400 年至 9999 年的历法日期，这意味着它支持 1582 年以前的日期。 因此，Boost.DateTime 可用于任一历法日期，只要该日期在转换为格里历后是在 1400 年之后。 如果需要更早的年份，就必须使用其它库来代替。

用于处理历法日期的类和函数位于名字空间 `boost::gregorian` 中，定义于 `boost/date_time/gregorian/gregorian.hpp`。 要创建一个日期，请使用 `boost::gregorian::date` 类。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d(2010, 1, 30); 
  std::cout << d.year() << std::endl; 
  std::cout << d.month() << std::endl; 
  std::cout << d.day() << std::endl; 
  std::cout << d.day_of_week() << std::endl; 
  std::cout << d.end_of_month() << std::endl; 
} 
```

*   下载源代码

`boost::gregorian::date` 提供了多个构造函数来进行日期的创建。 最基本的构造函数接受一个年份、一个月份和一个日期作为参数。 如果给定的是一个无效值，则将分别抛出 `boost::gregorian::bad_year`, `boost::gregorian::bad_month` 或 `boost::gregorian::bad_day_of_month` 类型的异常，这些异常均派生自 `std::out_of_range`。

正如在这个例子中所示的， 有多个方法用于访问一个日期。 象 `year()`, `month()` 和 `day()` 这些方法访问用于初始化的初始值，象 `day_of_week()` 和 `end_of_month()` 这些方法则访问计算得到的值。

而 `boost::gregorian::date` 的构造函数则接受年份、月份和日期的值来设定一个日期，调用 `month()` 方法实际上会显示 `Jan`，而调用 `day_of_week()` 则显示 `Sat`。 它们不是普通的数字值，而分别是 `boost::gregorian::date::month_type` 和 `boost::gregorian::date::day_of_week_type` 类型的值。 不过，Boost.DateTime 为格式化的输入输出提供了全面的支持，可以将以上输出从 `Jan` 调整为 `1`。

请留意，`boost::gregorian::date` 的缺省构造函数会创建一个无效的日期。 这样的无效日期也可以通过将 `boost::date_time::not_a_date_time` 作为单一参数传递给构造函数来显式地创建。

除了直接调用构造函数，也可以通过自由函数或其它对象的方法来创建一个 `boost::gregorian::date` 类型的对象。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d = boost::gregorian::day_clock::universal_day(); 
  std::cout << d.year() << std::endl; 
  std::cout << d.month() << std::endl; 
  std::cout << d.day() << std::endl; 

  d = boost::gregorian::date_from_iso_string("20100131"); 
  std::cout << d.year() << std::endl; 
  std::cout << d.month() << std::endl; 
  std::cout << d.day() << std::endl; 
} 
```

*   下载源代码

这个例子使用了 `boost::gregorian::day_clock` 类，它是一个返回当前日期的时钟类。 方法 `universal_day()` 返回一个与时区及夏时制无关的 UTC 日期。 UTC 是世界时(universal time)的国际缩写。 `boost::gregorian::day_clock` 还提供了另一个方法 `local_day()`，它接受本地设置。 要取出本地时区的当前日期，必须使用 `local_day()`。

名字空间 `boost::gregorian` 中包含了许多其它自由函数，将保存在字符串中的日期转换为 `boost::gregorian::date` 类型的对象。 这个例子实际上是通过 `boost::gregorian::date_from_iso_string()` 函数对一个以 ISO 8601 格式给出的日期进行转换。 还有其它相类似的函数，如 `boost::gregorian::from_simple_string()` 和 `boost::gregorian::from_us_string()`。

`boost::gregorian::date` 表示的是一个特定的时间点，而 `boost::gregorian::date_duration` 则表示了一段时间。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d1(2008, 1, 31); 
  boost::gregorian::date d2(2008, 8, 31); 
  boost::gregorian::date_duration dd = d2 - d1; 
  std::cout << dd.days() << std::endl; 
} 
```

*   下载源代码

由于 `boost::gregorian::date` 重载了 `operator-()` 操作符，所以两个时间点可以如上所示那样相减。 返回值的类型为 `boost::gregorian::date_duration`，表示了两个日期之间的时间长度。

`boost::gregorian::date_duration` 所提供的最重要的方法是 `days()`，它返回一段时间内所包含的天数。

我们也可以通过传递一个天数作为构造函数的唯一参数，来显式创建 `boost::gregorian::date_duration` 类型的对象。 要创建涉及星期数、月份数或年数的时间段，可以相应使用 `boost::gregorian::weeks`, `boost::gregorian::months` 和 `boost::gregorian::years`。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date_duration dd(4); 
  std::cout << dd.days() << std::endl; 
  boost::gregorian::weeks ws(4); 
  std::cout << ws.days() << std::endl; 
  boost::gregorian::months ms(4); 
  std::cout << ms.number_of_months() << std::endl; 
  boost::gregorian::years ys(4); 
  std::cout << ys.number_of_years() << std::endl; 
} 
```

*   下载源代码

`boost::gregorian::months` 和 `boost::gregorian::years` 都无法确定其天数，因为某月或某年所含天数是可长的。 不过，这些类的用法还是可以从以下例子中看出。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d(2009, 1, 31); 
  boost::gregorian::months ms(1); 
  boost::gregorian::date d2 = d + ms; 
  std::cout << d2 << std::endl; 
  boost::gregorian::date d3 = d2 - ms; 
  std::cout << d3 << std::endl; 
} 
```

*   下载源代码

该程序将一个月加到给定的日期 January 31, 2009 上，得到 `d2`，其为 February 28, 2009。 接着，再减回一个月得到 `d3`，又重新变回 January 31, 2009。 如上所示，时间点和时间长度可用于计算。 不过，需要考虑具体的情况。 例如，从某月的最后一天开始计算，`boost::gregorian::months` 总是会到达另一个月的最后一天，如果反复前后跳，就可能得到令人惊讶的结果。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d(2009, 1, 30); 
  boost::gregorian::months ms(1); 
  boost::gregorian::date d2 = d + ms; 
  std::cout << d2 << std::endl; 
  boost::gregorian::date d3 = d2 - ms; 
  std::cout << d3 << std::endl; 
} 
```

*   下载源代码

这个例子与前一个例子的不同之处在于，初始的日期是 January 30, 2009。 虽然这不是 January 的最后一天，但是向前跳一个月后得到的 `d2` 还是 February 28, 2009，因为没有 February 30 这一天。 不过，当我们再往回跳一个月，这次得到的 `d3` 就变成 January 31, 2009! 因为 February 28, 2009 是当月的最后一天，往回跳实际上是返回到 January 的最后一天。

如果你觉得这种行为过于混乱，可以通过取消 `BOOST_DATE_TIME_OPTIONAL_GREGORIAN_TYPES` 宏的定义来改变这种行为。 取消该宏后，`boost::gregorian::weeks`, `boost::gregorian::months` 和 `boost::gregorian::years` 类都不再可用。 唯一剩下的类是 `boost::gregorian::date_duration`，只能指定前向或后向的跳过的天数，这样就不会再有意外的结果了。

`boost::gregorian::date_duration` 表示的是时间长度，而 `boost::gregorian::date_period` 则提供了对两个日期之间区间的支持。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d1(2009, 1, 30); 
  boost::gregorian::date d2(2009, 10, 31); 
  boost::gregorian::date_period dp(d1, d2); 
  boost::gregorian::date_duration dd = dp.length(); 
  std::cout << dd.days() << std::endl; 
} 
```

*   下载源代码

两个类型为 `boost::gregorian::date` 的参数指定了开始和结束的日期，它们被传递给 `boost::gregorian::date_period` 的构造函数。 此外，也可以指定一个开始日期和一个类型为 `boost::gregorian::date_duration` 的时间长度。 请注意，结束日期的前一天才是这个时间区间的最后一天，这对于理解以下例子的输出非常重要。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d1(2009, 1, 30); 
  boost::gregorian::date d2(2009, 10, 31); 
  boost::gregorian::date_period dp(d1, d2); 
  std::cout << dp.contains(d1) << std::endl; 
  std::cout << dp.contains(d2) << std::endl; 
} 
```

*   下载源代码

这个程序用 `contains()` 方法来检查某个给定的日期是否包含在时间区间内。 虽然 `d1` 和 `d2` 都是被传递给 `boost::gregorian::date_period` 的构造函数的，但是 `contains()` 仅对第一个返回 `true`。 因为结束日期不是区间的一部分，所以以 `d2` 调用 `contains()` 会返回 `false`。

`boost::gregorian::date_period` 还提供了其它方法，如移动一个区间，或计算两个重叠区间的交集。

除了日期类、时间长度类和时间区间类，Boost.DateTime 还提供了迭代器和其它有用的自由函数，如下例所示。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::gregorian::date d(2009, 1, 5); 
  boost::gregorian::day_iterator it(d); 
  std::cout << *++it << std::endl; 
  std::cout << boost::date_time::next_weekday(*it, boost::gregorian::greg_weekday(boost::date_time::Friday)) << std::endl; 
} 
```

*   下载源代码

为了从一个指定日期向前或向后一天一天地跳，可以使用迭代器 `boost::gregorian::day_iterator`。 还有 `boost::gregorian::week_iterator`, `boost::gregorian::month_iterator` 和 `boost::gregorian::year_iterator` 分别提供了按周、按月或是按年跳的迭代器。

这个例子还使用了 `boost::date_time::next_weekday()`，它基于一个给定的日期返回下一个星期几的日期。 以下程序将显示 `2009-Jan-09`，因为它是 January 6, 2009 之的第一个 Friday。

## 10.3\. 位置无关的时间

`boost::gregorian::date` 用于创建日期，`boost::posix_time::ptime` 则用于定义一个位置无关的时间。 `boost::posix_time::ptime` 会存取 `boost::gregorian::date` 且额外保存一个时间。

为了使用 `boost::posix_time::ptime`，必须包含头文件 `boost/date_time/posix_time/posix_time.hpp`。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt(boost::gregorian::date(2009, 1, 5), boost::posix_time::time_duration(12, 0, 0)); 
  boost::gregorian::date d = pt.date(); 
  std::cout << d << std::endl; 
  boost::posix_time::time_duration td = pt.time_of_day(); 
  std::cout << td << std::endl; 
} 
```

*   下载源代码

要初始化一个 `boost::posix_time::ptime` 类型的对象，要把一个类型为 `boost::gregorian::date` 的日期和一个类型为 `boost::posix_time::time_duration` 的时间长度作为第一和第二参数传递给构造函数。 传给 `boost::posix_time::time_duration` 构造函数的三个参数决定了时间点。 以上程序指定的时间点是 January 5, 2009 的 12 PM。

要查询日期和时间，可以使用 `date()` 和 `time_of_day()` 方法。

象 `boost::gregorian::date` 的缺省构造函数会创建一个无效日期一样，如果使用缺省构造函数，`boost::posix_time::ptime` 类型的对象也是无效的。 也可以通过传递一个 `boost::date_time::not_a_date_time` 给构造函数来显式创建一个无效时间。

和使用自由函数或其它对象的方法来创建 `boost::gregorian::date` 类型的历法日期一样，Boost.DateTime 也提供了相应的自由函数和对象来创建时间。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt = boost::posix_time::second_clock::universal_time(); 
  std::cout << pt.date() << std::endl; 
  std::cout << pt.time_of_day() << std::endl; 

  pt = boost::posix_time::from_iso_string("20090105T120000"); 
  std::cout << pt.date() << std::endl; 
  std::cout << pt.time_of_day() << std::endl; 
} 
```

*   下载源代码

类 `boost::posix_time::second_clock` 表示一个返回当前时间的时钟。 `universal_time()` 方法返回 UTC 时间，如上例所示。 如果需要本地时间，则必须使用 `local_time()`。

Boost.DateTime 还提供了一个名为 `boost::posix_time::microsec_clock` 的类，它返回包含微秒在内的当前时间，供需要更高精度时使用。

要将一个保存在字符串中的时间点转换为类型为 `boost::posix_time::ptime` 的对象，可以用 `boost::posix_time::from_iso_string()` 这样的自由函数，它要求传入的时间点以 ISO 8601 格式提供。

除了 `boost::posix_time::ptime`, Boost.DateTime 也提供了 `boost::posix_time::time_duration` 类，用于指定一个时间长度。 这个类前面已经提到过，因为 `boost::posix_time::ptime` 的构造函数实际上需要一个 `boost::posix_time::time_duration` 类型的对象作为其第二个参数。 当然，`boost::posix_time::time_duration` 也可以单独使用。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::time_duration td(16, 30, 0); 
  std::cout << td.hours() << std::endl; 
  std::cout << td.minutes() << std::endl; 
  std::cout << td.seconds() << std::endl; 
  std::cout << td.total_seconds() << std::endl; 
} 
```

*   下载源代码

`hours()`, `minutes()` 和 `seconds()` 均返回传给构造函数的各个值，而象 `total_seconds()` 这样的方法则返回总的秒数，以简单的方式为你提供额外的信息。

可以传递任意值给 `boost::posix_time::time_duration`，因为没有象 24 小时这样的上限存在。

和历法日期一样，时间点与时间长度也可以执行运算。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt1(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(12, 0, 0)); 
  boost::posix_time::ptime pt2(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(18, 30, 0)); 
  boost::posix_time::time_duration td = pt2 - pt1; 
  std::cout << td.hours() << std::endl; 
  std::cout << td.minutes() << std::endl; 
  std::cout << td.seconds() << std::endl; 
} 
```

*   下载源代码

如果两个 `boost::posix_time::ptime` 类型的时间点相减，结果将是一个 `boost::posix_time::time_duration` 类型的对象，给出两个时间点之间的时间长度。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt1(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(12, 0, 0)); 
  boost::posix_time::time_duration td(6, 30, 0); 
  boost::posix_time::ptime pt2 = pt1 + td; 
  std::cout << pt2.time_of_day() << std::endl; 
} 
```

*   下载源代码

正如这个例子所示，时间长度可以被增加至一个时间点上，以得到一个新的时间点。 以上程序将打印 `18:30:00` 到标准输出流。

你可能已经留意到，Boost.DateTime 对于历法日期和时间使用了相同的概念。 就象有时间类和时间长度类一样，也有一个时间区间的类。 对于历法日期，这个类是 `boost::gregorian::date_period`; 对于时间则是 `boost::posix_time::time_period`。 这两个类均要求传入两个参数给构造函数： `boost::gregorian::date_period` 要求两个历法日期，而 `boost::posix_time::time_period` 则要求两个时间。

```cpp
#include <boost/date_time/posix_time/posix_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt1(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(12, 0, 0)); 
  boost::posix_time::ptime pt2(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(18, 30, 0)); 
  boost::posix_time::time_period tp(pt1, pt2); 
  std::cout << tp.contains(pt1) << std::endl; 
  std::cout << tp.contains(pt2) << std::endl; 
} 
```

*   下载源代码

大致上说，`boost::posix_time::time_period` 非常象 `boost::gregorian::date_period`。 它提供了一个名为 `contains()` 的方法，对于位于该时间区间内的每一个时间点，它返回 `true`。 由于传给 `boost::posix_time::time_period` 的构造函数的结束时间不是时间区间的一部分，所以上例中第二个 `contains()` 调用将返回 `false`。

`boost::posix_time::time_period` 还提供了其它方法，如 `intersection()` 和 `merge()` 分别用于计算两个重叠时间区间的交集，以及合并两个相交区间。

最后，迭代器 `boost::posix_time::time_iterator` 用于对时间点进行迭代。

```cpp
#include <boost/date_time/local_time/local_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::posix_time::ptime pt(boost::gregorian::date(2009, 1, 05), boost::posix_time::time_duration(12, 0, 0)); 
  boost::posix_time::time_iterator it(pt, boost::posix_time::time_duration(6, 30, 0)); 
  std::cout << *++it << std::endl; 
  std::cout << *++it << std::endl; 
} 
```

*   下载源代码

以上程序使用了迭代器 `it` 从时间点 `pt` 开始向前跳 6.5 个小时 。 由于迭代器被递增两次，所以相应的输出分别为 `2009-Jan-05 18:30:00` 和 `2009-Jan-06 01:00:00`。

## 10.4\. 位置相关的时间

和前一节所介绍的位置无关时间不一样，位置相关时间是要考虑时区的。 为此，Boost.DateTime 提供了 `boost::local_time::local_date_time` 类，它定义于 `boost/date_time/local_time/local_time.hpp`, 并使用 `boost::local_time::posix_time_zone` 来保存时区信息。

```cpp
#include <boost/date_time/local_time/local_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::local_time::time_zone_ptr tz(new boost::local_time::posix_time_zone("CET+1")); 
  boost::posix_time::ptime pt(boost::gregorian::date(2009, 1, 5), boost::posix_time::time_duration(12, 0, 0)); 
  boost::local_time::local_date_time dt(pt, tz); 
  std::cout << dt.utc_time() << std::endl; 
  std::cout << dt << std::endl; 
  std::cout << dt.local_time() << std::endl; 
  std::cout << dt.zone_name() << std::endl; 
} 
```

*   下载源代码

`boost::local_time::local_date_time` 的构造函数要求一个 `boost::posix_time::ptime` 类型的对象作为其第一个参数，以及一个 `boost::local_time::time_zone_ptr` 类型的对象作为第二个参数。 后者只不过是 `boost::shared_ptr&lt;boost::local_time::posix_time_zone&gt;` 的类型定义。 换句话说，并不是传递一个 `boost::local_time::posix_time_zone` 对象，而是传递一个指向该对象的智能指针。 这样，多个 `boost::local_time::local_date_time` 类型的对象就可以共享时区信息。 只有当最后一个对象被销毁时，相应的表示时区的对象才会被自动释放。

要创建一个 `boost::local_time::posix_time_zone` 类型的对象，就要将一个描述该时区的字符串作为单一参数传递给构造函数。 以上例子指定了欧洲中部时区：CET 是欧洲中部时间(Central European Time)的缩写。 由于 CET 比 UTC 早一个小时，所以时差以 +1 表示。 Boost.DateTime 并不能解释时区的缩写，也就不知道 CET 的意思。 所以，必须以小时数给出时差；传入 +0 表示没有时差。

在执行时，该程序将打印 `2009-Jan-05 12:00:00`, `2009-Jan-05 13:00:00 CET`, `2009-Jan-05 13:00:00` 和 `CET` 到标准输出流。 用以初始化 `boost::posix_time::ptime` 和 `boost::local_time::local_date_time` 类型的值缺省总是与 UTC 时区相关的。 只有当一个 `boost::local_time::local_date_time` 类型的对象被写出至标准输出流时，或者调用 `local_time()` 方法时，才使用时差来计算本地时间。

```cpp
#include <boost/date_time/local_time/local_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::local_time::time_zone_ptr tz(new boost::local_time::posix_time_zone("CET+1")); 
  boost::posix_time::ptime pt(boost::gregorian::date(2009, 1, 5), boost::posix_time::time_duration(12, 0, 0)); 
  boost::local_time::local_date_time dt(pt, tz); 
  std::cout << dt.local_time() << std::endl; 
  boost::local_time::time_zone_ptr tz2(new boost::local_time::posix_time_zone("EET+2")); 
  std::cout << dt.local_time_in(tz2).local_time() << std::endl; 
} 
```

*   下载源代码

通过使用 `local_time()` 方法，时区的偏差才被考虑进来。 为了计算 CET 时间，需要往保存在 `dt` 中的 UTC 时间 12 PM 上加一个小时，因为 CET 比 UTC 早一个小时。 `local_time()` 会相应地输出 `2009-Jan-05 13:00:00` 到标准输出流。

相比之下，`local_time_in()` 方法是在所传入参数的时区内解释保存在 `dt` 中的时间。 这意味着 12 PM UTC 相当于 2 PM EET，即东部欧洲时间，它比 UTC 早两个小时。

最后，Boost.DateTime 通过 `boost::local_time::local_time_period` 类提供了位置相关的时间区间。

```cpp
#include <boost/date_time/local_time/local_time.hpp> 
#include <iostream> 

int main() 
{ 
  boost::local_time::time_zone_ptr tz(new boost::local_time::posix_time_zone("CET+0")); 
  boost::posix_time::ptime pt1(boost::gregorian::date(2009, 1, 5), boost::posix_time::time_duration(12, 0, 0)); 
  boost::local_time::local_date_time dt1(pt1, tz); 
  boost::posix_time::ptime pt2(boost::gregorian::date(2009, 1, 5), boost::posix_time::time_duration(18, 0, 0)); 
  boost::local_time::local_date_time dt2(pt2, tz); 
  boost::local_time::local_time_period tp(dt1, dt2); 
  std::cout << tp.contains(dt1) << std::endl; 
  std::cout << tp.contains(dt2) << std::endl; 
} 
```

*   下载源代码

`boost::local_time::local_time_period` 的构造函数要求两个类型为 `boost::local_time::local_date_time` 的参数。 和其它类型的时间区间一样，第二个参数所表示的结束时间并不包含在区间之内。 通过如 `contains()`, `intersection()`, `merge()` 以及其它的方法，时间区间可以与其它 `boost::local_time::local_time_period` 相互操作。

## 10.5\. 格式化输入输出

本章中的所有例子在执行后都提供形如 `2009-Jan-07` 这样的输出结果。 有的人可能更喜欢用其它格式来显示结果。 Boost.DateTime 允许 `boost::date_time::date_facet` 和 `boost::date_time::time_facet` 类来格式化历法日期和时间。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 
#include <locale> 

int main() 
{ 
  boost::gregorian::date d(2009, 1, 7); 
  boost::gregorian::date_facet *df = new boost::gregorian::date_facet("%A, %d %B %Y"); 
  std::cout.imbue(std::locale(std::cout.getloc(), df)); 
  std::cout << d << std::endl; 
} 
```

*   下载源代码

Boost.DateTime 使用了 locales 的概念，它来自于 C++ 标准，在 第五章 *字符串处理* 中有概括的介绍。 要格式化一个历法日期，必须创建一个 `boost::date_time::date_facet` 类型的对象并安装在一个 locale 内。 一个描述新格式的字符串被传递给 `boost::date_time::date_facet` 的构造函数。 上面的例子传递的是 `%A, %d %B %Y`，指定格式为：星期几后跟日月年全名： `Wednesday, 07 January 2009`。

Boost.DateTime 提供了多个格式化标志，标志由一个百分号后跟一个字符组成。 Boost.DateTime 的文档中对于所支持的所有标志有一个完整的介绍。 例如，%A 表示星期几的全名。

如果应用程序的基本用户是位于德国或德语国家，最好可以用德语而不是英语来显示星期几和月份。

```cpp
#include <boost/date_time/gregorian/gregorian.hpp> 
#include <iostream> 
#include <locale> 
#include <string> 
#include <vector> 

int main() 
{ 
  std::locale::global(std::locale("German")); 
  std::string months[12] = { "Januar", "Februar", "März", "April", "Mai", "Juni", "Juli", "August", "September", "Oktober", "November", "Dezember" }; 
  std::string weekdays[7] = { "Sonntag", "Montag", "Dienstag", "Mittwoch", "Donnerstag", "Freitag", "Samstag" }; 
  boost::gregorian::date d(2009, 1, 7); 
  boost::gregorian::date_facet *df = new boost::gregorian::date_facet("%A, %d. %B %Y"); 
  df->long_month_names(std::vector<std::string>(months, months + 12)); 
  df->long_weekday_names(std::vector<std::string>(weekdays, weekdays + 7)); 
  std::cout.imbue(std::locale(std::cout.getloc(), df)); 
  std::cout << d << std::endl; 
} 
```

*   下载源代码

星期几和月份的名字可以通过分别传入两个数组给 `boost::date_time::date_facet` 类的 `long_month_names()` 和 `long_weekday_names()` 方法来修改，这两个数组分别包含了相应的名字。 以上例子现在将打印 `Mittwoch, 07\. Januar 2009` 到标准输出流。

Boost.DateTime 在格式化输入输出方面是非常灵活的。 除了输出类 `boost::date_time::date_facet` 和 `boost::date_time::time_facet` 以外，类 `boost::date_time::date_input_facet` 和 `boost::date_time::time_input_facet` 可用于格式化输入。 所有这四个类都提供了许多方法，来为 Boost.DateTime 所提供的各种不同对象配置输入和输出的方式。 例如，可以指定 `boost::gregorian::date_period` 类型的时间长度如何输入和输出。 要弄清楚各种格式化输入输出的可能性，请参考 Boost.DateTime 的文档。

## 10.6\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  创建一个程序，打印下一个 Christmas Eve, Christmas Day 及其后一天是星期几到标准输出流。

2.  以天数计算你的年龄。 该程序应该自动根据当前日期来计算。