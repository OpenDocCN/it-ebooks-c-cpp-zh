# 第十三章数据结构类

存储和处理数据是所有应用程序必须的一环.从最简单的用于保存大小和位置信息的类,到复杂的数据类型比如链表和哈希表等,wxWidgets 提供了一整套可选的数据结构类.本章将展示很多数据结构类,并着重展示每种数据结构类最常用的函数及方法.对于那些比较少用到的数据结构类及其方法,请参考 wxWidgets 的相关文档.

另外,这本书将不会涉及这些数据结构的内部实现,尽管如此,就算不知道其内部是怎样实现的,每个人也都需要知道这些结构应该怎样被使用.

# 13.1 为什么没有使用 STL?

首先,我们来回答一个关于 wxWidgets 的数据结构类问的最多的一个问题:"为什么它们不采用基于 STL(标准模板库)的实现?".最主要的原因是历史原因:wxWidgets 从 1992 年就存在了,这比可以稳定而可靠的支持跨平台交叉编译的 STL 库要早很久.不过随着 wxWidgets 的发展,它的许多数据结构类已经拥有了一个和标准 STL 非常相似的 API,希望有一天,wxWidgets 中的某些数据结构类可以实现完全的 STL 兼容.

尽管这样,你还是可以在你的 wxWidgets 应用程序中使用 STL,这需要你将 setup.h 中的 wxUSE_STL 置为 1(或者在配置 wxWidgets 的时候使用 enable-stl 选项),以便使得 wxString 和别的容器类使用等价的 STL 实现.不过需要事先声明,在 wxWidgets 中允许 STL 将加大 wxWidgets 库的大小,并且将延长 wxWidgets 的编译时间,尤其在使用 GCC 的时候更加明显.

# 13.2 字符串类型

使用字符串类来代替标准的字符串指针的好处是被普遍接受的.而 wxWidgets 就提供了它自己的字符串类:wxString,无论在 wxWidgets 内部还是在其提供的 API 接口上,这个类都被很广泛的使用.wxString 类拥有你对一个字符串类期待的所有的操作,包括:动态内存管理,从其它字符串类型构建,赋值操作,单个字符访问,字符串连接和比较,子字符串获取,大小写转换,空格字符的修剪和补齐,查找和替换,类似 C 语言 printf 的操作以及类似流一样的插入函数等等.

除了上述的这些字符串处理常用功能,wxString 还支持一些额外的特性.wxString 完美支持 Unicode,包括 ANSI 字符和 Unicode 的互相转换,这个特性是和 wxWidgets 的编译配置无关的.使用 wxString 还使得你的代码拥有直接将字符串传递给库函数以及直接从库函数返回字符串的能力.另外,wxString 已经实现了 90%的 STL 中的 std::string 类的函数,这意味着对 STL 熟悉的用户基本上不需要重新学习 wxString 的使用方法.

使用 wxString

在你的应用程序中使用 wxString 类型是非常简单而直接的.将你程序中使用 std::string 或者是别的你习惯的字符串类的地方,全部用 wxString 代替基本就可以了.要注意的是,所有参数中使用字符串的地方,最好使用 const wxString&这样的声明(这使得函数内部对于字符串的赋值操作由于使用了引用记数器技术而变得更快速),而所有返回值中使用的字符串则最好直接使用 wxString 类型,这使得在函数内部即使返回一个局部变量也是安全的.

C 和 C++的程序员通常都已经很熟悉字符串的各种操作了,因此 wxString 的详细的 API 参考就不在这里罗嗦了,请参考 wxWidgets 的相关文档.

你可能会注意到 wxString 的好多函数具有同样的功能,比如 Length,Len 和 length 函数都返回这个字符串的长度.在这种情况下,你最好使用标准 STL 兼容的函数形式.这会让你的代码对别的程序员来说更亲切,并且将使你的代码更容易转换为别的不使用 wxWidgets 库的代码,你甚至可以直接使用 typedef 将 wxString 重定义为 std::string.另外 wxWidgets 某一天可能会开始使用标准的 std:: string,因此这种作法也会让你的代码更容易保持前向兼容.(当然,wxString 的函数也会保留以保证后向兼容.)

wxString,字符以及字符串常量

wxWidgets 定义了一个 wxChar 类型,在不同的编译选项(ANSI 或 Unicode)下,这个类型用来映射 char 类型或者 wchar_t 类型.象前面提到的那样,你不必使用单独的 char 类型或者 wchar_t 类型,wxString 内部存储数据的时候使用的是相应的 C 类型.在任何时候,如果你需要对单个字符进行操作,你应该使用 wxChar 类型,这将使得你的代码在 ANSI 版本和 Unicode 版本中保持一致,而不必使用大量的预定义宏.

如果 wxWidgets 被编译成 Unicode 的版本,和标准字符串常量是不兼容的,因为标准字符串常量无论在哪种版本中都是 char*类型.如果你想在 Unicode 版本中直接使用字符串常量,你应该使用一个转义宏 L.wxWidgets 提供了一个宏 wxT(或者 _T)来封装字符串常量,这个宏在 ANSI 版本中被定义为什么事情也不做,而在 Unicode 版本中则用来封装 L 宏,因此无论在哪种版本中,你都可以使用下面的方法使用字符串常量:

```cpp
wxChar ch = wxT('*');
wxString s = wxT("Hello, world!");
wxChar* pChar = wxT("My string");
wxString s2 = pChar; 
```

关于使用 Unicode 版本的更详细的信息,请参考第十六章:"编写国际化应用程序".

wxString 到 C 指针的转换基础

因为有时候你需要直接以 C 类型访问 wxString 的内部数据进行底层操作,wxWidgets 提供了几种对应的访问方法:

*   mb_str 函数无论在 ANSI 版本 还是 Unicode 版本都返回一个 const char*类型的指针 const char*, 如果是 Unicode 版本,则字符串首先经过转换,转换过程可能导致数据丢失.
*   wc_str 函数无论在 ANSI 版本还是 Unicode 版本都返回一个 wchar_t*类型,如果是 ANSI 版本,则字符串首先被转换成 Unicode 版本然后再返回.
*   c_str 则返回一个指向内部数据的指针 (ANSI 版本为 const char*类型, Unicode 版本为 const wchar_t*类型).不进行任何转换.

你可以使用 c_str 函数的特性实现 wxString 和 std::string 之间的转换,如下所示:

```cpp
std::string str1 = wxT("hello");
wxString str2 = str1.c_str();
std::string str3 = str2.c_str(); 
```

使用 wxString 经常遇到的一个陷井是过度使用对 const char*类型的隐式的类型强制转换.我们建议你在任何需要使用这种转换的时候,显式使用 c_str 来指明这种转换,下面的代码演示了两个常见的错误:

```cpp
// 这段代码将输入的字符串转换为大写函数,然后将其打印在屏幕上
// 并且返回转换以后的值 (这是一个充满 bug 的代码)
const char *SayHELLO(const wxString& input)
{
    wxString output = input.Upper();
    printf("Hello, %s!\n", output);
    return output;
} 
```

上面这四行代码有两个危险的缺陷,第一个是对 printf 函数的调用.在类似 puts 这样的函数中,隐式的类型强制转换是没有问题的,因为 puts 声明其参数为 const char*类型,但是对于 printf 函数,它的参数采用的是可变参数类型,这意味着上述 printf 代码的执行结果可能是任何一个种结果(包括正确打印出期待结果),不过最常见的一种结果是程序异常退出,因此,应该使用下面的代码代替上面的 printf 语句:

```cpp
printf(wxT("Hello, %s!\n"), output.c_str()); 
```

第二个错误在于函数的返回值.隐式类型强制转换又被使用了以此,因为这个函数的返回值是 const char*类型,这样的代码编译是没有问题的,但是它返回的将是一个局部变量的内部指针,而这个局部变量在函数返回以后就很快被释放了,因此返回的指针将变成一个无效指针.解决的方法很简单,应该将返回类型更改为 wxString 类型,下面列出了修改了以后的代码:

```cpp
// 这段代码将输入的字符串转换为大写函数,然后将其打印在屏幕上
// 并且返回转换以后的值 (这是正确的代码)
wxString SayHELLO(const wxString& input)
{
    wxString output = input.Upper();
    printf(wxT("Hello, %s!\n"), output.c_str());
    return output;
} 
```

标准 C 的字符串处理函数

因为大多数的应用程序都要处理字符串,因此标准 C 提供了一套相应的函数库.不幸的是,它们中的一部分是有缺陷的(比如 strncpy 函数有时候不会添加结束符 NULL),另外一部分则可能存在缓冲区溢出的危险.而另一方面,一些很有用的函数却没能够进入标准的 C 函数库.这些都是为什么 wxWidgets 要提供自己的额外的全局静态函数的原因,wxWidgets 的一些静态函数视图避免这些缺陷:wxIsEmpty 函数增加了对字符串是否为 NULL 的校验, 在这种情况下也返回 True.wxStrlen 函数也可以处理 NULL 指针,而返回 0.wxStricmp 函数则是一个平台相关的大小写敏感字符串比较函数,它在某些平台上使用 stricmp 函数而在另外一些平台上则使用 strcasecmp 函数.

"wx/string.h"头文件中定义了 wxSnprintf 函数和 wxVsnprintf,你应该使用它们代替标准的 sprintf 函数以避免一些 sprintf 函数先天的危险.带"n"的函数使用了 snprintf 函数,这个函数在可能的时候对缓冲区进行大小检查.你还可以使用 wxString::Printf 而不必担心遭受可能受到的针对 printf 的缓冲区溢出攻击.

和数字的相互转换

应用程序经常需要实现数字和字符串之间的转换,比如将用户的输入转换成数字或者将计算的结果显示在用户街面上.

ToLong(long* val, int base=10)函数可以将字符串转换成一个给定进制的有符号整数.它在成功的时候返回 True 并将结果保存在 val 中,如果返回值是 False,则表明字符串不是一个有效的对应的进制的数字.指定的进制必须是 2 到 36 的整数,0 意味着根据字符串的前导符决定: 0x 开头的字符串被认为是 16 进制的, 0-则被认为是 8 进制的, 其它情况下认为是 10 进制的.

ToULong(unsigned long* val, int base=10)的工作模式和 ToLong 函数一致,不过它的转换结果为无符号类型.

ToDouble(double* val)则实现字符串到浮点数的转换.返回值为 Bool 类型.

Printf(const wxChar* pszFormat, ...) 和 C 语言的 sprintf 函数类似,将格式化的文本作为自己的内容.返回值为填充字符串的长度.

静态函数 Format(const wxChar* pszFormat, ...)则将格式化的字符串作为返回值.因此你可以使用下面的代码:

```cpp
int n = 10;
wxString s = "Some Stuff";
s += wxString::Format(wxT("%d"),n ); 
```

操作符"<<"可以用来在 wxString 中添加一个 int,float 或者是 double 类型的值.

wxStringTokenizer

wxStringTokenizer 帮助你将一个字符串分割成几个小的字符串,它被用类代替和扩展标准 C 函数 strtok,它的使用方法是:传递一个字符串和一个可选的分割符(默认为空白符),然后循环调用 GetNextToken 函数直到 HasMoreTokens 返回 False,如下所示:

```cpp
wxStringTokenizer tkz(wxT("first:second:third:fourth"), wxT(":"));
while ( tkz.HasMoreTokens() )
{
    wxString token = tkz.GetNextToken();
    // 处理单个字符串
} 
```

默认情况下,wxStringTokenizer 对于全空字符串的处理和 strtok 的处理相同,但是和标准函数不同的是,如果分割符为非空字符,它将把空白部分也作为一个子字符串返回.这对于处理那些格式化的表格数据(每一行的列数相同但是单元格数据可能为空)是比较有好处的,比如使用 tab 或者逗号作为分割符的情况.

wxStringTokenizer 的行为还受最后一个参数的影响,相关的描述如下:

*   wxTOKEN_DEFAULT: 如前所述的默认处理方式; 如果分割符为空白字符则等同于 wxTOKEN_STRTOK,否则等同于 wxTOKEN_RET_EMPTY.
*   wxTOKEN_RET_EMPTY: 在这种模式,空白部分将作为一个子字符串部分被返回,例如"a::b:"如果用":"分割则返回三个子字符串 a, ""和 b.
*   wxTOKEN_RET_EMPTY_ALL: 在这种模式下,最后的空白部分也将作为一个子字符串返回. 这样"a::b:"使用":"分割将返回四个子字符串,其三个和 wxTOKEN_RET_EMPTY 返回的相同,最后一个则为一个"".
*   wxTOKEN_RET_DELIMS: 在这种模式下,分割符也作为子字符串的一部分(除了最后一个子字符串,它是没有分割符的),其它方面类似 wxTOKEN_RET_EMPTY.
*   wxTOKEN_STRTOK: 这种情况下,子字符串的产生结果和标准 strtok 函数完全相同.空白字符串将不作为一个子字符串.

wxStringTokenizer 还有下面两个有用的成员函数:

*   CountTokens 函数返回分割完的子字符串的数目.
*   GetPosition 返回某个位置的子字符串.

wxRegEx

wxRegEx 类用来实现正则表达式.这个类支持的操作包括正则表达式查找和替换.其实现方式有基于系统正则表达式库(比如现代的类 Unix 系统以及 Mac OSX 支持的 POSIX 标准正则表达式库)或者基于由 Henry Spencer 提供的 wxWidgets 内建库.POSIX 定义的正则表达式有基础和扩展两套版本.内建的版本支持这两种模式而基于系统库的版本则不支持扩展模式.

即使是对于那些支持正则表达式库的系统,wxWidgets 默认的 Unicode 版本也采用了内建的正则表达式版本,ANSI 版本则使用系统提供的版本.记住只有内建版本的正则表达式库才能完全支持 Unicode.当编译 wxWidgets 的时候,覆盖这种默认设置是被允许的.如果在使用系统正则表达式库的 Unicode 版本中,在使用对应函数的之前,表达式和要匹配的数据都将被转换成 8-bit 编码的 Unicode 方式.

使用 wxRegEx 的方法和其它所有使用正则表达式的方法没有区别.因为正则表达式的内容较为罗嗦而且又鉴于正则表达式的只在特定情况下使用,请参考 wxWidgets 手册中的相关内容了解具体的 API.

# 13.3 wxArray

wxWidgets 使用 wxArray 提供了一种动态的数组结构.和 C 语言的数组结构类似,对于其数组项的访问时间为一个常量.对然这样,其内存仍然是动态分配的,换句话说,如果其内存不够再增加子项的时候,它将动态分配内存.在提前分配内存的前提下,其增加数组项的时间也大体上是一个常量. wxArray 类型还提供了边界检查的功能,在调试版本中,越界访问将会导致断言错误,而在正式版本中,越界访问将不会出现任何提示,而这种访问的结果可能会是一个随机值.

数组类型

wxWidgets 提供了三种不同类型的数组.它们都是 wxBaseArray 的派生类,在它们的数值类型未定义之前是不能直接使用的. 换句话说,你必须使用 WX_DEFINE_ARRAY, WX_DEFINE_SORTED_ARRAY 和 WX_DEFINE_OBJARRAY 宏来定义一个它们的派生类才可以使用它们.这种基类的名称分别为 wxArray,wxSortedArray 和 wxObjArray,但是你应该有这个概念:这三个类其实是不存在的,并没有这样的类,这个名称只是为了用来说明问题用的.

wxArray 这种类型,它的派生类可以用来存储整数类型以及指针类型,这种类型永远不会将它的成员按照对象来对待,也就是说:数组中的元素(无论是整数还是指针)从数组移除的时候并不会被释放.还应该提到的是:wxArray 类型的成员函数都是内联函数,因此程序的大小和运行效率完全不受其派生类个数的影响.这种类型最大的限制在于,它只能存储整形数据,比如 bool,char,int,long 和它们的无符号变体或者任何类型的指针.而浮点行或者 double 型的数据是不可以用 wxArray 来存储的.

wxSortedArray 和 wxArray 比较,区别在于,当你需要很频繁的数组进行查找操作时,你应该使用前者.它需要你定义一个比较函数,通过这个比较函数,它将把其内部的元素始终按顺序排列,如果你拥有大量数据并且需要频繁查找,那么使用它性能比使用 wxArray 要好的多.其它方面两者拥有同样的限制,wxSortedArray 也只能存储整形数据.

wxObjArray 类则将其内部存储的元素按照对象来对待.它知道在元素从数组中移除的时候释放相应的内存(通过调用其析构函数), 并且使用用于拷贝的构造函数实现拷贝.要定义一个这种类型的派生对象需要两个步骤.首先,使用 WX_DECLARE_OBJARRAY 宏来声明一个 wxObjArray,然后包含其实现文件<wx/arrimpl.cpp>并且在完整声明了其数据元素对象的地方调用 WX_DEFINE_OBJARRAY 宏.读起来有些绕口,不过我们很快会举一个简单的例子.

wxArrayString

wxArrayString 是存储 wxString 类型的一个很经济有效的类,它拥有和 wxArray 类完全一致的功能.它占用的空间也要比直接使用 C 数组类型 wxString[]占用的空间要小的多(这是因为它使用了一些直接对 wxString 类内部进行操作的方法).所有在 wxArray 中可以使用的函数都可以在 wxArrayString 中使用.

这个类使用上也和其它类差别不大,唯一的区别在于不需要象别的类那样使用 WX_DEFINE_ARRAY 宏,而可以直接在代码中使用. 当一个 wxString 实例被插入这个数组时,wxArrayString 将创建一份这个字符串的拷贝,应该在成功插入以后,你可以放心的释放原来的字符串.一般情况下,你也不需要关心 wxArrayString 的内存分配问题,它可以自己释放它所占用的所有的内存.

另外注意 Item, Last 和操作符[]返回的只是引用而不是拷贝,因此,对它们返回值的操作将导致数组内部数据的改变,如下所示:

```cpp
array.Last().MakeUpper(); 
```

对应的也有一个 wxSortedArrayString 对象,这个对象中的字符串总是按照子母顺序排序的.在得到对应字符串的 Index 时,wxSortedArrayString 使用二进制搜索方式,性能很高.因此如果你的程序中插入字符串的操作很少,而对其进行搜索的操作很多,你可以考虑使用这个类.需要注意的是不要调用这个类的 Insert 和 Sort 函数,这两个函数可能会搅乱 wxSortedArrayString 的内部排序.

数组的构造,析构和内存管理

数组对象也是通用的 C++对象,也有对应的构造和赋值操作.拷贝一个 wxArray 对象意味着其内部元素的拷贝而拷贝一个 wxObjArray 对象则是直接拷贝这个数组的子项.不过,出于内存效率的考虑,这两个类都没有虚的析构函数.对于 wxArray 来说这并不是很重要, 因为其析构函数不需要作什么太多的事情,而对于 wxObjArray 来说,绝对不要通过删除一个指向 wxBaseArray 类型的指针来释放相应的数组 (译者注:这将导致对应的析构函数不被调用),并且也永远不要从你自己的数组类再派生新的类型(译者注:同样的原因,新类型的释放将导致使用宏定义的类型的析构函数不被调用).

数组的内存自动管理机制也是很简单的:它在开始的时候会预分配一块小的内存(由宏 WX_ARRAY_DEFAULT_INITIAL_SIZE 指定).当发现不够用的时候,就增加当前内存的 50%(但是不超过 ARRAY_MAXSIZE_INCREMENT).Shrink 函数可以用来在没有新数据插入的时候释放多余的内存.而 Alloc 函数可以在你知道总共需要多少内存的时候被调用以便数组对象不那么频繁的进行分配内存的操作.

数组示例:

下面的例子演示了使用数组最复杂的情况,定义和使用针对自定义类型的 wxObjArray 数组.使用 wxArray 的基本原则和例子中演示的是相似的,只不过 wxArray 类永远不需要和自己内部存储的数据发生什么关系.

```cpp
// 我们自定义的数据类型
class Customer
{
public:
    int CustID;
    wxString CustName;
};
// 这一部分的代码可以位于头文件或者源文件(.cpp)文件中
// 用于声明我们的自定义数组:
WX_DECLARE_OBJARRAY(Customer, CustomerArray);
// 增加下面语句的唯一的要求是,自定义的 Customer 类型已经完全声明
// (对于 WX_DECLARE_OBJARRAY 来说,前面的声明已经足够了)
// 而且通常它应该被放在源文件中而不是头文件中.
#include &lt;wx/arrimpl.cpp&gt;
WX_DEFINE_OBJARRAY(CustomerArray);
// 用于排序的时候对两个对象进行比较
int arraycompare(Customer** arg1, Customer** arg2)
{
    return ((*arg1)->CustID &lt; (*arg2)->CustID);
}
// 数组测试例子
void ArrayTest()
{
    // 定义一个我们数组的实例
    CustomerArray MyArray;
    bool IsEmpty = MyArray.IsEmpty(); // will be true
    // 创建一些自定义对象实例
    Customer CustA;
    CustA.CustID = 10;
    CustA.CustName = wxT("Bob");
    Customer CustB;
    CustB.CustID = 20;
    CustB.CustName = wxT("Sally");
    Customer* CustC = new Customer();
    CustC->CustID = 5;
    CustC->CustName = wxT("Dmitri");
    // 将其中两个增加到数组中
    MyArray.Add(CustA);
    MyArray.Add(CustB);
    // 将最后一个插入到数组的任意位置.
    // 数组将会产生一个自定义对象的拷贝
    MyArray.Insert(*CustC, (size_t)0);
    int Count = MyArray.GetCount(); // will be 3
    // 如果找不到将返回 wxNOT_FOUND
    int Index = MyArray.Index(CustB); // will be 2
    // 依次处理数组中的对象
    for (unsigned int i = 0; i &lt; MyArray.GetCount(); i++)
    {
        Customer Cust = MyArray[i]; // 或者使用 MyArray.Item(i);
        // 进行一些处理
    }
    // 按照自己提供的比较函数进行排序
    MyArray.Sort(arraycompare);
    // 移除但不释放 A 对象
    Customer* pCustA = MyArray.Detach(1);
    // 如果使用 Detach 函数,我们需要自己释放对象
    delete pCustA;
    // 如果使用 Remove 函数,就不需要了
    MyArray.RemoveAt(1);
    // Clear 函数也不需要
    MyArray.Clear();
    // 数组从来就不会理会我们自己产生的 C 对象,要自己释放它
    delete CustC;
} 
```

# 13.4 wxList 和 wxNode

wxList 类是一个双向链表,可以用来存储任何类型的数据.wxWidgets 需要你显式的定义一个针对某种数据类型的新的类来使用它,以便对存储于其中的数据提供足够的类型检查.wxList 类还允许你指定一个索引类型以便进行基本的查找操作(如果你想使用基于结构的快速随机访问,请参考下一节的 wxHashMap 类).

wxList 使用了一个虚类 wxNode.当你定义一个新的 wxList 派生类的时候,你同时定义了一个派生自 wxNodeBase 的类,以便对节点提供类型安全检查.节点类最重要的函数包括:GetNext,GetPrevious 和 GetData.它们的功能显而易见,分别为:获取下一个子项,获取前一个子项以及获取子项的数据.

唯一值得说明的是 wxList 的删除操作,默认情况下,从链表中移除一个节点并不会导致节点内部数据的释放.你需要调用 DeleteContents 函数来改变这种默认的行为,设置数据随着节点一起释放.如果你想清除整个链表并且释放其中的数据,你应该先调用 DeleteContents,参数为 True,然后再调用 Clear 函数.

我们用不着在这里把手册的内容重新粘贴一遍.我们将举一个简单的例子来演示怎样创建你自己的链表类型.注意 WX_DECLARE_LIST 宏通常应该位于头文件中,而 WX_DEFINE_LIST 宏通常应该位于源文件中.

```cpp
// 我们将存储于链表的数据类型
class Customer
{
public:
    int CustID;
    wxString CustName;
};
// 通常应该位于头文件中
WX_DECLARE_LIST(Customer, CustomerList);
// 下面的定义应该位于源文件中,并且通常位于所有 Customer 声明之后
#include &lt;wx/listimpl.cpp&gt;
WX_DEFINE_LIST(CustomerList);
// 用于排序的比较函数
int listcompare(const Customer** arg1, const Customer** arg2)
{
    return ((*arg1)->CustID &lt; (*arg2)->CustID);
}
// 链表操作举例
void ListTest()
{
    // 定义一个我们自定义链表的实例
    CustomerList* MyList = new CustomerList();
    bool IsEmpty = MyList->IsEmpty(); // will be true
    // 创建一些自定义对象实例
    Customer* CustA = new Customer;
    CustA->CustID = 10;
    CustA->CustName = wxT("Bob");
    Customer* CustB = new Customer;
    CustB->CustID = 20;
    CustB->CustName = wxT("Sally");
    Customer* CustC = new Customer;
    CustC->CustID = 5;
    CustC->CustName = wxT("Dmitri");
    // 将其增加到链表中
    MyList->Append(CustA);
    MyList->Append(CustB);
    // 实现随机插入
    MyList->Insert((size_t)0, CustC);
    int Count = MyList->GetCount(); // will be 3
    // 如果找不到,返回 wxNOT_FOUND
    int index = MyList->IndexOf(CustB); // will be 2
    // 自定义的节点里包含了我们自定义的类型
    CustomerList::Node* node = MyList->GetFirst();
    // 节点遍历
    while (node)
    {
        Customer* Cust = node->GetData();
        // 进行一些处理
        node = node->GetNext();
    }
    // 返回特定位置的节点
    node = MyList->Item(0);
    // 按照排序函数排序
    MyList->Sort(listcompare);
    // 移除包含某个对象的节点
    MyList->DeleteObject(CustA);
    // 我们需要自己释放这个对象
    delete CustA;
    // 找到包含某个对象的节点
    node = MyList->Find(CustB);
    // 指示内部数据随节点的删除而删除
    MyList->DeleteContents(true);
    // 删除 B 的节点的时候,B 也被释放了
    MyList->DeleteNode(node);
    // 现在调用 Clear,所有的节点和其中数据都将被释放
    MyList->Clear();
    delete MyList;
} 
```

# 13.5 wxHashMap

wxHashMap 类是一个简单的,类型安全的并且效率很不错的哈希映射类,它的接口是标准的 STL 容器接口的一个子集.实际上,它是在标准的 std:: map 和非标准的 std::hash_map 之后才可以设计的.通过用于创建哈希表的宏,你可以选择下面的几种类型及其组合作为哈希表的键类型或者数据类型:int,wxString 或 void*(任意类型).

有三个用来定义哈希映射类的宏.要定义一个名字为 CLASSNAME,键类型为 wxString,值类型为 VALUE_T 类型的哈希表,你可以使用下面的语法:

```cpp
WX_DECLARE_STRING_HASH_MAP(VALUE_T, CLASSNAME); 
```

要定义一个名字为 CLASSNAME,键类型为 void*,值类型为 VALUE_T 类型的哈希表,使用下面的定义:

```cpp
WX_DECLARE_VOIDPTR_HASH_MAP(VALUE_T, CLASSNAME); 
```

要定义一个名称为 CLASSNAME,键类型和值类型任意类型的哈希表,使用下面的定义:

```cpp
WX_DECLARE_HASH_MAP(KEY_T, VALUE_T, HASH_T, KEY_EQ_T, CLASSNAME); 
```

HASH_T 和 KEY_EQ_T 是用来作为哈希算法和比较算法的函数. wxWidgets 提供了三种预定义的哈希算法: wxIntegerHash 用来作为整数的哈希算法(int, long, short 和它们的无符号变体都可以), wxStringHash 用来作为字符串的哈希算法(wxString, wxChar*, char*都可以), wxPointerHash 用来作为任何指针类型的哈希算法.类似的也有三个预定义的比较函数: wxIntegerEqual, wxStringEqual 和 wxPointerEqual.

下面的代码演示了 wxHashMap 的使用方法:

```cpp
// 我们要存放在哈希表中的类
class Customer
{
    public:
        int CustID;
        wxString CustName;
};
// 定义和实现我们自定义的哈希表.
WX_DECLARE_HASH_MAP(int, Customer*, wxIntegerHash,
                    wxIntegerEqual, CustomerHash);
void HashTest()
{
    // 定义一个自定义哈希表的实例
    CustomerHash MyHash;
    bool IsEmpty = MyHash.empty(); // will be true
    // 创建几个对象
    Customer* CustA = new Customer;
    CustA->CustID = 10;
    CustA->CustName = wxT("Bob");
    Customer* CustB = new Customer;
    CustB->CustID = 20;
    CustB->CustName = wxT("Sally");
    Customer* CustC = new Customer;
    CustC->CustID = 5;
    CustC->CustName = wxT("Dmitri");
    // 将对象增加到哈希表
    MyHash[CustA->CustID] = CustA;
    MyHash[CustB->CustID] = CustB;
    MyHash[CustC->CustID] = CustC;
    int Size = MyHash.size(); // will be 3
    // count 函数返回 0 或 1, 含义为:20 这个关键值在哈希表中吗?
    int Present = MyHash.count(20); //将返回 1
    // 我们哈希表的自定义节点类型
    CustomerHash::iterator i = MyHash.begin();
    // 遍历哈希表
    while (i != MyHash.end())    {
        // first 函数返回键值,second 返回数据
        int CustID = i->first;
        Customer* Cust = i->second;
        // 作一些处理
        // 然后处理下一个数据
        i++;
    }
    // 将键值为 10 的数据移出哈希表
    MyHash.erase(10);
    // 移出不会导致数据自动释放
    delete CustA;
    // 返回指定键值的一个节点
    CustomerHash::iterator i2 = MyHash.find(21);
    // 判断是否找到节点
    bool NotFound = (i2 == MyHash.end()); // 将返回 True
    // 这次将返回有效的节点
    i2 = MyHash.find(20);
    // 直接移除节点
    MyHash.erase(i2);
    delete CustB;
    // 副作用: 下面语句导致哈希表中插入一个键值为 30,值为 NULL 的节点.
    Customer* Cust = MyHash[30]; // Cust 将等于 NULL
    // 清除哈希表中的节点
    MyHash.clear();
    delete CustC;
} 
```

# 13.6 存储和使用日期和时间

wxWidgets 提供了一个功能强大的类 wxDateTime 来进行时间和日期相关的操作,包括:格式化输出,时区,时间和日期计算等等.还提供了一些静态函数来提供当前的日期和时间以及查询某个给定的年份是不是闰年等.注意即使你只想操作日期或者是时间,你仍然可以使用 wxDateTime 类型. wxTimeSpan 和 wxDateSpan 类型提供了修改一个 wxDateTime 值的合适的方法.

wxDateTime

wxDateTime 类拥有很多的成员函数,每个函数的含义都很清晰.完整的 API 可以参考 wxWidgets 的相关手册,下面只对其中使用最频繁的函数进行一些介绍.

注意尽管时间在其内部总是以格林威治时间(GMT)存储的,但是你通常关心的是本地时区的时间而不是格林威治时间,因此, wxDateTime 的构造函数以及更改函数中各个组成时间的要素(比如小时,分钟和秒钟)等都指的是本地时区的时间.所有用于获取时间和日期的函数返回的要素(月,日,小时,分,秒等)也都是本地时间.因此,如果这是你需要的,你不需要作任何额外的操作,如果你希望操作不同时区的时间,请参考相关的文档.

wxDateTime 类的构造和更改

wxDateTime 可以通过 Unix 时间戳,仅包含时间的信息,仅包含日期的信息,完整的时间日期信息等途径创建.对于每一个构造函数,都有一个对应的 Set 函数用来更改已经设置了值的 wxDateTime 对象.也可以通过类似 SetMonth 或者 SetHour 等函数更改时间或者日期中的某个要素.

wxDateTime(time_t)函数根据一个指定的 Unix 时间戳来构造对象.

wxDateTime(const struct tm&)函数根据一个指定的 C 语言标准 tm 结构构造对象.

wxDateTime(wxDateTime_t hour, wxDateTime_t minute = 0, wxDateTime_t second = 0, wxDateTime_t millisec = 0)根据指定的时间要素构造对象.

wxDateTime(wxDateTime_t day, Month month = Inv_Month, int year = Inv_Year, wxDateTime_t hour = 0, wxDateTime_t minute = 0, wxDateTime_t second = 0, wxDateTime_t millisec = 0) 根据指定的时间和日期要素构造对象

wxDateTime 访问方法

大多数 wxDateTime 类的访问函数都是自解释的,比如: GetYear, GetMonth, Getday, GetWeekDay, GetHour, GetMinute, GetSecond, GetMillisecond, GeTDayOfYear, GetWeekOfYear, GetWeekOfMonth 和 GetYearDay 等. wxDateTime 还提供了下列一些访问函数:

*   GetTicks 返回一个 Unix 时间戳(也就是自从 1970 年 1 月 1 日午夜以来的秒数).
*   IsValid 返回时间日期类是不是已经被初始化(类自使用默认构造函数创建以后始终未被赋值).

获取当前时间

wxDateTime 提供了两个静态函数返回当前时间:

*   wxDateTime::Now 返回精度为秒的当前时间.
*   wxDateTime::UNow 返回精度为毫秒的当前时间.

时间和字符串的转换

下面介绍的这些函数用来实现时间和字符串的相互转换.将时间转化成字符串的方法是比较简单的:你可以将时间转换成本地格式的字符串 (FormatDate 和 FormatTime 函数),或者以定义在 ISO 8601 中的国际标准格式来显示(FormatISODate 函数和 FormatISOTime 函数),也可以以自定义的格式来显示(Format 函数).

而从文本到时间的转换则显得更复杂些,因为可能的时间格式太多了.最简单的函数是 ParseFormat 函数,它用来解析那些指定格式的时间文本.ParseRfc822Date 函数用来解析那些定义在 RFC822 中的时间表示方法,这种方法在 email 或者互联网上使用比较普遍.

最有趣的文本到时间转换函数是 ParseTime,ParseDate 和 ParseDateTime 函数,它们将尽量匹配各种格式的时间文本.除了预定义的那些标准格式以外,ParseDateTime 函数甚至可以支持那些类似"tomorrow"(明天), "March first"(三月一日)以及"next Sunday"(下个星期天)这样的时间.

日期比较

两个 wxDateTime 对象可以通过各种函数进行比较,这些函数都返回 bool 类型的值.这些函数包括:IsEqualTo, IsEarlierThan, IsLaterThan, IsSameDate 和 IsSameTime 等.

而 IsStrictlyBetween 和 IsBetween 则用来比较某个日期是不是在两个日期之间.这两个函数的区别在于,如果要比较的时间刚好等于其中的与其比较的边界值的时候,前者返回 False 而后者返回 True.

日期计算

wxWidgets 提供了两个非常灵活的类 wxTimeSpan 和 wxDateSpan 来辅助进行日期和时间的计算. wxTimeSpan 用来计算那些以毫秒为单位的,跨度不大的,快速的和精确的计算,而 wxDateSpan 则用来进行跨度比较大的比如几周或者几个月的计算.wxDateSpan 将尽可能使用更自然的方法来进行计算,因此其含义有时候并不象它看上去的那样精确.比如 1 月 31 日加上一个月将返回二月 28 日 (或 29 日),也就是说是二月的最后一天,而不是永远不可能存在的二月 31 日.通常,你比较喜欢这样的结果,不过有时候相应的减法运算可能也会把你搞糊涂,比如二月 28 日减去一个月的结果的一月 28 日而不是一月 31 日.

日期类型可以进行的操作很多,但是这些操作的组合却未必是有效的.比如:对一个日期进行乘法运算是无效的,而对于任何一个表示时间间隔的类(wxTimeSpan 或 wxDateSpan)进行乘法运算则没有任何问题.

*   加法: wxTimeSpan 或 wxDateSpan 可以和 wxDateTime 进行加法运算,返回一个新的 wxDateTime 对象. 两个相同类型的时间间隔类也可以进行加法运算,返回一个新的同样类型的对象.
*   减法: 减法适用和加法同样的规则,额外的一个规则是两个 wxDateTime 对象相减返回一个 wxTimeSpan 对象.
*   乘法: wxTimeSpan 或 wxDateSpan 对象可以乘以一个整数,返回一个同样类型的对象.
*   Unary 相减:wxTimeSpan 或 wxDateSpan 对象可以定义为负数,导致相反的时间方向上的同样的间隔.

下面的例子演示了 wxDateSpan 和 wxTimeSpan 的用法,更多的用法请参考 wxWidgets 的相关手册.

```cpp
void TimeTests()
{
    // 获取当前时间和日期
    wxDateTime DT1 = wxDateTime::Now();
    // 创建一个 2 星期零 1 天,或者说 15 天的间隔
    wxDateSpan Span1(0, 0, 2, 1);
    // 今天减去 15 天
    wxDateTime DT2 = DT1 - Span1;
    // 用静态方法创建一天的间隔
    wxDateSpan Span2 = wxDateSpan::Day();
    // Span3 将代表 14 天的间隔
    wxDateSpan Span3 = Span1 - Span2;
    // 0 天 (这个间隔将用 2 周来表示)
    int Days = Span3.GetDays();
    // 14 天 (2 周)
    int TotalDays = Span3.GetTotalDays();
    // 之前两周
    wxDateSpan Span4 = -Span3;
    // 3 个月的间隔
    wxDateSpan Span5 = wxDateSpan::Month() * 3;
    // 10 小时 5 分 6 秒的间隔
    wxTimeSpan Span6(10, 5, 6, 0);
    // DT2 增加固定的间隔 Span6
    wxDateTime DT3 = DT2 + Span6;
    // Span7 是相反方向上的 3 倍 Span6 的间隔.
    wxTimeSpan Span7 = (-Span6) * 3;
    // SpanNeg 将返回 True, 这个间隔是负方向的.
    bool SpanNeg = Span7.IsNegative();
    // 适用静态方法创建一个 1 小时的间隔
    wxTimeSpan Span8 = wxTimeSpan::Hour();
    // 1 小时当然小于 30 小时(这里使用绝对值)
    bool Longer = Span8.IsLongerThan(Span7);
} 
```

# 13.7 其它常用的数据类型

wxWidgets 内部使用了一些其它的数据类型,也在一些公用 API 中作为参数和返回值,并且 wxWidgets 也鼓励程序员在它们的代码中使用这些类型.

wxObject

wxObject 类是所有 wxWidgets 类的基类,它提供的功能包括运行期类型信息,引用技术,虚析构函数,可选的调试版本的 new 和 delete 函数等.某些 wxObject 对象的成员函数还使用了用于存储 meta-data 的 wxClassInfo 对象.

```cpp
MyWindow* window = wxDynamicCast(FindWindow(ID_MYWINDOW), MyWindow); 
```

IsKindOf 函数判断对象是否是传入的 wxClassInfo 指向的类型.

```cpp
bool tmp = obj->IsKindOf(CLASSINFO(wxFrame)); 
```

Ref 函数的参数为 const wxObject&类型, 它的作用是将当前对象的数据替换为参数对象的引用.当前对象的引用技术减一,如果需要则释放当前对象数据,参数对象的引用技术则加一.

UnRef 则将对象内部数据的引用记数器减一,如果已经减到 0 则释放当前对象数据.

wxLongLong

wxLongLong 类用来存储 64 位整数.如果本地系统支持 64 位长整数则使用本地系统提供的实现,否则将使用模拟的 64 位实现.这个类的使用和其它标准的数字类型没有区别.注意它是一个有符号数字,如果想使用它的无符号版本,可以使用 wxULongLong 类型,后者的 API 和前者几乎完全一样,除了个别的函数(比如求绝对值函数)可能返回不同的结果.除了一般的计算函数以外,另外的几个常用的函数包括:

*   Abs 函数返回 wxLongLong 的绝对值,如果是作为常量引用调用的这个函数,则返回源对象的一个拷贝,否则将修改源对象的内部数值.
*   ToLong 函数将其转换成一个长整型,如果由于存在精度丢失,在调试版本中将引发一个断言错误.
*   ToString 将内部存储的数据转换成一个 wxString 类型.

wxPoint 和 wxRealPoint

wxPoint 在 wxWidgets 中使用比较普遍,常用来代表屏幕或者窗口上的一个确定的位置.正如它的名字的意思一样,它内部的数据用 x 和 y 两个整数代表一个座标值.其数据成员是以 public 方式定义的,可以直接被其它对象访问.wxPoint 支持和另外一个 wxPoint 对象或者 wxSize 对象进行加法和减法的运算.wxRealPoint 对象和 wxPoint 对象类似,不过其内部成员是 double 类型,并且只支持和别的 wxRealPoint 类型进行加减运算.

构造 wxPoint 实例的方法很直接:

```cpp
wxPoint myPoint(50, 60); 
```

wxRect

wxRect 通常在绘画或者窗口类中使用(比如 wxDC 或者 wxtreeCtrl),用来定义一个矩形区域.其内部的数据成员除了 x 和 y 以外,还包括宽度和高度.所有的成员都是 public 类型,可以直接被其它的类访问.除了同类型之间的加减运算,wxRect 还支持一些其它运算:

GetRight 返回矩形最右边的 X 座标.

GetBottom 返回底端的 Y 座标.

GetSize 返回一个 wxSize 对象用来表征矩形区域的宽度和高度.

Inflate 函数增加矩形区域的大小,如果只有一个参数,则长和宽增加一样的大小,如果是两个参数,则长和宽分别增加对应的大小.

Inside 函数判断某个点是否位于矩形区域以内,点的格式可以是单独的 XY 座标,也可以是一个 wxPoint 类型.

Intersects 判断某个矩形是否和另外一个矩形有重叠区域.

Offset 将当前矩形偏移一段举例,偏移的位置既可以通过 X 和 Y 单独指定,也可以通过 wxPoint 来指定.

下面的代码演示了 wxRect 的三种构造函数:

```cpp
wxRect myRect1(50, 60, 100, 200);
wxRect myRect2(wxPoint(50, 60), wxPoint(150, 260));
wxRect myRect3(wxPoint(50, 60), wxSize(100, 200)); 
```

wxRegion

wxRegion 用来代表设备上下文或者窗口上的一个简单的或者复杂的区域.它使用了引用记数,因此拷贝和赋值操作是非常快速的.它的主要用途是用来定义或者查询某个需要裁剪或者更新的区域.

Contains 函数在其参数指定的座标, wxPoint, 矩形或 wxRect 被包含在区域内时返回 True.

GetBox 函数返回一个包含整个区域的 wxRect 对象.

Intersect 函数在指定的矩形,wxRect 或 wxRegion 参数和本区域有重叠的时候返回 True.

Offset 对区域进行指定 x 和 y 方向数量的平移.

Subtract, Union 和 Xor 函数提供了一种灵活的机制来改变当前区域.这三个函数的变体函数(函数名相同,参数不同)超过 10 个.所有这些函数都可以支持 wxRegion 参数或者 wxPoint 参数.请参考 wxWidgets 的相关手册内容.

下面的代码演示了四种创建区域的方法,所有这些方法创建的结果都是一样的区域:

```cpp
wxRegion myRegion1(50, 60, 100, 200);
wxRegion myRegion2(wxPoint(50, 60), wxPoint(150, 260));
wxRect myRect(50, 60, 100, 200);
wxRegion myRegion3(myRect);
wxRegion myRegion4(myRegion1); 
```

你可以使用 wxRegionIterator 类来遍历某个区域中的矩形区域,比如要在窗口重绘函数中只绘制那些需要绘制的矩形区域,你可以使用下面的代码:

```cpp
// 在窗口需要被重绘的时候调用
void MyWindow::OnPaint(wxPaintEvent& event)
{
    wxPaintDC dc(this);
    wxRegionIterator upd(GetUpdateRegion());
    while (upd)
    {
        wxRect rect(upd.GetRect());
        // 刷新这个矩形区域
        ...一些代码...
        upd ++ ;
    }
} 
```

wxSize

wxSize 类型在 wxWidgets 广泛用于指定窗口,控件,画布等等对象的大小.很多需要返回大小信息的函数也将返回这个对象类型.

GetHeight 和 GetWidth 函数返回高度和宽度.

SetHeight 和 SetWidth 函数设置整数的高度和宽度.

Set 函数则使用两个整数参数来同时改变高度和宽度.

wxSize 的创建也非常简单,如下所示:

```cpp
wxSize mySize(100, 200); 
```

wxVariant

wxVariant 类用来表示那些可以是任意类型的数据.数据的类型甚至可以动态的改变.这种类型在解决某些特定的问题的时候很有用处,比如要编辑不同类型的数据的编辑器或者用于实现远程过程调用.

wxVariant 类型可以存储的数据包括 bool, char, double, long, wxString, wxArrayString, wxList, wxDateTime, void* 和可变类型变量列表.不过,你还是可以通过实现 wxVariantData 的派生类的发生扩展 wxVariant 可以支持的数据类型.在构造和赋值的时候,只需要将其当成 wxVariantData 使用就可以了.当然,不方便的地方在于如果你要访问自定义的数据类型,需要先将其转换成 wxVariantData 对象,而不象内置支持的类型那样,有对应的类似于 GetLong 这样的函数支持.

另外,要记住不是所有的类型都可以互相转换,比如你不可能把一个 bool 型的数据转换成 wxDateTime 类型,也不可能把一个整数转换成 wxArrayString,你需要按照一些常识来判断哪些数据类型是可以互相转换的,并且你总是可以通过 GetType 函数来得到当前数据最合适的类型.下面举一个使用 wxVariant 类的简单的例子:

```cpp
wxVariant Var;
// 存储 wxDateTime 类型, 获取 wxString 类型
Var = wxDateTime::Now();
wxString DateAsString = Var.GetString();
// 存储一个 wxString 类型, 获取一个 double 类型
Var = wxT("10.25");
double StringAsDouble = Var.GetDouble();
// 当前的类型应该是"string"
wxString Type = Var.GetType();
// 下面演示一个无理取闹的转换
// 由于不能转换,所以转换的结果为 0
char c = Var.GetChar(); 
```

# 第十三章小结

wxWidgets 提供的多种数据结构类型让你可以很容易的从 wxWidgets 提供的 API 中或者你自己的应用程序中获取和使用各种类型的结构化数据. 通过功能强大的数据类型 wxRegEx,wxStringTokenizer,wxDateTime,wxVariant 等等,你几乎不需要使用任何第三方的库就可以处理各种数据.

接下来,我们来看看 wxWidgets 提供的从文件或者流中读取或者写入数据的方法.