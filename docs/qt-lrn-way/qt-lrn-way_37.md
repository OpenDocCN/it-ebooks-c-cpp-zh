# 三十七、Qt 容器类之关联存储容器

今天我们来说说 Qt 容器类中的关联存储容器。所谓关联存储容器，就是容器中存储的一般是二元组，而不是单个的对象。二元组一般表述为<key-value>，也就是“键-值对”。</key-value>

首先，我们看看数组的概念。数组可以看成是一种<int-object>形式的键-值对，它的 Key 只能是 int，而值的类型是 Object，也就是任意类型(注意，这里我们只是说数组可以是任意类型，这个 Object 并不必须是一个对象)。现在我们扩展数组的概念，把 Key 也做成任意类型的，而不仅仅是 int，这样就是一个关联容器了。如果学过数据结构，典型的关联容器就是散列(Hash Map，哈希表)。Qt 提供两种关联容器类型：QMap<K, T>和 QHash<K, T>。</int-object>

QMap<K, T>是一种键-值对的数据结构，它实际上使用跳表 skip-list 实现，按照 K 进行升序的方式进行存储。使用 QMap<K, T>的 insert()函数可以向 QMap<K, T>中插入数据，典型的代码如下：

```cpp

QMap<QString, int> map; 
map.insert("eins", 1); 
map.insert("sieben", 7); 
map.insert("dreiundzwanzig", 23);
```

同样，QMap<K, T>也重载了[]运算符，你可以按照数组的复制方式进行使用：

```cpp

map["eins"] = 1; 
map["sieben"] = 7; 
map["dreiundzwanzig"] = 23;
```

[]操作符同样也可以像数组一样取值。但是请注意，如果在一个非 const 的 map 中，使用[]操作符取一个不存在的 Key 的值，则这个 Key 会被自动创建，并将其关联的 value 赋予一个空值。如果要避免这种情况，请使用 QMap<K, T>的 value()函数：

```cpp

int val = map.value("dreiundzwanzig");
```

如果 key 不存在，基本类型和指针会返回 0，对象类型则会调用默认构造函数，返回一个对象，与[]操作符不同的是，value()函数不会创建一个新的键-值对。如果你希望让不存在的键返回一个默认值，可以传给 value()函数第二个参数：

```cpp

int seconds = map.value("delay", 30);
```

这行代码等价于：

```cpp

int seconds = 30; 
if (map.contains("delay")) 
        seconds = map.value("delay");
```

QMap<K, T>中的 K 和 T 可以是基本数据类型，如 int，double，可以是指针，或者是拥有默认构造函数、拷贝构造函数和赋值运算符的类。并且 K 必须要重载<运算符，因为 QMap<K, T>需要按 K 升序进行排序。

QMap<K, T>提供了 keys()和 values()函数，可以获得键的集合和值的集合。这两个集合都是使用 QList 作为返回值的。

Map 是单值类型的，也就是说，如果一个新的值分配给一个已存在的键，则旧值会被覆盖。如果你需要让一个 key 可以索引多个值，可以使用 QMultiMap<K, T>。这个类允许一个 key 索引多个 value，如：

```cpp

QMultiMap<int, QString> multiMap; 
multiMap.insert(1, "one"); 
multiMap.insert(1, "eins"); 
multiMap.insert(1, "uno"); 

QList<QString> vals = multiMap.values(1);
```

QHash<K, T>是使用散列存储的键-值对。它的接口同 QMap<K, T>几乎一样，但是它们两个的实现需求不同。QHash<K, T>的查找速度比 QMap<K, T>快很多，并且它的存储是不排序的。对于 QHash<K, T>而言，K 的类型必须重载了==操作符，并且必须被全局函数 qHash()所支持，这个函数用于返回 key 的散列值。Qt 已经为 int、指针、QChar、QString 和 QByteArray 实现了 qHash()函数。

QHash<K, T>会自动地为散列分配一个初始大小，并且在插入数据或者删除数据的时候改变散列的大小。我们可以使用 reserve()函数扩大散列，使用 squeeze()函数将散列缩小到最小大小(这个最小大小实际上是能够存储这些数据的最小空间)。在使用时，我们可以使用 reserve()函数将数据项扩大到我们所期望的最大值，然后插入数据，完成之后使用 squeeze()函数收缩空间。

QHash<K, T>同样也是单值类型的，但是你可以使用 insertMulti()函数，或者是使用 QMultiHash<K, T>类来为一个键插入多个值。另外，除了 QHash<K, T>，Qt 也提供了 QCache<K, T>来提供缓存，QSet<k>用于仅存储 key 的情况。这两个类同 QHash<K, T>一样具有 K 的类型限制。</k>

遍历关联存储容器的最简单的办法是使用 Java 风格的遍历器。因为 Java 风格的遍历器的 next()和 previous()函数可以返回一个键-值对，而不仅仅是值，例如：

```cpp

QMap<QString, int> map; 
... 
int sum = 0; 
QMapIterator<QString, int> i(map); 
while (i.hasNext()) 
        sum += i.next().value();
```

如果我们并不需要访问键-值对，可以直接忽略 next()和 previous()函数的返回值，而是调用 key()和 value()函数即可，如：

```cpp

QMapIterator<QString, int> i(map); 
while (i.hasNext()) { 
        i.next(); 
        if (i.value() > largestValue) { 
                largestKey = i.key(); 
                largestValue = i.value(); 
        } 
}
```

Mutable 遍历器则可以修改 key 对应的值：

```cpp

QMutableMapIterator<QString, int> i(map); 
while (i.hasNext()) { 
        i.next(); 
        if (i.value() < 0.0) 
                i.setValue(-i.value()); 
}
```

如果是 STL 风格的遍历器，则可以使用它的 key()和 value()函数。而对于 foreach 循环，我们就需要分别对 key 和 value 进行循环了：

```cpp

QMultiMap<QString, int> map; 
... 
foreach (QString key, map.keys()) { 
        foreach (int value, map.values(key)) { 
                doSomething(key, value); 
        } 
}
```

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)