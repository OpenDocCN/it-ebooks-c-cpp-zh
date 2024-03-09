# 第二十章完善你的应用程序

一个简朴可用的应用程序和一个方便优雅的应用程序之间有很大的不同.如果只是内部使用,基本的不带有很多修饰的应用程序也许是足够的,但是如果你的应用程序打算分发给世界范围的很多人使用,你应该让它作到更令人信服和更容易使用,就像大多数商业软件公司制作的软件一样.你的软件应该遵守很多约定俗称的惯例和标准,比如提供配置对话框和联机帮助等.在这本书的最后一章,我们将讨论下面几个话题,它们可以让你的软件显得更专业:

*   单实例程序还是多实例程序? 怎样阻止同时运行你的程序的多个实例.
*   更改事件处理机制.怎样更改事件的执行顺序.
*   降低闪铄.怎样通过降低闪铄的方法提高你的应用程序可视界面的观感.
*   实现在线帮助.将给你提供一些实现各种联机帮助的建议.
*   解析命令行参数. 让你的用户可以通过命令行参数更直接的控制你的应用程序的行为.
*   保存应用程序所用的资源. 介绍你打包各种资源文件的方法.
*   调用别的应用程序. 从简单的调用别的程序执行,到控制别的应用程序的输入和输出方法.
*   管理应用程序设置. 通过 wxConfig 类来保存和加载应用程序设置以及和应用程序设置相关的一些提示.
*   应用程序安装.一些关于怎么让你的用户可以方便的在它们的平台上安装你的软件包的建议.
*   遵循用户界面设计规范. 关于各个平台用户界面设计规范方面的一些建议.

# 20.1 单个实例和多个实例

依照你的应用程序的性质的不同,你可能允许你的用户同时运行多个你的应用程序的实例,或者你也可能希望同时只能存在一个你的应用程序的实例,如果用户试图打开第二个或者试图通过资源管理器打开多个和你的应用程序关联的文档,它们都将只看到一个应用程序在运行.一种很通用的作法是,让你的用户根据他的工作习惯,自己选择是否允许运行应用程序的多个实例.不过同时运行多个实例有一个大问题是,哪个程序实例首先将配置文件写入磁盘是不一定的,因此用户可能会丢失它们的设置.而且多个实例运行对一些新手来说也有麻烦,它们可能并没有意识到它们已经运行了应用程序的多个实例.允许同时运行应用程序的多个实例在所有的平台上都是默认选项(除了在 Mac 平台上通过 Finder(查找器)来启动应用程序打开文档的时候),因此,如果你不希望你的应用程序可以运行多个实例, 你需要一些额外的代码.

在 Mac OS(也仅在这个系统)上,使用单一实例打开多个文档是非常容易的.你只需要重载 MacOpenFile 函数,这个函数采用一个 wxString 类型的文件名作为参数,这个函数将在 Mac Os 的 Finder 打开和这个应用程序关联的文档的时候被调用.如果当前还没有运行任何这个应用程序的实例,Mac 系统会先启动一个实例,然后再调用这个函数(这个大多数系统不同,在别的系统上,打开文档通常是通过调用应用程序并且将文件名作为参数的方法).如果你使用的是文档/试图框架,你可能并不需要重载这个函数,因为其在 Mac OsX 上的实现代码如下:

```cpp
void wxApp::MacOpenFile(const wxString& fileName)
{
    wxDocManager* dm = wxDocManager::GetDocumentManager() ;
    if ( dm )
        dm->CreateDocument(fileName, wxDOC_SILENT) ;
} 
```

然后,即使在 Mac Os 上,用户还是可以通过直接多次运行程序的方法运行你的应用程序的多个实例. 如果你想检测和禁止运行超过一个应用程序实例,你可以在程序运行之初使用 wxSingleInstanceChecker 类.这个对象将保持在应用程序的整个生命周期,因此,在你的 OnInit 函数中,调用其 IsAnotherRunning 函数检测是否已经有别的实例正在运行,如果返回 true,你可以在警告你的用户之后,立刻退出应用程序.如下所示:

```cpp
bool MyApp::OnInit()
{
    const wxString name = wxString::Format(wxT("MyApp-%s"),
                              wxGetUserId().c_str());
    m_checker = new wxSingleInstanceChecker(name);
    if ( m_checker->IsAnotherRunning() )
    {
        wxLogError(_("Program already running, aborting."));
        return false;
    }
    ... more initializations ...
    return true;
}
int MyApp::OnExit()
{
    delete m_checker;
    return 0;
} 
```

但是,如果你想把旧的实例带到前台,或者你想使用旧的实例打开传递给你的新的实例作为命令行参数的文件,该怎么办呢?一般说来,这需要在这两个实例间进行通讯.我们可以使用 wxWidgets 提供的高层进程间通讯类来实现.

在下面的例子中,我们将实现应用程序多个实例之间的通讯,以便允许第二个实例询问第一个实例是否它自己打开相应的文件,还是将自己提到前台以便提醒用户它已经请求用这个应用程序来打开文件,下面的代码声明了一个连接类,这个类将被两个实例使用.一个服务器类将被老的实例使用以便监听别的实例的连接请求,一个客户端类将被后来的实例使用以便和老的实例通讯.

```cpp
#include "wx/ipc.h"
// Server 类,用来监听连接请求
class stServer: public wxServer
{
public:
    wxConnectionBase *OnAcceptConnection(const wxString& topic);
};
// Client 类,在 OnInit 函数中被后来的实例使用
class stClient: public wxClient
{
public:
    stClient() {};
    wxConnectionBase *OnMakeConnection() { return new stConnection; }
};
// Connection 类,被两个实例同时使用以实现通讯
class stConnection : public wxConnection
{
public:
    stConnection() {}
    ~stConnection() {}
    bool OnExecute(const wxString& topic, wxChar*data, int size,
                   wxIPCFormat format);
}; 
```

OnAcceptConnection 函数在老实例(Server)中当有新实例(Client)进行连接请求的时候被调用.我们应该首先检查老实例中没有显示任何模式对话框,因为如果有模式对话框,就不可能有别的行为可以引起用户的注意.

```cpp
// 接收到了来自别的实例的连接请求
wxConnectionBase *stServer::OnAcceptConnection(const wxString& topic)
{
    if (topic.Lower() == wxT("myapp"))
    {
        // 检查没有活动的模式对话框
        wxWindowList::Node* node = wxTopLevelWindows.GetFirst();
        while (node)
        {
            wxDialog* dialog = wxDynamicCast(node->GetData(), wxDialog);
            if (dialog && dialog->IsModal())
            {
                return false;
            }
            node = node->GetNext();
        }
        return new stConnection();
    }
    else
        return NULL;
} 
```

OnExecute 函数在客户端实例对其连接对象调用 Execute 函数的时候被调用. OnExecute 函数可以有一个空的参数,这表示它需要将自己提到前台就可以了,否则,它需要检测参数中的文件名指示的文件是否已经被它打开,如果已经打开,将这个文件显示给用户,否则,就打开这个文件,再将其显示给用户.

```cpp
// 打开别的实例传来的文件参数.
bool stConnection::OnExecute(const wxString& WXUNUSED(topic),
                             wxChar *data,
                             int WXUNUSED(size),
                             wxIPCFormat WXUNUSED(format))
{
    stMainFrame* frame = wxDynamicCast(wxGetApp().GetTopWindow(), stMainFrame);
    wxString filename(data);
    if (filename.IsEmpty())
    {
        // 只需要提升主窗口
        if (frame)
            frame->Raise();
    }
    else
    {
        // 检查文件是否已经打开并且将其显示给用户
        wxNode* node = wxGetApp().GetDocManager()->GetDocuments().GetFirst();
        while (node)
        {
            MyDocument* doc = wxDynamicCast(node->GetData(),MyDocument);
            if (doc && doc->GetFilename() == filename)
            {
                if (doc->GetFrame())
                    doc->GetFrame()->Raise();
                return true;
            }
            node = node->GetNext();
        }
        wxGetApp().GetDocManager()->CreateDocument(
                                         filename, wxDOC_SILENT);
    }
    return true;
} 
```

在 OnInit 函数中,应用程序应该首先象前面介绍的那样使用 wxSingleInstanceChecker 检查是否已经运行了多个实例,如果没有别的实例运行,这个实例可以将自己设置位一个 Server,等待别的应用程序实例的连接请求,如果已经有实例在运行,就创建一个和那个实例的连接,第二个实例请求第一个实例打开自己被请求的文件或者提升其主窗口.下面是相关的代码:

```cpp
bool MyApp::OnInit()
{
    wxString cmdFilename; // code to initialize this omitted
    ...
    m_singleInstanceChecker = new wxSingleInstanceChecker(wxT("MyApp"));
    // 如果使用单实例,用 IPC 检测是否有别的实例.
    if (!m_singleInstanceChecker->IsAnotherRunning())
    {
        // 创建一个服务器
        m_server = new stServer;
        if ( !m_server->Create(wxT("myapp") )
        {
            wxLogDebug(wxT("Failed to create an IPC service."));
        }
    }
    else
    {
        wxLogNull logNull;
        // OK, 已经有一个实例了,创建一个和它之间的连接,然后在自己退出之前发送文件名
        stClient* client = new stClient;
        // 下面的参数在使用 DDE 的时候被忽略,在使用基于 TCP/IP 的类的时候代表主机名.
        wxString hostName = wxT("localhost");
        // 创建连接
        wxConnectionBase* connection =
                     client->MakeConnection(hostName,
                                     wxT("myapp"), wxT("MyApp"));
        if (connection)
        {
            // 请求那个已经存在的实例打开文件或者提升它自己
            connection->Execute(cmdFilename);
            connection->Disconnect();
            delete connection;
        }
        else
        {
            wxMessageBox(wxT("Sorry, the existing instance may be
            too busy too respond.\nPlease close any open dialogs and retry."),
                wxT("My application"), wxICON_INFORMATION|wxOK);
        }
        delete client;
        return false;
    }
    ...
    return true;
} 
```

如果你想要了解更多这里用到的进程间通讯的细节,你可以在 wxWidgets 自带的 utils/helpview/src 目录中,找到另外一个用在独立的 wxWidgets 帮助阅读器中的例子,在那个例子中,别的应用程序会通过进程间通讯的方式请求帮助阅读器程序打开某个帮助文件,另外在 wxWidgets 的 samples/ipc 例子中,也演示了 wxServer, wxClient 和 wxConnection 的用法.

# 20.2 更改事件处理机制

在通常情况下,wxWidgets 将事件发送到产生这个事件的窗口(或者别的事件处理器).对于命令事件,还将以特定的方式遍历整个窗口继承树(详情参考附录 H,"wxWidgets 的事件处理机制")来处理.举例来说,如果你点击工具条上的任何一个工具按钮,产生的事件将首先发送给这个工具按钮的事件表处理,然后是包含这个工具条的 frame 窗口的时间表,然后是整个应用程序类的事件表.通常情况下,这样的作法是满足要求的,但是如果有时候,你希望超过一个控件都可以使用工具条上的拷贝命令的时候,就会有一些问题,比如,你的主程序中有一个主窗口(假设是一个绘画程序)和一个文本编辑框,这个文本编辑框可能永远也无法处理工具条上的拷贝命令,因为这个命令并不会调用这个编辑框的事件处理函数.在这种情况下,你可能希望事件能够首先交给当前处于活动状态的控件处理,然后再按照正常的处理顺序执行.这样,如果当前的活动控件内部实现了针对这个事件的处理函数(比如 wxID_COPY),那么这个函数就将被调用,否则就在窗口继承树中向上查找对应的处理函数,直到它找到一个这样的函数.这种命令总是针对当前活动控件的作法会更符合用户的使用习惯.

我们可以通过下面的方法重载主窗口的 ProcessEvent 函数,以便拦截命令事件并将其首先交给当前活动的控件处理,如下所示:

```cpp
bool MainFrame::ProcessEvent(wxEvent& event)
{
    static wxEvent* s_lastEvent = NULL;
    // 避免死循环
    if (& event == s_lastEvent)
        return false;
    if (event.IsCommandEvent() &&
       !event.IsKindOf(CLASSINFO(wxChildFocusEvent)) &&
       !event.IsKindOf(CLASSINFO(wxContextMenuEvent)))
    {
        s_lastEvent = & event;
        wxControl *focusWin = wxDynamicCast(FindFocus(), wxControl);
        bool success = false;
        if (focusWin)
            success = focusWin->GetEventHandler()
                                ->ProcessEvent(event);
        if (!success)
            success = wxFrame::ProcessEvent(event);
        s_lastEvent = NULL;
        return success;
    }
    else
    {
        return wxFrame::ProcessEvent(event);
    }
} 
```

就目前的情况来看,这种作法在那些当前活动控件可能为一个 wxTextCtrl 控件的时候显的更有用(在大多数平台上), wxWidgets 为这种控件实现了多种内置的命令,包括 wxID_COPY, wxID_CUT, wxID_PASTE, wxID_UNDO 和 wxID_REDO,还实现了一些默认的用户界面行为.不过,你也可以给任意的控件通过实现其派生类的方式增加这些默认的事件处理函数,比如说 wxStyledTextCtrl 控件(参考 examples/chap20/pipedprocess 中的例子,这个例子为 wxStyledTextCtrl 控件提供了这种增强处理).

# 20.3 降低闪烁

闪烁问题是所有 GUI 程序员心中永远的痛.通常所有的应用程序都需要想办法来降低可能的闪烁,下面我们就这方面的话题来给您一些有益的提示.

在 Windows 平台上,如果你的窗口正在使用 wxFULL_REPAINT_ON_RESIZE 类型,尽量去掉它.这将导致窗口系统只重绘那些由于改变大小或者别的操作而被"破坏"的那部分而不是整个窗口,这将降低擦除和重画操作的范围.否则,即使窗口只是改变一点点的大小,整个窗口都将被重画,从而导致屏幕闪烁.不过如果你的程序中,主窗口显式的内容是和窗口的大小有关的,这种情况下,这种方法毫无用处,因为整个窗口必须被重新绘制.

有时候你可以设置 wxCLIP_CHILDREN 类型,这将阻止一个窗口发生改变的时候同时刷新它的子窗口.不过这个类型在别的平台上没有影响.

当你在滚动窗口上绘制的时候,你可以采取很多步骤来提供重绘的效率以减小闪烁.首先,你需要优化你获得绘制数据的方式:你只需要搜集那些可以在当前的可视区域内显示的数据(参考 wxWindow::GetViewStart 函数和 wxWindow:: GetClientSize 函数手册),而且在你的重画函数内,你也可以只重画那些处于需要更新区域内的图形(参考 wxWindow:: GetUpdateRegion).在设计数据结构的时候,你应该考虑怎样设计才能使得你可以快速访问到当前视图对应的数据,比如说,如果你的每个数据项都有相同的宽度的时候,你可以使用链表或者数据来存放这些数据,这比你挨个搜索所有的数据要快速的多.如果计算当前位置是一个很耗时的操作,你可以考虑对最近访问的页面的起始数据位置进行缓存.然后你可以直接通过缓存快速的进行上一页下一页这样的起始数据定位动作.你也可以为每块数据增加一个用来记录其高度的字段,以便不必每次都需要计算这个数据段的高度.

当你需要实现可以滚动的图片时,你可以使用 wxWindow::ScrollWindow 函数,来对窗口进行物理滚动,这将使得你只需要更新剩下的很小的区域,这将大大降低闪烁.(wxScrolledWindow 类已经为你实现了这种机制,GetUpdateRegion 函数将反应你需要更新的这个很小的区域.)

正如我们在第五章,"绘画和打印"中介绍的那样,你还可以定义你自己的背景擦除事件处理函数,并将其留空,以禁止控件自己清除自己的背景.然后你可以直接在旧的图片上更新整个图片(包括整个背景所在的范围),以便降低由于在绘图前擦除背景导致的屏幕闪烁.使用 wxWindow:: SetBackgroundStyle 函数将背景类型设置为 wxBG_STYLE_CUSTOM 将阻止控件自作主张的更新背景.第五章还介绍了 wxBufferedDC 和 wxBufferedPaintDC,你可以结合前面提到的这些技术一起使用来降低闪烁.

另外一种情况是由于对一个窗口进行多次连续的更新导致窗口闪烁.wxWidgets 提供了 wxWindow::Freeze 函数和 wxWindow::Thaw 函数,这两个函数可以控制窗口在被更新的时候是否立即显示在屏幕上.比如说,在你需要往一个文本框中增加很多行文本的时候, 或者往一个列表框中增加很多个子项的时候,你可以使用这两个函数.当 Thaw 函数被调用的时候,窗口才会进行彻底刷新.在 Windows 系统和 Mac OSX 系统上,所有的 wxWindow 类都支持 Freeze 和 Thaw 函数,而在 GTK+平台上,wxTextCtrl 也支持这两个函数.你也可以在自己的控件中实现这两个函数来避免过度的更新用户界面(第十二章介绍的 wxThumbnailCtrl 例子实现了这两个函数,你可以参考 examples/chap12/thumbnail 目录中的代码).

# 20.4 实现联机帮助

虽然你应该尽可能的将你的应用程序界面设计的非常直观,以便用户根本不需要使用联机帮助,但是除了那些最最简单的应用程序外,对于大多数应用程序来说,提供联机帮助都是一个非常重要的事情.你可以提供 PDF 版的帮助文件或者是 HTML 格式的帮助文件以便用户可以使用他最习惯的方式浏览,不过如果你能够借助于某种帮助制作手段,使得你的联机帮助中的主题直接和对话框或者你的主窗口中的控件联系起来,这会让你的用户感觉更好.

wxWidgets 提供了帮助控制器,你的应用程序可以使用它来加载和显示帮助文件中的主题,它主要包括下面几个类:

*   wxWinHelpController,这个类用来提供对 windows 平台的基于 RTF 格式的帮助文件(扩展名为.hlp)的支持. 这中格式现在已经不推荐使用了,新的用来代替它的类是 wxCHMHelpController.
*   wxCHMHelpController, 用来提供对 windows 平台的 MS HTML 帮助文件(扩展名是.chm)的支持.
*   wxWinceHelpController,用来提供对 WindowsCE 上的帮助文件(扩展名是.htm)的支持.
*   wxHtmlHelpController, 用来提供对 wxWidgets 自定义的 HTML 帮助文件(扩展名是.htb)的支持.

wxHtmlHelpController 类和别的帮助控制类是不同的,别的类都只是封装了那个平台上相应的本地实现,而这个类是和 wxWidgets 的帮助实现机制集成在一起的,和主程序属于同一个进程.如果你想以不同进程的方式使用 wxWidgets 提供的帮助文件,你可以编译 wxWidgets 自带的 utils/src/helpview 目录下的 HelpView 程序.其中文件 remhelp.h 和 remhelp.cpp 实现了一个远程帮助文件控制器(wxRemoteHtmlHelpController),你可以将它和你的应用程序链接在一起,以便你的应用程序可以远程控制位于别的进程的帮助文件控制器实例.

注意到目前为止,还没有实现对 Mac OSX 平台上的帮助文件的支持,在这个平台上,你可以使用通用的 wxWidgets HTML 格式的帮助文件.

下面的两副图演示了 Windows 平台上同时显示 MS HTML 格式的帮助文件和 wxWidgets 的 HTML 帮助文件的样子.它们两个的外观很相似,都是右边显示 HTML 帮助的内容,左边则显示主题的继承关系以及一个用来搜索主题的文本框.稍微有一点不同的是:wxWidgets 格式的帮助控制器可以同时加载多个帮助文件.

![](img/mhtCB7B%281%29.tmp)

![](img/mhtCB8E%281%29.tmp)

使用帮助控制器

一般说来,你需要创建一个帮助文件控制器并且在整个应用程序的生命周期维护它,通常是在应用程序类中保存一个指针,在 OnInit 函数中初始化,在 OnExit 函数中释放.之所以使用指针是因为你可以自己控制什么时候释放这个指针,某些帮助控制器是依赖于某种动态链接库类的,这个类在应用程序对象被释放以后就不存在了.在创建这个帮助控制器指针以后,使用其 Initialize 指定一个帮助文件.你可以不用提供文件的扩展名, wxWidgets 将会提供针对当前平台的扩展名,比如:

```cpp
#include "wx/help.h"
#include "wx/fs_zip.h"
bool MyApp::OnInit()
{
    ...
    // wxWidgets HTML 需要这个
    wxFileSystem::AddHandler(new wxZipFSHandler);
    m_helpController = new wxHelpController;
    m_helpController->Initialize(helpFilePath);
    ...
    return true;
}
int MyApp::OnExit()
{
    delete m_helpController;
    ...
    return 0;
} 
```

注意这里我们让 wxWidgets 自己决定使用哪个帮助控制器类: wxHelpController 在 Windows 平台上定义为 wxCHMHelpController,而在别的平台上则定义为 wxHtmlHelpController.你可以在 windows 平台上也使用 wxHtmlHelpController 类,不过最好是尽可能使用本地平台提供的帮助控制器.

一旦帮助控制器初始化成功,你就可以使用下面的函数显示相应的帮助了:

```cpp
// 显示帮助内容
m_helpController->DisplayContents();
// 显示标题为"Introduction"的主题
m_helpController->DisplaySection(wxT("Introduction"));
// 显示包含在指定文件中的帮助主题.
m_helpController->DisplaySection(wxT("wx.html"));
// 显示指定 ID 的主题(仅适用于 WinHelp 和 MS HTML Help)
m_helpController->DisplaySection(500);
// 查找关键字
m_helpController->KeywordSearch(wxT("Getting started")); 
```

通常情况下,你将在帮助菜单的事件处理函数中调用 DisplayContents,也许在帮助菜单中你还会列举出其它的重要的主题,你可以在其对应的事件处理函数中使用 DisplaySection 函数,如果你希望通过标题使用 DisplaySection 函数,那么所有的标题必须是不重复的.

可能你还打算在所有的自定义对话框上增加一个帮助按钮,以便用来显示针对这个对话框的帮助主题.然而,这里有一个需要注意的事情:不是所有的平台都支持从一个模式对话框的事件处理函数中显示帮助.当帮助文件是通过一个外部程序显示的时候(比如 Windows 平台的 wxCHMHelpController),你可以放心的在模式对话框中调用帮助控件,但是如果帮助控制器只是在本进程中通过一个非模式对话框显示帮助文件时(比如 wxHtmlHelpController),你必须小心,因为通常我们不可以在一个模式对话框中创建一个非模式的 frame 窗口.模式对话框不允许你切换到另外一个非模式的对话框中.在 wxGTK 平台上,这种行为仅对于 wxHtmlHelpController 来说是可以的,但是在 Mac OS X 平台上,这种行为是不允许的,这时你可以使用前面介绍的使用自己编译的 HelpView 程序等外部程序来显示帮助文件,或者使用模式对话框来显示帮助文件,后一种方法我们稍后会进一步描述.

扩展 wxWidgets HTML 帮助

wxWidgets 的 HTML 帮助系统确实是很不错的,不过它有两个问题:首先,它只能在自己的 frame 窗口中显示帮助,因此你不可以在你的主窗口的某个 TAB 控件中显示帮助文件,另外一个缺陷是前面提到过的在模式对话框中使用的问题.

为了解决这两个问题而对 wxWidgets HTML 帮助系统进行的一个扩展将 wxWidgets 的帮助显示在一个自定义的窗口中,这个窗口可以是任何类型窗口的子窗口.你可以从光盘的 examples/chap20/htmlctrlex 目录中或者 ftp: //biolpc22.york.ac.uk/pub/contrib/helpctrlex 获取它的源代码.

如果你将它集成进你的应用程序,你可以将 wxHtmlHelpWindowEx 窗口集成进你的主窗口,然后使用 wxHtmlHelpControllerEx 类来对它进行和别的控制器一样的控制来显示帮助内容,下面是一个例子:

```cpp
#include "helpwinex.h"
#include "helpctrlex.h"
bool MyApp::OnInit()
{
    ...
    m_embeddedHelpController = new wxHtmlHelpControllerEx;
    m_embeddedHelpWindow = new wxHtmlHelpWindowEx;
    m_embeddedHelpWindow->SetController(m_embeddedHelpController);
    m_embeddedHelpController->SetHelpWindow(m_embeddedHelpWindow);
    m_embeddedHelpController->UseConfig(config, wxT("EmbeddedHelp"));
    m_embeddedHelpWindow->Create(parentWindow,
        wxID_ANY, wxDefaultPosition, wxSize(200, 100),
        wxTAB_TRAVERSAL|wxNO_BORDER, wxHF_DEFAULT_STYLE);
    m_embeddedHelpController->AddBook(wxT("book1.htb"));
    m_embeddedHelpController->AddBook(wxT("book2.htb"));
    return true;
}
int MyApp::OnExit(void)
{
    if (m_embeddedHelpController)
    {
        m_embeddedHelpController->SetHelpWindow(NULL);
        delete m_embeddedHelpController;
    }
    ...
    return 0;
} 
```

而为了解决模式对话框的问题,你可以使用 wxModalHelp 类来在模式对话框中显示某个帮助主题.当用户看完帮助以后,需要使用关闭按钮关闭这个帮助对话框,然后焦点才会重新返回上一个模式对话框中去.下面的代码就是你所需要作的全部:

```cpp
wxModalHelp help(parentWindow, helpFile, topic); 
```

在同一个程序使用两种不同的方法来显示帮助有时候是很不方便的,这时候你可以使用下面的函数来节约一些代码:

```cpp
// 如果 modalParent 不为空,则使用模式帮助显示方法,否则就使用普通的方法.
void MyApp::ShowHelp(const wxString& topic, wxWindow* modalParent)
{
#if USE_MODAL_HELP
    if (modalParent)
    {
        wxString helpFile(wxGetApp().GetFullAppPath(wxT("myapp")));
        wxModalHelp help(modalParent, helpFile, topic);
    }
    else
#endif
    {
        if (topic.IsEmpty())
            m_helpController->DisplayContents();
        else
            m_helpController->DisplaySection(topic);
    }
} 
```

宏 USE_MODAL_HELP 应该在那些使用 wxHtmlHelpController 控件的平台上被定义.当你在模式对话框中显示帮助的时候,将这个对话框的指针作为 ShowHelp 函数的第二个参数,这样,如果需要,帮助就会显示在一个模式的对话框中,而当你不是在模式对话框中显示帮助的时候,只需要传递 NULL 作为 ShowHelp 的第二个参数.

帮助文件中的声明

大多数现代的帮助文件系统都是基于 HTML 格式的.为了让跨平台的帮助文件制作更容易一些,wxWidgets 的 HTML 帮助文件使用了和 MS 的 HTML 帮助文件同样的工程,内容以及关键字文件输入格式.这可以让你在制作多平台的帮助文件时,只需要考虑一套文件就可以了.下面列举出为了创建帮助文件所需要的所有的文件:

*   一套 HTML 文件,每个主题是一个文件.
*   一个内容文件(扩展名是 hhc)以 XML 的格式描述了主题的继承关系.
*   一个可选的关键字文件(扩展名是 hhk)用来将关键字映射到帮助主题.
*   一个工程文件(扩展名是 hhp)用来描述工程中所有的其它文件以及整个工程的各个选项.

然后,你就可以将它们编译为 MS HTML 帮助文件格式(扩展名为 chm)或者 wxWidgets 的 HTML 帮助文件格式(扩展名是 htb).对于前者,你需要使用微软的 HTML 帮助制作软件(Microsoft's HTML Help Workshop),它既可以从命令行调用也可以从 GUI 界面中被调用,而对于后者来说,你只需要使用任何你喜欢的 zip 压缩工具将所有的这些文件打包成一个文件,将扩展名修改为.htb 就可以了.

当然你可以手动制作你的帮助文件,但是使用一个工具可以节省你的时间.有很多 MS HTML 帮助文件的制作工具,但是它们有时候会输出不兼容于 wxWidgets(不能被 wxWidgets 解析)的 HTML 标记.Anthemion Software 公司的 HelpBlocks 软件是目前唯一的可以同时支持 MS HTML 格式和 wxWidgets HTML 格式的软件,可以用来帮助你制作帮助文件和分析关键字.

要了解好的帮助文件的结构,最好是看看别人是怎么作的.你可以考虑增加下面的这些主题:内容,欢迎,联系信息,安装信息,注册信息,发布信息,教程,菜单使用帮助,工具条使用帮助,对话框使用帮助(将所有针对对话框的主题作为子标题),快捷键,命令行参数以及故障解决等内容.

记住,应用程序中各个帮助主题的风格最好设计成各自独立的.比如教程主题,最好采用一种比较现代的风格.

其它提供帮助的手段

你也可以考虑使用别的方法给你的用户提供帮助,比如使用 wxWidgets 的 HTML 类等.Anthemion Software 公司的 Writer's Caf�软件使用一个模式对话框来实现一些初始选项,它是用 wxHtmlWindow 实现的,这些选项包括显示一个快速教程来通过一系列的 HTML 文件演示这个程序的用途(如下图所示).这样作的好处是不言而喻的,对于您软件的使用新手来说,这样做可以给你的用户提供一个更狭长的学习路径以便你的用户不至于一开始就被浩如烟海的帮助给淹没,从而把自己至于迷失的状态.

![](img/mhtCBA1%281%29.tmp)

另外一个常用的方法是提供每日一学之类的启动提示对话框,这种方法把应用程序的功能分成一个一个的小片,然后每天学习一点点,这很符合某些人的口味.我们在第八章,"使用标准对话框"中已经介绍了怎样使用 wxShowTip 函数来显示这种帮助,它的参数包括一个父窗口,一个 wxTipProvider 指针以便告诉 wxWidgets 到哪里去寻找那些帮助信息,以及一个 boolean 变量以便指定除此显示这个对话框的时候,用于给用户选择是否显示这种帮助的复选框的初始选项.如下所示:

```cpp
#include "wx/tipdlg.h"
...
m_tipProvider = wxCreateFileTipProvider(tipFilename, currentTip);
wxShowTip(parent, m_tipProvider, showAtStart); 
```

上下文敏感帮助和工具提示

应用程序应该尽可能的提供上下文敏感帮助和工具提示.所谓工具提示是指一个很小的提示窗口,这个窗口是当鼠标在某个控件上提留一小段时间的时候探出来的.上下文敏感帮助也和它类似,不过它是由用户先点击某个帮助按钮或者是工具条上的系统按钮,然后再点击他感兴趣的控件来加以显示的.第九章,"创建定制的对话框"中,对这些内容有比较详细的介绍,在那里我们用来介绍怎样给对话框提供帮助信息,然而这些方法不仅仅只能应用于对话框,它可以应用于任何一种窗口类型.比如用户可以在工具条或者帮助菜单中提供一个"这是什么?"的选项,以便在用户选择这个菜单或者点击这个工具按钮以后,对用户随后点击的窗口控件提供工具提示.你也不必拘泥于系统默认提供的帮助显示方法,你可以自己重载 wxHelpProvider 类来实现你自己的 ShowHelp 函数.

一些控件支持使用更本地化的方法来提供上下文敏感帮助,如果你正在使用 wxCHMHelpController 控件,你可以使用它来提供上下文敏感的帮助,比如:

```cpp
#include "wx/cshelp.h"
m_helpController = new wxCHMHelpController;
wxHelpProvider::Set(
      new wxHelpControllerHelpProvider(m_helpController)); 
```

wxHelpControllerHelpProvider 类的实例将使用其 m_helpController 成员的 DisplayTextPopup 函数来提供上下文敏感帮助.

注意,提供上下文敏感帮助和 Mac OSX 的风格是格格不入的,因此在这个平台你应该忽略这种帮助.

菜单项提示

当你在菜单中加入菜单项的时候,你可以提供一个帮助信息字符串.如果这个菜单是菜单条的一部分,而这个菜单条所在的 frame 窗口拥有一个状态条,那么当用户鼠标在这个菜单项上划过的时候,这个帮助信息将显示在状态栏上.你可以通过 wxFrame::SetStatusBarPane 函数来指定显示这个帮助信息的状态条方格(如果设置为-1 则将禁止显示帮助信息).这个行为是在 wxFrame 的默认菜单项事件 EVT_MENU_HIGHLIGHT_ALL 的处理函数中实现的,因此你可以拦截这个事件来提供你自己的显示方式,比如将这个帮助信息显示在另外的一个窗口上.

# 20.5 解析命令行参数

允许程序在初始化的时候分析命令行参数是很有用的,对于一个文档视图架构的程序来说,你应该允许程序通过这样的方式打开文件.也可能你想让你的程序可以从命令行启动,以便进行一些自动化的工作,这时候你可以通过命令行参数告诉你的应用程序不要显示用户界面.虽然通常应用程序的大部分工作都是通过用户界面完成的,但是有时候命令行参数还是很有用的,比如用它来打开程序的调试开关.

wxWidgets 提供了 wxCmdLineParser 类用来简化这部分的编程工作,以避免你需要直接处理 wxApp::argc 和 wxApp::argv.这个可以处理开关类型参数(比如-verbose),选项类型参数(比如-debug:1)以及命令参数(比如 "myfile.txt")等.对于开关类型参数和选项类型参数,它允许你设置它们的长参数形式和短参数形式,你还可以给每个参数提供一个帮助字符串,这个字符串将在需要显示使用帮助的时候打印在当前的 Log 目标上.

下面的例子演示了怎样使用开关类型,选项类型等各种参数:

```cpp
#include "wx/cmdline.h"
static const wxCmdLineEntryDesc g_cmdLineDesc[] =
{
    { wxCMD_LINE_SWITCH, wxT("h"), wxT("help"),    wxT("displays help on the command line
 parameters") },
    { wxCMD_LINE_SWITCH, wxT("v"), wxT("version"), wxT("print version") },
    { wxCMD_LINE_OPTION, wxT("d"), wxT("debug"), wxT("specify a debug level") },

    { wxCMD_LINE_PARAM,  NULL, NULL, wxT("input file"), wxCMD_LINE_VAL_STRING,
 wxCMD_LINE_PARAM_OPTIONAL },
    { wxCMD_LINE_NONE }
};
bool MyApp::OnInit()
{
    // 分析命令行
    wxString cmdFilename;
    wxCmdLineParser cmdParser(g_cmdLineDesc, argc, argv);
    int res;
    {
        wxLogNull log;
        // 传递 False 参数以便在分析命名行发生错误的时候不显示使用帮助对话框.
        res = cmdParser.Parse(false);
    }
    // 检查是否用户正在询问使用帮助
    if (res == -1 || res &gt; 0 || cmdParser.Found(wxT("h")))
    {
        cmdParser.Usage();
        return false;
    }
    // 检查是否用户正在询问版本号
    if (cmdParser.Found(wxT("v")))
    {
#ifndef __WXMSW__
        wxLog::SetActiveTarget(new wxLogStderr);
#endif
        wxString msg;
        wxString date(wxString::FromAscii(__DATE__));
        msg.Printf(wxT("Anthemion DialogBlocks, (c) Julian Smart, 2005 Version %.2f, %s"),
 wbVERSION_NUMBER, (const wxChar*) date);
        wxLogMessage(msg);
        return false;
    }
    // 检查是否用户希望以调试模式启动
    long debugLevel = 0;
    if (cmdParser.Found(wxT("d"), & debugLevel))
    {
    }
    // 检查是否用户传递了一个工程名
    if (cmdParser.GetParamCount() &gt; 0)
    {
        cmdFilename = cmdParser.GetParam(0);
        // 在 windows 系统上,如果通过资源管理器打开一个文件的时候,
        // 传递的是短格式的文件名
        // 因此我们可以把它变成长格式文件名
        wxFileName fName(cmdFilename);
        fName.Normalize(wxPATH_NORM_LONG|wxPATH_NORM_DOTS|
                      wxPATH_NORM_TILDE|wxPATH_NORM_ABSOLUTE);
        cmdFilename = fName.GetFullPath();
    }
    ...
    return true;
} 
```

使用 wxFileName 对文件名进行正常化是必要的,因为有时候在以命令行方式启动程序的时候,windows 会传递短格式的文件名.

正如我们在前面介绍的那样,在 MacOSX 上,当打开一个文档的时候不使用命令行参数,而使用直接调用 wxApp:: MacOpenFile 函数的方法.但是命令行参数的方法确实在多数系统上是被使用的,因此,为了让你开发的程序适用于各种[平台,你还是应该提供命令行参数的支持.

# 20.6 存储应用程序资源

一个简单的小程序可能只有一个可执行文件.但是,更常见的情形是,你必须使用包括帮助文件,也许还有别的 HTML 文件和图片文件,以及应用程序自定义的数据文件.这些附带的文件怎么存储合适呢?

减少数据文件的数量

你可以通过一些方法来减少你需要使用的数据文件,以便创建一个更简洁的发行包.首先,对于 XPM 类型的图片,尽可能在你的代码中使用#include,而不是通过从文件中读取的方法来加载图片.其次,如果你正在使用 XRC 文件工具,位于 wxWidgets 发行版的 utils/wxrc 目录中的 wxrc 工具能将它变成 C++的代码,如下所示:

```cpp
wxrc resources.xrc --verbose --cpp-code --output resources.cpp 
```

然后,你就可以调用生成的 C++文件中的 InitXmlResource 函数来初始化这些资源了.

第三种方法是,你可以将所有的数据文件打包成一个 zip 文件,然后使用我们在前面介绍的流操作和虚拟文件系统的方法来访问它们.你可能还需要用到类 wxStandardPaths,它定义在"wx/stdpaths.h"文件中,它的一些静态的成员函数包括 GetConfigDir, GetInstallDir,GetDataDir,GetLocalDataDir 和 GetUserConfigDir 等.具体这些函数在各个平台上返回的目录的为止参考 wxWidgets 手册中的相关描述.

在 Mac OSX 平台上,你需要创建一个应用程序发布包文件,这个文件用来描述你的应用程序的可执行文件路径,数据文件等.本章稍后部分我们会详细讨论有关的情况.

找到应用程序所在的位置

经常会有 wxWidgets 的使用者希望能够提供一个函数用来找到应用程序所在的绝对路径,以便可以从同样的路径加载资源文件.不过, wxWidgets 并没有提供这样的函数,这部分是因为要在不同的平台上实现这些函数是有一定难度的,可能会返回不可靠的路径,也是部分出于鼓励开发者最好把数据文件放在系统标准的数据文件夹中(尤其是在 linux 系统上)的原因. 然而,将所有应用程序相关的文件都放在一个路径下也是可以理解的,因此,在随书光盘的 examples/chap20/findapppath 目录中,你可以找到一个函数 wxFindAppPath 的代码,用来实现这个功能,它的声明部分如下:

```cpp
// 返回当前正在运行的可执行文件的绝对路径
wxString wxFindAppPath(const wxString& argv0, const wxString& cwd,
                                      const wxString& appVariableName = wxEmptyString,
                                      const wxString& appName = wxEmptyString); 
```

argv0 的值等于 wxApp::argv[0],在某些平台上,它代表了当前执行文件的完整路径.

cwd 是当前工作目录(你可以通过调用 wxGetCwd 函数得到),在某些平台上我们需要根据这个参数作出一些判断.

appVariableName 是相关环境变量的值,比如环境变量 MYAPPDIR,这些变量可以被在程序外部设置用来指明应用程序查找位置.

appName 是你在发行包中指明的前缀,函数可能需要使用它来检查位于发行包中的某些路径.比如,DialogBlocks 程序的这个参数是 DialogBlocks,因此在 Mac OsX 系统上,这个函数会在<currentdir>/DialogBlocks.app/Content/MacOS 中寻找可执行文件的全路径.

下面是这个函数的使用方法举例:

```cpp
bool MyApp::OnInit()
{
    wxString currentDir = wxGetCwd();
    m_appDir = wxFindAppPath(argv[0], currentDir, wxT("MYAPPDIR"),
                             wxT("MyApp"));
    ...
    return true;
} 
```

在 Windows 平台和 Mac OSX 平台上,这个函数返回的路径都是可以信赖的,然而在 Unix 平台上,只有应用程序是从其所在的目录被启动的时候,返回的路径才是可以信赖的.或者如果你正确设置了 MYAPPDIR 这个环境变量,返回的路径也是可以信赖的.因此,为了让返回的值更正确,有些安装程序选择另外安装一个启动脚本,这个脚本会首先设置 MYAPPDIR 环境变量,然后再启动应用程序.你可以选择提示用户是否安装这个脚本,或者你可以直接把你的程序安装在标准的路径上,比如 /usr/local/bin/

# 20.7 调用别的应用程序

有时候你需要从你的应用程序中启动别的应用程序,可能是一个浏览器或者是你自己写的另外一个程序.wxExecute 函数是一个功能很强大的函数,它的功能包括: 带参数或者不带参数调用别的程序,同步或者异步执行程序,搜集别的程序的输出,以及重定向别的程序的输入和输出以便实现和当前程序的交互.

启动一个应用程序

下面是 wxExecute 函数的一个简单的例子:

```cpp
// 异步执行程序(默认行为),函数将会立即返回.
wxExecute(wxT("c:\\windows\\notepad.exe"));
// 同步执行程序,函数在 Notepad 程序退出以后才会返回.
wxExecute(wxT("c:\\windows\\notepad.exe c:\\temp\\temp.txt"),
          wxEXEC_SYNC); 
```

注意一般来说你可以将参数和可执行文件用引号括起来,这在路径中包含空格的时候是很有用的.

打开文档

如果你启动一个外部程序的目的是打开一个文档,在 Windows 或者 Linux 平台上,你可以使用 wxMimeTypesManager 类.你可以使用它来获得打开某种类型的文档所需要执行的程序的路径,然后使用它来构造 wxExecute 函数的参数,比如,如果你想打开一个 HTML 文件,你可以使用下面的方法:

```cpp
wxString url = wxT("c:\\home\\index.html");
bool ok = false;
wxFileType *ft = wxTheMimeTypesManager->
                 GetFileTypeFromExtension(wxT("html"));
if ( ft )
{
     wxString cmd;
      ok = ft->GetOpenCommand(&cmd,
                         wxFileType::MessageParameters(url, wxEmptyString));
      delete ft;
      if (ok)
      {
           ok = (wxExecute(cmd, wxEXEC_ASYNC) != 0);
       }
} 
```

不幸的是,这种方法不适用于 Mac OSX 平台,因为 Mac OSX 平台使用完全不同的文档打开机制.对于任何别的文件类型,最好使用系统提供的 Finder 程序来打开,而对于 HTML 文件,你可以直接使用系统函数 ICLaunchURL.wxExecute 有时候并不是最好的选择,在 windows 平台上,如果要打开 HTML 文件,你可以直接使用 ShellExecute 函数会更有效率.即使在 Unix 平台上,你可能也要作好指定的程序不存在的准备,如果它确实不存在,你可以考虑使用别的程序比如 htmlview.

为了避免上述的这些问题,我们在随书光盘的 examples/chap20/launch 目录中,实现了一些函数,比如: wxLaunchFile,wxViewHTMLFile,wxViewPDFFile,wxPlaySoundFile,它们的功能一目了然,并且它们可以同时支持 Windows,Linux 和 Mac OsX 平台.

wxLaunchFile 是一个普通意义上的文本打开函数.参数包括一个文档文件名或者一个可执行文件名附带可选的参数,以及一个可选的错误消息字符串,这个字符串在执行失败的时候显示给用户.如果当前正在打开的文档是 HTML 类型的文档,wxLaunchFile 函数将调用 wxViewHTMLFile 函数.在 Mac OsX 平台上,这个函数将使用 Finder 打开文档,而在别的平台上则使用 wxMimeTypesManager.注意在 Mac OSX 平台上,有时候文档会在非活动的窗口上打开,这时候你可以通过 osascript 这个命令行工具来将它提到前台,如下所示(比如):

```cpp
wxExecute(wxT("osascript -e \"tell application \\\"AcmeApp\\\"\" -e
\"activate\" -e \"end tell\"")); 
```

在 Linux 平台上, wxViewHTMLFile, wxViewPDFFile 和 wxPlaySoundFile 都包含 fallbacks 机制以便在相应的可执行文件不存在的时候使用.你可以按照自己的需要调整相应的 fallbacks 设置. wxPlaySoundFile 是用来使用外部程序播放那些大型的声音文件的,如果只是播放一个很小的声音文件,你可以直接使用 wxSound.

重定向进程的输入和输出

有时候,你希望捕获另外一个进程的输入和输出,以便你或者你的用户可以控制那个进程.比起重头写实现某个功能的代码来说,这样作显然可以给你减少不少的工作量.而 wxExecute 可以帮助实现捕获和控制那些控制台程序的输入和输出.

要实现这个功能,你需要在调用 wxExecute 函数的时候传递一个 wxProcess 的实例,这个实例的 OnTerminate 函数将在进程结束的时候被调用,这个实例可以用来捕获进程的输出或者控制进程的输入.

在 wxWidgets 自带的 samples/exec 目录中,你可以找到各种各样使用 wxExecute 的例子,我们也提供了另外一个例子,它将 GDB 集成进自己的程序中去,你可以参考 examples/chap20/pipedprocess 中的代码.我们没有提供用于工具条的那些小图片以及整个可编译的代码,如果提供了这些,它将可以支持包括 windows,linux 和 Mac OSX 在内的各种平台,只要那些平台上安装了 GDB.

debugger.h 和 debugger.cpp 文件实现了一个管道化的进程和一个窗口,这个窗口包含一个工具条和一个文本框,用来显示 GDB 的输出和从用户那里获取输入并且把它发送给 GDB.

textctrlex.h 和 textctrlex.cpp 则实现了一个派生自 wxStyledTextCtrl 的控件,包括一些和 wxTextCtrl 兼容的函数和标准事件处理函数比如复制,剪切,粘贴,重做和撤消等.

processapp.h 和 processapp.cpp 实现了一个应用程序类,这个类可以在空闲的时候处理来自多个进程的输入.

GDB 是通过下面的语句启动的:

```cpp
DebuggerProcess *process = new DebuggerProcess (this);
m_pid = wxExecute(cmd, wxEXEC_ASYNC, process); 
```

可以使用下面的代码杀死这个进程:

```cpp
wxKill(m_pid, wxSIGKILL, NULL, wxKILL_CHILDREN); 
```

要给调试器发送一个命令,将会设置一个内部的变量以便通知应用程序在空闲的时候处理这个输入.

```cpp
// 给调试器发送一个命令
bool DebuggerWindow::SendDebugCommand(const wxString& cmd,
                                             bool needEcho)
{
      if (m_process && m_process->GetOutputStream())
      {
           wxString c = cmd;
           c += wxT("\n");
           if (needEcho)
                AddLine(cmd);
           // 这个函数只是简单的对 m_input 变量赋值
           // OnIdle 函数中的 HasInput 函数将检查这个变量.
           m_process->SendInput(c);
           return true;
      }
      return false;
} 
```

HasInput 函数被应用程序在其空闲时间周期性的调用,它的责任是发送用户输入的命令到进程并且从进程读取来自标准输出和标准错误的输出:

```cpp
bool DebuggerProcess::HasInput()
{
       bool hasInput = false;
       static wxChar buffer[4096];
       if ( !m_input.IsEmpty() )
       {
            wxTextOutputStream os(*GetOutputStream());
             os.WriteString(m_input);
             m_input.Empty();
             hasInput = true;
        }
        if ( IsErrorAvailable() )
        {
              buffer[GetErrorStream()->Read(buffer, WXSIZEOF(buffer) -
1).LastRead()] = _T('\0');
               wxString msg(buffer);
                m_debugWindow->ReadDebuggerOutput(msg, true);
                hasInput = true;
         }
         if ( IsInputAvailable() )
         {
                 buffer[GetInputStream()->Read(buffer, WXSIZEOF(buffer) -
1).LastRead()] = _T('\0');
                wxString msg(buffer);
                m_debugWindow->ReadDebuggerOutput(buffer, false);
                hasInput = true;
         }
         return hasInput;
} 
```

注意上面这个例子和 wxWidgets 自带的 exec 例子的一个关键的不同在于,exec 例子每次从进程读取一行,如果进程的输出没有带换行符,将导致应用程序被阻塞.而在我们的例子中,使用了一个缓冲区来保存尽可能多的输入,这是一种更安全的作法.

ProcessApp 类可以直接被用作你的应用程序的基类,或者你可以拷贝它的成员函数到你的应用程序类中去.它维护了一个进程列表,进程可以通过 RegisterProcess 和 UnregisterProcess 函数登记和注销, 进程输入和输出的处理在系统空闲时间完成.如下所示:

```cpp
// 任何缓存的输入都在系统空闲时处理
bool ProcessApp::HandleProcessInput()
{
    if (!HasProcesses())
         return false;
    bool hasInput = false;
    wxNode* node = m_processes.GetFirst();
    while (node)
    {
        PipedProcess* process = wxDynamicCast(node->GetData(), PipedProcess);
        if (process && process->HasInput())
            hasInput = true;
        node = node->GetNext();
    }
    return hasInput;
}
void ProcessApp::OnIdle(wxIdleEvent& event)
{
    if (HandleProcessInput())
        event.RequestMore();
    event.Skip();
} 
```

# 20.8 管理应用程序设置

大多数的应用程序都会给用户一些选项,以便用户自己决定一些应用程序的行为,比如是否显示每日提示,文本应用什么字体,或者是否显示启动画面等.而程序员需要作的决定是怎样保存和显示这些配置数据.关于怎样存储,通常我们需要使用 wxConfig 家族的类,这些类让你可以直接处理类型化的配置数据.至于如何显示,则是非常灵活的,我们将简短的介绍一些可能的选项.

保存配置数据

所有 wxWidgets 提供的用于处理配置数据的类都是 wxConfigBase 的派生类,因此你可以在这个基类的手册中找到相关的使用方法.而 wxConfig 则被定义为各个平台上推荐使用的用于处理配置数据的类:在 windows 平台,它被定义为 wxRegConfig(这个类使用 windows 的注册表),在所有别的平台上它被定义为 wxFileConfig(它使用文本文件).另外还有 wxIniConfig 类,它使用一个 Windows 3.1 风格的.ini 配置文件,不过这个类很少被使用到.而 wxFileConfig 则可以支持各个平台.

wxConfig 类提供了各种 Read 和 Write 函数的重载函数,用来直接读写各种数据类型,包括 wxString, long, double 和 bool 类型等. 配置文件中的每个项目都需要提供一个路径,这个路径由"/"分割并且最后必须是一个名称,比如"/General/UseTooltips".你可以使用 wxConfig::SetPath 函数设置一个当前路径,这样的话,在后续的读写中,如果没有指定绝对路径(以"/"开头),所有的路径都被认为是相对于这个路径的路径. 使用路径的目的是为了对配置项进行分组.

wxConfig 的构造函数需要使用应用程序名和供应商名称,这两个名称用来决定配置项的位置,比如:

```cpp
#include "wx/config.h"
wxConfig config(wxT("MyApp"), wxT("Acme")); 
```

wxRegConfig 将会从应用程序名和供应商名称构造一个注册表项,比如前面的例子中将会导致注册表项 HKEY_CURRENT_USER/Software/Acme/MyApp 被创建.而如果是 Unix 系统上的 wxFileConfig 类,配置文件默认被保存在文件~/.MyApp 中. 而在 Mac OSX 上,则保存在/Library/Preferences/MyApp/Preferences 中.这些缺省位置可以通过给 wxConfig 传递第三个参数来改变:

下面是一些 wxConfig 的用法:

```cpp
// 读取
wxString str;
if (config.Read(wxT("General/DataPath"), & str))
{
    ...
}
bool useToolTips = false;
config.Read(wxT("General/ToolTips"), & useToolTips));
long usageCount = 0;
config.Read(wxT("General/Usage"), & usageCount));
// 写入
config.Write(wxT("General/DataPath"), str))
config.Write(wxT("General/ToolTips"), useToolTips));
config.Write(wxT("General/Usage"), usageCount)); 
```

其它一些可以使用的操作包括比例组和选项条目,查询某个组或者某个选项是否存在,删除一个条目或者一个组等.

你可以临时使用 wxConfig 来读取一些存放在某个地方的数据,你也可以创建一个 wxConfig 的实例并且在应用程序的整个生命周期维持它.wxWidgets 也有一个称为默认 wxConfig 对象的机制,这个默认的对象可以通过 wxConfig::Set 函数设置.如果设置了这个默认对象,一些 wxWidgets 的内部实现将会使用这个对象,比如 wxFontMapper 类或者平台通用的 wxFileDialog 实现.

编辑选项

如果你只有很少的选项,那么普通的对话框也许就足够了.但是有时候,选项有很多,并且非常复杂,这时候,你可能需要很多对话框或者面板, 这种情况下最通常的作法是使用包含一个 wxNotebook 的模式对话框,这个对话框的底部应该有 OK,Cancel 或者 Help 按钮.其中 Help 按钮的处理函数将会查询当前正在显示的页面并显示一个相应的帮助文件主题.wxWidgets 提供了一个叫做 wxPropertySheetDialog 的对话框来处理这种情形.wxWidgets 自带的 samples/dialogs 例子中演示了它的使用方法.在 Pocket PC 上,这个对话框里的 notebook 控件将显示成屏幕底部标准的属性页面.

你也可以使用 wxListbook 和 wxChoicebook 来代替 wxNotebook,它们是控制多页控件的又一个选择.尤其是 wxListbook,它的 API 和 wxNotebook 几乎相同,但是它使用 wxListCtrl 而不是 TAB 来控制页标签,因此你可以使用图标和标签来代替 TAB 按钮.这在你拥有很多页面的时候也很有用.尤其是在 Mac OSX 平台上,这个平台的 wxNotebook 控件不能够自己滚动标签按钮,因此标签按钮的数目受限于 wxNotebook 的宽度和标签的宽度.另外你也可以下载第三方的 awxOutbarDialog 控件,它实现了一个类似 Outlook 外观的那种多页控件,使用图标来在页面间切换.

你也可以创建自定义的分页管理对话框,比如你可以使用 wxtreeCtrl 控件,这可以让你的各个页面保持一种树状的继承关系.要实现这个自定义控件,你可以维护一组面板列表,每一个都绑定一个名字.当用户点击树状控件上的某个子项的时候,隐藏当前正在显示的面板,而显示树状控件子项对应的面板.另外一个方案是使用 Jorgen Bodde 制作的 wxTreeMultiCtrl 控件,这个控件实现了上面所介绍的内容,因此你可以以更直观的方法使用树状分页控件,而不比自己处理每个单独的页面.

你也可以考虑使用一个属性编辑框:这是一个拥有一系列子项,每个子项左边拥有一个文本标签,右边拥有一个编辑框.这个控件的好处在于增加和删除设置是非常容易的,并且不影响界面的布局.不好的地方在于,如果你要编辑多行文本或者编辑一个列表就比较困难,尽管你可以拦截子项的双击事件,使用定制的对话框来显示其内容. 你可以实现自己的属性编辑框,或者你可以考虑使用 wxGrid.,或者使用第三方的属性编辑控件比如 Jaakko Salli 的 wxPropertyGrid.有些应用程序混合使用了对话框和属性列表,比如 DialogBlocks 设置对话框的配置页面.

你最好不要使用带有滚动条的面板或者对话框来避免配置项控件超出范围之外,因为这会是人感觉迷惑并且也是很丑陋的.

你应该考虑将你应用程序的所有设置保存在一个统一的类中,并且为这个类实现一个拷贝构造函数,一个等于操作以及一个赋值操作.通过这种方法,你可以很容易的创建一个所有配置项的副本,将其传递给你的配置对话框,并且仅仅在用户点击了配置对话框上的 OK 按钮的时候,才将修改的数据保存回你的全局配置中.

如果你没有单独的保存各个配置项,你需要给你的用户提供一种直接修改配置的方法.参考光盘中的 examples/chap20/valconfig 例子.其中包含了一个类 wxConfigValidator,这个类可以用来作为普通控件的验证器, 它的参数包括配置项路径,配置项类型以及一个指向 wxConfig 对象的指针.其中值类型可以是 wxVAL_BOOL, wxVAL_STRING 或者 wxVAL_LONG.如下所示:

```cpp
void MyDialog::SetValidators(wxConfig* config)
{
    FindWindow( ID_LOAD_LAST_DOCUMENT )->SetValidator(
        wxConfigValidator(wxT("LoadLastDoc"), wxVAL_BOOL, config));
    FindWindow( ID_LAST_DOCUMENT )->SetValidator(
        wxConfigValidator(wxT("LastDoc"), wxVAL_STRING, config));
    FindWindow( ID_MAX_DOCUMENTS)->SetValidator(
        wxConfigValidator(wxT("MaxDocs"), wxVAL_LONG, config));
} 
```

第九章中介绍了更多关于验证器的知识.本节提到的那些第三方控件可以在附录 E,"wxWidgets 的三方控件"中找到.

# 20.9 应用程序安装

如果你的应用程序可以很顺利的安装到用户的电脑上,这无疑在用户开始使用你的程序之前就给用户一个很不错的第一印象.在这一节里,我们将依次介绍在 Windows,Linux 和 OsX 平台上怎样制作安装程序,其中涉及到的一些第三方工具可以在附录 E 中找到.

在 Windows 系统上安装你的程序

在 windows 平台上,我们尤其需要一个安装程序,这不只是因为用户期待这样,而且安装程序还需要作一些类似文件类型绑定和创建快捷方式这样的动作.

不够这实在和 wxWidgets 所关注的邻域差别太大,因此 wxWidgets 并不准备自己提供这样的工具.一些另外的工具可以用来创建安装程序,比如 NSIS 和 InstallShield;另外一个广受好评的软件是 Inno Setup,它是一个非常强大的,免费的安装程序制作工具,它可以通过脚本来定制安装文件选项,通过 Pascal 语言来对其现有功能进行扩展.它的网站上也列举了一些用来创建安装脚本的图形界面工具.

如果你需要很频繁的发布新的版本,你可能想要通过一个脚本自动创建安装程序.随书光盘的 examples/chap20/install 目录中提供了一个用于创建这样的脚本的例子,你可以按你的需要进行更改.因为它们是 Unix 风格的 Shell 脚本,需要你有 MingW 或者 MSys 的环境,这些环境也有在随书光盘中提供.你需要提供的包括一个放置文件的目录,makeinno.sh 脚本将会创建 Inno Setup 的脚本中枚举子目录和文件的部分.而安装脚本的头和尾部那些需要你自己按照自己软件的情况提供的部分将不会被自动创建.你可以使用下面的命令来创建安装文件:

```cpp
sh makeinno.sh c:/temp/imagedir innotop.txt innobott.txt myapps.iss 
```

这将会基于文件夹 c:/temp/imagedir 中的文件创建 Inno Setup 的脚本文件 myapp.iss.

你可以调整 makesetup.sh 脚本来创建你需要的安装程序,这个文件首先将需要的文件拷贝到一个"images"文件夹(就是前面 makeinno.sh 脚本需要的那个文件夹),然后创建 setup.exe.这个脚本使用了定义在 setup.var 中的变量.你可以按照你自己的情况增加新的功能,比如编译你的应用程序或者使用 Curl 工具拷贝文件到你的 FTP 站点等.

当你发布应用程序的时候,别忘了增加一个 WindowsXp 的"manifest"文件.这个文件是一个 Xml 格式的文件,用来告诉 WindowsXp 应该给这个程序应用什么风格.你可以通过在你的程序的资源文件(.rc)中增加 wxWidgets 标准资源文件的方式来增加这个文件. 如下所示:

```cpp
aardvarkpro ICON aardvarkpro.ico
#include "wx/msw/wx.rc" 
```

这将包含一个标准的 manifest 文件,如果你希望使用自己定义的 manifest 文件,在包含 wx.rc 之前,你需要定义 wxUSE_NO_MANIFEST 宏,然后再指定你自己的 manifest 文件,如下所示:

```cpp
aardvarkpro ICON aardvarkpro.ico
#define wxUSE_NO_MANIFEST 1
#include "wx/msw/wx.rc"
1 24 "aardvark.manifest" 
```

你也可以直接将 manifest 文件放在你的应用程序目录中,详情可参考 wxWidgets 发行版自带的 docs/msw/winxp.txt 文件.

在 Linux 系统上制作安装程序

在 Linux 系统,你可以选用图形界面的安装程序,定制的 shell 脚本或者某个特定发行版的软件包,比如 RPM 格式(基于 Red Hat 发行版)和 Debian 发布包(基于 Debian 发行版),你甚至可以直接将所有需要的文件压缩成一个包含路径的压缩文件(.tar.gz 或者. tar,bz2),安装的时候只需要保持路径解压这个文件就可以了.

Linux 环境下的图形界面安装程序包括 Loki Setup(免费),Zero G 公司的 InstallAnywhere 和 InstallShield 等.

基于 GTK+的 wxWidgets 图形界面应用程序是桌面不可感知的:它不依赖于 GNOME 或者 KDE,因此无论在哪种桌面环境下它都可以运行.大多数 KDE 桌面的发行版都会包括 GTK+的库文件.然而,因为它们使用不同的桌面风格,GTK+程序在 KDE 桌面上看上去可能会有些不适应,某些控件可能超出边界,这种情况下你可以建议你的用户安装一个 KDE 下的 GTK 风格的皮肤,比如 GTK-Qt(不过,在你把它介绍给你的用户之前,最好自己先测试一下).

你可能会希望在桌面上安装一个图标,以便你的用户可以直接使用它来启动你的程序.要在 KDE 桌面环境中增加一个图标,你需要拷贝一个合适的 APP.desktop 文件到 PREFIX/share/applications 文件夹,其中 APP 代表你的应用程序,PREFIX 则通常代表 /usr,/usr/local 或者其它定义在 KDEDIR 环境变量中的路径.下面演示了一个叫做 Acme 的 Desktop 文件,其中架设 Acme 被安装在/opt/Acme 中.

```cpp
[Desktop Entry]
BinaryPattern=Acme;
MimeType=
Name=Acme
Exec=/opt/Acme/acme
Icon=/opt/Acme/acme32x32.png
Type=Application
Terminal=0 
```

而要在 GNOME 桌面上增加一个图标,语法和 KDE 中相似不过放置的位置应该是~/.gnome-desktop(只对单个用户有效). 更多关于 GNOME 和 KDE 桌面文件的定义可以在下面的网址看到:http: //www.freedesktop.org/wiki/Standards_2fdesktop_2dentry_2dspec.

如何制作 RPM 包的信息可以在[`www.rpm.org 找到,那里还包含一个免费的在线电子书.而创建 Debian 包的信息可以在 http://www.debian.org 找到.这俩中方法创建的安装包可以允许系统进行依赖性检查,也使得用户可以很容易的浏览软件包的内容和安装软件包.如果需要创建 RPM,.deb 或者其它格式发行包的软件,可以试一下 EPM`](http://www.rpm.org 找到,那里还包含一个免费的在线电子书.而创建 Debian 包的信息可以在 http://www.debian.org 找到.这俩中方法创建的安装包可以允许系统进行依赖性检查,也使得用户可以很容易的浏览软件包的内容和安装软件包.如果需要创建 RPM,.deb 或者其它格式发行包的软件,可以试一下 EPM).

关于使用 shell 脚本创建 Linux 的安装文件的方法,随书光盘的 examples/chap20/install 目录中包含了一个用来安装 Acme 的示例文件 installacme.这个脚本作的事情包括安装整个程序并且创建一个叫做 acme 脚本,这个脚本在运行实际的可执行文件之前会先设置当前位置环境变量.这样作的好处在于你既不需要将软件所在的目录的路径增加到 PATH 环境变量中,也不需要将可执行文件直接拷贝到系统路径下就可以执行.所有的数据文件保持在可执行文件所在的目录中.这使得卸载软件变得容易.你当然也可以选择让安装脚本将数据放置到 Linux 的标准数据目录中.

在 examples/chap20/install 目录中还包含一个脚本叫做 maketarball.sh,它演示了怎样创建一个用户发行的 tar 格式的压缩包,installacme 脚本将包含在这个压缩包内以及另外一个包含所有数据文件的压缩包.你可以修改 maketarball.sh 以满足你自己的需要.

Linux 环境上的动态链接库的问题

因为 Linux 系统上并没有标准的 GUI 库,各个发行版按照自己的喜好来添加它喜欢的库和程序,因此你可能会发现在某些系统上,你的应用程序不能运行,提示的原因是无法找到动态链接库.因此,静态链接所有需要的库文件实在是一个很诱人的想法.但是这样又会导致别的一些问题.虽然你不应该静态链接 GTK+的那些库文件,但是静态链接 wxWidgets 的库是可行的,你可以在运行 configure 脚本的时候选择开关--disable- shared 以达到这个目的.你也可以考虑将 wxWdigets 提供的那些动态链接库以及所需要的 GTK+相关的库和你的应用程序打包在一起发布.

另外就是不要在太老的 linux 发行版(太老的那些动态链接库在新版上已经不提供了)或者太新的 Linux 发行版(需要一些老的发行版上还没有的库)上编译你的软件.同时考虑在给你的链接器增加-lsupc++ 选项,以便你的程序可以静态链接一些基本的 C++的库,而不是需要完全的依赖动态链接库,这样作可能会解决一些潜在的问题(不过,请注意静态链接 GPL 库时候的版权问题).

最后,如果你想要针对各个发行版发布不同的软件包,如果你不想老是重新启动电脑以切换到不同的 Linux,你可以考虑使用一个工具比如伟大的 VMware,它可以让你同时在你的机器上运行多个 linux 的发行版.

在 Mac OSX 上安装程序

在 Mac OSX 上,你真正需要作的就是确认你的软件的目录结构是正确的,然后将它作成一个合适的磁盘镜像文件,我们将简单的介绍一下.Mac OSX 上没有安装程序制作工具这种东西,你只需要将你的文件夹拖到磁盘的合适的位置就可以了.

下面我们大概来介绍一下 Mac 的软件包结构,你可以在苹果公司的网站 [`developer.apple.com/documentation/MacOSX/Conceptual/SystemOverview/Bundles/chapter_4_section_3.html`](http://developer.apple.com/documentation/MacOSX/Conceptual/SystemOverview/Bundles/chapter_4_section_3.html) 找到更多的信息.

一个软件包包含一个标准的目录结构和一个 Info.plist 文件,这个文件用来描述软件包的某些属性.

一个小的软件包结构如下所示:

```cpp
DialogBlocks.app/                     ; top-level directory
    Contents/
        Info.plist                    ; the property list file
        MacOS/
            DialogBlocks              ; the executable
        Resources/
            dialogblocks-app.icns     ; the app icon
            dialogblocks-doc.icns     ; the document icon(s) 
```

下面是一个用于 DialogBlocks 软禁的 Info.plist 文件的例子:

```cpp
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist SYSTEM "file://localhost/System/Library/DTDs/PropertyList.dtd">
<plist version="0.9">
<dict>
      <key>CFBundleInfoDictionaryVersion</key>
      <string>6.0</string>
      <key>CFBundleIdentifier</key>
      <string>uk.co.anthemion.dialogblocks</string>
      <key>CFBundleDevelopmentRegion</key>
      <string>English</string>
      <key>CFBundleDocumentTypes</key>
      <array>
            <dict>
                  <key>CFBundleTypeExtensions</key>
                  <array>
                        <string>pjd</string>
                  </array>
                  <key>CFBundleTypeIconFile</key>
                  <string>dialogblocks-doc.icns</string>
                  <key>CFBundleTypeName</key>
                  <string>pjdfile</string>
                  <key>CFBundleTypeRole</key>
                  <string>Editor</string>
            </dict>
      </array>
      <key>CFBundleExecutable</key>
      <string>DialogBlocks</string>
      <key>CFBundleIconFile</key>
      <string>dialogblocks-app.icns</string>
      <key>CFBundleName</key>
      <string>DialogBlocks</string>
      <key>CFBundlePackageType</key>
      <string>APPL</string>
      <key>CFBundleSignature</key>
      <string>PJDA</string>
      <key>CFBundleVersion</key>
      <string>1.50</string>
      <key>CFBundleShortVersionString</key>
      <string>1.50</string>
      <key>CFBundleGetInfoString</key>
      <string>DialogBlocks version 1.50, (c) 2004 Anthemion Software Ltd.</string>
      <key>CFBundleLongVersionString</key>
      <string>DialogBlocks version 1.50, (c) 2004 Anthemion Software Ltd.</string>
      <key>NSHumanReadableCopyright</key>
      <string>Copyright 2004 Anthemion Software Ltd.</string>
      <key>LSRequiresCarbon</key>
        <true/>
      <key>CSResourcesFileMapped</key>
      <true/>
</dict>
</plist> 
```

程序使用的图标和支持的文档类型是通过 CFBundleIconFile 和 CFBundleTypeIconFile 属性指定的.正如我们在第十章,"使用图片编程"中介绍的那样,如果你主要实在 Windows 或 Linux 下编程,你可以创建各种不同大小的 (16x16,32x32,48x48 和 128x128)的图标文件.将其保存为透明的 PNG 文件,然后拷贝到 Mac 平台上,在 Finder 中打开这些文件,将其拷贝和粘贴到苹果公司的图标编辑器中,然后就可以另存为 icns 文件了.

前面我们介绍过的 maketarball.sh 脚本也可以用来创建 Mac OSX 上的磁盘镜像文件.比如 AcmeApp-1.50.dmg.它将已经准备好的 AcmeApp.app 包中的目录结构拷贝到新的目录结构,然后拷贝用于 Mac OSX 的可执行文件和数据文件,然后再使用下面的代码创建一个可以直接用于 Internet 安装的磁盘镜像文件:

```cpp
echo Making a disk image...
hdiutil create AcmeApp-$VERSION.dmg -volname AcmeApp-$VERSION -type UDIF -megabytes 50 -fs  HFS+
echo Mounting the disk image...
MYDEV=`hdiutil attach AcmeApp-$VERSION.dmg | tail -n 1 | awk '{print $1'}`
echo Device is $MYDEV
echo Copying AcmeApp to the disk image...
ditto --rsrc AcmeApp-$VERSION /Volumes/AcmeApp-$VERSION/AcmeApp-$VERSION
echo Unmounting the disk image...
hdiutil detach $MYDEV
echo Compressing the disk image...
hdiutil convert AcmeApp-$VERSION.dmg -format UDZO -o AcmeApp-$VERSION-compressed.dmg
echo Internet enabling the disk image...
hdiutil internet-enable AcmeApp-$VERSION-compressed.dmg
echo Renaming compressed image...
rm -f AcmeApp-$VERSION.dmg
mv AcmeApp-$VERSION-compressed.dmg AcmeApp-$VERSION.dmg 
```

之后,新创建的磁盘镜像文件就可以拷贝到你的 FTP 站点或者 CD-ROM 站点.当你的用户在一个浏览器中点击这个文件的时候,文件就会被自动下载,解包,加载成一个虚拟的磁盘,所有这些过程都不需要用户的干预,然后就等着用户把整个软件包拖拽到磁盘的合适的位置,就可以完成软件的安装了.

# 20.10 遵循用户界面设计规范

学习各个平台的界面设计规范是一件很值得一做的事情.虽然它们中的绝大多数差异都已经被 wxWidgets 自动屏蔽了,不过还是有一些细节是无法自动解决的.比如按钮的布局风格在不同的平台上是不一样的.苹果的 Mac OSX 操作系统上按钮的顺序和间隔的要求是非常严格的.下面只是我们认为值得特别说明的一些方面,包括一些平台相关的规则和一些一般的规则.另外你也可以通过多操作各个平台上的经典的程序,并观察他们外观的不同来帮助你设计你自己的程序在这些平台上的外观.

标准按钮

在 windows 和 Linux 平台上,按钮可以被整体居中或者右对齐,通常的顺序是 OK,Cancel 和 Help.而在 Mac OSX 上,帮助按钮(如果使用 wxID_HELP 会自动显示一个问号标记)通常应该是左对齐的,其它的按钮则是右对齐的,并且最右边的那个是默认按钮,也就是说是:?号,空格,Cancel,OK 这样的顺序.

尽可能使用 wxWidgets 提供的标准按钮标识符(比如 wxID_OK, wxID_CLOSE, wxID_APPLY 等),因为在某些平台上(尤其是 wxGTK 平台上),这些标准标识符会被自动添加一些合适的图形.

参考第七章"使用布局控件进行窗口布局"中的"平台相关布局"小节了解 wxStdDialogButtonSizer 类的使用方法,这个类能够对标准按钮进行平台相关的布局.

菜单

避免出现空的菜单条.小心的给各个菜单添加有意义的标签,并且使用"&"符号引导的通常是标签的第一个字符的加速键(比如 &File)和快捷键(比如 Ctrl+O).常用的那些菜单项命令应该尽可能的提供,比如拷贝,粘贴,撤消等.不要有太长或者太短的菜单项.通常 9 到 10 个菜单项是一个比较合理的最大值.如果确实有很多选项需要配置,可以考虑使用一个菜单项弹出一个对话框进行这些设置.

和按钮的使用一样,尽可能使用 wxWidgets 提供的标准的标识符,尤其是 wxID_HELP,wxID_PREFERENCES 等,在 Mac OSX 平台上,wxID_HELP 菜单将被移动到应用系统菜单中去,你应该注意这个问题,以便产生空的菜单条或者连续两个菜单分割条.

图标

你工具栏,frame 窗口和别的界面元素上的图标可以给你的应用程序一个很好的观感.忽略这一点可以让你的应用程序的界面效果大打折扣. 尤其是在 MAc OSX 平台上,这个平台对于美学的要求是很高的.你应该尽量给每一个项目创建自己的图标,或者,一个更简单的作法,直接购买别人设计好的图标,然后将那些非标准的图标按照统一的风格进行设计.在图标上的付出将会获得等价的回报,你的应用程序将会因为使用了这些图标而增光添彩,也会给用户留下很强烈的印象. 你也可以在网上找到一些图标,比如,遵循 L-GPL 协议发布的 Ximian 图标集: http: //www.novell.com/coolsolutions/feature/1637.html .

字体和颜色

不要在你的对话框上使用很多中字体和颜色,这除了导致你的界面看上去花里胡哨以外,还使得 wxWidgets 很难去进行针对各个平台的一些外观调整,以便给出你的应用程序以本地观感.不过,这并不防碍你给你的用户增加更改默认字体的选项,以便他们可以改变那些包含很多纹理信息的对话框的外观,比如用作报告的对话框.对于颜色的使用要遵循那个平台的规范.对于 wxWidgets 提供的对话框,wxWidgets 可以自己作一些平台适应工作, 但是对于你自定义的对话框,有些则需要你自己去注意这个问题.

应用程序中止时的行为

在大多数平台上,没有使用 MDI 界面或者将类似界面的基于文档的应用程序将在每个 frame 窗口中显示一个文档.当最后一个文档被关闭的时候应用程序就将退出.但是在 Mac OSX 平台上,正如我们在第十九章"使用文档和视图框架"中介绍的那样,用户并不期待这时候整个应用程序退出,应用程序应该还有一个菜单显示在系统菜单条上,以便用户可以通过它打开或者创建新的文档或者关闭应用程序.这可以通过创建一个不可见的 frame 主窗口的方法来实现,可能需要你增加一点点平台相关的代码.

在嵌入式开发系统中(比如 Pocket PC),应用程序在主窗口被关闭的时候仍然停留在内存中,用户通常没有办法让他们退出.你可以选择是否遵守这个规则还是允许用户退出应用程序以便给别的程序腾出内存.在 Pocket PC 上,wxWidgets 也会设置标准的快捷键 Ctrl+Q 用来退出应用程序,这个快捷键的默认处理动作是发送 wxID_EXIT 命令事件.

进一步阅读

下面列出了 wxWidgets 支持的各个主要平台上的用户界面设计规范,以及一些一般性 UI 设计建议的书籍:

*   苹果用户界面设计规范: [`developer.apple.com/documentation/UserExperience/Conceptual/OSXHIGuidelines/index.html`](http://developer.apple.com/documentation/UserExperience/Conceptual/OSXHIGuidelines/index.html)
*   Mac OSX 和 Windows 用户界面的关键差异: [`developer.apple.com/ue/switch/windows.html`](http://developer.apple.com/ue/switch/windows.html)
*   微软官方用户界面设计规范: [`msdn.microsoft.com/library/default.asp?url=/library/enus/dnwue/html/welcome.asp`](http://msdn.microsoft.com/library/default.asp?url=/library/enus/dnwue/html/welcome.asp)
*   GNOME 用户界面设计规范: [`developer.gnome.org/projects/gup/hig`](http://developer.gnome.org/projects/gup/hig)
*   GUI Bloopers: 软件开发和 Web 设计中要做和不要作的事, 作者:Jeff Johnson (Academic Press). ISBN 1-55860-582-7
*   程序员用户界面设计, 作者: Joel Spolsky (Apress). ISBN 1-893115-94-1
*   软件可用性: 以可用性为核心进行软件设计和建模(A Practical Guide to the Models and Methods of Usage-Centered Design), 作者:Larry L. Constantine and Lucy A.D. Lockwood (ACM Press). ISBN 0-201-92478-1

# 20.11 全书小结

本章我们介绍了完善你的程序相关的各种主题,演示了一些弥补 wxWidgets 不足之处的代码.最后我们介绍了一些用户界面设计的有益提示,介绍了一些更进一步介绍 UI 规范的书籍.

我们希望通过这些书籍的阅读,能够让你更加认同我们的工作,更加认同 wxWidgets,它是一个非常强大的工具集,它可以给予你的东西包括:

*   你的应用程序将拥有本地观感
*   大量的类控件,包括各种简单和复杂的窗口控件,轻量级的 HTML 支持,向导,联机帮助,多线程,进程间通信,流及虚拟文件系统等等,将让你开发出更加稳健的产品级的程序,并且让你享受开发的过程.
*   当然,在各个平台上使用同样的代码也将为你节省很多钱.
*   你可以很容易的将你的代码移植到别的你正打算移植的平台,比如 Pocket PC 和 Mac OS X,以便为它赢得更大的市场.
*   通过使用快速开发工具比如 DialogBlocks,以及强大的布局控件机制,你可以很快的创建出复杂而优雅的,可伸缩的并且是可移植的对话框和窗口.
*   wxWidgets 是开放源代码的,你可以更改它的代码或者理解它到底是怎样工作的.
*   你将从 wxWidgets 庞大的社区支持中受益,你的问题将很快被答复,你还可以使用很多第三方的控件和工具包(参考附录 E)

我们非常希望你能够享受阅读这本书的过程,并且在浏览了光盘中的例子和工具之后,愿意马上开始将你学到的这些知识应用到你的跨平台程序中去.祝你好运,我们期待很快能够在 wxWidgets 的邮件列表或者论坛上看到你的身影.