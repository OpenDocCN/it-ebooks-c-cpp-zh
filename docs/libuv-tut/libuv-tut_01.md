# 简介

# Introduction

本书由很多的 libuv 教程组成，libuv 是一个高性能的，事件驱动的 I/O 库，并且提供了跨平台（如 windows, linux）的 API。

本书会涵盖 libuv 的主要部分，但是不会详细地讲解每一个函数和数据结构。[官方文档](http://docs.libuv.org/en/v1.x/)中可以查阅到完整的内容。

本书依然在不断完善中，所以有些章节会不完整，但我希望你能喜欢它。

## Who this book is for

如果你正在读此书，你或许是：

> 1.  系统程序员，会编写一些底层的程序，例如守护进程或者网络服务器／客户端。你也许发现了 event-loop 很适合于你的应用场景，然后你决定使用 libuv。
> 2.  一个 node.js 的模块开发人员，决定使用 C/C++封装系统平台某些同步或者异步 API，并将其暴露给 Javascript。你可以在 node.js 中只使用 libuv。但你也需要参考其他资源，因为本书并没有包括 v8/node.js 相关的内容。

本书假设你对 c 语言有一定的了解。

## Background

[node.js](https://nodejs.org/en/)最初开始于 2009 年，是一个可以让 Javascript 代码离开浏览器的执行环境也可以执行的项目。 node.js 使用了 Google 的 V8 解析引擎和 Marc Lehmann 的 libev。Node.js 将事件驱动的 I/O 模型与适合该模型的编程语言(Javascript)融合在了一起。随着 node.js 的日益流行，node.js 需要同时支持 windows, 但是 libev 只能在 Unix 环境下运行。Windows 平台上与 kqueue(FreeBSD)或者(e)poll(Linux)等内核事件通知相应的机制是 IOCP。libuv 提供了一个跨平台的抽象，由平台决定使用 libev 或 IOCP。在 node-v0.9.0 版本中，libuv 移除了 libev 的内容。

随着 libuv 的日益成熟，它成为了拥有卓越性能的系统编程库。除了 node.js 以外，包括 Mozilla 的[Rust](http://rust-lang.org)编程语言，和许多的语言都开始使用 libuv。

本书基于 libuv 的 v1.3.0。

## Code

本书中的实例代码都可以在[Github](https://github.com/nikhilm/uvbook/tree/master/code)上找到。