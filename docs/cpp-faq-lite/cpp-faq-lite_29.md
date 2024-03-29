# [36] 序列化与反序列化

## FAQs in section [36]:

*   [36.1]“序列化”是什么东东？
*   [36.2] 如何选择最好的序列化技术？
*   [36.3] 如何决定是要序列化为可读的（“文本”）还是不可读的（“二进制”）格式？ or non-human-readable ("binary") format?")
*   [36.4] 如何序列化/反序列化数字，字符，字符串等简单类型？
*   [36.5] 如何读/写简单类型为可读的（“文本”）格式？ format?")
*   [36.6] 如何读/写简单类型为非可读的（“二进制”）格式？ format?")
*   [36.7] 如何序列化没有继承层次结构的对象，并且该对象不包含指向其他对象的指针？
*   [36.8] 如何序列化有继承层次结构的对象 ， 并且该对象不包含指向其他对象的指针？
*   [36.9] 我如何序列化包含指向其他对象指针的对象，但这些指针是没有回路和链接的树形结构？
*   [36.10] 我如何序列化包含指向其他对象指针的对象，这些指针是没有回路，但是有平凡链接的树形结构？
*   [36.11] 我如何序列化包含指向其他对象指针的对象，这些指针是可能含有回路或者非平凡链接的图？
*   [36.12] 序列化/ 反序列化对象的时候有什么注意事项？
*   [36.13] 什么是图，树，节点，回路，链接，叶链接与内部节点链接？

## 36.1 “序列化”是什么东东？

它可以让你把一个对象或组对象存储在磁盘；或通过有线或无线传输到另一台计算机，然后通过反向过程：唤醒原始的对象。基本机制是把对象扁平化(flatten)为一维的字节流，然后把字节流再变成原始的对象。

.如同星际迷航中的运输机，把复杂的东西扁平化为 1 和 0 的序列，然后把 1 和 0 的序列（可能在另一个地方，另一个时间）重新构造为原有的复杂“东西”。

## 36.2 如何选择最好的序列化技术？

有很多很多的条件限制，实际上，涉及到技术整体连贯性的多个方面。因为我时间有限（翻译：我没有报酬），我将之简化为“使用人类可读的（“文本”）或者非人类可读的（“二进制“）格式”，由或多或少按照技术成熟度排序的 5 个小技巧组成。

当然，不局限于上述五技术。你可能会想最终混合的几种技术。当然你也可以随时使用比实际需求更复杂（编号更高）的技术。事实上，使用比实际需求更复杂的技术是明智的，如果你认为未来的式样变更需要更大的复杂度。所以，这份名单仅仅是一个很好的起点。

最好准备！有许多东东在这里呢！

1.  决定使用人类可读的（“文本”）还是非可读（“二进制”）格式 or non-human-readable ("binary") format?")。折中很困难。后面的 FAQ 会告诉 如何序列化简单类型为文本格式 format?")和 如何编写简单类型的二进制格式 format?")。
2.  使用最简单的解决方案，当序列化对象不是继承层次结构的一部分（也就是说，当他们都派生自同一个类）和不包含指向其他对象指针的时候。
3.  使用较简单的解决方案，当序列化对象是继承层次结构的一部分，但是不包含指向其他对象的指针的时候。
4.  使用第三级复杂的解决方案 ，当序列化对象包含指向其他对象指针的对象，但这些指针是没有回路和链接的树形结构的时候。
5.  使用第四级复杂的解决方案， 当序列化对象包含指向其他对象指针的对象，但这些指针是没有回路，只有叶子链接的图的时候。
6.  使用最复杂的解决方案，当 序列化对象包含指向其他对象指针的对象，这些指针是可能含有回路或者链接的图的时候。

下面是同样的信息，但是使用算法格式：

1.  第一步是要决定使用文本还是二进制格式。
2.  如果对象不是继承层次结构的一部分，不包含指针， 那么使用解决方案#1 。
3.  否则，如果对象不包含指向其他对象的指针， 那么使用解决方案#2 。
4.  否则，如果指针的图不包含回路和链接，那么使用解决方案#3。
5.  否则，如果指针的图不包含循序，并且链接是叶子链接，那么使用解决方案#4。
6.  否则，使用解决方案#5。

记住：可以随意混合/增加上述列表，如果你能够判断使用更复杂的技术可以让额外成本最小。

还有一件事：对象继承和对象含有指针在逻辑上是不相关的，因此#2 比#3-5 简单并没有任何理论依据。然而在实践中常常（并不总是）总是这样。所以请不要认为这些分类是金科玉律—在某种程度上他们有些武断，你可能需要混合的这些解决方案以满足你的具体情况。序列化技术领域远非几个问题和解答能解决的。

## 36.3 如何决定是要序列化为可读的（“文本”）还是不可读的（“二进制”）格式？

要小心。

这个问题没有“正确”回答，它取决于你的目标。下面是人类可读（“文本”） 与非人类可读的（“二进制”）格式的一些利弊：

*   **文本格式**是比较容易检查。这意味着你将不必编写额外的工具来调试输入和输出，你可以使用文本编辑器打开序列化输出，来检查输出内容是否正确。
*   **二进制格式**使用较少的 CPU 周期。但是，这种相关仅仅当你的应用程序是与 CPU 绑定，并且序列化和/或反序列化是在一个内部循环和 bottleneck 之中。记住：90％的 CPU 时间花费在 10%的代码中，这意味着这将不会有任何实际意义，除非你 “CPU”使用量总是 100％，你的序列化和/或反序列化代码仅仅花费 100％的一小部分。
*   **文本格式**不用担心一些编程问题，例如 sizeof、小印第安编码与大印第安编码。
*   **二进制格式**不用担心相邻值的分隔符，因为许多值具有固定长度。
*   **文本格式**可以生成更小的结果集，当大多数数值很小和你需要文本化二进制编码的时候，例如 uuencode 的或 Base64。
*   **二进制格式**可以生成更小的结果集，当大多数数值很大或不需要文本化二进制编码的时候。

你也许会有补充添加其他的优缺点…重要的是要知道一鞋难合众人脚-请作出自己慎重决定。

还有：无论你如何选择，你要在每个文件/流的开头加上“magic”标签和版本号。版本号将显示格式规则。这样，如果你决定对格式做重大改变的时候，将仍然可以读取旧的软件产生的输出。

## 36.4 如何序列化/反序列化数字，字符，字符串等简单类型？

答案取决于使用人类可读（“文本”）还是非格式人类可读（“二进制”）格式的决定：

*   下面是如何读/写简单类型为可读的（“文本”）格式。
*   下面是如何读/写简单类型为非可读的（“二进制”）格式。

在本节几乎所有的其他 FAQ 中都需要上述 FAQ 中的讨论作为基础。

## 36.5 如何读/写简单类型为可读的（“文本”）格式？

阅读之前，请确保要评估人类可读和非人类可读的格式之间的平衡。这种均衡评估是非常重要的，所以不能因为上一个项目没有这么做而下意识地抵制它—一鞋难合众人脚。

如果你已明确决定使用人类可读（“文本”）格式，你应该记住这些要点：

*   你最好使用`iostream`的`>>`和`<<`，而不是它的`read()`和`write()`方法。`>>`和`<<`比较适合文本模式，而`read()`和`write()`则更适合二进制模式。
*   当储存数值时，你需要添加分隔符，以防止数值连在一起。一个简单的方法是在每个数字之前添加一个空格（``），这一数字 1 和数字 2 不会连在一起看起来像数值 12。由于前导空格自动会被`>>`操作符忽略，你不需要显示的处理空格当读入序列化的结果的时候。
*   字符串需要一些诀窍，因为你必须明确地知道什么时候该字符串结束。你不能明确地使用`'\n'`或`'”'`甚至`'\ 0'`来终止所有字符串，因为一些字符串可能包含这些字符。你要使用 C++字符转义序列，例如，当你看到一个新行时写`'\ '`接着`'n'`等等。做了这个转换之后，你可以输出字符串一直到行尾（这意味着它们使用`'\n'`分割），也可以以`'”'`来分割字符串。
*   如果对于字符串你使用 C++字符转义序列，对于 16 进制数一定要始终`'\x'`和`'\u'`之后使用相同的位数。我通常分别使用 2 为和 4 位。原因：如果你写入的十六进制数值较少，假如你只是使用`stream` `<<` `"\\x"` `<<` `hex` `<<` `unsigned(theChar)`，当字符串中的下一个字符碰巧是十六进制数位时就会得到错误的结果。例如，如果字符串包含`'\F'`之后是`’A’`，那么你应该写入`“\x0FA”`而不是`“\xFA”`。
*   对于像`”\n”`的字符，如果你不使用字符序转义序列，那么请确保操作系统不会搅乱你的字符串数据。特别是，如果你打开一个没有`std::ios::binary`的`std::fstream`时候，有的操作系统将会企图翻译行结束字符。
*   处理字符串数据的另一个方法是在字符串前面添加长度前缀，例如，写入“`now is the time`”为”`15:now is the time`” 。请注意，这可能使人很难读/写文件，因为长度值之后可能会有不可见的分隔字符。尽管这样这种方法也很有用。

请记住，这些是在本节中的其他 FAQ 中需要的一些基础知识。

## 36.6 如何读/写简单类型为非可读的（“二进制”）格式？

阅读之前，请确保要评估人类可读和非人类可读的格式之间的平衡。这种均衡评估是非常重要的，所以不能因为上一个项目没有这么做而下意识地抵制它—一鞋难合众人脚。

如果你已明确决定使用非人类可读（“文本”）格式，你应该记住这些要点：

*   请确保打开输入和输出流时使用`std::ios::binary`。即使在 Unix 系统也需要这么做，因为它很容易做到并且它说明了你的意图，它也更容易移植和修改。
*   你最好使用的`iostream`的`read()`和`write()`方法而不是它的`>>`和`<<`运算符。 `read()`和`write()`更适合二进制模式，`>>`和`<<`更适合文本模式。
*   如果二进制数据有可能被另外一台电脑读取，必须小心不同的计算机上的印第安编码问题（小印第安编码与大印第安编码）和`sizeof`问题。最简单的处理方法是，选定一种编码作为正式的“网络”格式，并创建一个包含依赖计算机实现的头文件（我通常称之为`machine.h`）。该头文件应该定义诸如`readNetworkInt`这样的内联函数（`std::istream& istr`）来读“网络 `int`”类型。对于其他所有的基本类型，都要定义相应的函数。你可以以任何你想要的方式来定义这些类型的格式。例如，你可以定义一个“网络 `int` ”类型为 32 位的小印第安编码格式。在任何情况下，`machine.h`内的函数将做任何必要的印第安编码转换，`sizeof`转换等等。你要么对每台机器定义不同的`machine.h`，要么在`machine.h`格式中使用# `ifdef`宏。但无论如何，所有这些丑陋的编码将被放置在一个头文件，所有其余代码将会变得很清晰。注：浮点数误差的处理是最微妙最棘手的。可以做到，但你必须小心像`NaN`，缓冲区向上溢出和向下溢出，尾数的位数和指数等。
*   如果空间成本是个问题的话，例如你要储存序列化数据到一个小存储设备的或通过慢速链接发送序列化数据，你可以压缩流和/或你可以使用一些小技巧。最简单的是使用最少的字节存储小的数值。例如，要在只有有 8 位字节的流中存储无符号整数，你可以劫持每个字节的第 8 位，来判断是否有另一个字节。.这意味着你可以存储 0 ... 127 在 1 个字节，128 ... 16384 在 2 个字节等等。如果平均值小于 500，000，000 左右，这比使用 4 字节存储无符号数。对于这一问题有许多其他演变方式，例如，对于排序的数值数组可以存储每个数值之差，存储极小值为 unary 格式等等。
*   字符串数据是棘手的，因为你必须明确地知道什么时候该字符串的本身结束。你不能明确地使用`'\ 0'`来结束所有字符串，回想一下`std::string`可以存储`'\0'`字符。最简单的方法是在字符串之前写入字符串长度。确保整形长度是“网络格式”，这样可以避免`sizeo`f 和印第安编码问题（参见前一帖的解决方案）。

请记住，这些是在本节中的其他 FAQ 中需要的一些基础知识。

## 36.7 如何序列化没有继承层次结构的对象，并且该对象不包含指向其他对象的指针？

这是最简单的问题，毫不奇怪它也最容易解决：

*   每个类应处理好自己的序列化和反序列化。你通常会创建一个成员函数，把对象序列化到存储体（如`std::ostream`），另一个成员函数来生成一个新的对象，或者修改现有对象，读取数据源（如一个`std::IStream`）并设置对象的成员。
*   如果你的对象本身包含另一个对象，例如一个`Car`对象可能有一个类型为`Engine`的成员变量，外层对象的`serialize()`成员函数应该简单地调用成员变量的相应函数来序列化。
*   使用前面描述的基本知识来以文本或者二进制格式来读/写简单类型或二进制 格式。
*   如果一个类的数据结构将来可能发生变化，在对象的序列化输出的开头类应该输出一个版本号。版本号仅仅代表序列化的格式 ，不应该递增类的版本号如果只是类的行为发生了变化。 就是说，该版本数字并不需要太花哨-通常不需要主版本号和次版本号。

## 36.8 如何序列化有继承层次结构的对象 ， 并且该对象不包含指向其他对象的指针？

假设你要序列化`shape`对象，其中 shape 是一个抽象类，它的派生类有`Rectangle`, `Ellipse`, `Line`, `Text`, etc 等。在 shape 类中你将声明一个纯虚函数`serialize(std:: ostream&) const`，并确保每个重写首先会输出类的身份。例如，`Ellipse::serialize(std::ostream&) const`会输出`Ellipse`的标识符（可能只是一个简单的字符串，但也可以是下文讨论的几种方案）。

当反序列化对象时，事情有点棘手。通常首先从基类的静态成员函数如`Shape::unserialize(std::istream& istr)开始`。这是声明返回一个 `Shape *`或可能是像`shape::Ptr`的智能指针。它读取类名标识符 ，然后使用某些创建模式来创建对象。例如，你可能有类名到对象的映射表，然后使用虚拟构造函数用法来创建对象。

这里有一个具体的例子：在基类`shape`内添加一个纯虚函数`create(std::istream&) const`，并定义只有一行的重写函数，来生成一个适当的派生类对象。例如，`Ellipse::create(std::istream& istr) const`将是`{ return new Ellipse(istr); }`。 添加一个`static` `std::map<std::string,Shape*>`对象，映射类名到一个对应类的代表（又名原型 ）对象，例如，`Ellipse`将映射到`new Ellipse()`。函数`Shape::unserialize(std::istream& istr)` 将读取类名，然后查找关联的`Shape*`并调用它的`create()`方法：`return theMap[className]->create(istr)，`如果类名不再映射表里则抛出一个异常(`if (theMap.count(className) == 0) throw ...something...`)。

映射表通常采用静态初始化。例如，如果文件`Ellipse.cpp`包含了派生类`Ellipse`的代码，它还将包含一个静态的对象，其构造函数将类添加到映射表：`theMap["Ellipse"] = new Ellipse()`。

说明和注意事项：

*   如果`Shape::unserialize()传`递类名到`create()`会增加一些弹性。特别是，这将让派生类可以使用两个或两个以上的类名，每个都有自己的“网络”格式。例如，派生类`Ellipse`可以被传递“`Ellipse`”和“`Circle`”，这有益于输出时节省空间或者另有其他原因。
*   在序列化过程中通过抛出异常来处理错误通常是最容易的。如果你想你可以返回 `NULL`，但你需要把读取输入流的代码从派生类的构造函数到相应的`create()`方法中，最终结果往往是你的代码变得更复杂。
*   你必须小心`Shape::unserialize()`所使用的映射表，避免静态初始化顺序错误。这通常意味着对于映射表要使用初次使用时进行初始化的手法。
*   对于由`Shape::unserialize()`所使用的映射表，我个人更喜欢基于虚构造函数手法的命名构造函数手法—它能简化一些步骤。详细信息：我通常会在`shape`内定`typedef`，例如`typedef Shape* (*Factory)(std::istream&)`。这意味着`Shape::Factory`是一个函数指针，它接受`std::istream&`为参数并返回一个`shape*`。然后，我定义映射表为`std::map<std::string,Factory>`。最后，我使用类似`theMap"Ellipse"] = Ellipse::create`代码来生成映射表（其中`Ellipse::create(std::istream&`）是一个`Ellipse`类的静态成员函数，即命名构造函数用法）。你可能需要把`Shape::unserialize(std::istream& istr)`函数的返回值从`theMap[className]->create(istr)`变为`theMapclassName。`
*   序列化一个 `NULL`指针通常很容易，因为你已经输出了类标识符，你可以很容易地输出一个伪类标识符，比如`NULL`。你可能在`Shape::unserialize()`中需要一个额外`if`语句 ，但是如果使用我上面的写法的话，你可以消除这种特殊情况（通常可以保持代码干净整齐）通过定义一个静态成员函数`Shape* Shape::nullFactory(istream&) { return NULL; }`。你也需要和其他的类一样把`NULL`加入到映射表：`theMap["NULL"] = Shape::nullFactory;`。
*   如果标记类名的话，序列化形式可能会更小更快一些。例如，仅在第一次看到一个类名的时候写一个类名，在以后使用时只使用相应的整数索引。如`std::map<std::string,unsigned> unique`将使之变得简单：如果一个类名已经在映射表里，只用写`unique[className]`；否则设置一个变量`unsigned n = unique.size()`,写`n`，写类名，并设置`unique[className] = n`。（注： 一定要复制到一个单独的变量，不要使用`unique[className] = unique.size()`！别怪我没警告你！）。当反序列化的时候， 使用`std::vector<std::string> unique`，读出 `n`，如果 `n == unique.size()`，读出类名并将其添加到`vector` 。无论哪种方式类名都是`unique[n]`。您还可以预先填充前 `N`个最常见的类名，这样流就不需要包含任何字符串。

## 36.9 我如何序列化包含指向其他对象指针的对象，但这些指针是没有回路和链接的树形结构？

开始之前，你必须明白，“树”并不意味着对象存储在数据结构的树中。它只是意味着你的对象互相指向对方。没有回路是指，如果不断地从一个对象指针跟随到下一个对象，你永远不会回到一个较早的对象。你的对象不是在树的内部，它们是一棵“树”。如果你不理解这些话，在继续之前你应该阅读行话 FAQ。

第二，如果图将来可能包含回路或者链接不要使用此技术。

既无回路也无链接的图非常常见，即使像组合或者修饰样的“递归组合”设计模式。例如，表示 XML 文档或 HTML 文档的对象可以表示为一个没有回路和链接的图。

序列化图的关键是忽视节点的身份 ，但注重其内容 。算法（通常递归）遍历树并且不断地输出其内容。例如，如果当前节点恰好有一个整数 a，指针 B，浮点数 c 和另一个指针 d，那么你先输出整数 a， 然后递归遍历 b 指向的子节点，然后输出浮点数 c，最后递归遍历 d 指向的子节点 。 （你不必以声明顺序来进行序列化/反序列化，唯一的规则是，序列化和反序列化的顺序要一致。）

反序列化时你需要一个构造函数以 std::istream＆为参数。上述对象的构造函数将读入一个整数并存储结果到 a 中，然后分配一个对象存储在指针 b 中（需要传递 std::istream 到对象的构造函数中，然它也可以读取流的内容）， 读入一个浮点数到 c，最后将分配一个对象存储在指针 d 中 。一定要在对象中使用智能指针，否则，如果任何序列化抛出异常的话(除了第一个指针象之外)，将会出现内存泄露。

当创建对象的时候，使用命名构造函数惯用法很方便。这样做的好处是可以强制使用的智能指针。要在类 Foo 中实现这一点，需要添加一个如`FooPtr Foo::create(std::istream& istr) { return new Foo(istr); }` 的静态方法。（其中`FooPtr`是一个指向`Foo`的只能指针 ）。警觉的读者会注意到这与以前 FAQ 讨论的技术很相像 -这两种技术是完全兼容的。

如果一个对象包含可变数目的子对象，例如， 一个`std::vector`类型的指针，通常的做法是在递归之前输出子对象的数目。当反序列化时，只需要读入子对象的数目 , 然后就可以使用一个循环来创建适当数量的子对象。

如果一个子对象有可能是 `NULL`指针，一定要在序列化和反序列化时候做处理。这不应该是一个问题， 如果对象使用继承 ；详情见上面的解决方案。 否则，如果第一个序列化的成员有一个已知的范围，使用范围以外的东西来表示一个`NULL`指针。例如，如果一个序列化对象的第一个成员总是一个数字，那就使用比如`N`这样的非数字来表示一个 `NULL`指针。反序列化是偶可以使用`std::istream::peek()`来检查`'n`'标签。如果第一个成员没有范围，那就强加一个范围，例如，每个对象之前总是输出`'×'`，然后使用`’y’`来表示`NULL`。

如果一个对象内部本身包含另一个对象，而不是包含指向其他对象的指针，这与上述做法没有什么两样：你仍然需要递归子节点，就好像是通过指针一样。

## 36.10 我如何序列化包含指向其他对象指针的对象，这些指针是没有回路，但是有平凡链接的树形结构？

和之前一样，“树”并不意味着对象存储在数据结构的树中。它只是意味着你的对象互相指向对方。没有回路是指，如果不断地从一个对象指针跟随到下一个对象，你永远不会回到一个较早的对象。你的对象不是在树的内部，它们是一棵“树”。如果你不理解这些话，在继续之前你应该阅读行话 FAQ。

如果图包含叶节点的链接，但这些链接可以很容易地通过一个简单的查找表重建，那么使用此解决方案。举例来说，像`(3*(a+b) - 1/a)`样的算术表达式的解析树可能会有链接，因为变量名称（如`a`）出现不止一次。如果你想让图使用完全相同的节点对象来表示该变量的两次出现，那么你可以可能需要这种解决方案。

虽然上述限制，不适合于那些没有任何链接的解决方案。但是它是如此的接近，因此你可以想办法套用那个解决方案。差异是：

*   序列化时候，完全忽视链接。
*   反序列化时候，创建一个查找表， 比如`std::map<std::string,Node*>`，映射变量名称到相关的节点。

警告：这里假设变量 a 的所有出现应该映射到同一个节点对象；如果情况比这复杂，即如果一部分 a 映射到一个对象，另一些部分映射到另一个对象，你可能需要使用一个更复杂的解决方案。

## 36.11 我如何序列化包含指向其他对象指针的对象，这些指针是可能含有回路或者非平凡链接的图？

警告：术语“树”并不意味着对象存储在数据结构的树中。它只是意味着你的对象互相指向对方。没有回路是指，如果不断地从一个对象指针跟随到下一个对象，你永远不会回到一个较早的对象。你的对象不是在树的内部，它们是一棵“树”。如果你不理解这些话，在继续之前你应该阅读行话 FAQ。

如果图包含回路，或者包含比平凡链接解决方案中更复杂的链接，那么使用此解决方案。该解决方案处理的两个核心问题：它避免了无限循环，除了节点的内容之外还写入/读取每个节点的身份 。

一个节点的身份在不同的输出流中通常不会一致。例如，如果某个文件使用数字 3 代表节点 `x`，一个不同的文件可以使用数字 3 代表一个不同的节点`y` 。

有一些巧妙的方法来序列化这种图，但最容易描述的是两阶段(two-pass)算法，该算法使用一个对象的 ID 映射表 ，例如`std::map<Node*,unsigned> oidMap`。在第一阶段设置`oidMap`，也就是说，它构建一个对象指针到对象整数 ID 的映射。 通过递归图，在每个节点检查该节点是否已经在`oidMap`，如果没有在`oidMap`中，就添加节点和一个唯一的整数 ID 到 oidMap，然后递归新的子节点。唯一的整数 ID 通常取`oidMap.size()`，如`unsigned n = oidMap.size(); oidMap[nodePtr] = n`。（是的，需要使用两条语句。你也必须这样做。 不要缩短为一条语句。莫怪我没有警告你！）

第二阶段遍历`oidMap`的所有节点，并在在每个节点输出入节点的身份（相关的整数 ID）之后输入节点内容。当输出节点内容时候，如果包含指向其他节点的指针，不要遍历这些子对象，只需要输出子节点的身份（相关整数 ID）。例如，当节点包含`Node* child`，只需输出整数`oidMap[child]`。第二阶段之后，该`oidMap`可以被丢弃。换句话说，节点`*`到无符号整数 ID 的映射通常不会活过任何给定图的序列化结束。

也有一些巧妙的方法来反序列化这种图，但在这里还是使用最容易描述的是两阶段(two-pass)算法。第一阶段设置一个含有正确类型的`std::vector<Node*> v`对象，但所有这些对象的子指针都是`NULL`指针。这意味着`v[3]`将指向对象 ID 是 3 的对象，但该对象内部的任何子指针将都是 `NULL`指针。第二阶段设置对象内部的的子指针，例如， 如果 `v [3]`有一个子指针叫`child`，应该指向对象 ID 是 5 的对象。在第二阶段的将`v[3].child`从`NULL`修改为`v [5]`（显然封装可能禁止直接访问`v[3].child`，但最终`v[3].child`需要被改为`v[5]`）。反序列化一个给定流之后， `vector v`通常被丢弃。换句话说，当序列化或反序列化不同的流时，对象 ID（3，5 等等）没有任何意义-这些数字只在一个给定流内有意义。

注意：如果你的对象包含多态性指针（*polymorphic pointers*） ，也就是基类指针可能指向派生类对象，那么使用以前描述的技术。 你还需要阅读的先前的关于处理`NULL`指针和版本号的一些技术。

注意：当递归遍历图的时候，你应该考虑访问者模式。因为序列化可能只是需要做递归遍历的原因之一，任何递归都需要避免无限循环。

## 36.12 序列化/ 反序列化对象的时候有什么注意事项？

除非在特殊情况下，不要在遍历时候改变节点数据。例如，有些人觉得通过简单地增加一个节点类的整型数据成员，他们就可以映射`Node*`到整数。有时也添加布尔型的`haveVisited`标志作为`Node`对象的另外一个数据成员。

*但是* ，这将导致大量的多线程和/或性能问题。你的`serialize()`方法可能不能再为`const`，所以即使它在逻辑上只是读取节点数据，尽管它在逻辑上可以有多个线程同时读取节点，但是实际的算法却写入数据到节点。如果你理解线程和读/写冲突，你只能寄希望于你的代码更复杂更慢（你必须阻止所有的读取线程，只要任何线程对图进行操作的话）。如果你(或者后继的代码维护者)不理解线程和读/写冲突，这可能引起非常严重和非常不容易觉察的错误。读/写和写/写冲突很不容易觉察，这些错误很难测试到。坏消息！相信我！如果你不相信我，找个你信任的人。但是切莫偷工减料！

现在还有很多更多要说，例如一些特殊情况。但我已经花了太多时间。如果想了解更多信息，花一些钱吧。

## 36.13 什么是图，树，节点，回路，链接，叶链接与内部节点链接？

当你的对象包含指向其他对象的指针，你就有了一种计算机科学家称之为*图*的东东 。你的对象都存储在一个像树的数据结构里面；他们是一个像树的数据结构。

图的*节点*又名顶点相当于你的对象 ，图的*边*相当于你的对象里面的指针 。 该图是一个树的一个特例，称为*带根有向图*。要序列化的根对象对应于图的*根节点* ，指针对应于*直接边*。

如果对象 x 有一个指向对象 Y 的指针，我们说 x 是 Y*双亲* 或 Y 是 X 的 *孩子* 。

图的*路径*是指从一个对象开始，顺着指针到另一个对象等等，可以是任意深度。如果存在一个从 x 到 z 的路径，我们说，x 是 z 的祖先 和/或 z 是 x 的 子孙。

图的*链接*是指有两个或两个以上不同的路径可以到达同一对象。例如， 如果 z 是 x 和 y 的孩子，则图有一个链接，z 是一个*链接节点* 。

图的*回路**(**环 __)*是指存在一个返回对象自己的路径： 如果 x 有一个指向自己的指针，或指向 Y 的指针，Y 又指向 x，或者为 Y，Y 指向 z，Z 又指向 x 等。图是*有环的*如果有一个或多个回路，否则是*无环的* 。

一个*内部节点*是有孩子的节点。*叶节点*是没有孩子的节点。

正如本节中使用的那样，*树*是指一个*带根的，有向的，无环图*。请注意，树的节点也是树。