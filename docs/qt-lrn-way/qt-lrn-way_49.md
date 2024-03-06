# 四十九、通用算法

关于 Qt 的 model-view 部分就告一段落，今天我们开始新的部分。或许有些朋友觉得前面的部分说得很简单。对此我也没有办法，毕竟，Qt 是一个很庞大的库，一时半会根本不可能穷尽所有内容，并且我也有很多东西不知道，有时候也必须去查找资料才能明白。

今天开始的部分是关于 Qt 提供的一些通用算法。这部分内容来自 C++ GUI Programming with Qt 4, 2nd Edition。

<qtalgorithms>提供了一系列通用的模板函数，用于实现容器上面的基本算法。这部分算法很多依赖于 STL 风格的遍历器(还记得前面曾经说过的 Java 风格的遍历器和 STL 风格的遍历器吗？)。实际上，C++ STL 也提供了很多通用算法，包含在<algorithm>头文件内。这部分算法对于 Qt 容器同样也是适用的。因此，如果你想使用的算法在 Qt 的<qtalgorithms>头文件中没有包含，那么就可以使用 STL 的算法代替，这并不会产生什么冲突。这里我们来说几个 Qt 中的通用算法。虽然这些算法都是很简单的，但是，库函数往往会比自己编写的更有效率，因此还是推荐使用系统提供的函数的。</qtalgorithms></algorithm></qtalgorithms>

首先是 qFind()函数。qFind()函数会在容器中查找一个特定的值。它的参数中有一个起始位置和终止位置，如果被查找的元素存在，函数返回第一个匹配项的位置，否则则返回终止位置。注意，我们这里说的“位置”，实际上是 STL 风格的遍历器。我们知道，使用 STL 风格遍历器是可以反映一个位置的。例如下面的例子，i 的值将是 list.begin() + 1，而 j 会是 list.end()：

```cpp

QStringList list; 
list << "Emma" << "Karl" << "James" << "Mariette"; 

QStringList::iterator i = qFind(list.begin(), list.end(), "Karl"); 
QStringList::iterator j = qFind(list.begin(), list.end(), "Petra");
```

qBinaryFind()的行为很像 qFind()，所不同的是，qBinaryFind()是二分查找算法，它只适用于查找排序之后的集合，而 qFind()则是标准的线性查找。通常，二分查找法使用条件更为苛刻，但是效率也会更高。

qFill()会使用给定值对容器进行填充。例如：

```cpp

QLinkedList<int> list(10); 
qFill(list.begin(), list.end(), 1009);
```

正如其他基于遍历器的算法一样，qFill()也可以针对容器的一部分进行操作，例如下面的代码将会把 vector 的前 5 位设置成 1009，而最后 5 位设置为 2013：

```cpp

QVector<int> vect(10); 
qFill(vect.begin(), vect.begin() + 5, 1009); 
qFill(vect.end() - 5, vect.end(), 2013);
```

qCopy()算法可以实现将一个容器中的元素复制到另一个容器，例如：

```cpp

QVector<int> vect(list.count()); 
qCopy(list.begin(), list.end(), vect.begin());
```

qCopy()也可以用于同一容器中的元素的复制。qCopy()操作成功的关键是源容器和目的容器的范围不会发生溢出。例如如下代码，我们将把一个列表的最后两个元素复制给前两个元素：

```cpp

qCopy(list.begin(), list.begin() + 2, list.end() - 2);
```

qSort()实现了容器元素的递增排序，使用起来也很简单：

```cpp

qSort(list.begin(), list.end());
```

默认情况下，qSort()将使用 < 运算符进行元素的比较。这暗示如果需要的话，你必须定义 < 运算符。如果需要按照递减排序，需要将 qGreater<t>()当作第三个参数传给 qSort()函数。例如：</t>

```cpp

qSort(list.begin(), list.end(), qGreater<int>());
```

注意，这里的 T 实际上是容器的泛型类型。实际上，我们可以利用第三个参数对排序进行定义。例如，我们自定义的数据类型中有一个大小写不敏感的 QString 的小于比较函数：

```cpp

bool insensitiveLessThan(const QString &str1, const QString &str2) 
{ 
        return str1.toLower() < str2.toLower(); 
}
```

那么，我们可以这样使用 qSort()从而可以利用这个函数：

```cpp

QStringList list; 
// ... 
qSort(list.begin(), list.end(), insensitiveLessThan);
```

qStableSort()函数类似与 qSort()，所不同之处在于它是稳定排序。稳定排序是算法设计上的一个名词，意思是，在排序过程中，如果有两个元素相等，那么在排序结果中这两个元素的先后顺序同排序前的原始顺序是一致的。举个例子，对于一个序列：a1, a5, a32, a31, a4，它们的大小顺序是 a1 < a31 = a32 < a4 < a5，那么稳定排序之后的结果应该是 a1, a32, a31, a4, a5，也就是相等的元素在排序结果中出现的顺序和原始顺序是一致的。稳定排序在某些场合是很有用的，比如，现在有一份按照学号排序的学生成绩单。你想按照成绩高低重新进行排序，对于成绩一样的学生，还是遵循原来的学号顺序。这时候就要稳定排序了。

qDeleteAll()函数将对容器中存储的所有指针进行 delete 操作。这个函数仅在容器元素是指针的情形下才适用。执行过这个函数之后，容器中的指针均被执行了 delete 运算，但是这些指针依然被存储在容器中，成为野指针，你需要调用容器的 clear()函数来避免这些指针的误用：

```cpp

qDeleteAll(list); 
list.clear();
```

qSwap()函数可以交换两个元素的位置。例如：

```cpp

int x1 = line.x1(); 
int x2 = line.x2(); 
if (x1 > x2) 
        qSwap(x1, x2);
```

最后，在<qtglobal>头文件中，也定义了几个有用的函数。这个头文件被其他所有的头文件 include 了，因此你不需要显式的 include 这个头文件了。</qtglobal>

在这个头文件中有这么几个函数：qAbs()返回参数的绝对值，qMin()和 qMax()则返回两个值的最大值和最小值。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)