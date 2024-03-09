# 第十八章使用 wxSocket 编程

socket 是一个数据传输的管道.socket 并不关心它正在传输什么类型的数据,也不关心数据从和而来,或者说要到哪里去.它的任务就是把数据从 A 点传输到 B 点.每次你访问 web,收发 email,登录你的即时消息帐号等等时候,你在都使用着 socket.socket 可以被用来再任何支持 socket 的设备之间建立连接,包括连接一台电脑和一台电冰箱(只要它支持 socket).

socket 编程的 API 最初是 BSD unix 系统的一部分,因为其起源的单一性,这个 API 变成了一种标准.所有现代的操作系统都会实现一个 socket 层,来提供按照 TCP 或者 UDP 协议通过网络(比如国际互联网)向外发送数据.使用 wxWidgets 提供的 wxSocket,你可以安全的从一台电脑向另外一台电脑发送任何数量的数据.本章也将涉及一些 socket 技术的基础知识,但是 socket 操作本身是非常简单明了的.

虽然基本的 socket 操作是非常简单的,在 Windows,Linux 和 Mac OSX 平台上也是非常类似的,但是每个平台在实现 socket 的时候还是有一些细微的差别,必须针对某个特定的平台作一些适配.基于事件的 socket 操作在各个平台上的差异就更为突出,这使得在各个平台上使用这种机制都成为一个挑战.而 wxWidgets 则使用 wxSocket 类屏蔽了这些差别,从而使得制作基于事件的跨平台的 socket 程序变得相对容易.

另外需要注意的是,到作者停笔前为止,wxWidgets 还不支持 UDP 协议的数据收发,也许在将来的版本中会增加 UDP 的支持.

# 18.1 Socket 类和功能概览

socket 操作的核心类是 wxSocketBase,它提供了类似发送和接收数据,关闭连接,错误报告等这样的功能.创建一个监听 socket 或者连接到一个 socket 服务器,你需要分别使用 wxSocketServer 和 wxSocketClient.wxSocketEvent 用来通知应用程序 socket 上有事件发生.虚类 wxSocketBase 和它的一些子类比如 wxIPV4address 让你可以指定特定的远端地址和端口.最后, wxSocketInputStream 和 wxSocketOutputStream 等这些流对象让你以流的方式处理 socket 上的数据移动和传输.关于流操作的更多内容参见第十四章,"文件和流操作"

正如我们在稍后的"Socket 标记"小节中即将讨论的那样,socket 可以以不同的方式使用.传统的使用线程的操作方式将禁止 socket 事件的产生和发送,而在线程中以阻塞的方式进行 socket 的操作.而另一方面,你也可能使用基于事件的方式以便逃避使用线程的复杂性. wxWdigets 将在需要的时候通过事件通知你需要对某个 socket 进行操作了.通过这种方式,数据的接收是放在后台的,你仅需要在有数据到来的时候处理它,它将不会阻塞你的 GUI 界面,也没有基于每个线程一个 socket 的实现的那种复杂性.

本章我们通过一个完整的例子来介绍 wxSocket 的这两种使用方法以及使用到的那些 wxSocket 类的 API.虽然仅仅是一个例子,但是例子中的代码都可以作为正式的代码来使用.

# 18.2 Socket 及其基本处理介绍

让我们直接开始一个基于事件的 socket 客户机和服务器的例子,作为对 wxWidgets 中 socket 编程的介绍.代码是相当直观的,只需要你有一点最基础的 socket 编程的背景.为了简洁起见,所有 GUI 操作的部分将被省略,我们只关注那些 Socket 有关的函数.完整的代码可以在光盘的 examples/chap18 目录中找到.例子中用到的 socket API 都附有详细的使用手册.

这个例子程序的功能是很简单的,服务器倾听连接请求,当有客户端建立连接的时候,服务器首先从 socket 上接收 10 个字符,然后再把这 10 个字符发送回去.相应的,客户端在建立连接以后先发送 10 个字符,然后等待接收 10 个响应字符.在例子中,这 10 个字符写死为 "0123456789".服务器端和客户端的程序运行的样子如下图所示:

![](img/mht8D5C%281%29.tmp)

![](img/mht8D5F%281%29.tmp)

客户端的代码

下面列出了客户端的关键代码

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU(CLIENT_CONNECT, MyFrame::OnConnectToServer)
    EVT_SOCKET(SOCKET_ID,    MyFrame::OnSocketEvent)
END_EVENT_TABLE()
void MyFrame::OnConnectToServer(wxCommandEvent& WXUNUSED(event))
{
    wxIPV4address addr;
    addr.Hostname(wxT("localhost"));
    addr.Service(3000);
    // 创建 Socket
    wxSocketClient* Socket = new wxSocketClient();
    // 设置要监视的 Socket 事件
    Socket->SetEventHandler(*this, SOCKET_ID);
    Socket->SetNotify(wxSOCKET_CONNECTION_FLAG |
                                        wxSOCKET_INPUT_FLAG |
                                        wxSOCKET_LOST_FLAG);
    Socket->Notify(true);
    // 等待连接事件
    Socket->Connect(addr, false);
}
void MyFrame::OnSocketEvent(wxSocketEvent& event)
{
    // 从事件获取 socket
    wxSocketBase* sock = event.GetSocket();
    // 所有事件共享的一块缓冲(Common buffer shared by the events)
    char buf[10];
    switch(event.GetSocketEvent())
    {
        case wxSOCKET_CONNECTION:
        {
            // 填充'0'-'9'的 ASCII 码
            char mychar = '0';
            for (int i = 0; i &lt; 10; i++)
            {
                buf[i] = mychar++;
            }
            // 发送 10 个字符到对端
            sock->Write(buf, sizeof(buf));
            break;
        }
        case wxSOCKET_INPUT:
        {
            sock->Read(buf, sizeof(buf));
            break;
        }
        // 服务器在发送 10 个字节以后关闭了连接
        case wxSOCKET_LOST:
        {
            sock->Destroy();
            break;
        }
    }
} 
```

服务器端代码

下面列出了服务器端的代码

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU(SERVER_START, MyFrame::OnServerStart)
    EVT_SOCKET(SERVER_ID,  MyFrame::OnServerEvent)
    EVT_SOCKET(SOCKET_ID,  MyFrame::OnSocketEvent)
END_EVENT_TABLE()
void MyFrame::OnServerStart(wxCommandEvent& WXUNUSED(event))
{
    // 创建地址,默认为 localhost:0
    wxIPV4address addr;
    addr.Service(3000);
    // 创建一个 Socket,保留其地址以便我们可以在需要的时候关闭它.
    m_server = new wxSocketServer(addr);
    // 检查 Ok 函数以判断服务器是否正常启动
    if (! m_server->Ok())
    {
        return;
    }
    // 设置我们需要监视的事件
    m_server->SetEventHandler(*this, SERVER_ID);
    m_server->SetNotify(wxSOCKET_CONNECTION_FLAG);
    m_server->Notify(true);
}
void MyFrame::OnServerEvent(wxSocketEvent& WXUNUSED(event))
{
    // 接受连接请求,并创建 Socket
    wxSocketBase* sock = m_server->Accept(false);
    // 告诉这个新的 Socket 它的事件应该被谁处理
    sock->SetEventHandler(*this, SOCKET_ID);
    sock->SetNotify(wxSOCKET_INPUT_FLAG | wxSOCKET_LOST_FLAG);
    sock->Notify(true);
}
void MyFrame::OnSocketEvent(wxSocketEvent& event)
{
    wxSocketBase *sock = event.GetSocket();
    // 处理事件
    switch(event.GetSocketEvent())
    {
        case wxSOCKET_INPUT:
        {
            char buf[10];
            // 读数据
            sock->Read(buf, sizeof(buf));
            // 写回数据
            sock->Write(buf, sizeof(buf));
            // 服务器接受的这个 socket 已经完成任务,释放它
            sock->Destroy();
            break;
        }
        case wxSOCKET_LOST:
        {
            sock->Destroy();
            break;
        }
    }
} 
```

连接服务器

这一小段我们来解释一下怎样创建一个客户端 Socket 并且用它连接某个 Server.

Socket 地址

所有的 socket 地址相关的类都是基于虚类 wxSockAddress,它提供了基于 socket 标准的所有地址相关的参数和操作.而 wxIPV4address 类则具体实现了当前应用最广泛的标准国际地址方案 IPV4.wxIPV6address 类是用来提供 IPv6 支持的,不过它的功能实现的并不完整,等到 IPv6 在全世界范围内广泛使用的那天,这个类当然会相应的变得完整.

注意:如果地址使用的是一个长整型,那么它期待的是网络序排列方式,它返回的长整型地址也总是网络序排列方式.网络序是对应的是 Big endian(Intel 和 AMD 的 x86 体系使用的是 little endian,而 Apple 的系统使用的是 big endian).你可以使用字节序转换宏 wxINT32_SWAP_ON_LE 来进行平台无关的字节续转换,这个宏只在使用 little endian 的平台上才进行相应的转换工作.如下所示:

```cpp
IPV4addr.Hostname(wxINT32_SWAP_ON_LE(longAddress)); 
```

Hostname 可以采用的参数包括一个 wxString 类型的字符串(比如 www.widgets.org)或者一个长整型的 IP 地址(前面已经提到过,采用 big endian),如果没有任何参数,则 Hostname 返回当前主机的主机名.

Service 用来设置远端端口,你可以指定一个 wxString 类型的已知服务名或者直接指定一个 short 类型的整数.如果不带任何参数,Service 返回当前指定的远端端口.

IPAddress 函数返回一个十进制的以点分割的 wxString 类型的远端 ip 地址.

AnyAddress 将地址设置为本机的任何 IP 地址,相当于将地址设置为 INADDR_ANY.

Socket 客户端

wxSocketClient 继承自 wxSocketBase 并且同时继承了所有的通用 Socket 操作函数.新增的少数几个函数主要用来发起和建立远端连接.

Connect 函数采用一个 wxSockAddress 参数以便知道要连接的远端地址和端口.正如前面提到的那样,你应该使用类似 wxIPV4address 这样的地址而不能直接使用 wxSockAddress.第二个参数是一个 bool 类型,默认为 true,指示是否应该等连接建立再返回.如果这个函数在主线程中运行,所有的 GUI 都将冻结直至这个函数返回.

WaitOnConnect 用来在 Connect 被以 false 作为第二个参数调用以后(不阻塞)调用.第一个参数指示要等待的秒数, 第二个参数则用来指示毫秒数.无论连接函数成功还是失败,这个函数都将返回成功.只有当连接函数返回超时的时候,这个函数才会返回失败.如果第一个参数是 -1,则代表使用默认的超时时长,通常是 10 分钟.也可以使用 SetTimeout 函数修改默认的超时时长.

Socket 事件

所有的 Socket 事件都是使用同一个事件映射宏 EVT_SOCKET 指定的.

EVT_SOCKET(identifier, function)宏将标识符为 identifier 的事件发送给指定的函数处理.处理函数的参数类型为 wxSocketEvent.

wxSocketEvent 事件非常简单,内部存储了事件的标识符和对应的 wxSocket 对象指针,这可以避免自己保存 socket 指针的麻烦.

Socket 事件类型

下表列出了 GetSocketEvent 函数可能返回的事件类型.

| wxSOCKET_INPUT | 指示 socket 上有数据可以接收.无论是 socket 数据缓存原本没有数据,新收到了数据,还是说原本就有数据,只是用户还没有读完,都将产生这个事件. |
| --- | --- |
| wxSOCKET_OUTPUT | 这个事件通常在 socket 的 Connect 函数第一次连接成功或者说 Accept 刚刚接受了一个新的 Socket 的时候产生,并且通常是产生在 socket 的写操作失败,缓冲区的数据又从无到有的时候. |
| wxSOCKET_CONNECTION | 对于客户端来说,用来只是 Connect 动作已经成功了,对于服务端来说,指示新接受了一个 Socket. |
| wxSOCKET_LOST | 用来指示接收数据时针对 socket 的关闭操作.这通常意味着对端已经关闭了 socket.这个事件在连接失败的时候也有可能产生. |

wxSocketEvent 的主要成员函数

GetSocket 返回指向产生这个事件的 wxSocketBase 对象的指针.

GetSocketEvent 返回对应的上表列出的事件类型.

使用 Socket 事件

要处理 socket 事件,你需要首先指定一个事件处理器并且指定你想要处理的事件类型.wxSocketBase 支持的各种事件宏,你可以在上面的服务器端例子中监听 socket 创建以后的代码中看到.需要注意的事,对 Socket 事件的设置仅对当前的 socket 起作用,如果你希望监听别的 socket 的相关事件,你需要对那个 socket 再次设置监听事件.

SetEventHandler 函数将某个事件标识符和相应的事件处理器关联起来. 事件标识符必须和事件处理器对应的事件表中指定的标识符相对应.

SetNotify 用来设置想要监听的事件,它的参数是一个 bit 为列表,比如 wxSOCKET_INPUT_FLAG | wxSOCKET_LOST_FLAG 将监听有数据到来以及 socket 被关闭事件.

Notify 使用一个 bool 类型的参数,来指示你是否想或者不想收到当前指定的事件.它的作用是让你在 SetNotify 之后可以不带事件指示来打开或者关闭事件监听.

Socket 状态和错误提醒

在讨论数据发送和接收之前,我们先来描述一下 socket 状态和 socket 的错误提醒,以便我们在讨论数据接收的时候可以引用他们.

Close 函数关闭 socket,禁止随后的任何数据传输并且会通知对端 socket 已经被关闭.注意可能在关闭之前已经缓存了一些 socket 事件,因此在 socket 被关闭之后你可能还要准备好处理可能缓存的 socket 事件.

Destroy 函数应该代替针对 socket 的 delete 操作,原因和 Window 对象类似,有可能队列中仍然有针对这个 socket 的事件,因此,在系统事件队列处理完以后再释放这个 socket 是一个安全的作法,Destroy 函数正是提供了这个功能.

Error 函数返回 True 如果上次的 socket 操作遇到某种错误.

GetPeer 返回一个 wxSockAddress 引用,它包含当前 socket 的对端信息比如 IP 地址和端口号.

IsConnected 返回是否这个 socket 已经成功连接.

LastCount 返回最近一次读写操作成功进行的字节数.

LastError 返回最近一次的错误码.注意如果操作成功并不会更新最近一次的错误码,因此你需要使用 Error 函数来判断最近一次操作是否成功.socket 所支持的错误码如下表所示:

| wxSOCKET_INVOP | 非法操作,比如使用了非法的地址类型. |
| --- | --- |
| wxSOCKET_IOERR | I/O 错误,比如无法创建和初始化 socket. |
| wxSOCKET_INVADDR | 不正确的地址, 比如试图连接空地址或者不完整的地址. |
| wxSOCKET_INVSOCK | socket 使用方法不正确或者尚未初始化. |
| wxSOCKET_NOHOST | 指定的地址不存在. |
| wxSOCKET_INVPORT | 无效端口. |
| wxSOCKET_WOULDBLOCK | socket 被指示为非阻塞 socket,但是操作将导致阻塞 (参见 socket 模式的讨论). |
| wxSOCKET_TIMEDOUT | socket 操作超时. |
| wxSOCKET_MEMERR | socket 操作时内存分配失败. |

Ok 返回 True 的条件是: 客户端 Socket 必须已经和 Server 建立连接或者服务端 Socket 已经成功绑定了本地地址并且开始监听客户端连接

SetTimeout 指定阻塞式访问的超时时长.默认为 10 分钟.

发送和接收 Socket 数据

wxSocketBase 提供了各种基本的或高级的读写 socket 操作.所有操作都将保存相关的数据并且支持使用 LastCount 返回成功操作的字节个数,LastError 返回最近一次遇到的操作错误码.

接收

Discard 函数删除所有的 socket 接收缓冲区数据.

Peek 函数让你可以读取缓冲区的数据但是不将 socket 缓冲区清除.你必须指定要 Peek 的数据的大小并且自己提供 Peek 目的地的缓冲区.

Read 函数和 Peek 一样,只是它在成功获取数据以后会清除相应的 Socket 接收缓冲区.

ReadMsg 函数对应于 WriteMsg 函数,将会完整的接收 WriteMsg 发送的数据,除非需要系统错误.注意如果 ReadMsg 开辟的缓冲区比 WriteMsg 发送的数据少,则多出的数据将被直接删除.

Unread 将数据放回接收缓冲区,你需要指定希望放回去的数据的字节数.

发送

Write 函数以参数中数据指针指向的缓冲作为开始位置,向 socket 写入参数中指定的数据大小.

WriteMsg 和 Write 的区别在于,wxWidgets 会增加一个消息头,以便接收端可以准确的知道消息的大小,WriteMsg 发送的数据必须由 ReadMsg 函数接收.

创建一个 Server

wxSocketServer 也只对其基类 wxSocketBase 增加了少数几个函数用来创建和监听连接请求.要创建一个 Server,你必须指定要监听的端口.wxSocketServer 使用和 wxSocketClient 一样的 wxIPV4address 类型,只是前者不需要指定远端地址.在大多数情况下,你需要调用 Ok 函数来判断是否绑定和监听动作已经成功.

wxSocketServer 的主要成员函数

wxSocketServer 构造函数使用一个地址对象用来指定监听端口,以及一个可选的 Socket 标记(参见下一节"Socket Flags").

Accept 函数返回一个新的 socket 连接或者立即返回 NULL,如果没有连接请求.你可以设置可选的等待标记,如果你这样做,Accept 将导致程序阻塞.

AcceptWith 和 Accept 的功能相近,只是它提供一个额外的已存在的 wxSocketBase 对象(引用),并且其返回值为 bool 型,用来指示是否接受了一个新的连接.

WaitForAccept 采用一个秒参数和一个毫秒参数以指定在某个事件范围内等待新的连接请求,如果请求发生则返回 True,否则超时返回 False.

处理新的连接请求事件

当监听 socket 检测到一个新的连接请求的时候,将产生一个相应的事件.在其事件处理函数中,你可以接受这个请求并且执行任何必要的即时处理.你需要保证连接在其生命周期内不被立即关闭,你还需要为新接受的 socket 指定事件处理器.注意监听的 socket 在被关闭之前将一直在监听, 而每一个新的连接请求都会创建一个新的 socket.在 server 的整个生命周期内,同一个监听 socket 可以接受成千上万个新的 socket.

Socket 事件概述

从程序员的观点来说,基于事件的 socket 处理简化了 socket 编程,使得他们不需要关心线程的创建和释放.这个例子没有使用线程, 但是 GUI 界面同样不会阻塞,因为所有的数据读取都是在确信有数据到来的时候才进行的,因此会立即返回.如果有很大量的数据需要读取,你可以将它们分为多个小部分,然后一次读一部分并将其放入你自己的缓冲区.或者你可以使用 Peek 函数检查当前缓冲区的数据的数量,如果没有达到需要处理的范围,你可以什么也不做,静静等待下一次数据事件通知的到来.

在下一节,我们来看看怎样使用不同的 socket 标记来改变 socket 的行为.

# 18.3 Socket 标记

Socket 的行为可能随着创建时指定的不同而有很大的差异,下表列出了 Socket 可以指定的标记:

| wxSOCKET_NONE | 普通行为(行为和底层的 send 和 recv 函数一致). |
| --- | --- |
| wxSOCKET_NOWAIT | 读和写操作尽可能快速的返回. |
| wxSOCKET_WAITALL | 等待所有的读写数据完成操作,除非出现系统错误. |
| wxSOCKET_BLOCK | 在读写数据的时候阻塞 GUI 界面. |

如果没有指定任何标记(或者指定了 wxSOCKET_NONE 标记),I/O 操作将在部分数据被读写的时候返回,甚至在整个数据还没有传输完的情况下也是这样.这和使用阻塞方式调用底层的 recv 或者 send 函数的效果是一样的.注意这里所说的阻塞指的是函数被阻止返回,并不意味着图形用户界面被冻结.

如果指定了 wxSOCKET_NOWAIT 标记,I/O 将立刻返回.对于读操作,它将读取所有当前输入缓冲区拥有的数据后立刻返回,对于写操作,它将尽可能多的发送数据以后立刻返回,这取决于当前的输出缓冲区的大小,这种方式等同于使用非阻塞方式调用底层函数 recv 或 send.同样, 这里的阻塞也指的是函数返回,而不是用户界面阻塞.

如果指定了 wxSOCKET_WAITALL 标记,I/O 操作将在所有要求的数据被读取或者被写入以后(或者发生系统错误)才会返回. 如果需要的话,将以阻塞的方式调度底层系统函数.这相当于使用一个循环重复以阻塞的方式调用 recv 或者 send 函数以便传输所有的数据.同样,这里的阻塞也指的是阻塞底层函数而不是 GUI.注意 ReadMsg 和 WriteMsg 函数将隐式使用这种方式,并且忽略你可能设置的 wxSOCKET_NONE 或 wxSOCKET_NOWAIT 标记.

用来指示 wxSOCKET_BLOCK 是否在 I/O 操作的间隙执行 Yield 操作(译者注:参见前面关于线程的替代方案中的描述),如果指定了这个标记,socket 在底层操作间隙将不会执行 Yield 动作,反之,如果没有指定这个标记,那么你要非常小心这可能产生的代码重入的问题.

总的来说:

*   wxSOCKET_NONE 总是试图读取或者写入一些数据,但是不关心具体是多少数据.
*   wxSOCKET_NOWAIT 只关心尽快返回,即使没有读取或写任何数据.
*   wxSOCKET_WAITALL 将在所有的数据都被写入或者是读取的数据达到要求的数目的时候返回.
*   wxSOCKET_BLOCK 和前面的标记没有关系,只控制否则在底层操作的间隙执行 Yield 动作.

wxWidget 中的阻塞和非阻塞 socket

在 wxWidgets 中的阻塞方式有双重的含义.在一般的编程中,socket 阻塞意味着当前的线程被挂起直至 socket 操作超时或者数据操作完成.如果是主线程阻塞,则用于界面也相应阻塞.

而在 wxWidgets 中,有两种类型的阻塞:socket 阻塞和用户界面阻塞.wxSOCKET_BLOCK 标记指示在 socket 阻塞的时候,是否同时阻塞用户界面.你也许回问,这怎么可能呢,怎么可能作到阻塞了 socket 操作而不阻塞用户界面呢?这主要是因为在 socket 被阻塞的时候,wxWidgets 还可以处理所有的事件,因为 socket 的底层函数处理的间隙调用了 wxYield,这个函数可以处理队列中所有未处理的事件,包括用户界面相关的事件.虽然在 socket 操作未结束之前,代码一直在 socket 函数中运转,但是所有事件还是可以被有序的处理.

对于刚开始使用 wxWidgets 的人来说,听上去这是一个很美妙的事情.如果你是第一次使用 wxWidgets 进行 socket 编程,你可能会觉得,再也不需要使用任何单独的线程来处理 socket 了,你可以将 socket 设置为 wxSOCKET_WAITALL 和 wxSOCKET_BLOCK,这样你可以通过事件机制处理 socket 数据,而 GUI 也不会被阻塞,不幸的是,我必须先警告你,这种想法可能是你痛苦的开始.

让我们来假设一个服务端有两个活动连接,每一个都设置了 wxSOCKET_WAITALL 标记.更进一步,我们假设其中一个连接正在以一种很缓慢的速度传输一个很大量的数据.Socket 1 的读缓冲区没有数据了,因此它调用了 wxYield,而 Socket2 还有未处理的事件在队列里,这时候会发生的事情是:Socket1 调用的 wxYield 试图处理 Socket2 相关的消息,但是因为 Socket2 的连接很缓慢,它的事件总是结束不了,它也会调用 wxYield 来避免 GUI 阻塞,这将导致出现一个名声不太好的告警消息"wxYield called recursively"(wxYield 被递归调用),在 Socket2 的数据传输结束之前,应用程序的堆栈将最终被 wxYield 的递归调用给耗尽, 因为递归调用使用这些堆栈一直没有机会释放,于是人们开始联系 wxWidgets 社区,报告发现了一个 bug,而实际上,这应该是应用程序自己的问题而不是 wxWidgets 的问题.应用程序不应该以这种方式来编程,它应该避免这种情形出现,因此,恐怕 wxWidgets 永远没有办法改正这个问题.

另外一方面,性能也是一个问题,为了不阻止 GUI,你的应用程序将不得不浪费 CPU 的资源.试想一下,用户界面要立即响应, Socket 也要不停的监视是否有数据到来以便产生事件通知应用程序,要让两者都得到满足,wxWidgets 所能做的唯一的办法就是使用循环,不停的以非阻塞的方式去用系统操作 select 去测试 socket,然后再调用 wxYield 处理 GUI 事件.

不可用的标记组合

容我再罗嗦一句,不要天真到认为 wxWidgets 采用了一种神奇的 socket 处理机制.无论这些 Socket 的标记在你的第一印象中看起来是多么的诱人,你都不可能同时满足下面的这些要求:

*   wxSOCKET_WAITALL
*   不阻塞 GUI
*   少于 100%的 CPU 占用率
*   单线程

你可以指定 wxSOCKET_WAITALL 并且也不阻塞 GUI,但是这将导致 100%的 CPU 占用率.如果你愿意付出阻塞 GUI 作为代价(指定 wxSOCKET_BLOCK 标记),你将可以得到 wxSOCKET_WAITALL 和 0%的 CPU 占用率. 或者你可以使用多线程来实现同时使用 wxSOCKET_WAITALL 又不阻塞 GUI,并且也不用 100%的 CPU 占用率.你也可以不用 wxSOCKET_WAITALL 而使用 wxSOCKET_NOWAIT 以便可以既不占用 100%的 CPU 又不堵塞 GUI.总之一句话,上面的四个条件, 你总可以同时满足任意三个,但是你不可以四个条件同时满足.

这些标记是怎样影响 Socket 的行为的

wxSOCKET_NONE, wxSOCKET_NOWAIT 和 wxSOCKET_WAITALL 是互斥的,你不可以同时使用他们中间的任何两个,而 wxSOCKET_BLOCK 和 wxSOCKET_NOWAIT 的组合也是没有意义的 (如果任何函数都立即返回,怎么可能阻塞 GUI 呢?), 因此下面五种标记的组合是有意义的:

*   wxSOCKET_NONE | wxSOCKET_BLOCK: 这种组合和标准的 socket 调用(recv 和 send)的行为相同.
*   wxSOCKET_NOWAIT: 和标准的非阻塞的 socket 调用行为相同.
*   wxSOCKET_WAITALL | wxSOCKET_BLOCK: 和普通的阻塞式 socket 调用的行为相同,只不过 recv 和 send 函数将被连续多次调用以便接受或者发送完整的数据.
*   wxSOCKET_NONE: 和标准的 socket 调用行为相同,只是由于在完成系统调用之前(比如数据完整接收缓冲区数据之前)调用了 wxYield,因此看上去 GUI 并不会阻塞.
*   wxSOCKET_WAITALL: 和 wxSOCKET_WAITALL | wxSOCKET_BLOCK 的行为相同只不过 GUI 将不被阻塞.

只有最后两种情况可能出现前面介绍的 wxYield 函数重入的问题,不过这俩组标记也是在 wxWidgets 中基于事件的 socket 编程中最主要的两种方式(因为他们阻塞了 socket 但是却不阻塞 GUI).使用这两组标记的时候要非常小心避免这个问题,虽然它们很强大,很有用,却也往往是错误和麻烦的根源,因为它们太容易被误解了.

标准 socket 和 wxSocket

使用 wxSOCKET_NONE | wxSOCKET_BLOCK 或 wxSOCKET_NOWAIT 的时候和直接使用 socket 系统调用的效果并没有不同,唯一的不同是你使用的是 wxWidgets 提供的 API 而不是标准 C 的 API.不过,即使这样,还是有足够的理由要使用 wxSocket,这些理由包括:wxSocket 提供了一个面向对象的接口,隐藏了很多平台相关的初始化代码,还提供了一些高级的函数比如 WriteMsg 和 ReadMsg.另外,下一节我们也将看到, wxSocket 也使我们可以用流的方式来操作 socket.

# 18.4 使用 Socket 流

使用 wxWidgets 的流,你仅使用很少的代码就可以很容易传输很大量的数据.现在,假设我们要通过 socket 来传输一个文件.你可能使用的方法是: 打开这个文件,将所有的内容读入内存,然后将这块内存写入到 socket.这种方法对于小文件来说没什么问题,但是如果这个文件是一个很多兆的大文件,将其完整读入内存对于一个速度和内存都很小的电脑来说,显得有些不太现实.而且通常我们需要对大文件进行压缩然后才发往 socket 以便降低网络流量.怎么办法呢,把大文件读入内存,对其整个进行压缩,然后再一次性发送 socket,这样得作法在效率和实用性方面都值得怀疑.

OK,我们来想另外一种办法,每次从文件里读入一小段数据,比如几 K 数据,然后将其压缩,然后发往 socket,如此反复.不幸得是, 小段压缩比起整个文件一起压缩来,压缩效率是大打折扣的.因此我们需要更进一步,维护一个压缩的状态,以便后面的小段数据可以使用前面的压缩信息,也可以避免多次传递压缩头信息.可是到目前为止,你的代码已经变的很庞大了,要分段读取文件,维护压缩数据,压缩并且写入 socket. wxWidgets 提供了一种更简便的方法.

因为 wxWidgets 提供了 wxSocketInputStream 和 wxSocketOutputStream 类,通过别的流来将数据读出或者写入 socket 是非常方便的.因为 wxWidgets 提供了基于文件,字符串,文本,内存以及 zlib 压缩的流操作,将这些流和 socket 流结合起来使用,可以实现很有趣也是很强大的 socket 数据操作方法.现在,回过头来看看我们刚才说的通过 socket 压缩传输大文件的问题,我们可能已经找到了一个更方便的途径.要发送一个文件,我们可以现将来自文件的数据流通过 zlib 的压缩流以后发送到 socket 的发送流,这样我们一下在就有了强大的支持大文件,支持压缩的,每次只需要读几 K 的 socket 文件传输方法了.而在接收端,我们同样可以使用流操作将来自 socket 流的数据通过 zlib 解压缩流发送到文件输出流,最后还原为原来的文件.所有这些可能几行代码就足够了.

我们将使用线程来处理整个过程,以便我们可以既不占用 100%的 CPU,又不阻塞 GUI(正如上一小节讨论的那样),要知道在使用 socket 传输大型的数据的时候,(如果不使用多线程,)这种阻塞几乎是不可避免的.

完整的例子可以在光盘的 examples/chap18 目录中找到.

文件发送线程

下面的例子中演示了流对象在堆上创建,FileSendThread 派生自 wxThread.

```cpp
FileSendThread::FileSendThread(wxString Filename,
                                 wxSocketBase* Socket)
{
    m_Filename = Filename;
    m_Socket = Socket;
    Create();
    Run();
}
void* FileSendThread::Entry()
{
    // 如果 10 秒之内我们什么数据都发送不了,就超时退出
    m_Socket->SetTimeout(10);
    // 在所有数据发送完成之前,阻塞一切非 socket 操作
    m_Socket->SetFlags(wxSOCKET_WAITALL | wxSOCKET_BLOCK);
    // 从特定的文件流中读入数据
    wxFileInputStream* FileInputStream =
        new wxFileInputStream(m_Filename);
    // 用来写入 socket 的流对象
    wxSocketOutputStream* SocketOutputStream =
        new wxSocketOutputStream(*m_Socket);
    // 我们写入的将是压缩以后的数据
    wxZlibOutputStream* ZlibOutputStream =
        new wxZlibOutputStream(*SocketOutputStream);
    // 将文件的内容写入压缩流
    ZlibOutputStream->Write(*FileInputStream);
    // 写所有的数据
    ZlibOutputStream->Sync();
    // 释放 ZlibOutputStream 将导致发送 zlib 的压缩结束标记
    delete ZlibOutputStream;
    // 释放资源
    delete SocketOutputStream;
    delete FileInputStream;
    return NULL;
} 
```

文件接收线程

接收例子演示了相关流对象也可以在栈上创建.FileReceiveThread 派生自 wxThread.

```cpp
FileReceiveThread::FileReceiveThread(wxString Filename,
                                    wxSocketBase* Socket)
{
    m_Filename = Filename;
    m_Socket = Socket;
    Create();
    Run();
}
void* FileReceiveThread::Entry()
{
    // 如果 10 秒内什么也收不到,中止接收
    m_Socket->SetTimeout(10);
    // 在我们成功接收完数据之前,阻塞一切其它的代码
    m_Socket->SetFlags(wxSOCKET_WAITALL | wxSOCKET_BLOCK);
    // 用于输出数据到文件的流对象
    wxFileOutputStream FileOutputStream(m_Filename);
    // 从 socket 接收数据的流对象
    wxSocketInputStream SocketInputStream(*m_Socket);
    // zlib 解压缩流对象
    wxZlibInputStream ZlibInputStream(SocketInputStream);
    // 将解压缩以后的结果写入文件
    FileOutputStream.Write(ZlibInputStream);
    return NULL;
} 
```

# 18.5 替代 wxSocket

虽然 wxSocket 提供了很多灵活性并且被很好的集成进了 wxWidgets,但是它并不是实现进程间通信的唯一方法.如果你只是想进行 FTP 或者 HTTP 的操作,你可以直接使用 wxFTP 或 wxHTTP,它们在内部使用了 wxSocket,不过这些类是不完善的,你最好还是使用 CURL,它是一个通用的库,提供了使用各种 Internet 协议传递文件的非常直观的 API,有人已经对其进行了 wxWidgets 封装,名字叫做 wxCURL.

wxWidgets 也提供了一套高级的进程间通信机制,它使用类 wxServer,wxClient 和 wxConnection 以及基于微软的 DDE(动态数据交换)协议的 API.实际上,在 windows 上,这些类是用 DDE 实现的,而在其它平台上,则是用 socket 实现的.之所以要使用这些更高层的类,是因为它比直接使用 wxSocket 更方便,另外一个优点是在 windows 平台上,使用 DDE 可以和别的支持 DDE 的程序交换数据(别的程序不必要是使用 wxWidgets 制作的).它的一个缺点是在别的平台上,非 wxWidgets 编制的程序是不能识别这种协议的,不过,如果你只需要在 wxWidgets 制作的程序之间交换数据的话,它还是可以满足要求的.我们将在第二十章的"单个实例还是多个实例?"小节,演示一个简单的例子.

更多信息请参考 wxWidgets 手册中的"Interprocess Commun-ication Overview"(进程间通信概述)小节以及 wxWidgets 自带的 samples/ipc 中的例子.你也可以参考 wxWidgets 自带的独立帮助显示工具中的代码,它位于 utils/helpview/src 目录内.

# 第十八章小结

我们在这一章里讨论了在 wxWidgets 中集成了 wxSocket 类,并且描述了它和 C 语言中的 socket 层的关系.为了让 socket 编程更容易, wxWidgets 还实现了 socket 的流操作,以便可以容易的和别的流对象进行交互来操作数据.只要你注意我们在 socket 标记小节中讨论的那些可能出现误解的地方,相信你可以使用 wxSocket 和其它相关的类制作出稳定的可以在各种平台上交叉编译的 socket 程序.

在下一章里,我们来看看如何使用 wxWidgets 提供的文档/视图框架来简化你的应用程序设计.