# 第十九章使用文档/视图框架

本章来讨论一下 wxWidgets 提供的文档和视图框架,通过使用它,那些基于文档的应用程序的使用的代码可以大为减少.另外我们还将讨论和撤消/重做操作相关的实现,我们将介绍通过怎样的途径让这个看上去非常复杂的操作变成很自然的事情.

# 19.1 文档/视图基础

文档/视图框架在很多编程框架中都专门提供了支持,因为它可以很大程度的简化编写类似的程序需要的代码量.

文档/视图框架主要是让你用文档和视图两个概念来建模.所谓文档,指的是那些用来存储数据和提供用户界面无关的操作的类,而视图,指的是用来显示数据的那些类.这和所谓的 MVC 模型(模型-视图-控制器)很相似,只不过这里把视图和控制器合在一起,作为一个概念.

基于这个框架,wxWidgets 可以提供大量的用户界面控件和默认行为.你需要先定义自己的派生类以及它们之间的关系,框架本身则负责显示文件选择,打开和关闭文件,询问用户保存数据,将菜单项和对应的代码关联,甚至一些基本的打印和预览功能,还有就是重做/撤消功能的支持等.这个框架已经被高度的模块化了,允许你的应用程序通过重载和替换函数和对象的方式来更改这些默认的行为.

如果你觉得框架适合你即将制作的程序,你可以采用下面的步骤来使用这个框架.这些步骤的顺序并不是非常的严格的,你大可以先创建你的文档类,然后再考虑你的文档在应用程序中的表现形式.

1.  决定你要使用的用户界面: 微软的 MDI (多文档界面,所有的子文档窗口被包含在一个父窗口内), SDI (单文档界面,每个文档一个单独的 frame 窗口), 或者是单一界面(同时只能打开一个文档,就象 windows 的写字板程序那样).
2.  基于前面的选择来使用对应的父窗口和子窗口类,比如 wxDocParentFrame 和 wxDocChildFrame 类. 在 OnInit 函数中创建一个父窗口的实例,对应于每个文档视图创建一个子窗口的实例(如果不是单文档界面的话).使用标准的菜单标识符创建菜单(比如 wxID_OPEN 和 wxID_PRINT).
3.  定义你自己的文档和视图类,重载尽可能少的成员函数用于输入和输出,绘画以及初始化.如果你需要重做/撤消的支持,你应该尽早实现它而不要等到程序快完成的时候再回来返工.
4.  定义任意的子窗口(比如一个滚动窗口)用来显示视图.你可能需要将它的一些事件传递给视图或者文档类处理,比如通常它的重绘事件都需要传递给 wxView::OnDraw 函数.
5.  在你的 wxApp::OnInit 函数的开始部分创建一个 wxDocManager 实例以及足够多的 wxDocTemplate 实例,以便定义文档和视图之间的关系.对于简单的应用程序来说,一个 wxDocTemplate 的实例就可以了.

我们将用一个简单的叫做 Doodle(参见下图)的程序来演示上面的步骤.正如它的名字那样,它支持在一个窗口上任意乱画,并且支持将这些涂鸦保存在文件里或者从文件里读取.也支持简单的重做和撤消操作.

![](img/mhtCD41%281%29.tmp)

第一步:选择用户界面类型

传统上,windows 平台的多文档程序都使用的是多文档界面,我们已经在第四章,"窗口基础"中的"wxMDIParentFrame"小节对此有过描述.多文档界面使用一个父 frame 窗口管理和包含多个文档子 frame 窗口,而其菜单条则用来反应当前活动窗口或者父窗口(如果没有当前活动窗口的话) 相关联的菜单命令.

或者你也可以选择使用一个主窗口,多个顶层的用于显示文档的 frame 窗口的方式,这种方式下文档窗口可以不受主窗口的限制,在桌面上任意移动.这通常是 Mac OS 采用的风格,不过在 Mac OS 上,每次只能显示一个菜单条(当前活动窗口的菜单条).Mac OS 上另外一个和别的平台不同的地方在于,Mac 用户并不期望应用程序的所有的窗口被关闭以后退出应用程序.Mac 系统有一个应用程序菜单条,上面显示了的应用程序所有的窗口都隐藏时候可以的少数的几个命令,在 wxWidgets 上,要实现这种行为,你需要创建一个不可见的 frame 窗口,它的菜单条将在所有其它可见窗口被释放以后自动显示在那个位置.

这种技术的另外一种用法是显示一个主窗口之外的非文档视图的 frame 窗口.不过,这种用法非常罕见,一般都不会这样使用.另外一种方法是仅显示文档窗口,不显示主窗口,仅在最后一个文档窗口被关闭的时候显示主窗口以用来创建或者打开新的文档窗口,这种模型被近期的 Microsoft Word 采用,这其实是一个和 Mac OS 很接近的作法,只不过在 Mac OS 上,在这种情况下,没有任何可见的窗口,只有一个菜单条.

也许最简单的模型是只有一个主窗口,没有独立的子窗口,每次也只能打开一个文档:微软的写字板就是这样的一个例子.这也是我们的 Doodle 例子所选择的形式.

最后,你当然也可以创建自己的模型,或者你可以采用上面这些模型的组合方式.比如 DialogBlocks 就是这样的一个例子,它组合了几种方式以便用户自己作出选择.DialogBlocks 中最常用的是方式是每次只显示一个视图,,当你在工程树中选择了一个文档的时候,当前视图隐藏, 新的视图打开.你也可以打开多页面支持,以便快速的在你最感兴趣的几个文档之间切换.另外,你还可以通过拖拽标题栏的方式,将某个文档以单独的窗口拖动到桌面上,以便你可以同时看到几个文档的视图.在 DialogBlocks 程序内部,它自己管理视图和文档以及窗口之间的关系,采用的就是和标准的 wxWidgets 不同的方式.很明显,创建这样的定制文档视图管理系统需要很多时间,因此,你可能更愿意选择 wxWidgets 提供的标准方式.

第二步: 创建和使用 frame 窗口类.

对于 MDI 界面应用程序来说,你应该使用 wxDocMDIParentFrame 和 wxDocMDIChildFrame 窗口类,而对于主窗口和文档窗口分离的模型来说,你可以选择使用 wxDocParentFrame 和 wxDocChildFrame 类.如果你使用的是单个主窗口每次打开一个文档这种模型,你可以只使用 wxDocParentFrame 类.

如果你的应用程序没有主窗口,只有多个文档窗口,你既可以使用 wxDocParentFrame,也可以使用 wxDocChildFrame.不过,如果你使用的是 wxDocParentFrame,你需要拦截 EVT_CLOSE 事件,以便只删除和这个窗口绑定的文档视图,因为这个窗口类默认的 EVT_CLOSE 事件处理函数将删除所有文档管理器知道的视图(这将导致关闭所有的文档).

下面列出了 doodle 例子的窗口类定义.其中保存了一个指向 doodle 画布的指针和一个指向编辑菜单的指针,以便文档视图系统可以视情况更新重做和撤消菜单.

```cpp
// 定义一个新的 frame 窗口类.
class DoodleFrame: public wxDocParentFrame
{
    DECLARE_CLASS(DoodleFrame)
    DECLARE_EVENT_TABLE()
public:
    DoodleFrame(wxDocManager *manager, wxFrame *frame, wxWindowID id,
        const wxString& title, const wxPoint& pos,
        const wxSize& size, long type);
    /// 显示关于对话框
    void OnAbout(wxCommandEvent& event);
    /// 获得编辑菜单指针
    wxMenu* GetEditMenu() const { return m_editMenu; }
    /// 获得画布指针
    DoodleCanvas* GetCanvas() const { return m_canvas; }
private:
    wxMenu *        m_editMenu;
    DoodleCanvas*   m_canvas;
}; 
```

下面的代码演示了 DoodleFrame 的实现.其中构造函数创建了一个菜单条和一个 DoodleCanvas 对象,后者拥有一个铅笔状的鼠标指针.文件菜单被传递给文档视图模型的管理对象,以便其可以增加最近使用文件的显示.

```cpp
IMPLEMENT_CLASS(DoodleFrame, wxDocParentFrame)
BEGIN_EVENT_TABLE(DoodleFrame, wxDocParentFrame)
    EVT_MENU(DOCVIEW_ABOUT, DoodleFrame::OnAbout)
END_EVENT_TABLE()
DoodleFrame::DoodleFrame(wxDocManager *manager, wxFrame *parent,
                 wxWindowID id, const wxString& title,
                 const wxPoint& pos, const wxSize& size, long type):
wxDocParentFrame(manager, parent, id, title, pos, size, type)
{
    m_editMenu = NULL;
    m_canvas = new DoodleCanvas(this,
                    wxDefaultPosition, wxDefaultSize, 0);
    m_canvas->SetCursor(wxCursor(wxCURSOR_PENCIL));
    // 增加滚动条
    m_canvas->SetScrollbars(20, 20, 50, 50);
    m_canvas->SetBackgroundColour(*wxWHITE);
    m_canvas->ClearBackground();   
    // 增加图标
    SetIcon(wxIcon(doodle_xpm));
    // 创建菜单
    wxMenu *fileMenu = new wxMenu;
    wxMenu *editMenu = (wxMenu *) NULL;
    fileMenu->Append(wxID_NEW, wxT("&New..."));
    fileMenu->Append(wxID_OPEN, wxT("&Open..."));
    fileMenu->Append(wxID_CLOSE, wxT("&Close"));
    fileMenu->Append(wxID_SAVE, wxT("&Save"));
    fileMenu->Append(wxID_SAVEAS, wxT("Save &As..."));
    fileMenu->AppendSeparator();
    fileMenu->Append(wxID_PRINT, wxT("&Print..."));
    fileMenu->Append(wxID_PRINT_SETUP, wxT("Print &Setup..."));
    fileMenu->Append(wxID_PREVIEW, wxT("Print Pre&view"));
    editMenu = new wxMenu;
    editMenu->Append(wxID_UNDO, wxT("&Undo"));
    editMenu->Append(wxID_REDO, wxT("&Redo"));
    editMenu->AppendSeparator();
    editMenu->Append(DOCVIEW_CUT, wxT("&Cut last segment"));
    m_editMenu = editMenu;
    fileMenu->AppendSeparator();
    fileMenu->Append(wxID_EXIT, wxT("E&xit"));
    wxMenu *helpMenu = new wxMenu;
    helpMenu->Append(DOCVIEW_ABOUT, wxT("&About"));
    wxMenuBar *menuBar = new wxMenuBar;
    menuBar->Append(fileMenu, wxT("&File"));
    menuBar->Append(editMenu, wxT("&Edit"));
    menuBar->Append(helpMenu, wxT("&Help"));
    // 指定菜单条
    SetMenuBar(menuBar);
    // 历史文件访问记录的显示将使用这个菜单.
    manager->FileHistoryUseMenu(fileMenu);
}
void DoodleFrame::OnAbout(wxCommandEvent& WXUNUSED(event) )
{
    (void)wxMessageBox(wxT("Doodle Sample\n(c) 2004, Julian Smart"),
                       wxT("About Doodle"));
} 
```

第三步: 定义你的文档和视图类

你的文档类应该有一个默认的构造函数,而且应该使用 DECLARE_DYNAMIC_CLASS 和 IMPLEMENT_DYNAMIC_CLASS 宏来使其提供 RTTI 并且支持动态创建(否则你就需要重载 wxDocTemplate::CreateDocument 函数,以实现的文档实例创建函数).

你还需要告诉文档视图框架怎样保存和读取你的文档对象,如果你想直接使用 wxWidgets 流操作,你可以重载 SaveObject 和 LoadObject 函数,就象我们例子中的作法一样.或者你可以直接重载 DoSaveDocument 函数和 DoOpenDocument 函数,这两个函数的参数为文件名而不是流对象.wxWidget 流操作相关内容我们已经在第十四章,"文件和流操作"中介绍过.

注意:框架本身在保存数据的时候不使用临时文件系统.这也是为什么我们有时候需要重载 DoSaveDocument 函数的一个理由,我们可以通过流操作将文档保存在 wxTempFile 中,正如我们在第十四章中介绍的那样.

下面是我们的 DoodleDocument 类的声明部分:

```cpp
/*
 * 代表一个 Doodle 文档
 */
class DoodleDocument: public wxDocument
{
    DECLARE_DYNAMIC_CLASS(DoodleDocument)
public:
    DoodleDocument() {};
    ~DoodleDocument();
    /// 保存文档
    wxOutputStream& SaveObject(wxOutputStream& stream);
    /// 读取文档
    wxInputStream& LoadObject(wxInputStream& stream);
    inline wxList& GetDoodleSegments() { return m_doodleSegments; };
private:
    wxList m_doodleSegments;
}; 
```

你的文档类也许要包含文档内容对应的数据.在我们的例子中,我们的数据就是一个 doodle 片断的列表,每一个数据片断代表从鼠标按下到鼠标释放过程中鼠标划过的所有的线段.这些片断所属的类知道怎样将自己保存在流中,这使得我们实现文档保存和读取的流操作变的相对容易.下面是用来代表这些线段片断的类的声明:

```cpp
/*
 * 定义了一个两点之间的线段
 */
class DoodleLine: public wxObject
{
public:
    DoodleLine(wxInt32 x1 = 0, wxInt32 y1 = 0,
               wxInt32 x2 = 0, wxInt32 y2 = 0)
    { m_x1 = x1; m_y1 = y1; m_x2 = x2; m_y2 = y2; }
    wxInt32 m_x1;
    wxInt32 m_y1;
    wxInt32 m_x2;
    wxInt32 m_y2;
};
/*
 * 包含一个线段的列表,用来代表一次鼠标绘画操作
 */
class DoodleSegment: public wxObject
{
public:
    DoodleSegment(){};
    DoodleSegment(DoodleSegment& seg);
    ~DoodleSegment();
    void Draw(wxDC *dc);
    /// 保存一个片断
    wxOutputStream& SaveObject(wxOutputStream& stream);
    /// 读取一个片断
    wxInputStream& LoadObject(wxInputStream& stream);
    /// 获取片断中的线段列表
    wxList& GetLines() { return m_lines; }
private:
    wxList m_lines;
}; 
```

DoodleSegment 类知道怎么在某个设备上下文上绘制自己,这有助于我们实现我们的 doodle 绘制代码.

下面的代码是这些类的实现部分:

```cpp
/*
 * DoodleDocument
 */
IMPLEMENT_DYNAMIC_CLASS(DoodleDocument, wxDocument)
DoodleDocument::~DoodleDocument()
{
    WX_CLEAR_LIST(wxList, m_doodleSegments);
}
wxOutputStream& DoodleDocument::SaveObject(wxOutputStream& stream)
{
    wxDocument::SaveObject(stream);
    wxTextOutputStream textStream( stream );
    wxInt32 n = m_doodleSegments.GetCount();
    textStream &lt;&lt; n &lt;&lt; wxT('\n');
    wxList::compatibility_iterator node = m_doodleSegments.GetFirst();
    while (node)
    {
        DoodleSegment *segment = (DoodleSegment *)node->GetData();
        segment->SaveObject(stream);
        textStream &lt;&lt; wxT('\n');
        node = node->GetNext();
    }
    return stream;
}
wxInputStream& DoodleDocument::LoadObject(wxInputStream& stream)
{
    wxDocument::LoadObject(stream);
    wxTextInputStream textStream( stream );
    wxInt32 n = 0;
    textStream &gt;&gt; n;
    for (int i = 0; i &lt; n; i++)
    {
        DoodleSegment *segment = new DoodleSegment;
        segment->LoadObject(stream);
        m_doodleSegments.Append(segment);
    }
    return stream;
}
/*
 * DoodleSegment
 */
DoodleSegment::DoodleSegment(DoodleSegment& seg)
{
    wxList::compatibility_iterator node = seg.GetLines().GetFirst();
    while (node)
    {
        DoodleLine *line = (DoodleLine *)node->GetData();
        DoodleLine *newLine = new DoodleLine(line->m_x1, line->m_y1, line->m_x2, line->m_y2);
        GetLines().Append(newLine);
        node = node->GetNext();
    }
}
DoodleSegment::~DoodleSegment()
{
    WX_CLEAR_LIST(wxList, m_lines);
}
wxOutputStream &DoodleSegment::SaveObject(wxOutputStream& stream)
{
    wxTextOutputStream textStream( stream );
    wxInt32 n = GetLines().GetCount();
    textStream &lt;&lt; n &lt;&lt; wxT('\n');
    wxList::compatibility_iterator node = GetLines().GetFirst();
    while (node)
    {
        DoodleLine *line = (DoodleLine *)node->GetData();
        textStream
            &lt;&lt; line->m_x1 &lt;&lt; wxT(" ")
            &lt;&lt; line->m_y1 &lt;&lt; wxT(" ")
            &lt;&lt; line->m_x2 &lt;&lt; wxT(" ")
            &lt;&lt; line->m_y2 &lt;&lt; wxT("\n");
        node = node->GetNext();
    }
    return stream;
}
wxInputStream &DoodleSegment::LoadObject(wxInputStream& stream)
{
    wxTextInputStream textStream( stream );
    wxInt32 n = 0;
    textStream &gt;&gt; n;
    for (int i = 0; i &lt; n; i++)
    {
        DoodleLine *line = new DoodleLine;
        textStream
            &gt;&gt; line->m_x1
            &gt;&gt; line->m_y1
            &gt;&gt; line->m_x2
            &gt;&gt; line->m_y2;
        GetLines().Append(line);
    }
    return stream;
}
void DoodleSegment::Draw(wxDC *dc)
{
    wxList::compatibility_iterator node = GetLines().GetFirst();
    while (node)
    {
        DoodleLine *line = (DoodleLine *)node->GetData();
        dc->DrawLine(line->m_x1, line->m_y1, line->m_x2, line->m_y2);
        node = node->GetNext();
    }
} 
```

到目前为止,我们还没有介绍怎样将 doodle 片断增加到我们的文档中,除了从文件读取以外.我们需要将那些用来响应鼠标和键盘操作,以更改文档内容的命令代码模型化,这是实现重做/撤消操作的关键.DoodleCommand 是一个继承自 wxCommand 的类,它实现了虚函数 Do 和 Undo,这些函数将被框架在合适的时候调用.因此,我们将不会直接更改文档内容,取而代之的是在相应的事件处理函数中,创建一个一个的 DoodleCommand 对象,并将这些对象提交给文档命令处理器(一个 wxCommandProcessor 类的实例)处理.文档命令处理器在执行这些命令前会自动将这些命令保存在一个重做/撤消堆栈中.文档命令处理器对象是在文档被初始化的时候被框架自动创建的,因此在这个例子中你看不到显式创建这个对象的代码.

下面是 DoodleCommand 类的声明:

```cpp
/*
 * 一个 doodle 命令
 */
class DoodleCommand: public wxCommand
{
public:
    DoodleCommand(const wxString& name, int cmd, DoodleDocument *doc, DoodleSegment *seg);
    ~DoodleCommand();
    /// Overrides
    virtual bool Do();
    virtual bool Undo();
    /// 重做和撤消的命令是对称的,因此将它们组合在一起.
    bool DoOrUndo(int cmd);
protected:
    DoodleSegment*  m_segment;
    DoodleDocument* m_doc;
    int             m_cmd;
};
/*
 * Doodle 命令标识符
 */
#define DOODLE_CUT          1
#define DOODLE_ADD          2 
```

我们定义了两种类型的命令: DOODLE_ADD 和 DOODLE_CUT.用户可以删除最后一次的绘画操作或者增加新的绘画操作.这里我们的两个命令都使用同一个类,不过这不是必须的.每一个命令对象都会保存一个文档指针,一个 DoodleSegment(代表一次绘画操作)指针和一个命令标识符.下面是 DoodleCommand 类的实现部分:

```cpp
/*
 * DoodleCommand
 */
DoodleCommand::DoodleCommand(const wxString& name, int command,
                              DoodleDocument *doc, DoodleSegment *seg):
    wxCommand(true, name)
{
    m_doc = doc;
    m_segment = seg;
    m_cmd = command;
}
DoodleCommand::~DoodleCommand()
{
    if (m_segment)
        delete m_segment;
}
bool DoodleCommand::Do()
{
    return DoOrUndo(m_cmd);
}
bool DoodleCommand::Undo()
{
    switch (m_cmd)
    {
    case DOODLE_ADD:
        {
            return DoOrUndo(DOODLE_CUT);
        }
    case DOODLE_CUT:
        {
            return DoOrUndo(DOODLE_ADD);
        }
    }
    return true;
}
bool DoodleCommand::DoOrUndo(int cmd)
{
    switch (cmd)
    {
    case DOODLE_ADD:
        {
            wxASSERT( m_segment != NULL );

            if (m_segment)
                m_doc->GetDoodleSegments().Append(m_segment);
            m_segment = NULL;
            m_doc->Modify(true);
            m_doc->UpdateAllViews();
            break;
        }
    case DOODLE_CUT:
        {
            wxASSERT( m_segment == NULL );

            // Cut the last segment
            if (m_doc->GetDoodleSegments().GetCount() &gt; 0)
            {
                wxList::compatibility_iterator node = m_doc->GetDoodleSegments().GetLast();

                m_segment = (DoodleSegment *)node->GetData();
                m_doc->GetDoodleSegments().Erase(node);

                m_doc->Modify(true);
                m_doc->UpdateAllViews();
            }
            break;
        }
    }
    return true;
} 
```

因为在我们的例子中 Do 和 Undo 操作使用共用的代码,我们直接使用一个 DoOrUndo 函数来实现所有的操作.如果我们被要求执行 DOODLE_ADD 的撤消操作,我们可以直接执行 DOODLE_CUT,而要执行 DOODLE_CUT 的撤消操作,我们则直接执行 DOODLE_ADD.

当增加一个绘画片断(或者对某个 Cut 命令执行撤消操作)时,DoOrUndo 函数所做的事情就是把这个绘画片断增加到文档的绘画片断列表,并且将自己内部的绘画片断的指针清除,以便在释放这个命令对象的时候不需要释放这个绘画片断对象.相应的,当执行 Cut 操作(或者 Add 的 Undo 操作)的时候,将文档的片断列表中的最后一个片断从列表中移除,并且保存其指针,以便用于相应的恢复操作.DoOrUndo 函数做的另外一件事情是将文档标记为已修改状态(以便在应用程序退出时提醒用户保存文档)以及告诉文档需要更新和自己相关的所有的视图.

要定义自己的视图类,你需要实现 wxView 的派生类,同样的,需要使用动态创建的宏,并至少重载 OnCreate,OnDraw,OnUpdate 和 OnClose 函数.

OnCreate 函数在视图和文档对象刚被创建的时候调用,你应该执行的动作包括:创建 frame 窗口,使用 SetFrame 函数将其和当前视图绑定.

OnDraw 函数的参数为一个 wxDC 指针,用来实现窗口绘制操作. 实际上,这一步不是必须的,但是一旦你不使用重载 OnDraw 函数的方法来实现窗口绘制,默认的打印/预览机制将不能正常工作.

OnUpdate 函数的参数是一个指向导致这次更新操作的视图的指针以及一个指向一个用来帮助优化视图更新操作的对象的指针.这个函数在视图需要被更新的时候调用,这通常意味着由于执行某个文档命令导致相关视图需要更新,或者应用程序显式调用了 wxDocument:: UpdateAllViews 函数.

OnClose 函数在视图需要被关闭的时候调用,默认的实现是调用 wxDocument::OnClose 函数关闭视图绑定的文档.

下面是 DoodleView 的类声明.我们重载了前面介绍的四个函数,并且增加了一个 DOODLE_CUT 命令的处理函数.为什么这里没有 DOODLE_ADD 命令的处理函数,是因为绘画片断是随着鼠标的操作而增加的,因此 DOODLE_ADD 命令对应的视图动作已经在 DoodleCanvas 对象的鼠标处理函数中实现了.我们很快就会看到.

```cpp
/*
 * DoodleView 是文档和窗口之间的桥梁.
 */
class DoodleView: public wxView
{
    DECLARE_DYNAMIC_CLASS(DoodleView)
    DECLARE_EVENT_TABLE()
public:
    DoodleView() { m_frame = NULL; }
    ~DoodleView() {};
    /// 当文档被创建的时候调用
    virtual bool OnCreate(wxDocument *doc, long flags);
    /// 当需要绘制文档的时候被调用
    virtual void OnDraw(wxDC *dc);
    /// 当文档需要更新的时候被调用
    virtual void OnUpdate(wxView *sender, wxObject *hint = NULL);
    /// 当视图被关闭的时候调用
    virtual bool OnClose(bool deleteWindow = true);
    /// 用于处理 Cut 命令
    void OnCut(wxCommandEvent& event);
private:
    DoodleFrame*    m_frame;
}; 
```

下面的代码是其实现部分:

```cpp
IMPLEMENT_DYNAMIC_CLASS(DoodleView, wxView)
BEGIN_EVENT_TABLE(DoodleView, wxView)
    EVT_MENU(DOODLE_CUT, DoodleView::OnCut)
END_EVENT_TABLE()
// 当视图被创建的时候需要做的动作
bool DoodleView::OnCreate(wxDocument *doc, long WXUNUSED(flags))
{
    // 将当前主窗口和视图绑定
    m_frame = GetMainFrame();
    SetFrame(m_frame);
    m_frame->GetCanvas()->SetView(this);
    // 让视图管理器感知当前视图
    Activate(true);
    // 初始化编辑菜单中的重做/撤消项目
    doc->GetCommandProcessor()->SetEditMenu(m_frame->GetEditMenu());
    doc->GetCommandProcessor()->Initialize();
    return true;
}
// 这个函数被默认的打印/打印预览以及窗口绘制函数共用
void DoodleView::OnDraw(wxDC *dc)
{
    dc->SetFont(*wxNORMAL_FONT);
    dc->SetPen(*wxBLACK_PEN);
    wxList::compatibility_iterator node = ((DoodleDocument *)GetDocument
())->GetDoodleSegments().GetFirst();
    while (node)
    {
        DoodleSegment *seg = (DoodleSegment *)node->GetData();
        seg->Draw(dc);
        node = node->GetNext();
    }
}
void DoodleView::OnUpdate(wxView *WXUNUSED(sender), wxObject *WXUNUSED(hint))
{
    if (m_frame && m_frame->GetCanvas())
        m_frame->GetCanvas()->Refresh();
}
// 清除用于显式这个视图的窗口
bool DoodleView::OnClose(bool WXUNUSED(deleteWindow))
{
    if (!GetDocument()->Close())
        return false;
    // 清除画布
    m_frame->GetCanvas()->ClearBackground();
    m_frame->GetCanvas()->SetView(NULL);
    if (m_frame)
        m_frame->SetTitle(wxTheApp->GetAppName());
    SetFrame(NULL);
    // 告诉文档管理器不要再给我发送任何事件了.
    Activate(false);
    return true;
}
void DoodleView::OnCut(wxCommandEvent& WXUNUSED(event))
{
    DoodleDocument *doc = (DoodleDocument *)GetDocument();
    doc->GetCommandProcessor()->Submit(
        new DoodleCommand(wxT("Cut Last Segment"), DOODLE_CUT, doc, NULL));
} 
```

第 4 步: 定义你的窗口类

通常你需要创建特定的编辑窗口来维护你视图中的数据.在我们的例子中,DoodleCanvas 用来显示对应的数据,和相关的事件交互等,wxWidgets 的事件处理机制也要求我们最好创建一个新的派生类.DoodleCanvas 类的声明如下:

```cpp
/*
 * DoodleCanvas 是用来显示文档的窗口类
 */
class DoodleView;
class DoodleCanvas: public wxScrolledWindow
{
    DECLARE_EVENT_TABLE()
public:
    DoodleCanvas(wxWindow *parent, const wxPoint& pos,
                 const wxSize& size, const long style);
    /// 绘制文档内容
    virtual void OnDraw(wxDC& dc);
    /// 处理鼠标事件
    void OnMouseEvent(wxMouseEvent& event);
    /// 设置和获取视图对象
    void SetView(DoodleView* view) { m_view = view; }
    DoodleView* GetView() const { return m_view; }
protected:
    DoodleView *m_view;
}; 
```

DoodleCanvas 包含一个指向对应视图对象的指针(通过 DoodleView::OnCreate 函数初始化),以便在绘画和鼠标事件处理函数中使用.下面是这个类的实现部分:

```cpp
/*
 * Doodle 画布的实现
 */
BEGIN_EVENT_TABLE(DoodleCanvas, wxScrolledWindow)
    EVT_MOUSE_EVENTS(DoodleCanvas::OnMouseEvent)
END_EVENT_TABLE()
// 构造函数部分
DoodleCanvas::DoodleCanvas(wxWindow *parent, const wxPoint& pos,
                           const wxSize& size, const long style):
    wxScrolledWindow(parent, wxID_ANY, pos, size, style)
{
    m_view = NULL;
}
// 定制重绘行为
void DoodleCanvas::OnDraw(wxDC& dc)
{
    if (m_view)
        m_view->OnDraw(& dc);
}
// 这个函数实现了主要的涂鸦操作,主要用了鼠标左键事件.
void DoodleCanvas::OnMouseEvent(wxMouseEvent& event)
{
    // 上一次的位置
    static int xpos = -1;
    static int ypos = -1;
    static DoodleSegment *currentSegment = NULL;   
    if (!m_view)
        return;
    wxClientDC dc(this);
    DoPrepareDC(dc);
    dc.SetPen(*wxBLACK_PEN);
    // 将滚动位置计算在内
    wxPoint pt(event.GetLogicalPosition(dc));
    if (currentSegment && event.LeftUp())
    {
        if (currentSegment->GetLines().GetCount() == 0)
        {
            delete currentSegment;
            currentSegment = NULL;
        }
        else
        {
            // 当鼠标左键释放的时候我们获得一个绘画片断,因此需要增加这个片断
            DoodleDocument *doc = (DoodleDocument *) GetView()->GetDocument();
            doc->GetCommandProcessor()->Submit(
                new DoodleCommand(wxT("Add Segment"), DOODLE_ADD, doc, currentSegment));
            GetView()->GetDocument()->Modify(true);
            currentSegment = NULL;
        }
    }
    if (xpos &gt; -1 && ypos &gt; -1 && event.Dragging())
    {
        if (!currentSegment)
            currentSegment = new DoodleSegment;
        DoodleLine *newLine = new DoodleLine;
        newLine->m_x1 = xpos;
        newLine->m_y1 = ypos;
        newLine->m_x2 = pt.x;
        newLine->m_y2 = pt.y;
        currentSegment->GetLines().Append(newLine);
        dc.DrawLine(xpos, ypos, pt.x, pt.y);
    }
    xpos = pt.x;
    ypos = pt.y;
} 
```

正如你看到的那样,当鼠标处理函数检测到一个新的绘画片断被创建的时候,它提交给对应的文档对象一个 DOODLE_ADD 命令,这个命令将被保存以便支持撤消(以及将来的重做)动作.在我们的例子中,它被保存在文档的绘画片断列表中.

第 5 步,使用 wxDocManager 和 wxDocTemplate

你需要在应用程序的整个生命周期内维持一个 wxDocManager 实例,这个实例负责整个文档视图框架的协调工作.

你也需要至少一个 wxDocTemplate 对象.这个对象用来实现文档视图模型中文档和视图相关联的那部分工作.每一个文档/视图对, 对应一个 wxDocTemplate 对象,wxDocManager 对象将管理一个 wxDocTemplate 对象的列表以便用来创建文档和视图. wxDocTemplate 对象知道怎样的文件扩展名对应目前的文档对象以及怎样创建相应的文档或者视图对象等.

举例来说,如果我们的 Doodle 文档支持两种视图:图形视图和绘图片断列表视图,那么我们就需要创建两种视图对象 (DoodleGraphicView 和 DoodleListView),相应的我们也需要创建两种文档模板对象,一个用于图形视图,一个用于列表视图. 你可以给这两个 wxDocTemplate 使用同样的文档类和同样的文件扩展名,但是传递不同的视图类型.当用户点击应用程序的打开菜单时,文件选择对话框将额外显示一组可用的文件过滤器,每一个过滤器对应一个 wxDocTemplate 类型,当某个文件被选中打开的时候,wxDocManager 将使用对应的 wxDocTemplate 创建相应的文档类和视图类.同样的逻辑也被应用于创建新对象的时候.当然,在我们的例子中,只有一种 wxDocManager 对象,因此打开和新建文档的对话框就显得简单一些了.

你可以在你的应用程序种存储一个 wxDocManager 指针,但是对于 wxDocTemplate 通常没有这个必要,因为后者是被 wxDocManager 管理和维护的.下面是我们的 DoodleApp 类的定义部分:

```cpp
/*
 *声明一个应用程序类
 */
class DoodleApp: public wxApp
{
public:
    DoodleApp();
    virtual bool OnInit();
    virtual int OnExit();
private:
    wxDocManager* m_docManager;
};
DECLARE_APP(DoodleApp) 
```

在 DoodleApp 的实现部分,我们在 OnInit 函数种创建 wxDocManager 对象和一个和我们的 DoodleDocument 和 DoodleView 绑定的 wxDocTemplate 对象.我们给 wxDocTemplate 传递的参数包括: wxDocManager 对象,描述字符串,文件过滤器(在文件对话框种使用),默认打开目录(在文件对话框种使用),默认的文件扩展名(.drw,用来区分我们的文件类型)以及我们的文档和视图类型以及对应的类型信息.DoodleApp 的实现部分如下所示:

```cpp
IMPLEMENT_APP(DoodleApp)
DoodleApp::DoodleApp()
{
    m_docManager = NULL;
}
bool DoodleApp::OnInit()
{
    // 创建一个 wxDocManager
    m_docManager = new wxDocManager;
    // 创建我们需要的 wxDocTemplate
    (void) new wxDocTemplate(m_docManager, wxT("Doodle"), wxT("*.drw"), wxT(""), wxT
("drw"), wxT("Doodle Doc"), wxT("Doodle View"),
        CLASSINFO(DoodleDocument), CLASSINFO(DoodleView));
    // 在 Mac 系统上登记文档类型
#ifdef __WXMAC__
    wxFileName::MacRegisterDefaultTypeAndCreator( wxT("drw") , 'WXMB' , 'WXMA' ) ;
#endif
    // 对于我们的单文档界面,我们只支持最多同时打开一个文档
    m_docManager->SetMaxDocsOpen(1);
    // 创建主窗口
    DoodleFrame* frame = new DoodleFrame(m_docManager, NULL, wxID_ANY, wxT("Doodle
 Sample"), wxPoint(0, 0), wxSize(500, 400), wxDEFAULT_FRAME_STYLE);   
    frame->Centre(wxBOTH);
    frame->Show(true);
    SetTopWindow(frame);
    return true;
}
int DoodleApp::OnExit()
{
    delete m_docManager;
    return 0;
} 
```

因为我们只支持同时显示一个文档,我们需要通过函数 SetMaxDocsOpen 告诉文档管理器这一点.为了在 Mac OS 上提供一些额外的系统特性,我们也通过 MacRegisterDefaultTypeAndCreator 函数在 Mac 系统中注册了我们的文件类型. 这个函数的参数为文件扩展名,文档类型标识符以及创建标识符(按照惯例通常采用四个字节的字符串来标识,你也可以在苹果公司的网站上注册这种类型以避免可能存在的冲突).

Doodle 例子完整的代码请参考附带光盘的 examples/chap19/doodle 目录.

# 19.2 文档/视图框架的其它能力

上一节中我们通过一个简单的例子演示了使用文档视图框架所必须的一些步骤,这一节我们来讨论这个框架中一些更深入的话题.

标准标识符

文档/视图系统支持很多默认的标识符,比如 wxID_OPEN, wxID_CLOSE, wxID_CLOSE_ALL, wxID_REVERT, wxID_NEW, wxID_SAVE, wxID_SAVEAS, wxID_UNDO, wxID_REDO, wxID_PRINT 和 wxID_PREVIEW,为了更大的发挥框架的威力,你应该尽可能在你的菜单或者工具栏中使用这些标准的标识符.这些标识符的处理函数大多已经在 wxDocManager 类中实现,比如 OnFileOpen,OnFileClose 和 OnUndo 等.对应的处理函数将自动调用当前文档相应的处理函数.如果你愿意,你当然可以在你的 frame 窗口类或者 wxDocManager 的派生类中重载这些处理函数,不过通常都没有这个必要.

打印和打印预览

默认情况下,wxID_PRINT 和 wxID_PREVIEW 使用标准的 wxDocPrintout 类来实现打印和打印预览,以便直接重用 wxView:: OnDraw 函数.然而,这种用法的一个最大的缺陷是仅适用于只有一页的文档的情形.因此你可以创建你自己的 wxPrintout 类来重载标准的 wxID_PRINT 和 wxID_PREVIEW 处理,最快速的方法的方法当然是使用 wxHtmlEasyPrinting 类,我们在第十二章,"高级窗口类"的"HTML 打印"小结有比较详细的介绍.

文件访问历史

当你的应用程序初始化的时候,可以直接通过 wxConfig 对象使用 wxDocManager::FileHistoryLoad 函数在文件菜单的最下方加载一个文件访问历史列表,也可以在应用程序退出之前使用 wxDocManager::FileHistorySave 函数保存这个列表.比如,要加载文件访问历史,你可以这样做:

```cpp
// 加载文件访问历史
wxConfig config(wxT("MyApp"), wxT("MyCompany"));
config.SetPath(wxT("FileHistory"));
m_docManager->FileHistoryLoad(config);
config.SetPath(wxT("/")); 
```

如果你是在创建主窗口或者其主菜单之前加载的文件访问历史,你可以显式的通过 wxDocManager::FileHistoryAddFilesToMenu 函数将其增加在菜单中.

你也可以通过 wxFileHistory 类或者你自己的方法来实现文件访问历史功能,比如有时候你可能需要为每个文档窗口实现不同的文件访问历史.

显式创建文档类

有时候你需要显式的创建一个文档对象,比如说,有时候你想打开上次显式的文档,你可以通过下面的方式直接打开一个已经存在的文档:

```cpp
wxDocument* doc = m_docManager->CreateDocument(filename,
                                               wxDOC_SILENT); 
```

或者象下面这样创建一个新的文档:

```cpp
wxDocument* doc = m_docManager->CreateDocument(wxEmptyString,
                                               wxDOC_NEW); 
```

无论是上面哪种情况,都将自动创建一个相应的视图对象.

# 19.3 实现 Undo/Redo 的策略

你的应用程序的 Undo/Redo 机制的实现方法通常和你文档的数据类型以及用户操作数据的方式有关.在我们的例子中,我们每次只操作一整块的数据,操作也是很简单的.然而在很多应用程序中,用户可以对多种类型的数据进行操作.在这种情况下,我们可能需要使用另外一种命令表示方法,可以称之为 CommandState(状态命令),其中包含了文档中特定对象的信息.你的命令类应该维护一个状态列表,并且也可以在其构造函数中接受这样的状态列表参数.Do 和 Undo 操作将根据列表中的状态并将当前的命令应用到对应的状态.

实现 Redo/Undo 操作的关键,不外乎以每次一步的方式向前或者向后遍历每个历史命令.因此,你的 Undo/Redo 实现可以任意对文档状态进行快照操作,以用于将来的恢复操作.通过这种方法无论用户进行多少次向前或者向后的历史命令都没有问题,你的代码所需要作的只是怎样确定"完成"和"未完成"状态.

一个常用的方法是,在每个命令状态中保存文档中每个对象的一个拷贝,以及一个指向实际对象的指针.而 Do 和 Undo 操作只是简单的交换当前状态和历史状态.举例来说,比如对象是一个图形,用户将它的颜色从红色改为蓝色.应用程序于是创建了一个新的标识符用来指示当前红色的对象,但是它将其内部的颜色属性由红色设置为蓝色.当这个命令第一次执行的时候,Do 函数制作一个当前的红色对象的拷贝,并且对其应用这个新的命令(包括那个蓝颜色), 并将其置为可视状态,然后重绘这个对象,Undo 作的也是同样的事情,它制作一个当前蓝色对象的拷贝,然后对齐应用原来的状态(红色),然后重绘这个对象.因此 Do 和 Undo 函数其实是完全一样的.不光如此,这个函数还可以用于除更改颜色以外的其它操作,因为整个对象包括其属性在内都被制作了一份拷贝, 你可以通过给你的应用程序所能编辑的对象单独实现各自实现赋值和拷贝操作的构造函数的方法,使得这整个过程更直观.

让我们举个例子,以便说的更明白些.假如说,我们的文档包含多种形体,用户可以更改当前选中的所有形体的颜色.我们可以在相应的更改颜色菜单命令处理函数(比如 ShapeView::OnChangeColor)中,在这个命令应用于整个文档之前,为当前选种的各个对象创建一个新的状态拷贝,如下所示:

```cpp
// 更改当前选中的形体的颜色
void ShapeView::OnChangeColor(wxCommandEvent& event)
{
    wxColour col = GetSelectedColor();
    ShapeCommand* cmd = new ShapeCommand(wxT("Change color"));
    ShapeArray arrShape;
    for (size_t i = 0; i &lt; GetSelectedShapes().GetCount(); i++)
    {
        Shape* oldShape = GetSelectedShapes()[i];
        Shape* newShape = new Shape(*oldShape);
        newShape->SetColor(col);
        ShapeState* state = new ShapeState(SHAPE_COLOR, newShape, oldShape);
        cmd->AddState(state);
    }
    GetDocument()->GetCommandProcessor()->SubmitCommand(cmd);
} 
```

因为对于我们的 ShapeState 对象来说,Do 和 Undo 的操作都是一样的,因此我们可以使用一个共用的 DoAndUndo 函数,用来进行状态交换的动作:

```cpp
// 不完整的 DoAndUndo 实现
// 对于某些命令来说,Do 和 Undo 共用同样的代码
void ShapeState::DoAndUndo(bool undo)
{
    switch (m_cmd)
    {
    case SHAPE_COLOR:
    case SHAPE_TEXT:
    case SHAPE_SIZE:
        {
            Shape* tmp = new Shape(m_actualShape);
            (* m_actualShape) = (* m_storedShape);
            (* m_storedShape) = (* tmp);
            delete tmp;
            // 进行重绘动作
            ...
            break;
        }
    }
} 
```

上面的代码中我们没有列出 ShapeCommand::Do 和 ShapeCommand:Undo 函数,这两个函数所做的事情就是遍历其成员列表中的所有状态的相应函数.

(译者注:这一小节翻译的似曾相识,云里雾里)

# 第十九章小结

在这一章里,我们看到了怎样通过 wxWidgets 提供的文档/视图框架来简化这种类型的应用程序的编程,让 wxWidgets 来自动处理那些显示文件对话框或者创建文档和视图对象这种琐碎的工作.你也对如果实现 Undo/Redo 机制有了一个大概的印象.这个机制你应该在你的应用程序的初期就给予充分的考虑,否则到了应用程序的开发后期,这些功能的实现可能导致你重写大部分的代码.

在我们的最后一章中,我们将讨论一下如何让你的应用程序显得更完美.