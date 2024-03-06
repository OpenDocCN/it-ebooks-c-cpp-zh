# 五十一、QByteArray 和 QVariant

前面我们在介绍 QString 的最后部分曾经提到了 QByteArray 这个类。现在我们就首先对这个类进行介绍。

QByteArray 具有类似与 QString 的 API。它也有相应的函数，比如 left(), right(), mid()等。这些函数不仅名字和 QString 一样，而且也具有几乎相同的功能。QByteArray 可以存储原生的二进制数据和 8 位编码的文本数据。这句话怎么理解呢？我们知道，计算机内部所有的数据都是以 0 和 1 的形式存储的。这种形式就是二进制。比如一串 0、1 代码：1000，计算机并不知道它代表的是什么，这需要由上下文决定：它可以是整数 8，也可以是一个 ARGB 的颜色(准确的说，整数 8 的编码并不是这么简单，但我们姑且这个理解吧)。对于文件，即便是一个文本文件，读出时也可以按照二进制的形式读出，这就是二进制格式。如果把这些二进制的 0、1 串按照编码解释成一个个字符，就是文本形式了。因此，QByteArray 实际上是原生的二进制，但是也可以当作是文本，因此拥有文本的一些操作。但是，我们还是建议使用 QString 表示文本，重要的原因是，QString 支持 Unicode。

为了方便期间，QByteArray 自动的保证“最后一个字节之后的那个位”是'\0'。这就使得 QByteArray 可以很容易的转换成 const char *，也就是上一章节中我们提到的那两个函数。同样，作为原生二进制存储，QByteArray 中间也可以存储'\0'，而不必须是'\0'在最后一位。

在有些情况下，我们希望把数据存储在一个变量中。例如，我有一个数组，既希望存整数，又希望存浮点数，还希望存 string。对于 Java 来说，很简单，只要把这个数组声明成 Object[]类型的。这是什么意思呢？实际上，这里用到的是继承。在 Java 中，int 和 float 虽然是原生数据类型，但是它们都有分别对应一个包装类 Integer 和 Float。所有这些 Integer、Float 和 String 都是继承于 Object，也就是说，Integer、Float 和 String 都是一个(也就是 is-a 的关系)Object，这样，Object 的数组就可以存储不同的类型。但是，C++中没有这样一个 Object 类，原因在于，Java 是单根的，而 C++不是。在 Java 中，所有类都可以上溯到 Object 类，但是 C++中没有这么一个根。那么，怎么实现这么的操作呢？一种办法是，我们都存成 string 类，比如 int i=10，我就存"10"字符串。简单的数据类型固然可以，可复杂一些的呢？比如一个颜色？难道要把 ARGB 所有的值都转化成 string？这种做法很复杂，而且失去了 C++的类型检查等好处。于是我们想另外的办法：创建一个 Object 类，这是一个“很大很大的”类，里面存储了几乎所有的数据类型，比如下面的代码：

```cpp

class Object  
{  
public:  
    int intValue;  
    float floatValue;  
    string stringValue;  
};
```

这个类怎么样？它就足以存储 int、float 和 string 了。嗯，这就是我们的思路，也是 Qt 的思路。在 Qt 中，这样的类就是 QVariant。

QVariant 可以保存很多 Qt 的数据类型，包括 QBrush、QColor、QCursor、QDateTime、QFont、QKeySequence、QPalette、QPen、QPixmap、QPoint、QRect、QRegion、QSize 和 QString，并且还有 C++基本类型，如 int、float 等。QVariant 还能保存很多集合类型，如 QMap<QString, QVariant>, QStringList 和 QList<qvariant>。item view classes，数据库模块和 QSettings 都大量使用了 QVariant 类，，以方便我们读写数据。 QVariant 也可以进行嵌套存储，例如</qvariant>

```cpp

QMap<QString, QVariant> pearMap;  
pearMap["Standard"] = 1.95;  
pearMap["Organic"] = 2.25;
```

```cpp

QMap<QString, QVariant> fruitMap;  
fruitMap["Orange"] = 2.10;  
fruitMap["Pineapple"] = 3.85;  
fruitMap["Pear"] = pearMap;
```

QVariant 被用于构建 Qt Meta-Object，因此是 QtCore 的一部分。当然，我们也可以在 GUI 模块中使用，例如

```cpp

QIcon icon("open.png");  
QVariant variant = icon;  
// other function  
QIcon icon = variant.value<QIcon>();
```

我们使用了 value<t>()模版函数，获取存储在 QVariant 中的数据。这种函数在非 GUI 数据中同样适用，但是，在非 GUI 模块中，我们通常使用 toInt()这样的一系列 to...()函数，如 toString()等。</t>

如果你觉得 QVariant 提供的存储数据类型太少，也可以自定义 QVariant 的存储类型。被 QVariant 存储的数据类型需要有一个默认的构造函数和一个拷贝构造函数。为了实现这个功能，首先必须使用 Q_DECLARE_METATYPE()宏。通常会将这个宏放在类的声明所在头文件的下面：

```cpp

Q_DECLARE_METATYPE(BusinessCard)
```

然后我们就可以使用：

```cpp

BusinessCard businessCard;  
QVariant variant = QVariant::fromValue(businessCard);  
// ...  
if (variant.canConvert<BusinessCard>()) {  
    BusinessCard card = variant.value<BusinessCard>();  
    // ...  
}
```

由于 VC 6 的编译器限制，这些模板函数不能使用，如果你使用这个编译器，需要使用 qVariantFromValue(), qVariantValue<t>()和 qVariantCanConvert<t>()这三个宏。 如果自定义数据类型重写了<<和>>运算符，那么就可以直接在 QDataStream 中使用。不过首先需要使用 qRegisterMetaTypeStreamOperators<t>().宏进行注册。这就能够让 QSettings 使用操作符对数据进行操作，例如</t></t></t>

```cpp

qRegisterMetaTypeStreamOperators<BusinessCard>("BusinessCard");
```

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)