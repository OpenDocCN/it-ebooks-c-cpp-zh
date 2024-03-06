# 第七章使用布局控件进行窗口布局

# 第七章使用布局控件进行窗口布局

所有的图形界面设计者都会告诉你,人们对于可以看见的对象的排列是非常敏感的.一个好的 GUI 设计框架一定要允许创建具有吸引力的窗口布局.但是和打印布局不同,应用程序的窗口可能由于大小,字体设置甚至是语言的不同进行不同的变化.对于一个平台无关的设计来说,还要考虑同一个控件在不同的平台上可能有不同的外观和尺寸.这意味着使用绝对大小和位置来进行窗口布局几乎是行不通的.本章描述的 wxWidgets 的布局控件,让你可以灵活的进行非常复杂的窗口布局.如果这听上去有一点让人畏惧,那么记住有一些工具可以帮助你以图形化的方式使用布局控件创建布局,比如包含在附赠光盘中的 DialogBlocks 工具,使用它们你几乎用不着手动设置任何布局.

# 7.1 窗口布局基础

# 7.1 窗口布局基础

在深入介绍窗口布局之前,我们先来大概了解一下什么时候你需要进行窗口布局以及你拥有的几种选择.

一个最简单的情况是你拥有一个 frame 窗口,其中只有一个客户窗口位于这个 frame 的客户区.这是最简单的情况,wxWidgets 将为你完成所有的布局工作,将那个客户区窗口缩放到刚好适合 frame 的客户区的大小.wxWidgets 也会管理 frame 的菜单条,工具条和状态条(如果有的话).当然,如果你想使用两个工具条,你至少要管理其中的一条,而如果 frame 窗口的客户区拥有超过一个窗口,你将不得不自己单独管理它们所有的大小和位置,比如你可能需要在 OnSize 事件中计算每一个窗口的大小并且设置它们的新位置.当然,你也可以使用窗口布局控件.类似的,如果你创建了一个定制的控件,这个控件拥有多个子窗口,你也需要安排这些子窗口的布局以便当你的这个控件被别人使用而且默认大小改变的时候,对那些子窗口进行合理的布局.

另外,大多数应用程序都需要创建自己的对话框,有时候有些程序会创建很多个定制的对话框,这些对话框可能被改变大小,在这种情况下,对话框上所有的控件的大小也应该发生相应的改变,以便即使这个对话框已经比最初设计的时候大了很多,这些控件看上去也不会显得很奇怪.另外应用程序的语言也可能改变,某些在默认语言中很短的标签,在另外一种语言中可能会变得很长.如果你的应用程序要应付成百上千中这种对话框,相信即使使用窗口布局控件,维护它们也是一个令人望而生畏的工作,还好幸运的是我们还可以使用一些工具让所有这些事情变得不那么恐怖甚至还有一点点的乐趣在其中.

如果你选择使用布局控件,你需要自己决定怎样创建和发布它们.你可以自己写或者使用工具来创建 C++或者其它语言的代码,或者你可以直接使用 XRC 文件,XRC 文件用来将布局的定义保存在一个 Xml 文件中,可以被应用程序动态加载,也可以通过 wxrc 工具将其编译成 C++源文件以便和别的源文件编译成一个单独的可执行文件.大多数的 wxWidgets 对话框编辑器都既支持产生 C++的代码,又支持生成 XRC 文件.怎样作决定完全取决于你自己的审美观,有些人喜欢把一切都放在一个 C++代码中以保持足够的灵活性,而另外一些人则喜欢把实现功能的代码和产生界面的代码分开.

接下来的小节用来描述布局控件的原理,再往后一个小节则用来描述怎样使用各种具体的布局控件来编程.

# 7.2 窗口布局控件

# 7.2 窗口布局控件

wxWidgets 使用的窗口布局控件的算法和其它 GUI 程序开发框架的算法,例如 Java 的 AWT,GTK+以及 Qt 等是非常相似的.它们都是基于这样的一个假设,那就是每一个窗口可以报告它们自己需要的最小尺寸以及当它们父窗口的大小发生改变的时候它们的可伸缩能力.通常这也意味着程序代码中没有给对话框中的控件设定固定的大小,取而代之的是设置了一个窗口布局控件,这个窗口布局控件会被寻问它最需要的合适大小,而窗口布局控件则会一次询问它内部的那些窗口,空白区域,控件以及其它的窗口布局控件最合适的大小,以此类推.要注意布局控件并非是 wxWindow 的派生类,因此不具有 TAB 顺序,所需要的系统开销也比一个真实的窗口要小的多.布局控件建立的是一种包含继承关系,在一个复杂的对话框里,这种包含继承关系的层级可能会很深,但是所有这些布局控件中的窗口或控件在窗口继承关系中可能都是兄弟控件,它们都以这个对话框作为自己的父窗口.

在对话框编辑软件中,布局控件的包含继承关系以一种更直观的图形化方式表示.下图演示了 Anthemion 软件公司的 DialogBlocks 软件编辑一个个人对话框时候的样子,这个对话框我们会在第九章,"创建自己的对话框"中作为一个例子.一个红色的方框围绕在当前选择的控件上,而蓝色的方框则用来指示其直接父容器的范围,左边显示的是对话框的容器继承关系树,当然所有这些控件还是有它们自己的窗口继承关系的,正如我们前面提到的那样,窗口继承关系树和容器继承关系树有很大的区别.

![](img/mht3736%281%29.tmp)

为了更清楚的说明容器继承关系和窗口继承关系的不同,我们用下图来大概的表示上图中的容器继承关系.下图中,阴影部分代表实际的窗口,而白色区域则代表布局控件.可以看出,对话框首先使用了两个垂直布局控件,以便在对话框周围释放出一个合理的边界区域,里面的那个垂直布局控件中有两个水平布局控件和一些其它的控件,其中一个水平布局控件中还定义了一截空白区域以便使得其中一个控件远离同组中另外的控件.正象你看到的那样,使用布局控件就象使用一堆大小不等的硬纸片进行摆放,然后把各个窗口控件放置到硬纸片的合适的位置.当然这个比喻并不完全贴切因为,硬纸片的大小是不会伸缩的.

![](img/mht3749%281%29.tmp)

目前为止 wxWidgets 总共支持五类布局控件,每一中布局控件或者用来实现一种特殊的布局方式,或者用来实现和布局相关的一种特殊的功能比如在某些控件周围围绕一个静态的文本框.接下来的小节我们会对它们进行一一的介绍.

布局控件的通用特性

所有的布局控件都是容器,这就是说,它们都是用来容纳一个或多个别的窗口或者元素的,不论每个单独的布局控件怎样排放它们的子元素,所有的子元素都必然有下面这些通用的特性.

最小大小: 布局控件中的每个元素都有计算自己的最小大小的能力(这往往是通过每个元素的 DoGetBestSize 函数计算出来的).这是这个元素的自然大小.举例来说,一个复选框的自然大小等于其复选框图形的大小加上其标签的最合适大小.当然,你可以给某个控件在其构造函数中指定固定的大小,并且在把它增加到布局控件中时指定 wxFIXED_MINSIZE 以改变自动计算的最小大小.需要注意的是,不是每个控件都可以计算自己的大小,对于类似列表框这样的控件来说,你必须清晰的指明它的大小,因为它们没有自然大小.另外一些控件则只拥有自然高度不拥有自然宽度,比如一个单行的文本框.下图演示了当对话框中只有一个控件的时候,以上三种控件怎样扩展对话框以适合自己的最小大小.

![](img/mht375C%281%29.tmp)

![](img/mht375F%281%29.tmp)

![](img/mht3762%281%29.tmp)

边界: 每个元素都应该有一个边界.所谓边界指的是用来和别的元素分开的空白区域,边界的最小大小必须被显式的指定,一般设置为 5 个象素.下图演示了对话框只有一个按钮控件但是指定了 0,5 和 10 作为最小边界值的时候的样子.

![](img/mht3774%281%29.tmp)

![](img/mht3777%281%29.tmp)

![](img/mht378A%281%29.tmp)

对齐方式: 每个元素都可以被以居中或者对齐某个边的方式放置.下图演示了一个水平的布局控件,在其中增加了一个列表框,一个和三个按钮,其中第一个按钮以居中方式增加,第二个则为上对齐,第三个则为下对齐方式.对齐既可以是水平方向的也可以是垂直方向的,但是对于大多数布局控件来说,只有一个方向是有效的.比如对于水平布局控件来说,只有垂直方向是有效的,因为水平方向的空间是被所有的子元素分割的,因此设置水平对齐方式是没有意义的(当然,为了达到水平对齐的效果,我们可能需要插入一个水平方向的空白区域,关于这点我们不作太多的说明).

![](img/mht378D%281%29.tmp)

伸缩因子: 如果一个布局控件的空间大于它所有子元素所需要的空间,那么我们需要一个机制来分割多余的空间.为了实现这个目的,布局控件中的每一个元素都可以指定一个伸缩因子,如果这个因子设置为默认值 0,那么子元素将保持其原本的大小,大于 0 的值用来指定这个子元素可以分割的多余空间的比例,因此如果两个子元素的伸缩因子为 1,其它子元素的伸缩机制为 0,那么这两个子元素将会各占用多余空间的 50%的大小,下图演示了一个对话框有三个按钮它们的初始大小和其中一个的伸缩因子设为 1 以后各自的大小.

![](img/mht3790%281%29.tmp)

![](img/mht37A3%281%29.tmp)

注意在 wxWidgets 的手册中,有时不使用伸缩因子(stretch factor)这个词,而用比例(proportion)这个词表示相同的含义.

# 7.3 使用布局控件进行编程

# 7.3 使用布局控件进行编程

现在我们开始使用布局控件进行窗口布局,首先,创建一个顶层的布局控件(任何类型的布局控件都可以),使用 wxWindow::SetSizer 函数将它和你的顶层窗口绑定.现在你可以在这个顶层布局控件中放置你的窗口或者其它控件元素了.如果你想让你的顶层窗口的大小适合所有控件所需要的大小,你可以调用 wxSizer::Fit 函数,将那个顶层窗口的指针作为其参数.想要顶层窗口在以后的执行过程中尺寸永远不小于初始尺寸,可以使用 wxSizer:: SetSizeHints 函数,将顶层窗口的指针作为参数,这将使得 wxWindow::SetSizeHints 函数以合适的参数被调用.

除了使用上面介绍的方法依次调用三个函数以外,你还可以直接通过调用 wxWindow::SetSizerAndFit 函数来达到同样的效果,这会使得上面的三个函数依次被调用.

如果在你的 frame 窗口里使用了 panel,你可能不知道到底该给 frame 还是 panel 指定布局控件.这个问题应该这样看,如果你的 frame 窗口中只有一个 panel,所有其它的窗口和控件都是 panel 的子窗口,那么 wxWidgets 已经知道怎样将这个 panel 以合适的大小和位置放置在 frame 上了,因此你只需要给 panel 绑定一个布局控件,以便其可以对所有 panel 的子窗口进行布局.而如果你的 frame 窗口中有多个 panel,那么首先你不得不为 frame 绑定一个布局控件以便对 panel 进行布局,然后针对每个 panel 还应该绑定一个布局控件,以便对 panel 中的子窗口进行布局.

接下来的小节里,我们来依次描述一下每一种布局控件类型以及使用它们的方法:

使用 wxBoxSizer 进行编程

wxBoxSizer 可以将它的容器子元素进行横向或者纵向的排列(具体的排列方式在构造函数中指定).如果采用横向排列的方法,则子元素在纵向上可以指定居中,顶部对齐,底部对齐,如果采用纵向排列的方法,子元素在横向上可以指定居中,左对齐或者右对齐的方式.前一小节提到过的缩放因子用来指示在主要方向上的缩放,比如对于横向排列来说,缩放因子指的就是在横向上子元素的缩放比例. 下图演示了上一小节最后一幅图采用纵向排列的样子.

![](img/mhtDB34%281%29.tmp)

你可以使用 wxBoxSizer 的 Add 方法增加一个子元素:

```cpp
// 增加一个窗口
void Add(wxWindow* window, int stretch = 0, int flags = 0,
         int border = 0);
// 增加一个布局控件
void Add(wxSizer* window, int stretch = 0, int flags = 0,
         int border = 0); 
```

第一个参数是要增加的窗口或者布局控件的指针

第二个参数是前面说过的缩放因子

第三个参数是一个比特位列表,用来指示新增的子元素的对齐和边界的行为.对齐比特位用来指示当垂直排列的布局控件的宽度发生改变时子元素的水平对齐方式,或者是水平排列的布局控件的高度改变时子元素的垂直对齐方式,默认的值为 wxALIGN_LEFT | wxALIGN_TOP,可选的值列举在下表中:

| 0 | 子元素保留原始大小. |
| --- | --- |
| wxGROW | 子元素随这布局控件一起改变大小. 等同于 wxEXPAND. |
| wxSHAPED | 子元素保持原有比例按缩放因子缩放. |
| wxALIGN_LEFT | 左对齐. |
| wxALIGN_RIGHT | 右对齐. |
| wxALIGN_TOP | 顶端对齐. |
| wxALIGN_BOTTOM | 底部对齐. |
| wxALIGN_CENTER_HORIZONTAL | 水平居中. |
| wxALIGN_CENTER_VERTICAL | 垂直居中. |
| wxALIGN_CENTER | 水平或者垂直居中. wxALIGN_CENTER_HORIZONTAL &#124; wxALIGN_CENTER_VERTICAL. |
| wxLEFT | 边界间隔位于子元素左面. |
| wxRIGHT | 边界间隔位于子元素右面. |
| wxTOP | 边界间隔位于子元素上面. |
| wxBOTTOM | 边界间隔位于子元素下面. |
| wxALL | 边界间隔位于子元素四周.wxLEFT &#124; wxRIGHT &#124; wxTOP &#124; wxBOTTOM. |

第四个参数指定边界间隔的大小

当然你也可以直接增加一段空白,下面演示了增加空白区域的几种方法:

```cpp
// 增加一段空白 (旧方法)
void Add(int width, int height, int stretch = 0, int flags = 0,
         int border = 0);
// 增加一段固定大小的空白
void AddSpacer(int size);
// 增加一个可缩放的空白
void AddStretchSpacer(int stretch = 1); 
```

上面第二种方法相当于调用 Add(size, size, 0),第三种则相当于调用 Add(0, 0, stretch).

我们来举这样一个例子,一个对话框包含一个多行文本框和两个位于底端的按钮.我们可以以这样的角度去看待这些窗口,首先是一个顶层的垂直布局,包含一个多行文本框和一个底层的子布局控件,这个子布局控件是一个水平的布局控件,它包含一个 OK 按钮,被放置在左面,和一个 Cancel 按钮,被放置在右面.当对话框的大小发生变化的时候,我们希望多行文本框随着对话框大小的变化而变化,而按钮则保持它们原来的大小,并且在水平方向上居中排列,如下图所示:

![](img/mhtDB37%281%29.tmp)

下面列出了实现上述对话框所使用的代码:

```cpp
MyDialog::MyDialog(wxWindow *parent, wxWindowID id,
                     const wxString &title )
        :  wxDialog(parent, id, title,
                     wxDefaultPosition, wxDefaultSize,
                     wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER)
{
    wxBoxSizer  *topSizer = new wxBoxSizer( wxVERTICAL );
    // 创建一个最小大小为 100x60 的多行文本框
    topSizer->Add(
        new wxTextCtrl( this, wxID_ANY, "My text.",
            wxDefaultPosition, wxSize(100,60), wxTE_MULTILINE),
        1,         // 垂直方向可缩放,缩放因子为 1
        wxEXPAND|  // 水平方向可缩放
        wxALL,     // 四周都由空白边框
        10 );      // 空白边框大小为 10
    wxBoxSizer *buttonSizer = new wxBoxSizer( wxHORIZONTAL );
    buttonSizer->Add(
       new wxButton( this, wxID_OK, "OK" ),
       0,          // 水平方向不可缩放
       wxALL,     // 四周有空白边框:(注意默认为顶部对齐)
       10 );      // 空白边框大小为 10
    buttonSizer->Add(
       new wxButton( this, wxID_CANCEL, "Cancel" ),
       0,         // 水平方向不可缩放
       wxALL,     // 四周有空白边框:(注意默认为顶部对齐)
       10 );      // 空白边框大小为 10
    topSizer->Add(
       buttonSizer,
       0,                // 垂直方向不可缩放
       wxALIGN_CENTER ); // 无边框并且居中对齐
    SetSizer( topSizer ); // 绑定对话框和布局控件
    topSizer->Fit( this );          // 调用对话框大小
    topSizer->SetSizeHints( this ); // 设置对话框最小大小
} 
```

使用 wxStaticBoxSizer 编程

wxStaticBoxSizer 是一个继承自 wxBoxSizer 的布局控件,除了实现 wxBoxSizer 的功能,另外还在整个布局的范围以外增加了一个静态的边框 wxStaticBox,这个 wxStaticBox 需要手动创建并且在 wxStaticBoxSizer 的构造函数中作为参数传入,Add 函数和 wxBoxSizer 的 Add 函数用法相同.

下图演示了使用 wxStaticBoxSizer 对一个复选框进行布局的样子:

![](img/mhtDB49%281%29.tmp)

对应的代码:

```cpp
MyDialog::MyDialog(wxWindow *parent, wxWindowID id,
                   const wxString &title )
        : wxDialog(parent, id, title,
                   wxDefaultPosition, wxDefaultSize,
                   wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER)
{
    // 创建一个顶层布局控件
    wxBoxSizer* topLevel = new wxBoxSizer(wxVERTICAL);
    // 创建静态文本框和静态文本框布局控件
    wxStaticBox* staticBox = new wxStaticBox(this,
        wxID_ANY, wxT("General settings"));
    wxStaticBoxSizer* staticSizer = new wxStaticBoxSizer(staticBox,
        wxVERTICAL);
    topLevel->Add(staticSizer, 0,
        wxALIGN_CENTER_HORIZONTAL|wxALL, 5);
    // 在其中增加一个复选框
    wxCheckBox* checkBox = new wxCheckBox( this, ID_CHECKBOX,
        wxT("&Show splash screen"), wxDefaultPosition, wxDefaultSize);
    staticSizer->Add(checkBox, 0, wxALIGN_LEFT |wxALL, 5);
    SetSizer(topLevel);
    topLevel->Fit(this);
    topLevel->SetSizeHints(this);
} 
```

使用 wxGridSizer 编程

wxGridSizer 布局控件可以以二维表的方式排列它的子元素,这个二维表的每个表格的大小都是相同的,都等于最长的那个表格的长度和最高的那个表格的高度.创建一个 wxGridSizer 需要指定它的行数和列数,以及一个额外的行间距和列间距.Add 方法和 wxBoxSizer 的用法相同.

下图演示了一个两行三列的网格布局控件,由于有一个很大的按钮,导致这个格的大小很大,从而导致所有的表格的大小都跟着变大:

![](img/mhtDB4C%281%29.tmp)

代码如下:

```cpp
MyDialog::MyDialog(wxWindow *parent, wxWindowID id,
                   const wxString &title )
        : wxDialog(parent, id, title,
                   wxDefaultPosition, wxDefaultSize,
                   wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER)
{
    // 创建一个顶层网格布局控件
    wxGridSizer* gridSizer = new wxGridSizer(2, 3, 0, 0);
    SetSizer(gridSizer);
    wxButton* button1 = new wxButton(this, ID_BUTTON1, wxT("One"));
    gridSizer->Add(button1, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button2 = new wxButton(this, ID_BUTTON2, wxT("Two (the second button)"));
    gridSizer->Add(button2, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button3 = new wxButton(this, ID_BUTTON3, wxT("Three"));
    gridSizer->Add(button3, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button4 = new wxButton(this, ID_BUTTON4, wxT("Four"));
    gridSizer->Add(button4, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button5 = new wxButton(this, ID_BUTTON5, wxT("Five"));
    gridSizer->Add(button5, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button6 = new wxButton(this, ID_BUTTON6, wxT("Six"));
    gridSizer->Add(button6, 0, wxALIGN_CENTER_HORIZONTAL|
                               wxALIGN_CENTER_VERTICAL|wxALL, 5);
    gridSizer->Fit(this);
    gridSizer->SetSizeHints(this);
} 
```

使用 wxFlexGridSizer 编程

wxFlexGridSizer 同样采用二维表来对其子元素进行布局,和 wxGridSizer 不同的是,它不要求所有的表格的大小都是一样的,只要求同一列上所有表格的宽度是相同的并且同一行上所有表格的高度是相同的,也就是说,行的高度或者列的宽度仅由这一行或者这一列上的子元素决定.另外还可以给行和列指定是否缩放,这意味着当整个布局控件的大小发生变化的时候,可以指定某些行或者列随着整个布局控件的缩放而缩放.

创建一个 wxFlexGridSizer 可以指定行数,列数额外的垂直间距和水平间距.调用 Add 函数的方法和 wxBoxSizer 相同.

下图演示了一个使用 wxFlexGridSizer 进行布局的对话框的样子,正如你看到的那样,和 wxGridSizer 相比整个布局显的更紧凑了,因为中间很宽的那一列不再影响其它列的宽度了.

![](img/mhtDB4C%281%29.tmp)

初始情况下,我们看不出来设置第一列可以改变大小的效果,不过如果我们如下图所示的那样改变这个对话框的水平方向的大小,,我们就可以看到第一列占用了额外增加的空间,并且第一列的子元素也为居中方式显式.

![](img/mhtDB5F%281%29.tmp)

代码如下:

```cpp
MyDialog::MyDialog(wxWindow *parent, wxWindowID id,
                   const wxString &title )
        : wxDialog(parent, id, title,
                   wxDefaultPosition, wxDefaultSize,
                   wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER)
{
    //创建一个复杂网格布局控件
    wxFlexGridSizer* flexGridSizer = new wxFlexGridSizer(2, 3, 0, 0);
    this->SetSizer(flexGridSizer);
    //让第一列可变大小
    flexGridSizer->AddGrowableCol(0);
    wxButton* button1 = new wxButton(this, ID_BUTTON1, wxT("One"));
    flexGridSizer->Add(button1, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button2 = new wxButton(this, ID_BUTTON2, wxT("Two (the second button)"));
    flexGridSizer->Add(button2, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button3 = new wxButton(this, ID_BUTTON3, wxT("Three"));
    flexGridSizer->Add(button3, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button4 = new wxButton(this, ID_BUTTON4, wxT("Four"));
    flexGridSizer->Add(button4, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button5 = new wxButton(this, ID_BUTTON5, wxT("Five"));
    flexGridSizer->Add(button5, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    wxButton* button6 = new wxButton(this, ID_BUTTON6, wxT("Six"));
    flexGridSizer->Add(button6, 0, wxALIGN_CENTER_HORIZONTAL|
                                   wxALIGN_CENTER_VERTICAL|wxALL, 5);
    flexGridSizer->Fit(this);
    flexGridSizer->SetSizeHints(this);
} 
```

使用 wxGridBagSizer 编程

这种布局控件用来模拟现实世界中的那种固定位置和大小的基于布局控件的布局.它将它的子元素按照一个虚拟的网格进行排列,不过子元素的位置是通过 wxGBPosition 对象指定的,对象的大小使用 wxGBSpan 指定,对象的大小不仅限于一个网格.

创建 wxGridBagSizer 的可选参数包括垂直和水平方向的间隔(默认为 0),Add 函数需要提供的参数包括子元素的位置和大小,另外的可选标记和边框大小参数的意义和 wxBoxSizer 是一样的.

下图演示了一个使用 wxGridBagSizer 进行布局的例子,我们指定了其中一个按钮的大小为两个单元列,我们还指定了第二行和第三列的大小是可以变化的,这样当我们改变对话框的大小的时候,就会出现如下面另外一幅图的效果.

![](img/mhtDB62%281%29.tmp)

![](img/mhtDB65%281%29.tmp)

相关代码如下:

```cpp
MyDialog::MyDialog(wxWindow *parent, wxWindowID id,
                   const wxString &title )
        : wxDialog(parent, id, title,
                   wxDefaultPosition, wxDefaultSize,
                   wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER)
{
    wxGridBagSizer* gridBagSizer = new wxGridBagSizer();
    SetTopSizer(gridBagSizer);
    wxButton* b1 = new wxButton(this, wxID_ANY, wxT("One (0,0)"));
    gridBagSizer->Add(b1, wxGBPosition(0, 0));
    wxButton* b2 = new wxButton(this, wxID_ANY, wxT("Two (2,2)"));
    gridBagSizer->Add(b2, wxGBPosition(2, 2), wxGBSpan(1, 2),
                      wxGROW);
    wxButton* b3 = new wxButton(this, wxID_ANY, wxT("Three (3,2)"));
    gridBagSizer->Add(b3, wxGBPosition(3, 2));
    wxButton* b4 = new wxButton(this, wxID_ANY, wxT("Four (3,3)"));
    gridBagSizer->Add(b4, wxGBPosition(3, 3));
    gridBagSizer->AddGrowableRow(3);
    gridBagSizer->AddGrowableCol(2);
    gridBagSizer->Fit(this);
    gridBagSizer->SetSizeHints(this);
} 
```

# 7.4 更多关于布局的话题

# 7.4 更多关于布局的话题

这一节里,我们将讨论一些更深入的话题,在进行窗口布局的时候,你可以在脑子里考虑这些事情.

对话框单位

尽管布局控件可以让基本控件的大小随着平台的不同语言的不同进行相应的改变,但是有些情况下,你还是需要手动指定控件的大小(比如在对话框中增加一个列表框的时候).如果你希望这些手动指定的大小也随着平台的不同字体的不同进行相应的变化,你应该使用对话框单位来代替象素单位.对话框单位是基于应用程序当前字体的字符宽度和高度所取的一个平均值的,因此总能很好的和当前的字体对应.wxWidgets 也提供了相关的转换函数包括: ConvertDialogToPixels,ConvertPixelsToDialog 等,还包括一个宏 wxDLG_UNIT(window, ptOrSz)用来直接将使用对话框单位 wxPoint 对象或者 wxSize 对象转换为象素单位.所以你可以使用下面的代码来指定那些你不得不指定的控件大小:

```cpp
wxListBox* listBox = new wxListBox(parent, wxID_ANY,
    wxDefaultPosition, wxDLG_UNIT(parent, wxSize(60, 20))); 
```

你也可以在 XRC 文件中使用对话框单位,只需要在相应的值前面增加一个"d"字符就可以了.

平台自适应布局

尽管不同平台的对话框的绝大部分都是相同的,但是在风格上确是存在着一些不同.比如在 Windows 和 Linnx 平台上,右对齐或者居中放置的 OK,Cancel 和 Help 按钮都是可以接受的,但是在 Mac OsX 上,Help 按钮通常位于左面,而 Cancel 和 OK 按钮则通常依序位于右面.

要作到这种不同平台上按钮顺序的自适应,你需要使用 wxStdDialogButtonSizer 布局控件,这个控件继承自 wxBoxSizer,因此使用方法并没有太大的不同,只是它依照平台的不同对某些按钮进行特殊的排列.

这个布局控件的构造函数没有参数,要增加按钮可以使用两种方法:传递按钮指针给 AddButton 函数,或者(日过你没有使用标准的标识符的话),使用 SetAffirmativeButton, SetNegativeButton, and SetCancelButton 来设置按钮的特性.如果使用 AddButton,那么按钮应使用下面的这些标识符: wxID_OK, wxID_YES, wxID_CANCEL, wxID_NO, wxID_SAVE, wxID_APPLY, wxID_HELP 和 wxID_CONTEXT_HELP.

然后,在所有的按钮都增加到布局控件以后,调用 Realize 函数以便布局控件调整按钮的顺序,如下面的代码所示:

```cpp
wxBoxSizer* topSizer = new wxBoxSizer(wxVERTICAL);
dialog->SetSizer(topSizer);
wxButton* ok = new wxButton(dialog, wxID_OK);
wxButton* cancel = new wxButton(dialog, wxID_CANCEL);
wxButton* help = new wxButton(dialog, wxID_HELP);
wxStdDialogButtonSizer* buttonSizer = new wxStdDialogButtonSizer;
topSizer->Add(buttonSizer, 0, wxEXPAND|wxALL, 10);
buttonSizer->AddButton(ok);
buttonSizer->AddButton(cancel);
buttonSizer->AddButton(help);
buttonSizer->Realize(); 
```

或者作为一个更方便的手段,你可以使用 wxDialog::CreateButtonSizer 函数,它基于一些按钮标记的列表来自动创建平台自适应的按钮,并将其放在一个布局控件中,如果你查看 src/generic 目录中的对话框代码的实现,你会发现大量的地方使用了 CreateButtonSizer 函数.这个函数支持的按钮标记如下表所示:

| wxYES_NO | 增加 YES 和 No 按钮各一个. |
| --- | --- |
| wxYES | 增加一个标识符为 wxID_YES 的 Yes 按钮. |
| wxNO | 增加一个标识符为 wxID_NO 的 No 按钮. |
| wxNO_DEFAULT | 让 No 按钮作为默认按钮,否则 Yes 或 OK 按钮将成为默认按钮. |
| wxOK | 增加一个标识符为 wxID_OK 的 OK 按钮. |
| wxCANCEL | 增加一个标识符为 wxID_CANCEL 的 Cancel 按钮. |
| wxAPPLY | 增加一个标识符为 wxID_APPLY 的 Apply 按钮. |
| wxHELP | 增加一个标识符为 wxID_HELP 的 Help 按钮. |

使用 CreateButtonSizer 函数,上面例子中的代码可以简化为:

```cpp
wxBoxSizer* topSizer = new wxBoxSizer(wxVERTICAL);
dialog->SetSizer(topSizer);
topSizer->Add(CreateButtonSizer(wxOK|wxCANCEL|wxHELP), 0,
              wxEXPAND|wxALL, 10); 
```

另外一种给不同的平台指定不同布局的方法是在 XRC 文件中指定平台属性.其中的参数部分的值可以通过一个"|"符号加上 unix, win,mac 或者 os2 来指定特定平台上的界面布局.在应用程序运行的时候,XRC 文件将只会创建那些和当前运行平台符合的控件.另外如果没有使用 XRC 的话,DialogBlocks 程序还支持针对不同的平台生成预置条件的 C++代码.

当然你也可以给不同的平台指定不同的 XRC 文件,不过这样作的话维护起来就有点不方便了.

动态布局

有时候你可能需要动态更改对话框的布局,比如你可以会增加一个"Detail"按钮,当这个按钮被按下的时候显式更多的选项,当然你可以使用平常的办法,调用 wxWindow::Show 函数来隐藏某个控件,不过 wxSizer 也提供了一个单独的方法,你可以使用 wxSizer:: Show 函数并且传递 False 参数,以便告诉 wxSizer 不要计算其中的窗口的大小,当然调用这个函数以后,你需要调用 wxSizer:: Layout 函数来强制更新对应的窗口.

# 第七章小结

# 第七章小结

布局控件可能需要花点时间才能慢慢习惯使用,因此,如果你觉得这一章的内容有点沉重,不要担心.掌握它们最好的办法是使用光盘中附带的 DialogBlocks 软件进行各种各样的窗口布局,然后观看其自动产生的代码.你也可以参考 wxWidgets 自带的 samples/layout 中的例子.在你可以熟练使用它们以后,你会发现,它们在不同平台和不同语言下进行布局的能力,可以让你的产品极大的受益.

在下一章中,我们来看一看 wxWidgets 提供的标准对话框.