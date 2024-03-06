# 五十七、二进制文件读写

今天开始进入 Qt 的另一个部分：文件读写，也就是 IO。文件读写在很多应用程序中都是需要的。Qt 通过 QIODevice 提供了 IO 的抽象，这种设备(device)具有读写字节块的能力。常用的 IO 读写的类包括以下几个：

| QFlie | 访问本地文件系统或者嵌入资源 |
| :-- | :-- |
| QTemporaryFile | 创建和访问本地文件系统的临时文件 |
| QBuffer | 读写 QByteArray |
| QProcess | 运行外部程序，处理进程间通讯 |
| QTcpSocket | TCP 协议网络数据传输 |
| QUdpSocket | 传输 UDP 报文 |
| QSslSocket | 使用 SSL/TLS 传输数据 |

QProcess、QTcpSocket、QUdpSoctet 和 QSslSocket 是顺序访问设备，它们的数据只能访问一遍，也就是说，你只能从第一个字节开始访问，直到最后一个字节。QFile、QTemporaryFile 和 QBuffer 是随机访问设备，你可以从任何位置访问任意次数，还可以使用 QIODevice::seek() 函数来重新定位文件指针。

在访问方式上，Qt 提供了两个更高级别的抽象：使用 QDataStream 进行二进制方式的访问和使用 QTextStream 进行文本方式的访问。这些类可以帮助我们控制字节顺序和文本编码，使程序员从这种问题中解脱出来。

QFile 对于访问独立的文件是非常方便的，无论是在文件系统中还是在应用程序的资源文件中。Qt 同样也提供了 QDir 和 QFileInfo 两个类，用于处理文件夹相关事务以及查看文件信息等。 这次我们先从二进制文件的读写说起。

以二进制格式访问数据的最简单的方式是实例化一个 QFile 对象，打开文件，然后使用 QDataStream 进行访问。QDataStream 提供了平台独立的访问数据格式的方法，这些数据格式包括标准的 C++ 类型，如 int、double 等；多种 Qt 类型，如 QByteArray、QFont、QImage、QPixmap、QString 和 QVariant，以及 Qt 的容器类，如 QList <t>和 QMap<K, T>。先看如下的代码：</t>

```cpp

QImage image("philip.png");  

QMap<QString, QColor> map;  
map.insert("red", Qt::red);  
map.insert("green", Qt::green);  
map.insert("blue", Qt::blue);  

QFile file("facts.dat");  
if (!file.open(QIODevice::WriteOnly)) {  
    std::cerr << "Cannot open file for writing: " 
              << qPrintable(file.errorString()) << std::endl;  
    return;  
}  

QDataStream out(&file);  
out.setVersion(QDataStream::Qt_4_3);  

out << quint32(0x12345678) << image << map;
```

这里，我们首先创建了一个 QImage 对象，一个 QMap<QString, QColor>，然后使用 QFile 创建了一个名为 "facts.dat" 的文件，然后以只写方式打开。如果打开失败，直接 return；否则我们使用 QFile 的指针创建一个 QDataStream 对象，然后设置 version，这个我们以后再详细说明，最后就像 std 的 cout 一样，使用 << 运算符输出结果。

0x12345678 成为“魔术数字”，这是二进制文件输出中经常使用的一种技术。我们定义的二进制格式通常具有一个这样的“魔术数字”，用于标志文件格式。例如，我们在文件最开始写入 0x12345678，在读取的时候首先检查这个数字是不是 0x12345678，如果不是的话，这就不是可识别格式，因此根本不需要去读取。一般二进制格式都会有这么一个魔术数字，例如 Java 的 class 文件的魔术数字就是 0xCAFE BABE(很 Java 的名字)，使用二进制查看器就可以查看。魔术数字是一个 32 位的无符号整数，因此我们使用 quint32 宏来得到一个平台无关的 32 位无符号整数。

在这段代码中我们使用了一个 qPrintable() 宏，这个宏实际上是把 QString 对象转换成 const char *。注意到我们使用的是 C++ 标准错误输出 cerr，因此必须使用这个转换。当然，QString::toStdString() 函数也能够完成同样的操作。

读取的过程就很简单了，需要注意的是读取必须同写入的过程一一对应，即第一个写入 quint32 型的魔术数字，那么第一个读出的也必须是一个 quint32 格式的数据，如

```cpp

quint32 n;  
QImage image;  
QMap<QString, QColor> map;  

QFile file("facts.dat");  
if (!file.open(QIODevice::ReadOnly)) {  
    std::cerr << "Cannot open file for reading: " 
              << qPrintable(file.errorString()) << std::endl;  
    return;  
}  

QDataStream in(&file);  
in.setVersion(QDataStream::Qt_4_3);  

in >> n >> image >> map;
```

好了，数据读出了，拿着到处去用吧！

这个 version 是干什么用的呢？对于二进制的读写，随着 Qt 的版本升级，可能相同的内容有了不同的读写方式，比如可能由大端写入变成了小端写入等，这样的话旧版本 Qt 写入的内容就不能正确的读出，因此需要设定一个版本号。比如这里我们使用 QDataStream::Qt_4_3，意思是，我们使用 Qt 4.3 的方式写入数据。实际上，现在的最高版本号已经是 QDataStream::Qt_4_6。如果这么写，就是说，4.3 版本之前的 Qt 是不能保证正确读写文件内容的。那么，问题就来了：我们以硬编码的方式写入这个 version，岂不是不能使用最新版的 Qt 的读写了？

解决方法之一是，我们不仅仅写入一个魔术数字，同时写入这个文件的版本。例如：

```cpp

QFile file("file.xxx");  
file.open(QIODevice::WriteOnly);  
QDataStream out(&file);  

// Write a header with a "magic number" and a version  
out << (quint32)0xA0B0C0D0;  
out << (qint32)123;  

out.setVersion(QDataStream::Qt_4_0);  

// Write the data  
out << lots_of_interesting_data;
```

这个 file.xxx 文件的版本号是 123。我们认为，如果版本号是 123 的话，则可以使用 Qt_4_0 版本读取。所以我们的读取代码就需要判断一下：

```cpp

QFile file("file.xxx");  
 file.open(QIODevice::ReadOnly);  
 QDataStream in(&file);  

 // Read and check the header  
 quint32 magic;  
 in >> magic;  
 if (magic != 0xA0B0C0D0)  
     return XXX_BAD_FILE_FORMAT;  

 // Read the version  
 qint32 version;  
 in >> version;  
 if (version < 100)  
     return XXX_BAD_FILE_TOO_OLD;  
 if (version > 123)  
     return XXX_BAD_FILE_TOO_NEW;  

 if (version <= 110)  
     in.setVersion(QDataStream::Qt_3_2);  
 else 
     in.setVersion(QDataStream::Qt_4_0);  

 // Read the data  
 in >> lots_of_interesting_data;  
 if (version >= 120)  
     in >> data_new_in_XXX_version_1_2;  
 in >> other_interesting_data;
```

这样，我们就可以比较完美的处理二进制格式的数据读写了。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)