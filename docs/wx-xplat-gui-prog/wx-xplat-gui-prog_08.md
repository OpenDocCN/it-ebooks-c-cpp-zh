# 第八章使用标准对话框

本章描述了 wxWidgets 提供的标准的对话框,使用这些这些对话框,你只用很少的代码就可以用来显示一些信息或者从用户那里获取数据.熟悉这些标准的对话框将会节省你大量的时间,并且让你的程序显得更专业.wxWidgets 在任何可能的时候使用本地原生的对话框,但是有些时候,wxWidgets 也是一些自己的对话框,比如 wxTextEntryDialog,这些对话框统称为一般的对话框.在这一章中,对于那些在各个平台上外观有较大差别的对话框,也给予了单独的图片说明.

我们人为的把标准对话框分成了以下四类:信息对话框,文件和目录对话框,选项和选择对话框以及输入对话框.

# 8.1 信息对话框

在这一小节中,我们来看看以下四种用来提供信息的对话框:wxMessageDialog, wxProgressDialog, wxBusyInfo, and wxShowTip.

wxMessageDialog

这种对话框显示一个消息和一组按钮,按钮可以为 OK, Cancel, Yes 或者 No,还可以有一个可选的图标,用来显示一个惊叹号或者一个问号.消息文本中还可以包含换行符"\n".

wxMessageDialog::ShowModal 函数的返回值用来表征哪个按钮被按下了.

下图显示了 wxMessageDialog 在 windows 平台上的样子:

![](img/mhtE8C9%281%29.tmp)

在 GTK+平台上:

![](img/mhtE8CC%281%29.tmp)

在 Mac 平台上:

![](img/mhtE8CF%281%29.tmp)

要创建一个这种对话框,你需要提供父窗口指针,要显示的消息,可选的标题,类型和位置参数,然后调用 ShowModal 函数显示这个对话框,然后判断这个函数的返回值进行进一步的动作.

其中的类型参数是一个比特位列表,其值如下表所示:

| wxOK | 显示一个 OK 按钮. |
| --- | --- |
| wxCANCEL | 显示一个 Cancel 按钮. |
| wxYES_NO | 显示 Yes 和 No 按钮. |
| wxYES_DEFAULT | 设置 Yes 为默认按钮. 和 wxYES_NO 一起使用.这是 wxYES_NO 类型的默认行为. |
| wxNO_DEFAULT | 设置 No 按钮为默认按钮,和 wxYES_NO 一起使用. |
| wxICON_EXCLAMATION | 显示一个惊叹号. |
| wxICON_ERROR | 显示一个错误图标. |
| wxICON_HAND | 和 wxICON_ERROR 相同. |
| wxICON_QUESTION | 显示一个问号. |
| wxICON_INFORMATION | 显示一个信息图标. |
| wxSTAY_ON_TOP | 在 windows 平台上,这个对话框将在所有的窗口(包括那些其它应用程序的窗口)之上. |

wxMessageDialog 使用举例

```cpp
#include "wx/msgdlg.h"
wxMessageDialog dialog( NULL, wxT("Message box caption"),
      wxT("Message box text"),
      wxNO_DEFAULT|wxYES_NO|wxCANCEL|wxICON_INFORMATION);
switch ( dialog.ShowModal() )
{
  case wxID_YES:
      wxLogStatus(wxT("You pressed \"Yes\""));
      break;
  case wxID_NO:
      wxLogStatus(wxT("You pressed \"No\""));
      break;
  case wxID_CANCEL:
      wxLogStatus(wxT("You pressed \"Cancel\""));
      break;
  default:
      wxLogError(wxT("Unexpected wxMessageDialog return code!"));
} 
```

wxMessageBox

你可以使用更方便的 wxMessageBox 函数,它的参数为一个消息文本,标题文本,类型和父窗口.例如:

```cpp
if (wxYES == wxMessageBox(wxT("Message box text"),
    wxT("Message box caption"),
    wxNO_DEFAULT|wxYES_NO|wxCANCEL|wxICON_INFORMATION,
    parent))
{
    return true;
} 
```

要注意 wxMessageBox 的返回值和 wxMessageDialog::ShowModal 的返回值是不一样的,前者返回 wxOK, wxCANCEL, wxYES 或 wxNO,而后者返回 wxID_OK, wxID_CANCEL, wxID_YES 或 wxID_NO.

wxProgressDialog

wxProgressDialog 可以用来显示一个短的消息文本和一个进度条用来指示用户还需要等待多久.它还可以显示一个 Cancel 按钮用来中止正在进行的处理,还可以显示已经过去的时间,估计剩余的时间和估计全部的时间.这个对话框是 wxWidgets 在各个平台上自己实现的.下图显示了这个对话框在 windows 上的样子:

![](img/mhtE8E2%281%29.tmp)

你既可以用全局变量或者 new 函数来创建这种对话框,也可以直接使用局部变量创建这种对话框,需要传递的参数包括:标题,消息文本,进度条的最大值,父窗口和类型.

类型的值如下表所示:

| wxPD_APP_MODAL | 设置为模式对话框.如果没有设置这个类型,对话框为非模式对话框,意味着除了其父窗口以外,应用程序中其它的窗口还可以输入数据. |
| --- | --- |
| wxPD_AUTO_HIDE | 导致这个进度条对话框在进度达到最大值的时候自动消失. |
| wxPD_CAN_ABORT | 告诉对话框显示一个取消按钮,当用户点击这个按钮以后,下次调用对话框的 Update 函数将会返回失败. |
| wxPD_ELAPSED_TIME | 显示已逝去时间标签. |
| wxPD_ESTIMATED_TIME | 显示估计全部时间标签. |
| wxPD_REMAINING_TIME | 显示估计剩余时间标签. |

在进度对话框被创建以后,其父窗口将被禁用,如果设置了 wxPD_APP_MODAL 类型,则应用程序中其它的窗口也将被禁用.应用程序应用调用 Update 函数来更新进度条以及提示信息,如果设置了时间显示标签,则它们的值将被自动计算并在每次调用 Update 的时候刷新.

如果设置了 wxPD_AUTO_HIDE 类型,对话框会在进度达到最大值的时候自动隐藏,你应该自己根据其创建方式释放这个对话框.对于由于用户点击取消按钮而导致 Update 失败的情况,如果你愿意,你可以使用 Resume 函数还恢复中止的进度条.

wxProgressDialog 使用举例

```cpp
#include "wx/progdlg.h"
void MyFrame::ShowProgress()
{
    static const int max = 10;
    wxProgressDialog dialog(wxT("Progress dialog example"),
                              wxT("An informative message"),
                              max,    // range
                              this,   // parent
                              wxPD_CAN_ABORT |
                              wxPD_APP_MODAL |
                              wxPD_ELAPSED_TIME |
                              wxPD_ESTIMATED_TIME |
                              wxPD_REMAINING_TIME);
    bool cont = true;
    for ( int i = 0; i &lt;= max; i++ )
    {
        wxSleep(1);
        if ( i == max )
            cont = dialog.Update(i, wxT("That's all, folks!"));
        else if ( i == max / 2 )
            cont = dialog.Update(i, wxT("Only a half left (very long message)!"));
        else
            cont = dialog.Update(i);

        if ( !cont )
        {
            if ( wxMessageBox(wxT("Do you really want to cancel?"),
                               wxT("Progress dialog question"),
                               wxYES_NO | wxICON_QUESTION) == wxYES )
                break;

            dialog.Resume();
        }
    }
    if ( !cont )
        wxLogStatus(wxT("Progress dialog aborted!"));
    else
        wxLogStatus(wxT("Countdown from %d finished"), max);
} 
```

wxBusyInfo

wxBusyInfo 其实不是一个对话框不过它的表现和对话框非常相似,当这个对象被创建的时候,屏幕上将显示一个窗口以及一条让用户耐心等待的消息,这个窗口将存在于 wxBusyInfo 的整个生命周期.在 windows 平台上,它的长相类似下面的样子:

![](img/mhtE8E5%281%29.tmp)

wxBusyInfo 也可以以全局和局部的方式创建,需要传递给构造函数的参数包括一个消息文本和一个父窗口.

wxBusyInfo 使用举例

在下面的例子中,首先使用 wxWindowDisabler 对象禁用应用程序当前创建的所有的窗口,然后显示了一个 wxBusyInfo 窗口:

```cpp
#include "wx/busyinfo.h"
wxWindowDisabler disableAll;
wxBusyInfo info(wxT("Counting, please wait..."), parent);
for (int i = 0; i &lt; 1000; i++)
{
    DoCalculation();
} 
```

wxShowTip

许多应用程序都会在程序启动的时候显示一个附加的窗口,用来给出一些如何使用这个应用程序的提示信息,那些不愿阅读沉闷的文档的人会非常喜欢这样的学习方式的.

启动提示窗口在 windows 平台上的样子如下图所示:

![](img/mhtE8E8%281%29.tmp)

和大多数对话框不同,这个对话框是使用 wxShowTip 函数显示的,传递的参数为一个父窗口指针,一个指向 wxTipProvider 对象的指针和一个可选的 bool 参数用来指示是否显示一个复选框,以便用户可以选择是否在应用程序启动的时候显示这个提示框.而函数的返回值则为用户的选择.

你必须实现一个 wxTipProvider 的派生类,实现其中的 GetTip 函数才可以使用 wxShowTip 函数,幸运的是, wxWidgets 已经实现了一个这样的基于文本文件的类.你可以直接使用 wxCreateFileTipProvider 函数,传递以文本文件(每行一个提示文本)路径和默认选择索引来创建一个这样的类.

应用程序应负责在 wxTipProvider 对象不需要的时候释放这个对象.

wxShowTip 使用举例

```cpp
#include "wx/tipdlg.h"
void MyFrame::ShowTip()
{
    static size_t s_index = (size_t)-1;
    if ( s_index == (size_t)-1 )
    {
        // 随机化...
        srand(time(NULL));
        // ...选择一个提示
        s_index = rand() % 5;
    }
    // 传递一个提示文件以及提示索引
    wxTipProvider *tipProvider =
        wxCreateFileTipProvider(wxT("tips.txt"), s_index);
    m_showAtStartup = wxShowTip(this, tipProvider, true);
    delete tipProvider;
} 
```

# 8.2 文件和目录对话框

如果想让用户选择文件和目录,你可以使用下面这两种对话框:wxFileDialog 和 wxDirDialog.

wxFileDialog

wxFileDialog 用来让用户选择一个或多个文件.它还有一个专门用来打开文件或者保存文件的变体.

下图演示了 windows 平台上文件对话框的样子:

![](img/mht8C73%281%29.tmp)

GTK+ 版本:

![](img/mht8C85%281%29.tmp)

GTK+ 2.4 以上版本:

![](img/mht8C88%281%29.tmp)

Mac Os X 版本:

![](img/mht8C9B%281%29.tmp)

创建一个文件对话框需要传递的参数包括一个父窗口,一个显示在对话框标题栏的消息文本,默认的目录,默认文件名,通配符,对话框类型和显示位置(有些平台上忽略这个参数).然后调用其 ShowModal 函数并且判断函数返回值,如果返回值为 wxID_OK 则表明用户确认了文件的选择.

目录名和文件名组成一个文件全路径.如果目录名为空,则默认为当前工作目录,如果文件名为空,则没有默认文件名.

通配符用来确定哪种类型的文件应该显示在对话框中.通配符可以用"|"分割多种文件类型,并且可以提供文件类型的描述文字,具体格式如下所示:

```cpp
BMP files (*.bmp)|*.bmp|GIF files (*.gif)|*.gif 
```

如果在文件名文本区输入一个带有通配符("*","?")的文件名,然后点确定按钮,将会导致只有符合这个通配名的文件名被显示.

wxFileDialog 的类型

如下表所示:

| wxSAVE | 指定为一个保存文件对话框. |
| --- | --- |
| wxOPEN | 指定为一个打开文件对话框(默认行为). |
| wxOVERWRITE_PROMPT | 对于保存文件对话框,如果目标文件已经存在,则提示是否覆盖. |
| wxFILE_MUST_EXIT | 用户只能选择已经存在的文件. |
| wxMULTIPLE | 用户可以选择多个文件. |

wxFileDialog 的成员函数

Getdirectory 返回默认的目录名或者单选文件对话框中选中文件的所在的目录名,使用 SetDirectory 设置默认目录.

GetFilename 返回不包括目录部分的默认文件名或者单选文件对话框中选中的文件的文件名.使用 SetFilename 设置默认文件名.

GetFilenames 使用 wxArrayString 类型返回多选文件对话框中所有选中的文件名.通常,这些文件名是不含有路径的, 但是在 windows 平台上,如果选中的是一个快捷方式文件,windows 可能会增加上一个全路径,因为应用程序无法通过通过增加当前的目录的方式来得到其全路径.使用 GetPaths 可以得到包含全路径在内的已选中文件的列表.

GetFilterIndex 用来返回默认的基于 0 的过滤器的索引.过滤器通常以一个下拉框的方式显示.使用 SetFilterIndex 设置默认的过滤器索引.

GetMessage 返回对话框的标题文本. 使用 SetMessage 函数来设置标题文本.

GetPath 以全路径的方式返回单选文件框中默认文件或者选中文件名.对于多选框,应使用 GetPaths 函数.

GetWildcard 返回指定的通配符, SetWildcard 用来设置通配符.

wxFileDialog 例子

下面的代码用来创建和显示一个用来打开 BMP 或者 GIF 类型文件的对话框:

```cpp
#include "wx/filedlg.h"
wxString caption = wxT("Choose a file");
wxString wildcard =
   wxT("BMP files (*.bmp)|*.bmp|GIF files (*.gif)|*.gif");
wxString defaultDir = wxT("c:\\temp"));
wxString defaultFilename = wxEmptyString;
wxFileDialog dialog(parent, caption, defaultDir, defaultFilename,
    wildcard, wxOPEN);
if (dialog.ShowModal() == wxID_OK)
{
    wxString path = dialog.GetPath();
    int filterIndex = dialog.GetFilterIndex();
} 
```

wxDirDialog

wxDirDialog 用来让用户选择一个本地或者网络文件夹.如果在构造函数中设置了可选类型 wxDD_NEW_DIR_BUTTON,则对话框将显示一个用来创建新文件夹的按钮.

下图演示了 windows 平台上的目录对话框的样子,这是 windows 系统提供的原生控件:

![](img/mht8C9E%281%29.tmp)

linux 平台上通常使用 GTK+版本的 wxDirDialog,如下图所示:

![](img/mht8CB0%281%29.tmp)

Mac 平台上的 wxDirDialog 和文件选择对话框非常相似,如下图所示:

![](img/mht8CB3%281%29.tmp)

创建一个目录对话框需要传递的参数包括:一个父窗口,一个标题文本,一个默认路径,一个窗口类型以及一个位置和大小(最后两个参数在某些平台上被忽略),然后调用 ShowModal 函数,判断返回值是否为 wxID_OK 以确定用户是否进行了选择.

wxDirDialog 成员函数

SetPath 和 GetPath 用来获得和设置用户选择的目录.

SetMessage 和 GetMessage 用来获取和设置标题文本.

wxDirDialog 使用举例

```cpp
#include "wx/dirdlg.h"
wxString defaultPath = wxT("/");
wxDirDialog dialog(parent,
    wxT("Testing directory picker"),
    defaultPath, wxDD_NEW_DIR_BUTTON);
if (dialog.ShowModal() == wxID_OK)
{
    wxString path = dialog.GetPath();
    wxMessageBox(path);
} 
```

# 8.3 选择和选项对话框

在这一小节中,我们来看看那些让你作出选择或者允许你给出选项的对话框,包括:wxColourDialog, wxFontDialog, wxSingleChoiceDialog 和 wxMultiChoiceDialog.

wxColourDialog

这个对话框允许用户从标准颜色或者是一个颜色范围中选择一种颜色.

在 windows 系统上,颜色选择对话框使用的是 windows 系统提供的原生控件,这个对话框主要包含三个区域,左上角是一个由 48 中常用颜色组成的区域,左下角是一个拥有 16 个选项的定制颜色区,用户的应用程序可以自行设置这 16 种定制的颜色,右边则可以展开用来精确的选择一个颜色.下图演示了这个对话框完全展开以后的样子:

![](img/mhtD2D6%281%29.tmp)

通用的颜色选择对话框如下图所示,可以在 GTK+ 1 或者 X11 系统上使用.它包含 48 个常用颜色和 16 个可定制颜色,右边包含三个滑条用来选择 RGB 三原色的值.选好的值用来替换当前选中的定制颜色或者 (如果当前没有选中任何定制颜色的话)第一个定制颜色.和 windows 系统原生的颜色选择框不同,右边的滑条区域是不可以隐藏的.在 Windows 平台和其它平台上,你也可以通过 wxGenericColourDialog 类来使用这个通用颜色选择对话框.

![](img/mhtD2E9%281%29.tmp)

下图演示了 GTK+的原生颜色选择对话框:

![](img/mhtD2EC%281%29.tmp)

Mac OsX 平台上的颜色选择对话框如下图所示:

![](img/mhtD2FE%281%29.tmp)

要使用颜色选择对话框,你可以创建一个 wxColourDialog 局部变量,传递的参数包括父窗口指针和一个 wxColourData 指针,后者用来设置一些默认数据,然后调用 ShowModal 函数,当这个函数返回时,通过 GetColourData 函数获取用户对 wxColourData 所做的更改.

wxColourData 的成员函数

SetChooseFull 定义在 windows 平台上,颜色选择对话框应该是完全展开的方式显示.否则将只显示左边的部分.GetChooseFull 函数则用来获取 Bool 格式的这个选项的值.

SetColour 用来设置默认的颜色, GetColour 函数用来返回用户当前选择的颜色.

SetCustomColour 使用一个 0 到 15 之间的索引和一个 wxColour 类型的颜色值来设置颜色选择对话框的定制颜色.使用 GetCustomColour 函数返回当前的可定制范围的颜色值,这个值可能在用户操作颜色对话框的过程中被改变.

wxColourDialog 使用举例

下面是一个使用 wxColourDialog 的例子,它将首先设置 wxColourData 的各种参数,包括 16 个灰阶色彩的定制颜色.如果用户没有取消这个对话框,就用用户选择的颜色来设置当前的背景色.

```cpp
#include "wx/colordlg.h"
wxColourData data;
data.SetChooseFull(true);
for (int i = 0; i &lt; 16; i++)
{
    wxColour color(i*16, i*16, i*16);
    data.SetCustomColour(i, color);
}
wxColourDialog dialog(this, &data);
if (dialog.ShowModal() == wxID_OK)
{
    wxColourData retData = dialog.GetColourData();
    wxColour col = retData.GetColour();
    myWindow->SetBackgroundColour(col);
    myWindow->Refresh();
} 
```

wxFontDialog

wxFontDialog 可以让用户来选择一个字体(在某些平台上,还可以选择字体的颜色).

在 Windows 平台上,使用的是 windows 系统提供的原生控件,这个控件给用户的选择包括字体的名称,点大小,类型,粗细,下划线,中划线等属性以及字体的前景颜色.一个白色区域还显示了当前选择字体的样子.注意在 windows 的字体转换到 wxWidgets 的字体过程中,中划线属性将被忽略,字体名称则被相应的字体家族名称取代. 而在 GTK+平台上,使用的是 GTK+的标准字体对话框,这个对话框不允许选择字体前景颜色.

下图演示了 windows 平台上字体选择对话框的样子:

![](img/mhtD301%281%29.tmp)

下图演示了 GTK+上的标准字体选择对话框的样子:

![](img/mhtD314%281%29.tmp)

除了上述两个平台以外,其它平台字体选择对话框的样子都很相似,允许用户选择的项目包括字体家族名称,大小,类型,粗细,下划线以及前景颜色等,还包括一个示例区用来显示当前字体的样子.这种字体选择框在各种平台都是可以使用的,类的名字为 wxGenericFontDialog. 下图演示了 Mac 平台上的这种字体选择对话框的样子:

![](img/mhtD317%281%29.tmp)

要使用字体选择对话框,需要传递的参数包括父窗口和一个 wxFontData 对象,然后调用其 ShowModal 函数并且测试其返回值是否为 wxID_OK,然后从对话框中获取 wxFontData 数据,然后视需要调用其 GetChosenFont 和 GetChosenColour 函数.

wxFontData 的成员函数

EnableEffects 函数允许 windows 平台上的字体选择对话框或者通用版本的字体选择对话框显示颜色和下划线属性控制.在 GTK 版本上则没有任何效果.GetEnableEffects 函数则用来判断当前是否设置了这个标志. 不过即使禁止了这个标志,字体颜色选项还是会被保留.

SetAllowSymbols 允许选择符号字体(仅适用于 Windows), GetAllowSymbols 则返回这个选项的当前设置值.

SetColour 设置默认字体颜色, GetColour 则返回当前用户选择的颜色.

SetInitialFont 返回对话框被创建时候的默认字体. GetChosenFont 则返回用户选择的字体(wxFont).

SetShowHelp 用来指示应该在字体对话框上显示帮助按钮(仅适用于 Windows). GetShowHelp 则返回这个选项的当前设置.

SetRange 设置用户可以选择的字体大小的范围, 默认值(0, 0)表示任何大小的字体都可以被选择.仅适用于 windows.

字体选择使用举例

在下面的例子中,应用程序使用字体选择对话框返回的字体在窗口上绘画.

```cpp
#include "wx/fontdlg.h"
wxFontData data;
data.SetInitialFont(m_font);
data.SetColour(m_textColor);
wxFontDialog dialog(this, &data);
if (dialog.ShowModal() == wxID_OK)
{
    wxFontData retData = dialog.GetFontData();
    m_font = retData.GetChosenFont();
    m_textColor = retData.GetColour();
    // 更新当前窗口以便使用当前选择的字体和颜色进行重绘
    myWindow->Refresh();
} 
```

wxSingleChoiceDialog

wxSingleChoiceDialog 给用户提供了一组字符串以便用户可以选择其中的一个.它的外观如下图所示:

![](img/mhtD32A%281%29.tmp)

传递给构造函数的参数包括其父窗口指针,一个消息文本,对话框的标题文本以及一个 wxArrayString 类型的字符串选项列表.其中最后一个参数可以使用一个大小和一个 C 类型的字符串指针的指针(wxChar**)代替.

SetSelection 用来在对话框显示之前设置一个默认的选项,使用 GetSelection(返回当前选项的索引)或者 GetStringSelection(返回对应的字符串)来在对话框关闭以后获取用户的选择.

你还可以在这种对话框的构造函数中传递一组 char*类型的客户区数据,在对话框被关闭以后,使用 GetSelectionClientData 函数获得和用户选项对应的客户区数据.

wxSingleChoiceDialog 使用举例

```cpp
#include "wx/choicdlg.h"
const wxArrayString choices;
choices.Add(wxT("One"));
choices.Add(wxT("Two"));
choices.Add(wxT("Three"));
choices.Add(wxT("Four"));
choices.Add(wxT("Five"));
wxSingleChoiceDialog dialog(this,
                            wxT("This is a small sample\nA single-choice convenience dialog"),
                            wxT("Please select a value"),
                            choices);
dialog.SetSelection(2);
if (dialog.ShowModal() == wxID_OK)
    wxMessageBox(dialog.GetStringSelection(), wxT("Got string")); 
```

wxMultiChoiceDialog

wxMultiChoiceDialog 和 wxSingleChoiceDialog 很相似,不过它允许用户进行多项选择.这种对话框的外观如下图所示:

![](img/mhtD32D%281%29.tmp)

传递给这种对话框的构造函数的参数包括一个父窗口指针,一个消息文本,对话框标题文本和一个 wxArrayString 类型的选项列表. 和 wxSingleChoiceDialog 一样,最后一个参数可以使用字符串指针的列表 wxChar**和数量代替.和 wxSingleChoiceDialog 不同的是,不可以在构造函数中提供客户区数据.

使用函数 SetSelections 还设置默认选中的选项,其参数为 wxArrayInt 类型,代表一个整数数组.使用 GetSelections 来获取用户的选择,返回值也为 wxArrayInt 类型.

wxMultiChoiceDialog 使用举例

```cpp
#include "wx/choicdlg.h"
const wxArrayString choices;
choices.Add(wxT("One"));
choices.Add(wxT("Two"));
choices.Add(wxT("Three"));
choices.Add(wxT("Four"));
choices.Add(wxT("Five"));
wxMultiChoiceDialog dialog(this,
                            wxT("A multi-choice convenience dialog"),
                            wxT("Please select several values"),
                            choices);
if (dialog.ShowModal() == wxID_OK)
{
    wxArrayInt selections = dialog.GetSelections();
    wxString msg;
    msg.Printf(wxT("You selected %u items:\n"),
        selections.GetCount());

    for ( size_t n = 0; n &lt; selections.GetCount(); n++ )
    {
        msg += wxString::Format(wxT("\t%d: %d (%s)\n"),
                              n, selections[n],
                              choices[selections[n]].c_str());
    }
    wxMessageBox(msg, wxT("Got selections"));
} 
```

# 8.4 输入对话框

这一类对话框让用户自己输入信息,包括:wxNumberEntryDialog, wxTextEntryDialog, wxPasswordEntryDialog 和 wxFindReplaceDialog.

wxNumberEntryDialog

wxNumberEntryDialog 提示用户输入一个固定范围内的数字,这个对话框包含一个 spin 控件因此,用户既可以手动输入数字,也可以通过鼠标点击 spin 按钮来调整数字的值,这个对话框是 wxWidgets 自己实现的,因此在各个平台上的表现都是相似的.

创建 wxNumberEntryDialog 需要提供的参数包括一个父窗口,消息文本,提示文本(显示在 spin 控件的前面),标题文本,默认值,最小值和最大值,位置等,然后调用 ShowDialog 函数,如果返回 wxID_OK,则可以调用 GetValue 函数返回用户输入的数字的值.

下图演示了其在 windows 平台上的样子:

![](img/mht9AB%281%29.tmp)

wxNumberEntryDialog 使用举例

上图中的对话框是用下面的代码创建的:

```cpp
#include "wx/numdlg.h"
wxNumberEntryDialog dialog(parent,
  wxT("This is some text, actually a lot of text\nEven two rows of text"),
  wxT("Enter a number:"), wxT("Numeric input test"), 50, 0, 100);
if (dialog.ShowModal() == wxID_OK)
{
  long value = dialog.GetValue();
} 
```

wxTextEntryDialog 和 wxPasswordEntryDialog

wxTextEnTRyDialog 和 wxPasswordEntryDialog 提供一个消息文本和一个单行文本框控件,以便用户可以输入文本,它们的功能很类似,只不过在 wxPasswordEntryDialog 中输入的文本被以掩码的方式显示,因此是不能直接看到的. 下图演示了 wxTextEntryDialog 对话框在 windows 平台上的例子:

![](img/mht9AE%281%29.tmp)

创建这两个对话框需要提供的参数包括父窗口指针,消息文本,标题文本,默认文本和一个类型参数.类型参数是一个比特位列表,其值为 wxOK, wxCANCEL,wxCENTRE(或者 wxCENTER)等,你还可以传递 wxTextCtrl 的窗口类型 wxTE_CENTRE(或 wxTE_CENTER)等.

你可以使用 SetValue 函数单独设置其默认文本,还可以使用 GetValue 函数获取用户输入的文本.

wxTextEntryDialog 使用举例

上图演示的对话框是用下面的代码创建的:

```cpp
#include "wx/textdlg.h"
wxTextEntryDialog dialog(this,
                           wxT("This is a small sample\n")
                           wxT("A long, long string to test out the text entrybox"),
                           wxT("Please enter a string"),
                           wxT("Default value"),
                           wxOK | wxCANCEL);
if (dialog.ShowModal() == wxID_OK)
    wxMessageBox(dialog.GetValue(), wxT("Got string")); 
```

wxFindReplaceDialog

wxFindReplaceDialog 是一个非模式对话框,它允许用户用来输入用于搜索的文本以及(如果需要的话)用来替换的文本.实际的搜索动作需要在其派生类或者其父窗口作为这个对话框某个按钮时间的响应来完成.和大多数标准对话框不同,这种对话框必须拥有一个父窗口(非空),并且这个对话框必须是非模式显示的,无论是基于设计还是实现来说.

下图演示了 windows 系统上的查找和替换对话框:

![](img/mht9C1%281%29.tmp)

在其它平台(比如 GTK+或 Mac OS X)上, wxWidgets 使用自己实现的通用版对话框,如下图所示:

![](img/mht9C4%281%29.tmp)

处理这个对话框相关的事件

wxFindReplaceDialog 对话框在用户点击其上的按钮的时候产生一些命令事件.事件处理函数采用 wxFindDialogEvent 类型的参数,事件映射宏中的窗口标识符为这个对话框的标识符,这些宏如下表所示:

| EVT_FIND(id, func) | 当"查找"按钮被按下时产生. |
| --- | --- |
| EVT_FIND_NEXT(id, func) | 当"下一个"按钮被按下时产生. |
| EVT_FIND_REPLACE(id, func) | 当"替换"按钮被按下时产生. |
| EVT_FIND_REPLACE_ALL(id, func) | 当"替换全部"按钮被按下时产生. |
| EVT_FIND_CLOSE(id, func) | 当用户通过取消或者别的途径关闭对话框的时候产生. |

wxFindDialogEvent 的成员函数

GetFlags 返回下列值的一组比特位列表:wxFR_DOWN, wxFR_WHOLEWORD 和 wxFR_MATCHCASE.

GetFindString 返回用户输入的要查找的文本.

GetreplaceString 返回用户输入的要替换的文本.

Getdialog 返回一个指向产生这个事件的对话框的指针.

向对话框传递数据

创建 wxFindReplaceDialog 需要传递的参数包括一个父窗口,一个指向 wxFindReplaceData 的指针,标题文本和一个类型,类型是下表所示比特值的列表:

| wxFR_REPLACEDIALOG | 指定对话框是查找替换对话框,而不是查找对话框. |
| --- | --- |
| wxFR_NOUPDOWN | 只是查找方向不允许被改变. |
| wxFR_NOMATCHCASE | 支持仅允许大小敏感的搜索或替换. |
| wxFR_NOWHOLEWORD | 指定不支持整字搜索的选项. |

wxFindReplaceData 保存了所有 wxFindReplaceDialog 相关的信息.用来对 wxFindReplaceDialog 对象进行初始化的动作以及用来在 wxFindReplaceDialog 对话框关闭以后保存其相关的信息,它的值也会在每次产生 wxFindDialogEvent 事件的时候自动更新,因此你可以直接使用它的成员函数来代替使用 wxFindDialogEvent 事件的成员函数.使用对话框的 GetData 函数可以返回在构造对话框的时候填充的 wxFindReplaceData 对象指针.

wxFindReplaceData 的成员函数

下面列出了 wxFindReplaceData 的用来设置或者获取相关数据的函数,注意那些用于设置的函数只在这个对话框显示之前有用,在对话框显示以后,调用这些用于设置的函数是没有任何效果的.

GetFindString 和 SetFindString 用来设置或者获取要查找的字符串.

GetFlags 和 SetFlags 用来设置或者获取查找替换对话框选项的相应状态(前面已经有具体描述).

GetreplaceString 和 SetReplaceString 用来设置或者获取要替换成的字符串.

查找和替换使用举例

下面演示了查找和替换对话框的使用方法,其中 DoFind 和 DoReplace 函数的代码没有列出来,它们用来进行应用程序相关的查找和替换动作.同时,这些函数还应该维护一组应用程序相关的变量,用来保存当前查找的位置,以便下次查找在这之后进行,这些函数还应该完成文档视图相匹配部分的高亮显示.

```cpp
#include "wx/fdrepdlg.h"
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
     EVT_MENU(ID_REPLACE, MyFrame::ShowReplaceDialog)
     EVT_FIND(wxID_ANY, MyFrame::OnFind)
     EVT_FIND_NEXT(wxID_ANY, MyFrame::OnFind)
     EVT_FIND_REPLACE(wxID_ANY, MyFrame::OnReplace)
     EVT_FIND_REPLACE_ALL(wxID_ANY, MyFrame::OnReplaceAll)
     EVT_FIND_CLOSE(wxID_ANY, MyFrame::OnFindClose)
END_EVENT_TABLE()
void MyFrame::ShowReplaceDialog( wxCommandEvent& event )
{
    if ( m_dlgReplace )
    {
        delete m_dlgReplace;
        m_dlgReplace = NULL;
    }
    else
    {
        m_dlgReplace = new wxFindReplaceDialog
                           (
                            this,
                            &m_findData,
                            wxT("Find and replace dialog"),
                            wxFR_REPLACEDIALOG
                           );
        m_dlgReplace->Show(true);
    }
}
void MyFrame::OnFind(wxFindDialogEvent& event)
{
    if (!DoFind(event.GetFindString(), event.GetFlags()))
    {
        wxMessageBox(wxT("No more matches."));
    }
}
void MyFrame::OnReplace(wxFindDialogEvent& event)
{
    if (!DoReplace(event.GetFindString(), event.GetReplaceString(),
        event.GetFlags(), REPLACE_THIS))
    {
        wxMessageBox(wxT("No more matches."));
    }
}
void MyFrame::OnReplaceAll(wxFindDialogEvent& event)
{
    if (DoReplace(event.GetFindString(), event.GetReplaceString(),
        event.GetFlags(), REPLACE_ALL))
    {
        wxMessageBox(wxT("Replacements made."));
    }
    else
    {
        wxMessageBox(wxT("No replacements made."));
    }
}
void MyFrame::OnFindClose(wxFindDialogEvent& event)
{
    m_dlgReplace->Destroy();
    m_dlgReplace = NULL;
} 
```

# 8.5 打印对话框

你可以使用的打印对话框包括 wxPageSetupDialog 和 wxPrintDialog 以用来打印文档. 不过,如果你使用 wxWidgets 的打印框架(包括 wxPrintout, wxPrinter 以及其它一些类)的话,你的代码中很少需要显式的调用这些对话框.更多关于打印的细节请参考第五章, "绘画和打印."

wxPageSetupDialog

wxPageSetupDialog 包含一些用来设置纸张大小(A4 或者信纸大小等),打印方向(横向或者纵向),以毫米为单位的边框大小等控件还包括一个用来调用另外一个更详细打印设置对话框的按钮.

下图演示了 wxPageSetupDialog 对话框在 windows 系统上的样子:

![](img/mht65D2%281%29.tmp)

而下面的这个图则是 wxWidgets 自己实现的使用 GTK+的通用的打印设置对话框:

![](img/mht65E5%281%29.tmp)

如果使用了 Gnome 的打印库,则 GTK 版本中的打印设置对话框将使用 Gnome 的原生对话框,如下图所示:

![](img/mht65E8%281%29.tmp)

Mac 版本的打印对话设置对话框图下图所示:

![](img/mht65FA%281%29.tmp)

要创建一个打印设置对话框,你需要传递的参数包括一个父窗口和一个指向 wxPageSetupDialogData 对象的指针,后者用于指定以及从对话框获取打印设置相关的数据,你可以以局部变量或者全局指针的方式创建打印设置对话框.构造函数中的打印设置数据将会被拷贝到打印设置对话框的内部数据中去,而 GetPageSetupData 函数可以用来获取一个打印设置对话框内部数据的引用.

wxPageSetupData 成员函数

Ok 函数在其内部数据有效的情况下返回 True,在 windows 平台上,如果没有设置默认的打印机,这个函数可能返回 False,在其它平台上,这个函数总是返回 True.

SetMarginTopLeft 函数使用 wxPoint 类型的参数以毫米为单位指定打印页面的左边距和上边距. GetMarginTopLeft 则用来获取这两个边距.

SetMarginBottomRight 函数使用 wxPoint 类型的参数以毫米为单位设置打印页面的右边距和下边距. GetMarginBottomRight 则用来获取这两个边距.

SetPaperId 使用标识符来代替具体的大小来设置页面大小. 参考这个函数在手册中的描述来获取相关的标识符. GetPaperId 则用来获取当前设置的标识符.

SetPaperSize 采用一个 wxSize 参数来设置以毫米为单位的页面大小. GetPaperSize 则用来获取对应的设置.

EnableMargins 允许或者禁用对话框上的边界控制控件(仅适用于 Windows). GetEnableMargins 用来获取这个设置的值.

EnableOrientation 用来允许或者禁止对话框上的打印方向控制控件(仅适用于 Windows). GetEnableOrientation 用来获取这个设置的值.

EnablePaper 用来允许或者禁止对话框上选择纸张大小的控件(仅适用于 Windows). GetEnablePaper 用来获取这个设置的值.

EnablePrinter 用来允许或者禁止打印机按钮,这个按钮用来调用另外一个打印设置对话框. GetEnablePrinter 用来获取这个设置的值.

wxPageSetupDialog 使用举例

```cpp
#include "wx/printdlg.h"
void MyFrame::OnPageSetup(wxCommandEvent& event)
{
    wxPageSetupDialog pageSetupDialog(this, & m_pageSetupData);
    if (pageSetupDialog.ShowModal() == wxID_OK)
        m_pageSetupData = pageSetupDialog.GetPageSetupData();
} 
```

wxPrintDialog

这个对话框用来显式打印及打印设置的标准对话框,当这个对话框关闭的时候你可以从中获得一个 wxPrinterDC 对象的实例.

下图演示了这个对话框在 windows 系统上的外观:

![](img/mht65FD%281%29.tmp)

下面的两幅图分别演示了 GTK 版本中没有使用 Gnome 打印库和使用了 GNome 打印库两种情况下对应的这个对话框的样子:

![](img/mht6610%281%29.tmp)

![](img/mht6613%281%29.tmp)

下图则演示了 Mac OSX 上对应的样子,从图中可以看到,Mac 在标准对话框中提供了预览以及存为 PDF 文件的选项,你可以直接使用这个预览功能.

![](img/mht6625%281%29.tmp)

要创建一个 wxPrintDialog 对象,你需要提供的参数包括一个父窗口指针,一个 wxPrintDialogData 对象的指针, 后者的内容将被拷贝给打印对话框内部的对象.如果你希望显示一个打印设置对话框来代替打印对话框,你可以以 True 为参数调用 wxPrintDialogData:: SetSetupDialog 函数,然后再将其传递给 wxPrintDialog 的构造函数.按照微软的说法,虽然打印设置对话框已经被 wxPageSetupDialog 所取代,但是一些老的程序可能还在使用以前的标准,因此,这样作可以保证兼容性.

打印对话框被成功关闭的时候,你可以使用 GetPrintDialogData 函数来获取一个 wxPrintDialogData 的引用.

调用对话框的 GetPrintDC 函数来获取一个基于用户选项的打印设备上下文,如果这个函数的返回值不为空,应用程序应该自己释放这个被返回的对象.

Ok 函数在打印对话框内部数据有效的时候返回 True,在 windows 平台上,如果没有设置默认的打印机,则 Ok 返回 False,在其它平台上,这个函数总是返回 True.

wxPrintDialogData 的成员函数

EnableHelp 允许或者禁止对话框上的帮助按钮. GetEnableHelp 用来获取对应的设置.

EnablePageNumbers 允许或者禁止页码设置控件, GetEnablePageNumbers 用来获取对应的设置.

EnablePrintToFile 允许或者禁止打印到文件按钮. GetEnablePrintToFile 用来获取对应的设置.

EnableSelection 允许或者禁止用于给用户选择打印范围的单选框. GetEnableSelection 用来获取这个设置的值.

SetCollate 用来设置 Collate 复选框的值为 True 或者 false. GetCollate 来获取这个复选框的值.

SetFromPage 和 SetToPage 用来设置打印的起始页和终至页. 使用 GetFromPage 和 GetToPage 来获取相应的值.

SetMinPage 和 SetMaxPage 用来设置可以打印的最小页数和最大页数. GetMinPage 和 GetMaxPage 则用来获取相应的值.

SetNoCopies 用来设置默认打印份数. GetNoCopies 用来获取当前设置的打印份数.

SetPrintToFile 设置打印到文件的复选框的值. GetPrintToFile 则用来获取这个值的当前设定.

SetSelection 用来设置打印范围单选框选项. GetSelection 则用来返回这个选项的值.

SetSetupDialog 用来指示显示打印设置对话框还是打印对话框. GetSetupDialog 来获取这个设置.

SetPrintData 设置内部的 wxPrintData 对象. GetPrintData 则用来返回内部的 wxPrintData 对象的一个引用.

wxPrintDialog 使用举例

下面的例子演示了怎样显示一个打印对话框以便获取对应的打印上下文:

```cpp
#include "wx/printdlg.h"
void MyFrame::OnPrint(wxCommandEvent& event)
{
    wxPrintDialogData dialogData;
    dialogData.SetFromPage(0);
    dialogData.SetToPage(10);
    wxPrintDialog printDialog(this, & m_dialogData);
    if (printDialog.ShowModal() == wxID_OK)
    {
        // 在调用 GetPrintDC()以后, 应用程序
        // 负责管理这个设备上下文
        wxDC* dc = printDialog.GetPrintDC();
        // 在这个设备上下文上绘画
        ...
        // 然后释放它
        delete dc;
    }
} 
```

不过,通常你不需要自己直接调用打印对话框 .你应该使用 wxWidgets 提供的高层打印框架(参考第五章).在你调用 wxPrinter::Print 函数的时候将会自动显示打印对话框.

# 第八章小结

在这一章里,你学习了怎样使用标准对话框,以便通过很少的代码来向你的用户提供信息或者从用户那里获得输入.关于更多使用标准对话框的例子,你可以参考 wxWidgets 自带的 samples/dialogs 中的例子.在下一章中,我们会介绍一下怎样创建你自己的对话框.