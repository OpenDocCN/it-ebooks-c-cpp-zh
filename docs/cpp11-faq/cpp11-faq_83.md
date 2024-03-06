# 无序容器（unordered containers）

一个无序容器实际上就是某种形式的蛤希表。C++0x 提供四种标准的无序容器：

*   unordered_map
*   unordered_set
*   unordered_multimap
*   unordered_multiset

实际上，它们应该被称为 hash_map 等。但是因为有很多地方已经在使用 hash_map 这样的名字了，为了保证其兼容性，标准委员会不得不选择新的名字。而 unordered_map 是我们所能够找到的最好的名字了。无序（“unordered”）代表着 map 和 unordered_map 之间一个最本质的差别：当你使用 map 容器的时候，容器中的所有元素都是通过小于操作（默认情况下使用“`<`”操作符）排好序的，但是 unordered_map 并没有对元素进行排序，所以它并不要求元素具有小于操作符。并且，一个哈希表也并添言地提供排序的功能。相反，map 容器中的元素也并不要求具有哈希函数。 基本上，当代码的优化是可能的并且我们有理由对其进行优化时，我们可以把 unordered_map 当作一个优化之后的 map 容器来使用。例如：

```cpp
map<string,int> m {
    {“Dijkstra”,1972}, {“Scott”,1976},
    {“Wilkes”,1967}, {“Hamming”,1968}
};
m["Ritchie"] = 1983;
for(auto x : m)
    cout << ‘{‘ << x.first << ‘,’ << x.second << ‘}’;

// 使用优化之后的 unordered_map
unordered_map<string,int> um {
    {“Dijkstra”,1972}, {“Scott”,1976},
    {“Wilkes”,1967}, {“Hamming”,1968}
};
um["Ritchie"] = 1983;
for(auto x : um)
    cout << ‘{‘ << x.first << ‘,’ << x.second << ‘}’; 
```

map 容器 m 的迭代器将以字母的顺序访问容器中的所有元素，而 unordered_map 容器 um 则并不按照这样的顺序（除非通过一些特殊的操作）。m 和 um 两者的查找功能的实现机制是非常不同的。对于 m，它使用的是复杂度为 log2(m.size())的小于比较，而 um 只是简单地调用了一个哈希函数和一次或多次的相等比较。如果容器中的元素比较少（比如几打），很难说哪一种容器更快，但是对于大量数据（比如数千个）而言，unordered_map 容器的查找速度要比 map 容器快很多。

更多内容稍后提供。

参考：

? Standard: 23.5 Unordered associative containers.

(翻译：Yibo Zhu)