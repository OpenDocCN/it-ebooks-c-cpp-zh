# 五十二、Qt 容器和算法拾遗

Qt 提供了另外的容器，比如 QPair<T1, T2>，可以存储两个值，类似于 std::pair<T1, T2>。还有 QVarLengthArray<T, Prealloc>，这是一个 QVactor<t>的低级实现。因为它需要预分配内存，并且没有隐式的内存共享机制。但是它的开销低于 QVector<t>，更适合资源紧张的情况。 关于 Qt 的通用算法，还有 qCopyBackward()和 qEqual()两个。具体可以查阅 Qt 文档中 Algorithnms 一章。</t></t>

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)