# 第九章创建定制的对话框

也许很快,也许在将来什么时候,你就需要创建你自己的对话框.也许只是很简单的拥有一些按钮或者文本的对话框,也许是很复杂的包含 notebook 控件, 多面板,定制控件并且拥有上下文敏感帮助的对话框等.在这一章里,我们将介绍创建定制的对话框以及将数据在对话框控件和 C++变量之间传输的一般原理.我们还会介绍一下 wxWidgets 的资源管理体系,这个体系允许你从 XML 文件中加载并创建对话框或者是其它的用户界面.

# 9.1 创建定制对话框的步骤

当你开始创建你自己的对话框的时候,真正有趣的事情才算是刚刚开始.下面是通常你需要采取的步骤:

1.  从 wxDialog 派生一个新类.
2.  决定数据的存放位置以及应用程序以怎样的方式访问用户选择的数据.
3.  编写代码来创建和布局相关控件.
4.  增加代码以便在 C++变量和控件之间进行数据传输.
5.  增加事件映射和相应的处理函数以处理那些来自控件的事件.
6.  增加用户界面更新处理函数,以便将控件设置为正确的状态.
7.  增加帮助,尤其是工具提示以及上下文敏感帮助(在 Mac OS 中还没有实现),以及在你的应用程序的用户手册中添加你的对话框的使用方法.
8.  在你的应用程序中调用你定制的对话框.

让我们通过一个具体的例子来说明一下这些步骤.

# 9.2 一个例子:PersonalRecordDialog

正如我们在前面的章节看到的那样,对话框通常有两种:模式对话框和非模式对话框.我们将创建一个定制的模式的对话框,因为这是最常用的一种对话框,而且需要注意的方面也会更少.因为应用程序调用 ShowModal 函数以后,直到这个函数返回,应用程序将禁止除了*这个窗口以及这个窗口产生的别的模式对话框窗口*以外的所有别的窗口处理任何用户输入,而将所有的用户交互限制在我们这个模式对话框的小世界里面.

创建一个自定义对话框的许多步骤,可以通过使用一个对话框编辑器而变得简单.比如 wxDesigner 或者 DialogBlocks (译者注:当然,它们都是收费的.一个不收费的开源的对话框编辑器是 wxGlade,效果好像也不错).如果使用了对话框编辑器,那么剩余的需要你自己写代码的部分的复杂程度将和你的对话框的复杂程序相关.不过在这里,我们将以全部手写的方式创建这个对话框,以便于演示创建定制对话框的一些基本原理,但是我们强烈推荐你使用任何一种对话框编辑器因为它将节省你大量的时间.

我们将通过这样的一个对话框来演示创建自定义对话框的步骤: 用户需要使用这个对话框输入它的姓名,年龄和性别以及它是否想投票.这个对话框叫做 PersonalRecordDialog,它的样子如下图所示:

![](img/mht3AF6%281%29.tmp)

其中的 Reset 按钮将所有控件的值复位到它们的默认值.Ok 按钮关闭对话框并且以 wxID_OK 值从 ShowModal 函数返回. Cancel 按钮关闭对话框并且以 wxID_CANCEL 值从 ShowModal 函数返回,也不会用用户输入的值来更新对话框的内部变量.而 Help 按钮则显示一段文本用来大概描述这个对话框(当然,在实际的程序中,这个按钮应该调用一个更漂亮的格式化过的帮助文件).

一个好的用户界面应该不允许用户进行当前上下文中不允许的操作.在这个例子中,如果年龄的值小于 18,则投票按钮应该是不可用的(根据英国或者美国的法律).

派生一个新类

下面的代码定义了这个新的对话框类:PersonalRecordDialog,我们使用 DECLARE_CLASS 宏来提供运行期类型信息,而使用 DECLARE_EVENT_TABLE 宏来定义了一个事件表.

```cpp
/*!
 * PersonalRecordDialog 类定义
 */
class PersonalRecordDialog: public wxDialog
{
    DECLARE_CLASS( PersonalRecordDialog )
    DECLARE_EVENT_TABLE()
public:
    // 构造函数
    PersonalRecordDialog( );
    PersonalRecordDialog( wxWindow* parent,
      wxWindowID id = wxID_ANY,
      const wxString& caption = wxT("Personal Record"),
      const wxPoint& pos = wxDefaultPosition,
      const wxSize& size = wxDefaultSize,
      long style = wxCAPTION|wxRESIZE_BORDER|wxSYSTEM_MENU );
    //用来初始化内部变量
    void Init();
    // 创建窗体
    bool Create( wxWindow* parent,
      wxWindowID id = wxID_ANY,
      const wxString& caption = wxT("Personal Record"),
      const wxPoint& pos = wxDefaultPosition,
      const wxSize& size = wxDefaultSize,
      long style = wxCAPTION|wxRESIZE_BORDER|wxSYSTEM_MENU );
    // 创建普通控件和布局控件
    void CreateControls();
}; 
```

依照 wxWidgets 的惯例,我们同时支持了单步创建和两步窗口的方式,单步创建使用的是一个复杂的构造函数,而两步创建则使用的是一个简单的额构造函数和一个复杂的 Create 函数.

设计数据存储

我们有四个数据要存储:姓名(字符串),年龄(整数),性别(bool)和是否投票(bool).因为我们想用选择框来作为性别的用户界面,所以我们将性别的类型更改为整数类型,但是它可以对外表现为 bool 类型. 现在让我们来给 PersonalRecordDialog 类增加这些成员:

```cpp
// 数据成员
wxString    m_name;
int         m_age;
int         m_sex;
bool        m_vote;
// 姓名访问控制
void SetName(const wxString& name) { m_name = name; }
wxString GetName() const { return m_name; }
// 年龄访问控制
void SetAge(int age) { m_age = age; }
int GetAge() const { return m_age; }
// 性别访问控制 (男性 = false, 女性 = true)
void SetSex(bool sex) { sex ? m_sex = 1 : m_sex = 0; }
bool GetSex() const { return m_sex == 1; }
// 是否投票?
void SetVote(bool vote) { m_vote = vote; }
bool GetVote() const { return m_vote; } 
```

编码产生控件和布局

现在我们增加一个 CreateControls 函数来创建控件,这个函数将被 Create 函数调用,它将增加一个静态文本框,按钮,一个 wxSpinCtrl 控件和一个 wxTextCtrl 控件,一个 wxChoice 控件,一个 wxCheckBox 控件,参见上图中的最终效果.

我们使用了布局控件来进行布局,这是为什么它比你想像中的显得复杂了一点的原因(你应该还记得布局控件,我们在第七章对齐进行了描述, 简单的说就是让你的布局拥有随主窗口的大小,语言以及平台的变化而变化的能力).当然你也可以使用别的方法来进行布局,比如你可以使用从 wxWidgets 资源文件(XRC)中读取布局的方法.

我们来大概刷新一下你关于布局控件的记忆,布局控件把所有的控件分为一个一个的小格子,每个格子刚好可以放置一个控件,这些控件虽然可以拥有非常复杂的布局继承关系,但是它们的窗口继承关系非常单纯,都是继承自同一个父窗口.你可以参考第七章中的第二幅图来刷新你的记忆.

在 CreateControls 函数中,我们使用了一个垂直盒子布局控件嵌套了另外一个垂直盒子布局控件以便使对话框产生边界.一个水平的盒子布局控件用来放置 wxSpinCtrl, wxChoice 和 wxCheckBox,以及另外一个水平盒子布局控件来放置四个按钮.

```cpp
/*!
 * PersonalRecordDialog 控件创建
 */
void PersonalRecordDialog::CreateControls()
{
    // 一个顶层的布局控件
    wxBoxSizer* topSizer = new wxBoxSizer(wxVERTICAL);
    this->SetSizer(topSizer);
    // 第二个顶层布局控件用来产生边界
    wxBoxSizer* boxSizer = new wxBoxSizer(wxVERTICAL);
    topSizer->Add(boxSizer, 0, wxALIGN_CENTER_HORIZONTAL|wxALL, 5);
    // 一个友善的提示文本
    wxStaticText* descr = new wxStaticText( this, wxID_STATIC,
        wxT("Please enter your name, age and sex, and specify whether you wish to\nvote in
 a general election."), wxDefaultPosition, wxDefaultSize, 0 );
    boxSizer->Add(descr, 0, wxALIGN_LEFT|wxALL, 5);
    // 空格
    boxSizer->Add(5, 5, 0, wxALIGN_CENTER_HORIZONTAL|wxALL, 5);
    // 产生静态文本
    wxStaticText* nameLabel = new wxStaticText ( this, wxID_STATIC,
        wxT("&Name:"), wxDefaultPosition, wxDefaultSize, 0 );
    boxSizer->Add(nameLabel, 0, wxALIGN_LEFT|wxALL, 5);
    // 一个用于输入用户名的文本框
    wxTextCtrl* nameCtrl = new wxTextCtrl ( this, ID_NAME, wxT("Emma"), wxDefaultPosition,
 wxDefaultSize, 0 );
    boxSizer->Add(nameCtrl, 0, wxGROW|wxALL, 5);
    // 一个水平布局控件用来放置年龄,性别和是否投票
    wxBoxSizer* ageSexVoteBox = new wxBoxSizer(wxHORIZONTAL);
    boxSizer->Add(ageSexVoteBox, 0, wxGROW|wxALL, 5);
    // 年龄控件的标签
    wxStaticText* ageLabel = new wxStaticText ( this, wxID_STATIC,
        wxT("&Age:"), wxDefaultPosition, wxDefaultSize, 0 );
    ageSexVoteBox->Add(ageLabel, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // 用于输入年龄的 spin 控件
    wxSpinCtrl* ageSpin = new wxSpinCtrl ( this, ID_AGE,
        wxEmptyString, wxDefaultPosition, wxSize(60, -1),
        wxSP_ARROW_KEYS, 0, 120, 25 );
    ageSexVoteBox->Add(ageSpin, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // 性别标签
    wxStaticText* sexLabel = new wxStaticText ( this, wxID_STATIC,
        wxT("&Sex:"), wxDefaultPosition, wxDefaultSize, 0 );
    ageSexVoteBox->Add(sexLabel, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // 性别选择框
    wxString sexStrings[] = {
        wxT("Male"),
        wxT("Female")
    };
    wxChoice* sexChoice = new wxChoice ( this, ID_SEX,
        wxDefaultPosition, wxSize(80, -1), WXSIZEOF(sexStrings),
            sexStrings, 0 );
    sexChoice->SetStringSelection(wxT("Female"));
    ageSexVoteBox->Add(sexChoice, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // 增加一个可拉升的空白区域
    // 以便让投票选项出现在右边
    ageSexVoteBox->Add(5, 5, 1, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxCheckBox* voteCheckBox = new wxCheckBox( this, ID_VOTE,
       wxT("&Vote"), wxDefaultPosition, wxDefaultSize, 0 );
    voteCheckBox ->SetValue(true);
    ageSexVoteBox->Add(voteCheckBox, 0,
        wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // OK 和 Cancel 按钮之间的分割线
    wxStaticLine* line = new wxStaticLine ( this, wxID_STATIC,
        wxDefaultPosition, wxDefaultSize, wxLI_HORIZONTAL );
    boxSizer->Add(line, 0, wxGROW|wxALL, 5);
    // 用来放置四个按钮的水平盒子布局控件
    wxBoxSizer* okCancelBox = new wxBoxSizer(wxHORIZONTAL);
    boxSizer->Add(okCancelBox, 0, wxALIGN_CENTER_HORIZONTAL|wxALL, 5);
    // Reset 按钮
    wxButton* reset = new wxButton( this, ID_RESET, wxT("&Reset"),
        wxDefaultPosition, wxDefaultSize, 0 );
    okCancelBox->Add(reset, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // Ok 按钮
    wxButton* ok = new wxButton ( this, wxID_OK, wxT("&OK"),
        wxDefaultPosition, wxDefaultSize, 0 );
    okCancelBox->Add(ok, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // Cancel 按钮
    wxButton* cancel = new wxButton ( this, wxID_CANCEL,
        wxT("&Cancel"), wxDefaultPosition, wxDefaultSize, 0 );
    okCancelBox->Add(cancel, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
    // Help 按钮
    wxButton* help = new wxButton( this, wxID_HELP, wxT("&Help"),
        wxDefaultPosition, wxDefaultSize, 0 );
    okCancelBox->Add(help, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
} 
```

数据传输和验证

现在我们已经大概建立起来了这个对话框,但是控件和它的内部成员还没有连接起来,怎样完成这种连接呢?

当对话框第一次显示的时候,wxWidgets 会调用 InitDialog 函数,这个函数产生一个 wxEVT_INIT_DIALOG 事件.这个事件默认的处理函数是调用 transferDataToWindow 函数.要把数据从控件传回变量中,你可以在用户确认输入的时候调用 transferDataFromWindow 函数,wxWidgets 定义了一个默认的 wxID_OK 命令事件的处理函数,这个函数会在 EndModal 函数调用之前调用 transferDataFromWindow 函数.

因此,你可以通过重载 transferDataToWindow 和 transferDataFromWindow 函数以实现数据传输,对于我们这个对话框来说,你可以使用象下面这样的代码:

```cpp
/*!
 * 传输数据到控件
 */
bool PersonalRecordDialog::TransferDataToWindow()
{
    wxTextCtrl* nameCtrl = (wxTextCtrl*) FindWindow(ID_NAME);
    wxSpinCtrl* ageCtrl = (wxSpinCtrl*) FindWindow(ID_SAGE);
    wxChoice* sexCtrl = (wxChoice*) FindWindow(ID_SEX);
    wxCheckBox* voteCtrl = (wxCheckBox*) FindWindow(ID_VOTE);
    nameCtrl->SetValue(m_name);
    ageCtrl->SetValue(m_age);
    sexCtrl->SetSelection(m_sex);
    voteCtrl->SetValue(m_vote);
    return true;
}
/*!
 * 传输数据到变量
 */
bool PersonalRecordDialog::TransferDataFromWindow()
{
    wxTextCtrl* nameCtrl = (wxTextCtrl*) FindWindow(ID_NAME);
    wxSpinCtrl* ageCtrl = (wxSpinCtrl*) FindWindow(ID_SAGE);
    wxChoice* sexCtrl = (wxChoice*) FindWindow(ID_SEX);
    wxCheckBox* voteCtrl = (wxCheckBox*) FindWindow(ID_VOTE);
    m_name = nameCtrl->GetValue();
    m_age = ageCtrl->GetValue();
    m_sex = sexCtrl->GetSelection();
    m_vote = voteCtrl->GetValue();
    return true;
} 
```

然而,还有一个更容易的方法.wxWidgets 支持验证器,所谓验证器是一个把数据变量和对应的控件联系起来的对象.虽然不总是可以使用,但是只要可以使用,总是可以节省你大量的时间和代码来进行数据的传输和验证.在我们这个例子中,我们可以通过下面的代码来代替上面的额两个函数:

```cpp
FindWindow(ID_NAME)->SetValidator(
      wxTextValidator(wxFILTER_ALPHA, & m_name));
FindWindow(ID_AGE)->SetValidator(
      wxGenericValidator(& m_age));
FindWindow(ID_SEX)->SetValidator(
      wxGenericValidator(& m_sex);
FindWindow(ID_VOTE)->SetValidator(
      wxGenericValidator(& m_vote); 
```

这几行代码可以放在 CreateControls 函数的最后以便代替前面的两个函数,作为一个额外的好处,这样写代码还将增加校验以便用户不可以在姓名框中输入数字.

验证器可以执行两个任务,除了可以传输数据,它们还可以对数据进行校验.并且在数据不合法的时候进行错误提示.在这个例子中,除了姓名框以外,别的输入控件都没有设置校验.wxGenericValidator 是一个很简单的验证器,它只负责传输数据,不对数据进行校验,也正因为如此,它支持更多的基本控件.别的验证器则象 wxTextValidator 一样,提供了各种的校验方法,比如 wxTextValidator 就可以阻止那些不合法的按键事件传入文本框控件.在这个例子中,我们只是使用了标准校验类型 wxFILTER_ALPHA,但是通过验证器的 SetIncludes 和 SetExcludes 函数,我们还可以指定具体哪些字符是允许的哪些是不允许的.

我们还需要略微深入一点的来讨论 wxWidgets 怎样处理验证过程以便你更好的理解这儿到底发生了什么.正如我们看到的那样,默认的 OnOk 函数调用了 TransferDataToWindow 函数,但是在调用这个函数之前,它还调用了另外一个函数 Validate,下面的代码是默认的 OnOK 的代码:

```cpp
void wxDialog::OnOK(wxCommandEvent& event)
{
    if ( Validate() && TransferDataFromWindow() )
    {
        if ( IsModal() )
            EndModal(wxID_OK); // 如果是模式对话框
        else
        {
            SetReturnCode(wxID_OK);
            this->Show(false); // 如果是非模式对话框
        }
    }
} 
```

而默认的 Validate 函数的实现是遍历对话框所有的直接子窗口(如果设置了 wxWS_EX_VALIDATE_RECURSIVELY 类型的化,也会调用子窗口的子窗口),依次调用它们的 Validate 函数,如果任何一个调用失败,则整个 Validate 过程失败,那么对话框将不会被关闭,并且验证器自己会提供一个消息框以说明失败的原因.

类似的,TransferDataToWindow 和 TransferDataFromWindow 也将自动调用所有子控件的对应的函数.一个验证器可以不做校验,但是必须要可以支持作数据传输.

一个验证器其实是一个事件处理类,wxWidgets 的事件处理机制在将事件传递给控件本身之前,会检查这个控件是否被设置了验证器,如果已经设置了,则首先将这个事件传递给验证器,以便验证器可以通过 Veto 等方法阻止不合法的按键事件.在这种情况下,通常验证器应该调用 Beep 函数发出一声告警信号以便提示用户某个键已经被按下了但是是非法的.

wxWidgets 提供的两种验证器可能是不足够的,尤其是在你想创建自己的控件的时候,这种情况下你可能希望能够创建自己的验证器. 你需要从 wxValidator 类派生一个新类,这个新类必须带有一个自拷贝的构造函数和一个 Clone 函数用来返回自己的一份拷贝,还要实现自己的数据传输和验证机制.通常一个验证器都需要存储一个指向某个 C++变量的指针,并且构造函数中允许指定不同的模式以实现不同的验证机制.你可以参考 wxWidgets 源代码中的 include/wx/valtext.h 文件和 src/common/valtext.cpp 文件以了解怎样实现一个自己的验证器,也可以参考第十二章,"高级窗口类"中的"制作你自己的控件"这一小节.

处理事件

在这个例子中,wxWidgets 默认的处理对于 OK 和 Cancel 按钮来说已经是足够,不需要额外添加任何代码,只需要将其设置为对应的 wxID_OK 和 wxID_CANCEL 的标识符.不过,在自定义对话框中,进行相应的事件处理是很经常的事情.在这个例子中我们还有一个 Reset 按钮,当这个按钮被点击时,我们需要复位所有控件的值为它们的默认值.因此我们需要增加 OnResetClick 事件处理函数,以及一个对应的事件表条目. 这个处理函数的实现是非常简单的,首先我们调用我们自定义的 Init 成员函数来初始化所有的成员,然后调用 TransferDataToWindow 函数将数据传输到控件,如下所示:

```cpp
BEGIN_EVENT_TABLE( PersonalRecordDialog, wxDialog )
    ...
    EVT_BUTTON( ID_RESET, PersonalRecordDialog::OnResetClick)
    ...
END_EVENT_TABLE()
void PersonalRecordDialog::OnResetClick( wxCommandEvent& event )
{
    Init();
    TransferDataToWindow();
} 
```

处理 UI 更新

GUI 程序员设计代码的一大挑战是确保所有控件在其不可用的时候都是被禁用的.没有什么比弹出一个对话框告诉用户"这个按钮目前是不可用的"更能破坏程序的紧凑性的了.如果一个选项是不可用的,那么它看上去就应该是不可用的,用户点击它或者对它进行任何输入应该都是没有效果的.因此,我们应该在这种情况发生的时候,尽可能快的更新控件的状态使其不可用或者重新可用.

在我们这个例子里面,如果用户输入的年龄小于 18 岁,我们必须禁用投票按钮,使得这个选项对用户来说是不可用的.你的第一想法可能是要增加一个对于 spin 控件的事件的更新事件处理函数,并且在其中根据它的值来禁用或者可用投票复选框控件.当然,对于简单的情况来说,这样作完全没有问题但是试想一下,如果有很多种控件存在这种复杂的关联,或者更坏的情况,有时候我们根本不可能在影响某个控件的条件发生改变的时候获得事件通知,比如说我们的剪贴按钮,可能要随着剪贴板的有无数据情况来进行可用或者不可用的操作,而剪贴板是系统变量,它可能受别的应用程序的影响,因为我们自己的应用程序很难在其发生改变的时候收到通知事件.这种情况下唉该怎么办呢?

为了解决这个问题,wxWidgets 提供了一个称为 wxUpdateUIEvent 的事件类,这个事件实在系统空闲的时候(所有其它的事件都已经处理完的时候)发送给应用程序的.你可以使用 EVT_UPDATE_UI 宏来拦截这个事件,对应于每一个要和别的控件关联改变状态的控件,你需要在事件表中对应增加一个条目,每一个对应的处理函数中,可以检测这个世界当前的状况,从而改变某个控件当前的状态:允许或者禁止,选中或者未选中等, 这种机制允许你将某个控件相关的所有因素在某个特定的时机放在一个函数中处理.你可以松一口气了,因为你不必记住在每一个相关连的事件发生改变的时候去更新对应的控件了.

下面是我们用来处理 UI 更新事件的相关代码,要注意在代码中我们不能直接判断 m_age 成员,因为这个成员只有在用户点击了 OK 按钮以后才会将控件的数据传递过来.

```cpp
BEGIN_EVENT_TABLE( PersonalRecordDialog, wxDialog )
    ...
    EVT_UPDATE_UI( ID_VOTE, PersonalRecordDialog::OnVoteUpdate )
    ...
END_EVENT_TABLE()
void PersonalRecordDialog::OnVoteUpdate( wxUpdateUIEvent& event )
{
    wxSpinCtrl* ageCtrl = (wxSpinCtrl*) FindWindow(ID_AGE);
    if (ageCtrl->GetValue() &lt; 18)
    {
        event.Enable(false);
        event.Check(false);
    }
    else
        event.Enable(true);
} 
```

会不会有大量的这种界面更新事件而导致程序的性能降低呢?首先,这种情况是存在的,不过你不必过分担心这个问题,wxWidgets 已经作了大量的优化工作,使得这种系统开销将至了最低点.不过如果你的程序真的对性能有很大的要求并且真实感觉是因为这个界面更新的原因导致损失了性能,你还是可以通过 wxUpdateUIEvent 相关文档中提到的 SetMode 函数和 SetUpdateInterval 函数来对这种行为进行进一步的控制, 以减少界面更新事件的消耗的时间.

增加帮助信息

至少有三种帮助信息你可以给你的对话框提供.

*   工具提示
*   上下文敏感帮助
*   联机帮助

当然,你还可以使用更多更先进的没有被 wxWidgets 显式支持的技术. 我们已经在对话框上边放置了一段用来大概描述这个对话框用途的文本,对于更复杂的对呼框来说,你可以考虑使用 wxHtmlWindow 代替普通的文本以便可以加载一个 HTML 文件来提供更复杂的帮助信息.或者,你还可以在每个按钮旁边放置一个小按钮用来提供针对性的帮助信息.

下面我们来说一说 wxWidgets 明确支持的三种帮助形式:

工具提示

工具提示是在鼠标划过某个控件的时候弹出的一个小窗口,窗口里包含对这个控件用途的一个简短的提示.你可以使用 SetToolTip 函数来设置控件对应的工具提示.不过,由于对于一些有经验的使用者来说,这个提示可能会显得有些讨厌,因此你应该给你的应用程序增加一个全局的设置以便不显示这些工具提示(这里的意思是说,这个设置使得在对话框创建的时候不调用 SetToolTip 函数).

上下文敏感帮助

上下文敏感帮助提供的弹出窗口和工具提示窗口是非常相似的,不过这个窗口并不会自动探出来,而需要用户在点击了某个特殊按钮以后再点击对应的控件,或者在某个控件获得焦点的时候按 F1 键(windows 平台适用)的时候才会弹出.在 windows 平台上,你可以在创建对话框的时候指定 wxDIALOG_EX_CONTEXTHELP 类型以便在标题栏上出现这个上下文敏感帮助按钮,在其它平台上,你可以创建一个 wxContextHelpButton 类型的按钮,这个按钮通常应该位于 OK 和 Cancel 按钮的旁边.然后,在对话框初始话的时候,增加下面的代码:

```cpp
#include "wx/cshelp.h"
    wxHelpProvider::Set(new wxSimpleHelpProvider);

这将告诉 wxWidgets 怎样提供上下文敏感帮助,然后调用 SetHelpText 来设置某个控件的上下文敏感帮助文本.下面是我们例子中设置这两中帮助的代码:

void PersonalRecordDialog::SetDialogHelp()
{
    wxString nameHelp = wxT("Enter your full name.");
    wxString ageHelp = wxT("Specify your age.");
    wxString sexHelp = wxT("Specify your gender, male or female.");
    wxString voteHelp = wxT("Check this if you wish to vote.");
    FindWindow(ID_NAME)->SetHelpText(nameHelp);
    FindWindow(ID_NAME)->SetToolTip(nameHelp);
    FindWindow(ID_AGE)->SetHelpText(ageHelp);
    FindWindow(ID_AGE)->SetToolTip(ageHelp);
    FindWindow(ID_SEX)->SetHelpText(sexHelp);
    FindWindow(ID_SEX)->SetToolTip(sexHelp);
    FindWindow(ID_VOTE)->SetHelpText(voteHelp);
    FindWindow(ID_VOTE)->SetToolTip(voteHelp);
} 
```

如果你希望自己控制上下文敏感帮助,而不想通过对话框的上下文敏感帮助按钮或者 wxContextHelpButton 按钮,你可以在任何事件处理函数中使用下面的代码:

```cpp
wxContextHelp contextHelp(window); 
```

这将使 wxWidgets 进入一个检测左键单击的死循环,当单击事件发生时,wxWidgets 给对应的控件发送 wxEVT_HELP 事件,你可以拦截这个事件以便弹出你自己的帮助窗口.

你不必将自己限制于 wxWidgets 实现的存储显式帮助文本的方式,你可以创建自己的 wxHelpProvider 派生类,进而实现自己的 GetHelp, SetHelp, AddHelp, RemoveHelp 和 ShowHelp 函数.

联机帮助

大多数应用程序都会在一个帮助文件中提供使用应用程序的详细说明.wxWidgets 也对应于这种应用提供了几个对应的控件,这些控件都是 wxHelpControllerBase 的派生类.参考第二十章,"优化你的应用程序"来获得这方面更详细的信息.

针对我们这个应用程序而言,我们只是简单的在用户点击帮助按钮的时候显式一个消息框.

```cpp
BEGIN_EVENT_TABLE( PersonalRecordDialog, wxDialog )
    ...
    EVT_BUTTON( wxID_HELP, PersonalRecordDialog::OnHelpClick )
    ...
END_EVENT_TABLE()
void PersonalRecordDialog::OnHelpClick( wxCommandEvent& event )
{
    // 通常我们需要用下面注释的代码提供联机帮助
    /*
    wxGetApp().GetHelpController().DisplaySection(wxT("Personal record dialog"));
     */
    // 在这个例子中,我们只简单的提供一个消息框
    wxString helpText =
      wxT("Please enter your full name, age and gender.\n")
      wxT("Also indicate your willingness to vote in general elections.\n\n")
      wxT("No non-alphabetical characters are allowed in the name field.\n")
      wxT("Try to be honest about your age.");
    wxMessageBox(helpText,
        wxT("Personal Record Dialog Help"),
        wxOK|wxICON_INFORMATION, this);
} 
```

完整的例子

这个例子完整的代码列举在附录 J,"代码列表"中,你也可以在附送光盘的 examples/chap09 找到.

调用这个对话框

现在我们需要调用这个对话框来完成所有的编码,我们可以使用下面的代码来调用这个对话框:

```cpp
PersonalRecordDialog dialog(NULL, ID_PERSONAL_RECORD,
    wxT("Personal Record"));
dialog.SetName(wxEmptyString);
dialog.SetAge(30);
dialog.SetSex(0);
dialog.SetVote(true);
if (dialog.ShowModal() == wxID_OK)
{
    wxString name = dialog.GetName();
    int age = dialog.GetAge();
    bool sex = dialog.GetSex();
    bool vote = dialog.GetVote();
} 
```

# 9.3 在小型设备上调整你的对话框

wxWidgets 的几个不同的版本都可以用于支持移动设备和嵌入式平台,比如 GTK+版本,X11 的版本以及 WindowsCE 的版本(将来也许还会有别的版本)..在这些小型设备使用的最大的问题在于屏幕的大小,一个 smartphone 的屏幕的大小甚至只有 176x220 个象素.

对于这样的小型屏幕来说,大多数的对话框都需要一个替代的布局方案.甚至有些控件本身都需要被移除,尤其是那些相比较于桌面系统中其功能已经被移除的那部分控件.你可以使用 wxSystemSettings::GetScreenType 函数获取当前屏幕的尺寸类型,如下所示:

```cpp
#include "wx/settings.h"
bool isPda = (wxSystemSettings::GetScreenType() &lt;= wxSYS_SCREEN_PDA); 
```

这个函数的返回值列举在下表中,因为它随着屏幕实际尺寸的增长而增加,因此你可以直接对其通过简单的整数比较的方法,来判断是否为比某个屏幕更大或者更小的屏幕.

| wxSYS_SCREEN_NONE | 未定义屏幕类型 |
| --- | --- |
| wxSYS_SCREEN_TINY | 微小屏幕,尺寸小于 320x240 |
| wxSYS_SCREEN_PDA | PDA 屏幕, 尺寸在 320x240 到 640x480 之间 |
| wxSYS_SCREEN_SMALL | 小型屏幕,尺寸在 640x480 到 800x600 之间 |
| wxSYS_SCREEN_DESKTOP | 桌面屏幕,尺寸大于 800x600 |

如果你需要更具体的大小,你可以使用下面的方法:

1.  使用 wxSystemSettings::GetMetric, 传递 wxSYS_SCREEN_X 或 wxSYS_SCREEN_Y 参数.
2.  使用 wxGeTDisplaySize,得到一个 wxSize 类型的屏幕大小.
3.  创建一个 wxDisplay 对象,调用其 GetGeometry 成员函数, 将返回表征当前屏幕的一个矩形区域(wxRect 类型).

当你明确知道你的程序将在一个很小的屏幕上运行的时候,你可以作些什么呢?这里我们给出一些建议:

1.  使用从一个不同的 XRC 文件加载布局的方法来代替整个布局代码或者针对小型的屏幕使用不同的布局代码,只要你不更改控件的类型,事件处理函数的代码就不需要作任何改动.
2.  减小控件以及空白区域的大小.
3.  改变控件的类型(比如使用 wxComboBox 来代替 wxListBox),这将导致一些相关处理函数的代码的改动
4.  改变布局的方向.一些小型设备通常在和桌面屏幕相反的方向拥有更多的空间.

有时候你还需要使用针对特定平台的 API 函数.比如微软的 Smartphone 有两个特殊的按钮,你可以对其设置标签,比如"OK", "Cancel"之类的.在这种平台上,你可以使用调用 wxDialog::SetLeftMenu 和 wxDialog::SetRightMenu 函数 (参数为标识符,标签以及可选的子菜单)来取代产生两个单独的按钮.当然由于这些函数仅存在于这个平台,因此你需要使用宏开关来区分不同的平台:

```cpp
#ifdef __SMARTPHONE__
    SetLeftMenu(wxID_OK, wxT("OK"));
    SetRightMenu(wxID_OK, wxT("Cancel"));
#else
    wxBoxSizer* buttonSizer = new wxBoxSizer(wxHORIZONTAL);
    GetTopSizer()->Add(buttonSizer, 0, wxALL|wxGROW, 0);
    buttonSizer->Add(new wxButton(this, wxID_OK), 0, wxALL, 5);
    buttonSizer->Add(new wxButton(this, wxID_CANCEL), 0, wxALL, 5);
#endif 
```

# 9.4 一些更深入的话题

下面的这些提示可以让你创建的对话框看上去更专业.

键盘导航

尽量给你的对话框上的控件的标签增加"&"前导符,在某些平台(特别明显的是在 windows 和 GTK+平台上),这将使得用户可以通过键盘在控件之间移动.

总是给你的对话框提供一个取消操作,最好是使用 Escape 键来执行这个操作.如果你的对话框有一个标识符为 wxID_CANCEL 的按钮,那么默认情况下它的处理函数将在用户按下 Escape 键的时候被执行,因此,如果你有一个单纯用来关闭对话框的按钮,最好将它的标识符定义为 wxID_CANCEL.

提供一个默认按钮(通常为 OK 按钮),你可以通过 wxButton::SetDefault 函数指定一个默认按钮,这个按钮的处理函数将在用户按下回车键的时候被调用.

数据和用户界面分离

为了简化例子,我们在 PersonalRecordDialog 中采用了把数据存放在对话框内部的方法.然而,一个更好的设计应该是让用户数据和对话框分离,在构造函数中进行赋值操作,这样的话你就可以更方便的传递一组数据给这个对话框的构造函数,以及在用户确认修改的时候(按下 OK 按钮的时候)从对话框获取一整组数据.这也是大多数标准对话框采用的方法.作为一个练习,你可以使用 PersonalRecordData 类作为 PersonalRecordDialog 的数据成员重写 PersonalRecordDialog 的代码,以使得对话框的构造函数采用 PersonalRecordData 的引用作为参数,而对话框的 Getdata 函数则返回其内部数据的引用.

一般说来,你应该尽可能的将界面功能和非界面功能分开.这通常会让你的代码显得更紧凑.不要害怕增加新的类,它会让你的设计更优雅,也不要惧怕在类中使用复制或者赋值之类的操作,如果一个对象能够很容易的被赋值和赋值,应用程序可以少些很多底层的赋值代码.

除非你的对话框提供了一个类似"Apply"功能的按钮,否则,在对话框被取消之后,所有内部的数据应该保持原样.使用数据和界面分离的原则也使得实现这一点变得相对容易,因为通常在这种情况下,你只是操作数据的一份拷贝而不是数据本身.

布局

如果你的对话框看上去好像显得有些拥挤或者说有些丑陋,可能是因为你没有给予足够的空白区间.你可以尝试单独使用一个布局控件来增加一个大的边界区域,就象我们在例子中作的那样,在一组控件和另外一组控件之间增加空白,或者使用 wxStaticBoxSizer 和 wxStaticLine 将逻辑上处于两组的控件区隔开.使用 wxGridSizer 和 wxFlexGridSizer 布局控件来对齐控件及它们对应的标签以便它们的位置显得不那么零乱.在基于布局控件的布局中,还可以使用空白区域来实现对齐.比如,通常 OK,Cancel 按钮和 Help 按钮应该被设置为右对齐,你可以通过在水平 wxBoxSizer 的水平方向上增加一个可以缩放的(缩放因子不为负数的)空白区域,然后再增加这些按钮,以便在对话框的水平大小发生改变时实现按钮的右对齐.

尽可能的让你的对话框的边框是可以改变大小的,通常 windows 系统上的对话框的大小是不能被改变的,但是这样做实在是有些无厘头. 在一个大的对话框上使用很少的控件通常都令用户感到沮丧.在 wxWidgets 下通过布局控件创建可变大小的对话框是非常简单的事情,你应该尽可能的使用布局控件以便你的对话框可以自适应字体和语言以及对话框大小的改变.当然,你要小心的选择那些可以随着对话框大小的改变而改变大小的控件,例如,多行文本框控件就是一个不错的选择,它的大小随着对话框的大小一起增长可以给用户更多实用的空间,另外,你还可以考虑把增加的空间分给空白区域以实现对齐.注意我们这里说的控件增大,不是只的缩放控件或者是让控件的文本变的更大,而仅仅是指的增加控件的宽度和高度.参考第七章以得到更多关于布局控件的知识.

如果你发现你的对话框上放置了太多的控件,你应该考虑增加面板,并且使用 wxNotebook, wxListbook 或 wxChoicebook 来进行分页.使用太多菜单来打开很多互不相干的对话框通常是非常令用户感到不爽的,用分页控件,用户自己选择查看哪个页面,这样就显得方便多了.同时在面板里使用滚动条也是应该被避免的(除非你又充足的原因).这一方面是因为不是所有的平台都支持滚动控件, 另外一方面,这也会给用户一个印象:你对控件布局并没有进行充分的设计.如果你有很多的属性需要用户编辑,你可以考虑使用基于 wxGrid 的属性编辑器或者其它的第三方控件(参考 wxPropertyGrid,在附录 E,"wxWidgets 的第三方工具"中有提到这个控件).

美学

标签的大小写要保持一致.不要给对话框设置自定义的颜色和字体,这有时候会让你的用户感到抓狂而且也使得你的应用程序看上去有些特立独行 (贬义),和系统当前的风格或者你的应用程序中的其它对话框不一致.要在各个平台都取得不错的效果,最好让 wxWidgets 自己决定对话框的颜色或者字体.取而代之的,你可以把精力花费在设计一些新颖而紧凑的小图片上,并且在合适的位置使用 wxStaticBitmap 控件显式它.

对话框的替代品

最后,对于那些非模式的解决方案,你应该好好考虑是否要使用一个对话框,也许在应用程序的主窗口中使用 TAB 控件会是更好的选择. 尽管我们前面所说的大部分原则都适用于非模式的对框框,但是它们确实有一些不同之处,比如你需要额外考虑布局(通常这种窗口拥有原少于它的大小的控件)以及数据同步(对于非模式窗口来说,它在显示的时候不能以独占的方式访问它的数据了).

# 9.5 使用 wxWidgets 资源文件

你可以从一个 Xml 文件中加载对话框,frame 窗口,菜单条,工具条等等,而不一定非要用 C++代码来创建它们.这更符合界面和代码分离的原则,它可以让应用程序的界面在运行期改变.XRC 文件可以通过一系列用户界面设计的工具导出,比如:wxDesigner, DialogBlocks, XRCed 和 wxGlade.

加载资源文件

要使用 XRC 文件,你需要在你的代码中包含 wx/xrc/xmlres.h 头文件.

如果你打算将你的 XRC 文件转换成二进制的 XRS 文件(我们很快会介绍到),你还需要增加 zip 文件系统的处理函数,你可以在你的 OnInit 函数中增加下面的代码来作到这一点:

```cpp
#include "wx/filesys.h"
#include "wx/fs_zip.h"
wxFileSystem::AddHandler(new wxZipFSHandler); 
```

首先初始化 XRC 处理系统,你需要在 OnInit 中增加下面的代码:

```cpp
wxXmlResource::Get()->InitAllHandlers(); 
```

然后加载一个 XRC 文件:

```cpp
wxXmlResource::Get()->Load(wxT("resources.xrc")); 
```

这只是告诉 wxWidgets 这个资源文件的存在,要创建真实的用户界面,还需要类似下面的代码:

```cpp
MyDialog dlg;
wxXmlResource::Get()->LoadDialog(& dlg, parent, wxT("dialog1"));
dlg.ShowModal(); 
```

下面的代码则演示了怎样创建菜单条,菜单,工具条,位图,图标以及面板:

```cpp
MyFrame::MyFrame(const wxString& title): wxFrame(NULL, -1, title)
{
    SetMenuBar(wxXmlResource::Get()->LoadMenuBar(wxT("mainmenu")));
    SetToolBar(wxXmlResource::Get()->LoadToolBar(this,
                                                  wxT("toolbar")));
    wxMenu* menu = wxXmlResource::Get()->LoadMenu(wxT("popupmenu"));
    wxIcon icon = wxXmlResource::Get()->LoadIcon(wxT("appicon"));
    SetIcon(icon);
    wxBitmap bitmap = wxXmlResource::Get()->LoadBitmap(wxT("bmp1"));
    // 既可以先创建实例再加载
    MyPanel* panelA = new MyPanel;
    panelA = wxXmlResource::Get()->LoadPanel(panelA, this,
                                                       wxT("panelA"));
    // 又可以直接创建并加载
    wxPanel* panelB = wxXmlResource::Get()->LoadPanel(this,
                                                       wxT("panelB"));
} 
```

wxWidgets 维护一个全局的 wxXmlResource 对象,你可以直接拿来使用,也可以创建一个你自己的 wxXmlResource 对象,然后加载某个资源文件,然后使用和释放它.你还可以使用 wxXmlResource::Set 函数来让应用程序用某个 wxXmlResource 对象来取代全局资源对象,并释放掉那个旧的.

要为定义在资源文件中的控件定义事件表条目,你不能直接使用整数的标识符,因为资源文件中存放的其实是字符串,你需要使用 XRCID 宏,它的参数是一个资源名,返回值是这个资源对应的标识符.其实 XRCID 就是直接使用的 wxXmlResource::GetXRCID 函数,举例如下:

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU(XRCID("menu_quit"),  MyFrame::OnQuit)
    EVT_MENU(XRCID("menu_about"), MyFrame::OnAbout)
END_EVENT_TABLE() 
```

使用二进制和嵌入式资源文件

你可以把多个 wxWidgets 资源文件编译成一个二进制的压缩的 xrs 文件.使用的工具 wxrc 可以在 wxWidgets 的 utils/wxrc 目录中找到,使用方法如下:

```cpp
wxrc resource1.xrc resource2.xrc -o resource.xrs 
```

使用 wxXmlResource::Load 函数加载一个二进制的压缩的资源文件,和加载普通的文本 Xml 文件没有区别.

提示: 你可以不必把你的 XRC 文件单独制作一个 zip 压缩文件,而是把它放在其它一个可能包含 HTML 文件以及图片文件的普通的 zip 压缩文件中, wxXmlResource::Load 函数支持虚拟文件系统定义(参考第十四章:"文件和流"),因此你可以通过下面的方法来加载压缩文件中的 XRC 文件:

```cpp
wxXmlResource::Get()->Load(wxT("resources.bin#zip:dialogs.xrc")); 
```

你也可以将 XRC 文件编译为 C++的代码,通过和别的 C++的代码编译在一起,你就可以去掉某个单独的资源文件了.编译用的命令行如下所示:

```cpp
wxrc resource1.xrc resource2.xrc c -o resource.cpp 
```

编译方法和编译普通的 C++代码相同,这个文件包含一个 InitXmlResource 函数,你必须在你的主程序中调用这个函数:

```cpp
extern void InitXmlResource(); // defined in generated file
wxXmlResource::Get()->InitAllHandlers();
InitXmlResource(); 
```

下面列出了 wxrc 程序的命令行参数:

| 短命令格式 | 长命令格式 | 描述 |
| --- | --- | --- |
| -h | help | 显式帮助信息. |
| -v | verbose | 打印执行过程信息. |
| -c | cpp-code | 编译目标为 C++代码,而不是 XRS 文件. |
| -p | python-code | 编译目标为 Python 代码而不是 XRS 文件. |
| -e | extra-cpp-code | 和-c 一起使用,指示为 XRC 定义的窗口生成头文件. |
| -u | uncompressed | 不要压缩 Xml 文件(C++ only). |
| -g | gettext | 将相关的字符串翻译为 poEdit 或者 gettext 可以识别的格式.输出到标准输出或者某个文件中(如果指定了-o 参数的话). |
| -n | function <name> | 指定特定的 C++初始化函数(和-c 一起使用). |
| -o <filename> | output <filename> | 指定输出文件名,比如 resource.xrs or resource.cpp. |
| -l <filename> | list-of-handlers <filename> | 列举这个资源文件所需要的处理函数. |

资源翻译

如果 wxXmlResource 对象创建的时候指定了 wxXRC_USE_LOCALE 标记(默认行为),所有可显示的字符串都将被认为是需要翻译的,具体内容参考第十六章,"编写国际化应用程序",然后 poEdit 并能查找 XRC 文件来发现那些需要翻译的字符串,因此,必须使用"-g" 参数产生一个对应的 C++文件,以供 poEdit 使用,命令行如下:

```cpp
wxrc -g resources.xrc -o resource_strings.cpp 
```

然后你就可以使用 poEdit 来搜索这个和其它的 C++文件了.

XRC 的文件格式

这里显然不是完整描述 XRC 文件格式的地方,因此我们只举一个简单的使用了布局控件的例子:

```cpp
<?xml version="1.0"?>
<resource version="2.3.0.1">
<object class="wxDialog" name="simpledlg">
    <title>A simple dialog</title>
    <object class="wxBoxSizer">
      <orient>wxVERTICAL</orient>
      <object class="sizeritem">
        <object class="wxTextCtrl">
          <size>200,200d</size>
          <style>wxTE_MULTILINE|wxSUNKEN_BORDER</style>
          <value>Hello, this is an ordinary multiline\n textctrl....</value>
        </object>
        <option>1</option>
        <flag>wxEXPAND|wxALL</flag>
        <border>10</border>
      </object>
      <object class="sizeritem">
        <object class="wxBoxSizer">
          <object class="sizeritem">
            <object class="wxButton" name="wxID_OK">
              <label>Ok</label>
              <default>1</default>
            </object>
          </object>
          <object class="sizeritem">
            <object class="wxButton" name="wxID_CANCEL">
              <label>Cancel</label>
            </object>
            <border>10</border>
            <flag>wxLEFT</flag>
          </object>
        </object>
        <flag>wxLEFT|wxRIGHT|wxBOTTOM|wxALIGN_RIGHT</flag>
        <border>10</border>
      </object>
    </object>
  </object>
</resource> 
```

XRC 文件格式的详细描述可以在 wxWidgets 自带的文档目录 docs/tech/tn0014.txt 中找到.如果你使用对话框编辑器的话,你管它的文件格式干嘛呢.

你可能回问怎样在 XRC 文件中指定二进制的图片或者图标文件呢?实际上这些资源是通过 URLs 来指定的,wxWidgets 的虚拟文件系统将会从合适的地方(比如一个压缩文件中)获取指定的文件.举例如下:

```cpp
<object class="wxBitmapButton" name="wxID_OK">
  <bitmap>resources.bin#zip:okimage.png</bitmap>
</object> 
```

关于使用虚拟文件系统加载资源或者图片的细节,请参考第十章,"在程序中使用图片"以及第十四章"文件和流".

编写资源处理类

XRC 系统使用不同的资源处理类来识别 Xml 文件中定义的不同的资源.如果你编写了自己的控件,你就需要编写自己的资源处理类.

wxButton 的资源处理类如下所示:

```cpp
#include "wx/xrc/xmlres.h"
class wxButtonXmlHandler : public wxXmlResourceHandler
{
DECLARE_DYNAMIC_CLASS(wxButtonXmlHandler)
public:
    wxButtonXmlHandler();
    virtual wxObject *DoCreateResource();
    virtual bool CanHandle(wxXmlNode *node);
}; 
```

资源处理类的实现是非常简单的.在其构造函数的实现中,使用 XRC_ADD_STYL 宏来使得处理类可以识别控件相关的特殊的窗口类型,然后使用 AddWindowStyles 增加这些类型.然后在 DoCreateResource 函数中,使用两步法创建按钮实例,其中第一步要使用 XRC_MAKE_INSTANCE 函数,然后调用 Create 函数,参数需要使用对应的函数从 Xml 文件中获得.而 CanHandle 函数则用来回答是否这个处理类可以处理某个 Xml 节点的问题.使用一个处理类处理多种 Xml 节点是允许的.

```cpp
IMPLEMENT_DYNAMIC_CLASS(wxButtonXmlHandler, wxXmlResourceHandler)
wxButtonXmlHandler::wxButtonXmlHandler()
: wxXmlResourceHandler()
{
    XRC_ADD_STYLE(wxBU_LEFT);
    XRC_ADD_STYLE(wxBU_RIGHT);
    XRC_ADD_STYLE(wxBU_TOP);
    XRC_ADD_STYLE(wxBU_BOTTOM);
    XRC_ADD_STYLE(wxBU_EXACTFIT);
    AddWindowStyles();
}
wxObject *wxButtonXmlHandler::DoCreateResource()
{
   XRC_MAKE_INSTANCE(button, wxButton)
   button->Create(m_parentAsWindow,
                    GetID(),
                    GetText(wxT("label")),
                    GetPosition(), GetSize(),
                    GetStyle(),
                    wxDefaultValidator,
                    GetName());
    if (GetBool(wxT("default"), 0))
        button->SetDefault();
    SetupWindow(button);
    return button;
}
bool wxButtonXmlHandler::CanHandle(wxXmlNode *node)
{
    return IsOfClass(node, wxT("wxButton"));
} 
```

要使用某种处理类,应用程序需要包含相应的头文件并且登记这个处理类,就象下面这样:

```cpp
#include "wx/xrc/xh_bttn.h"
wxXmlResource::AddHandler(new wxBitmapXmlHandler); 
```

外来控件

XRC 文件还可以通过 class="unknown"来指定某个控件是外来的或者说是"未知的"控件.这可以用来实现在其父窗口已经加载到应用程序之中以后,使用 C++代码来创建这个未知的控件.当 XRC 文件加载一个未知控件的时候,它将创建一个用来占位的窗口,然后在代码中,可以使用 C ++先创建这个实际的控件,然后使用 AttachUnknownControl 函数替换掉那个用来占位的窗口.如下所示:

```cpp
wxDialog dlg;
// 加载对话框
wxXmlResource::Get()->LoadDialog(&dlg, this, wxT("mydialog"));
// 创建特殊控件
MyCtrl* myCtrl = new MyCtrl(&dlg, wxID_ANY);
// 增加到对话框里
wxXmlResource::Get()->AttachUnknownControl(wxT("custctrl"), myCtrl);
// 显示整个对话框
dlg.ShowModal(); 
```

外来的控件在 XRC 文件中可以这样定义:

```cpp
<object class="unknown" name="custctrl">
  <size>100,100</size>
</object> 
```

使用这种技术,你可以既不用创建新的资源处理类,又可以在资源文件中使用未知的控件.

# 第九章小结

在这一章里,你学习了怎样创建自定义对话框的一些基本原理,包括布局控件概览,使用验证器以及处理 UI 更新时间的优点等.在 wxWidgets 自带的 samples/dialogs 目录中你也可以看到一个自定义对话框的例子.另外 samples/validate 目录中的例子则演示了一般的文本验证器的用法.在下一章里,我们来看看怎样处理图片.