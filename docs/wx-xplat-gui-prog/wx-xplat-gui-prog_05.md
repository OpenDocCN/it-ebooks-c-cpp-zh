# 第五章绘画和打印

# 第五章绘画和打印

这一章里，我们来介绍一下设备上下文，也就是通常所说的在一个窗口或者一个打印页面上绘画的概念。我们将会讨论目前拥有的设备上下文类，以及 wxWidgets 提供的和字体，颜色，线条以及填充等相关的绘画工具集。接下来我们会描述每个设备上下文的绘画函数以及 wxWidgets 支持打印的机制。在本章的最后，我们会简单讨论一下 wxGLCanvas，这个类提供了一种在你的窗口中使用 OpenGL 技术绘制三维图形的方法。

# 5.1 理解设备上下文

# 5.1 理解设备上下文

在 wxWidgets 中，所有的绘画相关的动作，都是由设备上下文完成的。每一个设备上下文都是 wxDC 的一个派生类。从来就没有直接在窗口上绘画这种事情，每次在窗口上绘画，都要先创建一个窗口绘画设备上下文，然后在这个上下文上绘画。其它一些设备上下文是和 bitmap 图片或者打印机绑定的，你也可以设计自己的设备上下文。这样作一个最大的好处就是，你的绘画的代码是可以共享的，如果你的代码使用一个 wxDC 类型的参数，顶多再增加一个用于缩放的分辨率，那么这段代码就可以被同时用于在窗口绘制和在打印机上绘制。 让我们先来描述一下设备上下文的主要属性。

一个设备上下文拥有一个座标体系，它的原点通常在画布的左上角。不过这个位置可以通过调用 SetDeviceOrigin 函数改变，这样的效果就相当于对随后在这个上下文上的作的画进行平移。这种使用方法在对 wxScrolledWindow 进行绘画的时候非常常见。你还可以调用 SetAxisOrientation 来改变坐标系的放向，比如说你可以让 Y 轴是从下到上的放向而不是默认的从上到下的放向。

逻辑单位和设备单位是有区别的。设备单位是设备本地的单位，对于计算机屏幕来说，设备单位是一个象素，而对于一个打印机来说，设备单位是由它的分辨率决定的，这个分辨率可以用 GetSize（用来取得设备单位的一页的大小）或者 GetSizeMM(用来取得以毫米为单位的一页的大小)来获得。

设备上下文的映射模式定义了逻辑单位到设备单位的转换标准。要注意某些设备上下文（典型的例子比如：wxPostScriptDC）只支持 wxMM_TEXT 映射模式。下表列出了目前支持的映射模式：

| wxMM_TWIPS | 每个逻辑单位为 1/20 点, 或者 1/1440 英寸. |
| --- | --- |
| wxMM_POINTS | 每个逻辑单位为 1 点, 或 1/72 英寸. |
| wxMM_METRIC | 每个逻辑单位为 1 毫米. |
| wxMM_LOMETRIC | 每个逻辑单位为 1/10 毫米. |
| wxMM_TEXT | 每个逻辑单位为 1 象素.这是默认的映射模式. |

你可以通过 SetUserScale 函数给逻辑单位指定一个缩放比例，这个比例乘以映射模式中指定的单位，可以得到实际逻辑单位和设备单位之间的关系，比如在 wxMM_TEXT 映射模式下，如果用户缩放比例为(1.0,1.0)，那么逻辑单位和设备单位就是一样的，这也是设备上下文中映射模式和用户缩放比例的缺省值。

设备上下文还可以通过 SetClippingRegion 函数指定一个区域，这个区域以外的部分将不被显示，可以使用 DestroyClippingRegion 函数清除设备上下文当前指定的区域。举个例子，为了将一个字符串显示的范围限制在某个矩形区域以内，你可以先指定这个矩形区域为这个设备上下文的当前区域，然后调用函数在这个设备上下文上绘制这个字符串，即使这个字符串很长，可能会超出这个矩形区域，超出的部分也将被截掉不被显示，然后再调用函数清除这个区域就可以了。

和现实世界一样，为了绘画，你需要先选择绘画的工具，如果你要画线，那么你需要选择画笔，要填充一个区域，你需要选择画刷，而当前字体，文本前景色和背景色等，则会决定你画布上的文本怎样显示。晚些时候我们会详细讨论这些。我们先来看看目前都支持那些设备上下文：

可用的设备上下文

下面列出了你可以使用的设备上下文:

*   wxClientDC. 用来在一个窗口的客户区绘画。
*   wxBufferedDC. 用来代替 wxClientDC 来进行双缓冲区绘画。
*   wxWindowDC. 用来在窗口的客户区和非客户区（比如标题栏）绘画.这个设备上下文极少使用而且也不是每个平台都支持。
*   wxPaintDC. 仅用在重绘事件的处理函数中，用来在窗口的客户区绘画。
*   wxBufferedPaintDC. 和 wxPaintDC 类似，不过采用双缓冲区进行绘画。
*   wxScreenDC. 用来直接在屏幕上绘画。
*   wxMemoryDC. 用来直接在图片上绘画。
*   wxMetafileDC. 用来创建一个图元文件(只支持 Windows 和 Mac OS X).
*   wxPrinterDC. 用来在打印机上绘画。
*   wxPostScriptDC. 用来在 PostScript 文件上或者在支持 PostScript 的打印机上绘画。

接下来我们会描述一下怎样来创建和使用这些设备上下文。打印机设备上下文将会在本章稍后的“使用打印机的架构”小节中进行更详细的讨论。

使用 wxClientDC 在窗口客户区进行绘画

wxClientDC 用来在非重绘事件处理函数中对窗口的客户区进行绘制。例如，如果你想作一个信手涂鸦的程序，你可以在你的鼠标事件处理函数中创建一个 wxClientDC。这个设备上下文也可以用于擦除背景事件处理函数。

下面的例子演示了怎样使用鼠标在窗口上随便乱画：

```cpp
BEGIN_EVENT_TABLE(MyWindow, wxWindow)
    EVT_MOTION(MyWindow::OnMotion)
END_EVENT_TABLE()
void MyWindow::OnMotion(wxMouseEvent& event)
{
    if (event.Dragging())
    {
        wxClientDC dc(this);
        wxPen pen(*wxRED, 1); // red pen of width 1
        dc.SetPen(pen);
        dc.DrawPoint(event.GetPosition());
        dc.SetPen(wxNullPen);
    }
} 
```

在第十九章，“使用文档和视图”中，实现了一个更具有可用性的涂鸦工具，它使用线段代替了点，并且实现了重做和撤消的操作。而且还存储了所有的线段，以便在 windows 需要重绘的时候重绘所有的线段。而使用上面的代码，你所作的画在下次 windows 重绘的时候将会消失。当然你还可以使用 CaptureMouse 和 ReleaseMouse 函数，以便在鼠标按下的时候，直接把鼠标限制在你的绘画窗口的客户区范围内。

一种替代 wxClientDC 的方法是使用 wxBufferedDC，后者将你的所有绘画的结果保存在内存中，然后当自己被释放的时候一次性把所有的内容传输到窗口上。这样作的结果是使得整个绘画过程更平滑，如果你不希望用户看到绘画的结果一点一点的更新，你可以使用这种方法，使用 wxBufferedDC 和使用 wxClientDC 的代码是完全一样的，另外为了提高效率，你可以在 wxBufferedDC 的构造函数中传递一个已经创建好的 bitmap 对象，以避免在每次创建 wxBufferedDC 的时候创建一个新的图片。

擦除窗口背景

窗口类通常会收到两种和绘画相关的事件：wxPaintEvent 事件用来绘制窗口客户区的主要图形，而 wxEraseEvent 事件则用来通知应用程序擦除背景。如果你只拦截并处理了 wxPaintEvent 事件，则默认的 wxEraseEvent 事件处理函数会用最近一次的 wxWindow::SetBackgroundColour 函数调用设置的颜色或者别的合适的颜色清除整个背景。

这也许看上去有些混乱，但是这种把背景和前景分开的方法可以更好的支持那些使用这种架构的平台（比如 windows）上的控件。举个例子，如果你希望在一个窗口上使用某种纹路的背景，如果你在 OnPaint 函数中是平铺纹理贴图，你可能会看到一些闪烁，因为在绘画之前，系统进行了清除背景的动作，在这种背景和前景分离的架构下，你可以拦截 wxEraseEvent 事件，然后在其处理函数中什么事情都不做，或者你干脆在擦除背景事件处理函数中平铺你的背景图，而在前景绘制事件的处理函数中绘制前景（当然，这样也会带来一些双缓冲绘画方面的问题，我们接下来会谈到）.

在某些平台上。仅仅是拦截 wxEraseEvent 事件仍然不足以阻止所有的系统默认的重绘动作，让你的窗口的背景不只是一个纯色背景的最安全的方法是调用 wxWindow::SetBackgroundStyle 然后传递 wxBG_STYLE_CUSTOM 参数。这将告诉 wxWidgets 把所有背景重绘的动作留给应用程序自己处理。

如果你真的决定实现自己的背景擦除事件处理函数，你可以先调用 wxEraseEvent::GetDC 来尝试返回一个已经创建的设备上下文，如果返回的值为空，再创建一个 wxClientDC 设备上下文。这将允许 wxWidgets 不在擦除背景事件中固定创建一个设备上下文，象前面例子中提到的那样，擦除背景事件处理函数中可能根本不会用到设备上下文，因此在事件中创建一个设备上下文可能是不必要的。下面的代码演示了怎样在擦除背景事件使用平铺图片的方法绘制窗口背景：

```cpp
BEGIN_EVENT_TABLE(MyWindow, wxWindow)
  EVT_ERASE_BACKGROUND(MyWindow::OnErase)
END_EVENT_TABLE()
void MyWindow::OnErase(wxEraseEvent& event)
{
    wxClientDC* clientDC = NULL;
    if (!event.GetDC())
        clientDC = new wxClientDC(this);
    wxDC* dc = clientDC ? clientDC : event.GetDC() ;
    wxSize sz = GetClientSize();
    wxEffects effects;
    effects.TileBitmap(wxRect(0, 0, sz.x, sz.y), *dc, m_bitmap);
    if (clientDC)
        delete clientDC;
} 
```

和重绘窗口事件一样，wxEraseEvent::GetDC 返回的设备上下文已经设置了区域，使得只有需要重绘的部分才会被重绘。

使用 wxPaintDC 在窗口上绘画

如果你定义了一个窗口重画事件处理函数，则必须在这个处理函数中产生一个 wxPaintDC 设备上下文（即使你根本不使用它），并且使用它来进行你需要的绘画动作。产生这个对象将告诉 wxWidgets 的窗口体系这个窗口的需要重画的区域已经被重画了，这样窗口系统就不会重复的发送重画消息给这个窗口了。在重画事件中，你还可以调用 wxWindow::GetUpdateRegion 函数来获得需要重画的区域，或者使用 wxWindow:: IsExposed 函数来判断某个点或者某个矩形区域是否需要重画，然后优化代码使得仅在这个范围内的内容被重画，虽然在重画事件中创建的 wxPaintDC 设备上下文会自动将自己限制在需要重画的区域内，不过你自己知道需要重画的区域的话，可以对代码进行相应的优化。

重画事件是由于用户和窗口系统的交互造成的，但是它也可以通过调用 wxWindow::Refresh 和 wxWindow:: RefreshRect 函数手动产生。如果你准确的知道窗口的哪个部分需要重画，你可以指定只重画那一部分区域以便尽可能的减少闪烁。这样作的一个问题是，并不能保证窗口在调用 Refresh 函数以后会马上重画。如果你真的需要立刻调用你的重画事件处理函数，比如说你在进行一个很耗时的计算，需要即时显示一些进度，你可以在调用 Refresh 或者 RefreshRect 以后调用 wxWindow::Update 函数。

下面的代码演示了如何在窗口正中位置画一个黑边红色的矩形区域，并且会判断这个区域是否位于需要更新的区域范围内以便决定是否需要重画。

```cpp
BEGIN_EVENT_TABLE(MyWindow, wxWindow)
  EVT_PAINT(MyWindow::OnPaint)
END_EVENT_TABLE()
void MyWindow::OnPaint(wxPaintEvent& event)
{
    wxPaintDC dc(this);
    dc.SetPen(*wxBLACK_PEN);
    dc.SetBrush(*wxRED_BRUSH);
    // 获取窗口大小
    wxSize sz = GetClientSize();
    // 要绘制的矩形的大小
    wxCoord w = 100, h = 50;
    // 将我们的矩形设置在窗口正中，
    // 但是不为负数的位置
    int x = wxMax(0, (sz.xw)/2);
    int y = wxMax(0, (sz.yh)/2);
    wxRect rectToDraw(x, y, w, h);
    // 只有在需要的时候才重画以便提高效率
    if (IsExposed(rectToDraw))
        DrawRectangle(rectToDraw);
} 
```

注意在默认情况下，当窗口大小改变时，只有那些需要重画的地方才会被更新，指定 wxFULL_REPAINT_ON_RESIZE 窗口类型可以覆盖这种默认情况以使得整个窗口都被刷新。在我们上面的例子中，就需要指定这种情况，因为我们矩形的位置是根据窗口大小计算出来的，如果窗口变大而我们只更新需要更新的部位，则可能在原来的窗口中留下半个矩形或者在屏幕上出现两个矩形，这和我们的初衷是不一致的。

wxBufferedPaintDC 是 wxPaintDC 的双缓冲区版本。只需要简单的将重绘事件处理函数中的 wxPaintDC 换成 wxBufferedPaintDC 就可以了，它会首先将所有的图片画在一个内存位图上，然后在自己被释放的时候一次性将其画在窗口上以减小闪烁。

正象我们前面提到的那样，另外一个减少闪烁的方法，是把背景和前景统一在窗口重画事件处理函数中，而不是将它们分开处理，配合 wxBufferedPaintDC，那么所有的绘画动作在完成之前都是在内存中进行的，这样在窗口被重绘之前你将看不到窗口背景被更新。你需要增加一个空的背景擦除事件处理函数，并且使用 SetBackgroundStyle 函数设置背景类型为 wxBG_STYLE_CUSTOM 以便告诉某些系统不要自动擦除背景。在 wxScrolledWindow 中你还有注意需要对绘画设备上下文的原点进行平移，并据此重新计算你自己的图片位置。（译者注：在 wxScrolledWindow 窗口中的 wxPaintDC 的原点是当前窗口滚动位置下的原点，这通常不是我们所需要的，因为我们绘画通常要基于整个滚动窗口本身的原点，调用 PrepareDC 函数可以将其设置成滚动窗口本身的原点，在绘画的时候可以通过 CalcUnscrolledPosition 函数将当前客户区中的某个点转换成相对于整个滚动窗口区的座标，如下面的代码演示的那样）下面的代码演示了怎样在一个继承自 wxScrolledWindow 的窗口类中进行绘画并且尽可能的避免出现闪烁。

```cpp
#include "wx/dcbuffer.h"
BEGIN_EVENT_TABLE(MyCustomCtrl, wxScrolledWindow)
    EVT_PAINT(MyCustomCtrl::OnPaint)
    EVT_ERASE_BACKGROUND(MyCustomCtrl::OnEraseBackground)
END_EVENT_TABLE()
//重画事件处理函数
void MyCustomCtrl::OnPaint(wxPaintEvent& event)
{
    wxBufferedPaintDC dc(this);
    // 平移设备座标以便我们不需要关心当前滚动窗口的位置
    PrepareDC(dc);
    // 在重画绘制函数中绘制背景
    PaintBackground(dc);
    // 然后绘制前景
    ...
}
/// 绘制背景
void MyCustomCtrl::PaintBackground(wxDC& dc)
{
    wxColour backgroundColour = GetBackgroundColour();
    if (!backgroundColour.Ok())
        backgroundColour =
            wxSystemSettings::GetColour(wxSYS_COLOUR_3DFACE);
    dc.SetBrush(wxBrush(backgroundColour));
    dc.SetPen(wxPen(backgroundColour, 1));
    wxRect windowRect(wxPoint(0, 0), GetClientSize());   
    //我们需要平移当前客户区矩形的座标以便将其转换成相对于整个滚动窗口而不是当前窗口的座标
    //因为在前面我们已经对设备上下文进行了 PrepareDC 的动作。
    CalcUnscrolledPosition(windowRect.x, windowRect.y,
                           & windowRect.x, & windowRect.y);
    dc.DrawRectangle(windowRect);
}
// 空函数 只为了防止闪烁
void MyCustomCtrl::OnEraseBackground(wxEraseEvent& event)
{
} 
```

为了提高性能，当你使用 wxBufferedPaintDC 时，你可以维护一个足够大的（比如屏幕大小的）位图，然后将其传递给 wxBufferedPaintDC 的构造函数作为第二个参数，这可是避免每次使用 wxBufferedPaintDC 的时候创建和释放一个位图。

wxBufferedPaintDC 通常会从其缓冲区中拷贝整个客户区（用户可见部分）大小，在滚动窗口中，其内部创建的设备上下文并不会随着 PrepareDC 的调用平移其座标系。不过你可以通过在 wxBufferedPaintDC 的构造函数中指定 wxBUFFER_VIRTUAL_AREA（默认为 wxBUFFER_CLIENT_AREA）参数来避免这一问题。不过这种情况下，你需要提供一个和整个滚动窗口的虚拟大小一样的缓冲区，而这通常效率是很低的，应该尽量避免。另外一个需要注意的是对于设置为 wxBUFFER_CLIENT_AREA 的 wxBufferedPaintDC 到目前为止还不支持缩放（SetUserScale）。

你可以在随书光盘的 examples/chap12/thumbnail 例子的 wxThumbnailCtrl 控件中，找到使用 wxBufferedPaintDC 的完整的例子。

使用 wxMemoryDC 在位图上绘图

内存设备上下文是和一个位图绑定的设备上下文，在这个设备上下文上的所有绘画都将画在那个位图上面。使用的方法是先使用默认的构造函数创建一个内存设备上下文，然后使用 SelectObject 函数将其和一个位图绑定，在完成所有的绘画以后再调用 SelectObject 函数参数为 wxNullBitmap 来移除绑定的位图，代码如下所示：

```cpp
wxBitmap CreateRedOutlineBitmap()
{
    wxMemoryDC memDC;
    wxBitmap bitmap(200, 200);
    memDC.SelectObject(bitmap);
    memDC.SetBackground(*wxWHITE_BRUSH);
    memDC.Clear();
    memDC.SetPen(*wxRED_PEN);
    memDC.SetBrush(*wxTRANSPARENT_BRUSH);
    memDC.DrawRectangle(wxRect(10, 10, 100, 100));
    memDC.SelectObject(wxNullBitmap);
    return bitmap;
} 
```

你也可以使用 Blit 函数将内存设备上下文中的某一个区域拷贝到别的设备上下文上，在本章稍后的地方我们会对此进行介绍。

使用 wxMetafileDC 创建图元文件

wxMetafileDC 适用于 Windows 和 Mac OS X，它在这两个平台上分别提供了绘制 Windows 图元文件和 Mac PICT 的画布。允许在 wxMetafile 对象上绘画，这个对象保留了一组绘画动作记录，可以被其它应用程序使用或者通过 wxMetafile:: Play 函数将其绘制在别的设备上下文上。

使用 wxScreenDC 访问屏幕

使用 wxScreenDC 可以在整个屏幕的任何位置绘画。通常这在给拖放操作提供可见响应的时候比较有用（比如拖放分割窗口的分割条的时候）。处于性能方面的考虑。你可以将其操作的屏幕区域限制在一个矩形区域（通常是程序窗口所在的区域）内。当然，除了在屏幕上绘画，我们还可以把绘画从屏幕上拷贝到其它设备上下文中，以便实现屏幕捕获。因为无法限制别的应用程序的行为，所以 wxScreenDC 类通常在当前应用程序的窗口内工作的最好。

下面是将屏幕捕获到位图文件的例子：

```cpp
wxBitmap GetScreenShot()
{
    wxSize screenSize = wxGetDisplaySize();
    wxBitmap bitmap(screenSize.x, screenSize.y);
    wxScreenDC dc;
    wxMemoryDC memDC;
    memDC.SelectObject(bitmap);
    memDC.Blit(0, 0, screenSize.x, screenSize.y, & dc, 0, 0);
    memDC.SelectObject(wxNullBitmap);
    return bitmap;
} 
```

使用 wxPrinterDC 和 wxPostScriptDC 实现打印

wxPrinterDC 用来实现打印机接口。在 windows 和 Mac 平台上，这个接口实现到标准打印接口的映射。在其它类 Unix 系统上，没有标准的打印接口，因此需要使用 wxPostScriptDC 代替（除非打开了 Gnome 打印支持，参考接下来的小节，“在类 Unix 系统上使用 GTK+进行打印”）。

可以通过多种途径创建 wxPrinterDC 对象，你还可以传递设置了纸张类型，横向或者纵向，打印份数等参数的 wxPrintData 对象给它。一个简单的创建 wxPrinterDC 设备上下文的方法是显示一个 wxPrintDialog 对话框，在用户选择各种参数以后，使用 wxPrintDialog::GetPrintDC 的方法获取一个对应的 wxPrinterDC 对象。作为更高级的用法，你可以从 wxPrintout 定义一个自己的派生类，以便更精确定义打印以及打印预览的行为，然后把它的一个实例传递给 wxPrinter 对象（在后面小节中还会详细介绍）。

如果你要打印的内容主要是文本，你可以考虑使用 wxHtmlEasyPrinting 类，以便忽略 wxPrinterDC 和 wxPrintout 排版的细节：你只需要按照 wxWidgets'实现的那些 HTML 的语法编写 HTML 文件，然后创建一个 wxHtmlEasyPrinting 对象用来实现打印和预览，这通常可以节省你几天到几周的时候来对那些文本，表格和图片进行排版。详情请参考第十二章，“高级窗口类”。

wxPostScriptDC 用来打印到 PostScript 文件。虽然这种方式主要应用在类 Unix 系统上，不过在别的系统上你一样可以使用它。用这种方式打印需要先产生 PostScript 文件，而且还要保证你拥有一个支持打印 PostScript 文件的打印机。

你可以通过传递 wxPrintData 参数，或者传递一个文件名，一个 bool 值以确定是否显示一个打印设置对话框和一个父窗口来创建一个 wxPostScriptDC，如下所示：

```cpp
#include "wx/dcps.h"
wxPostScriptDC dc(wxT("output.ps"), true, wxGetApp().GetTopWindow());
if (dc.Ok())
{
    // 告诉它在哪里找到 AFM 字体文件。
    dc.GetPrintData().SetFontMetricPath(wxGetApp().GetFontPath());
    // 设置分辨率（每英寸多少个点，默认 720）
    dc.SetResolution(1440);
    // 开始绘画
    ...
} 
```

wxPostScriptDC 的一个特殊的地方在于你不能直接使用 GetTextExtent 来获取文本大小的信息。你必须先用 wxPrintData::SetFontMetricPath 指定 AFM（Adobe Font Metric）文件的路径，就象上面例子中的那样。你可以从下面的路径下载 GhostScript AFM 文件。

ftp://biolpc22.york.ac.uk/pub/support/gs_afm.tar.gz

# 5.2 绘画工具

# 5.2 绘画工具

在 wxWidgets 中，绘画操作就象是一个技术非常高超的艺术家，快速的选择颜色，绘画工具，然后画场景的一小部分，然后换不同的颜色，不同的工具，再绘画场景的其它部分，周而反复的操作。因此，接下来我们来介绍一下 wxColour,wxPen, wxBrush, wxFont 和 wxPalette 这些绘画工具。还有其它的一些内容也是有帮助的，比如 wxRect，wxRegion，wxPoint 和 wxSize，我们会在第十三章，�数据结构类�中对它们进行介绍。

注意这些类使用了�引用记数�的方法，使用内部指针以避免大块的内存拷贝，在大多数情况下，你可以直接以局部变量的方式定义颜色，画笔，画刷和字体对象而不用担心性能。如果你的程序确实因此而拥有性能上的问题，你才需要考虑采取一些方法来提高性能，比如将其中的一些局部变量改变成类的成员。

wxColour

你可以使用 wxColour 类来定义各种各样的颜色。（因为 wxWidgets 开始于爱丁堡，它的 API 使用英式拼写，不过对于那些对拼写很敏感的人，wxWidgets 还是给 wxColor 定义了一个别名 wxColour）。

你可以使用 SetTextForeground 和 SetTextBackground 函数来定义一个设备上下文中文本的颜色，也可以使用 wxColour 来创建画笔和刷子。

wxColour 对象有很多种创建方法，你可以使用 RGB 三元色的值（0 到 255）来构建 wxColour，或者通过一个标准的字符串，比如 WHITE 或者 CYAN，或者从另外一个 wxColour 对象创建。或者你还可以直接使用系统预定的颜色对象指针： wxBLACK, wxWHITE, wxRED, wxBLUE, wxGREEN, wxCYAN,和 wxLIGHT_GREY.还有一个 wxNullColour 对象用来代表未初始化的颜色，它的 Ok 函数总是返回 False。

使用 wxSystemSettings 类可以获取很多系统默认的颜色，比如 3D 表面颜色，默认的窗口背景颜色，菜单文本颜色等等。请参考相关文档中 wxSystemSettings::GetColour 的部分来获取详细的列表。

下面的例子演示了创建 wxColour 的方法：

```cpp
wxColour color(0, 255, 0); // green
wxColour color(wxT("RED")); // red
// 使用面板的三维表面系统颜色
wxColour color(wxSystemSettings::GetColour(wxSYS_COLOUR_3DFACE)); 
```

wxTheColourDatabase 指针用来在系统之类的颜色和颜色名之间建立映射，通过颜色名寻找对应的颜色对象或者通过颜色对象来寻找对应的颜色名，如下所示：

```cpp
wxTheColourDatabase->Add(wxT("PINKISH"), wxColour(234, 184, 184));
wxString name = wxTheColourDatabase->FindName(
                                         wxColour(234, 184, 184));
wxString color = wxTheColourDatabase->Find(name); 
```

下面列出了目前支持的标准颜色：aquamarine, black, blue, blue violet, brown, cadet blue, coral, cornflower blue, cyan, dark gray, dark green, dark olive green, dark orchid, dark slate blue, dark slate gray dark turquoise, dim gray, firebrick, forest green, gold, goldenrod, gray, green, green yellow, indian red, khaki, light blue, light gray, light steel blue, lime green, magenta, maroon, medium aquamarine, medium blue, medium forest green, medium goldenrod, medium orchid, medium sea green, medium slate blue, medium spring green, medium turquoise, medium violet red, midnight blue, navy, orange, orange red, orchid, pale green, pink, plum, purple, red, salmon, sea green, sienna, sky blue, slate blue, spring green, steel blue, tan, thistle, turquoise, violet, violet red, wheat, white, yellow, 和 yellow green.

wxPen

你可以使用 SetPen 函数指定一个设备上下文使用的画笔（wxPen）。画笔指定了随后的绘画操作中线条的颜色，粗细以及线条类型。wxPen 的开销很小，你可以放心的在你的绘图代码中创建局部变量类型的画笔对象而不用对它们进行全局存储。

下表列出了目前支持的画笔线条类型，其中 Hatch 和 stipple 类型目前的 GTK+版本不支持：

| 线形 | 示例 | 描述 |
| --- | --- | --- |
| wxSOLID | ![](img/mht8198%281%29.tmp) | 纯色线. |
| wxTRANSPARENT | 透明颜色. |
| wxDOT | ![](img/mht819B%281%29.tmp) | 纯点线. |
| wxLONG_DASH | ![](img/mht81AE%281%29.tmp) | 长虚线. |
| wxSHORT_DASH | ![](img/mht81B1%281%29.tmp) | 短虚线.在 windows 平台上等同于 wxLONG_SASH. |
| wxDOT_DASH | ![](img/mht81B4%281%29.tmp) | 短线和点间隔线. |
| wxSTIPPLE | ![](img/mht81C6%281%29.tmp) | 使用一个位图代替点的点虚线,这个位图是其构造函数的第一个参数 |
| wxUSER_DASH | ![](img/mht81C9%281%29.tmp) | 自定义虚线. 参考用户手册. |
| wxBDIAGONAL_HATCH | ![](img/mht81CC%281%29.tmp) | 反斜线虚线. |
| wxCROSSDIAG_HATCH | ![](img/mht81DF%281%29.tmp) | 交叉虚线. |
| wxFDIAGONAL_HATCH | ![](img/mht81E2%281%29.tmp) | 斜线虚线. |
| wxCROSS_HATCH | ![](img/mht81F4%281%29.tmp) | 十字虚线. |
| wxHORIZONTAL_HATCH | ![](img/mht81F7%281%29.tmp) | 水平线段虚线. |
| wxVERTICAL_HATCH | ![](img/mht820A%281%29.tmp) | 垂直线段虚线. |

使用 SetCap 定义粗线条的末端的样子：wxCAP_ROUND 是默认的设置，只是粗线条的末端应该使用圆形，wxCAP_PROJECTING 则只是使用方形并且有一个凸起，wxCAP_BUTT 则只是直接使用方形。

使用 SetJoin 函数来设置当有线段相连时候的联结方式，默认的值是 wxJOIN_ROUND，这种情况下转角是圆形的，其它可选的值还有 wxJOIN_BEVEL 和 wxJOIN_MITER.

你也可以直接使用预定的画笔对象：wxRED_PEN, wxCYAN_PEN, wxGREEN_PEN, wxBLACK_PEN, wxWHITE_PEN, wxtrANSPARENT_PEN, wxBLACK_DASHED_PEN, wxGREY_PEN, wxMEDIUM_GREY_PEN 和 wxLIGHT_GREY_PEN.这些都是指针，所以在 SetPen 函数中使用的时候，应该使用"*"号指向它们的实例。还有一个预定义的对象（不是指针）wxNullPen，可以用来复位设备上下文中的画笔。

下面是创建画笔的一些演示代码,都用来产生一个纯红色的画笔：

```cpp
wxPen pen(wxColour(255, 0, 0), 1, wxSOLID);
wxPen pen(wxT("RED"), 1, wxSOLID);
wxPen pen = (*wxRED_PEN);
wxPen pen(*wxRED_PEN); 
```

上面例子中的最后两行使用了引用记数的方法，实际上内部指向同一个对象。这种引用记数的方法在绘画对象中很常用，它使得对象赋值和拷贝的系统开销非常小，不过同时它意味着一个对象的改变将会影响到其它所有使用同一个引用的对象。

一个既可以简化画笔对象的创建和释放过程，又不需要将画笔对象存储在自己的对象中的方法，是使用全局指针 wxThePenList 来创建和存储所有你需要的画笔对象，如下所示：

```cpp
wxPen* pen = wxThePenList->FindOrCreatePen(*wxRED, 1, wxSOLID); 
```

这个 wxThePenList 指向的对象将负责存储所有的画笔对象并且在应用程序退出的时候自动释放所有的画笔。很显然，你应该小心不要过量使用这个对象以免画笔对象占用大量的系统内存，而且也要注意前面我们提到过的使用引用对象的问题，你可以使用 RemovePen 函数从 wxThePenList 中删除一个画笔但是却不释放它所占的内存。

wxBrush

设备上下文当前使用的画刷对象可以用 SetBrush 函数指定，它决定设备上下文中图像的填充方式。你也可以使用它来定义设备上下文的默认背景，这样的定义方式可以使得背景不只是简单的纯色。和画笔对象一样，画刷对象的系统消耗也非常小，你可以直接使用局部变量的方式定义它。

画刷的构造函数采用一个颜色参数和一个画刷类型参数，如下表所示：

| 画刷类型 | 例子 | 描述 |
| --- | --- | --- |
| wxSOLID | ![](img/mht820D%281%29.tmp) | 纯色画刷. |
| wxTRANSPARENT | 透明画刷. |
| wxBDIAGONAL_HATCH | ![](img/mht8210%281%29.tmp) | 反斜线画刷. |
| wxCROSSDIAG_HATCH | ![](img/mht8223%281%29.tmp) | 交叉画刷. |
| wxFDIAGONAL_HATCH | ![](img/mht8226%281%29.tmp) | 斜线画刷. |
| wxCROSS_HATCH | ![](img/mht8238%281%29.tmp) | 十字画刷. |
| wxHORIZONTAL_HATCH | ![](img/mht823B%281%29.tmp) | 水平线画刷. |
| wxVERTICAL_HATCH | ![](img/mht823E%281%29.tmp) | 垂直线画刷. |
| wxSTIPPLE | ![](img/mht8251%281%29.tmp) | 位图画刷, 其位图在构造函数中指定. |

你也可以直接使用下面这些系统预定义的画刷：wxBLUE_BRUSH, wxGREEN_BRUSH, wxWHITE BRUSH, wxBLACK_BRUSH, wxGREY_BRUSH, wxMEDIUM_GREY_BRUSH, wxLIGHT_GREY_BRUSH, wxtrANSPARENT_BRUSH, wxCYAN_BRUSH 和 wxRED_BRUSH. 这些都是指针，类似的还有 wxNullBrush 用来复位设备上下文的画刷。

下面是创建红色纯色画刷的例子：

```cpp
wxBrush brush(wxColour(255, 0, 0), wxSOLID);
wxBrush brush(wxT("RED"), wxSOLID);
wxBrush brush = (*wxRED_BRUSH); // a cheap operation
wxBrush brush(*wxRED_BRUSH); 
```

和画笔一样，画刷也有一个用来保存列表的全局指针指针： wxTheBrushList，你可以象下面这样使用它：

```cpp
wxBrush* brush = wxTheBrushList->FindOrCreateBrush(*wxBLUE, wxSOLID); 
```

同样要避免在应用程序过量使用以及要注意引用记数使用的问题。使用 RemoveBrush 来从 wxTheBrushList 中移除一个画刷而不释放其内存。

wxFont

你可以使用字体对象来设置一个设备上下文使用的字体，字体对象有下面一些属性：

字体大小用来以点（1/72 英寸）为单位指定字体中的最大高度。wxWidgets 会选择系统中最接近的字体。

字体家族用来指定一个家族系列，象下表中描述的那样，指定一个字体家族而不指定一个字体的名字是为了移植的方便，因为你不大可能要求某个字体的名字存在于所有的平台。

| 字体家族标识符 | 例子 | 描述 |
| --- | --- | --- |
| wxFONTFAMILY_SWISS | ![](img/mht8254%281%29.tmp) | 非印刷字体，依平台的不同通常是 Helvetica 或 Arial. |
| wxFONTFAMILY_ROMAN | ![](img/mht8267%281%29.tmp) | 一种正规的印刷字体. |
| wxFONTFAMILY_SCRIPT | ![](img/mht826A%281%29.tmp) | 一种艺术字体. |
| wxFONTFAMILY_MODERN | ![](img/mht827C%281%29.tmp) | 一种等宽字体.通常是 Courier |
| wxFONTFAMILY_DECORATIVE | ![](img/mht827F%281%29.tmp) | 一种装饰字体. |
| wxFONTFAMILY_DEFAULT | wxWidgets 选择一个默认的字体家族. |

字体类型可以是 wxNORMAL, wxSLANT 或 wxITALIC。其中 wxSLANT 可能不是所有的平台或者所有的字体都支持。

weight 属性的值则可以是 wxNORMAL, wxLIGHT 或 wxBOLD.

下划线可以被设置或者关闭。

字体名属性是可选参数，用来指定一个特定的字体，如果其为空，则将使用指定字体家族默认的字体。

可选的编码方式参数用来指定字体编码和程序用设备上下文绘画的文本的编码方式的映射，详情请参考第十六章，�编写国际化应用程序�。

你可以使用默认的构造函数创建一个字体，或者使用上表中列出的字体家族创建一个字体。

也可以使用下面这些系统预定义的字体： wxNORMAL_FONT, wxSMALL_FONT, wxITALIC_FONT 和 wxSWISS_FONT. 除了 wxSMALL_FONT 以外，其它的字体都使用同样大小的系统默认字体(wxSYS_DEFAULT_GUI_FONT), 而 wxSMALL_FONT 则比另外的三个字体小两个点.你可以使用 wxSystemSettings::GetFont 来获取当前系统的默认字体。

要使用字体，你需要在进行任何字体相关的操作（比如 DrawText 和 GetTextExtent）之前，使用 wxDC::SetFont 函数设置字体。

下面的代码演示了怎样构造一个字体对象：

```cpp
wxFont font(12, wxFONTFAMILY_ROMAN, wxITALIC, wxBOLD, false);
wxFont font(10, wxFONTFAMILY_SWISS, wxNORMAL, wxBOLD, true,
            wxT("Arial"), wxFONTENCODING_ISO8859_1));
wxFont font(wxSystemSettings::GetFont(wxSYS_DEFAULT_GUI_FONT)); 
```

和画笔和画刷一样，字体也有一个全局列表指针 wxTheFontList，用来查找以前创建的字体或者创建一个新的字体：

```cpp
wxFont* font = wxTheFontList->FindOrCreateFont(12, wxSWISS,
                                               wxNORMAL, wxNORMAL); 
```

同样的，避免大量使用这个全局指针，因为其分配的内存要到程序退出的时候才会释放。你可以使用 RemoveFont 从中移除一个字体但是不释放相关的内存。

在本章晚些时候我们会看到一些使用文本和字体的例子。同时你也可以看一下字体例子程序，它允许你选择一个字体然后看看某些文本是什么样子，也可以让你更改字体的大小和别的属性。

![](img/mht8292%281%29.tmp)

wxPalette

调色板是一个表，这个表的大小通常是 256,表中的每一个索引被映射到一个对应的 rgb 颜色值。通常在不同的应用程序需要在同一个显示设备上共享确定数目的颜色的时候使用。通过设置一个调色板，应用程序之间的颜色可以取得一个平衡。调色板也被用来把一个低颜色深度的图片映射到当前可用的颜色，因此每个 wxBitmap 都有一个可选的调色板。

因为现在大多数电脑的显示设备都支持真彩色，调色板已经很少使用了。应用程序定义的 RGB 颜色可以直接被显示设备映射到最接近的颜色。

创建一个调色板需要提供一个调色板大小参数和三个分别代表红绿蓝三种颜色的数组。你可以通过 GetColoursCount 函数得到当前调色板中条目的数量。GetRGB 函数通过索引找到其代替的颜色的 RGB 值，而 GetPixel 则通过 RGB 值得到其相应的索引。

使用 wxDC::SetPalette 函数给某个设备上下文指定一个调色板。比如，你可以给当前的设备上下文指定一个位于某个低颜色深度的 wxBitmap 对象中的调色板，一边让设备上下文知道怎样把这个图片中的索引颜色映射到真实的 RGB 颜色。当在一个指定了调色板的设备上下文中使用 wxColour 绘画的时候，系统会自动在设备上下文的调色板中查找最匹配的颜色的索引，因此你应该指定一个和你要用的颜色最接近的调色板。

wxPalette 的另外一个用法是用来查询一个低颜色深度图像文件（比如 GIF）中的图像的不同的颜色。如果这个图像拥有一个调色板，即时这个图片已经被转换成 RGB 格式，区分图像中的不同颜色也仍然是个很容易的事。类似的，通过调色板，你也可以把一个真彩色的图片转换成低颜色深度的图片，下面的代码演示了怎样将一个真彩色的 PNG 文件转换成 8bit 的 windows 位图文件：

```cpp
// 加载这个 PNG 文件
wxImage image(wxT("image.png"), wxBITMAP_TYPE_PNG);
// 创建一个调色板
unsigned char* red = new unsigned char[256];
unsigned char* green = new unsigned char[256];
unsigned char* blue = new unsigned char[256];
for (size_t i = 0; i &lt; 256; i ++)
{
    red[i] = green[i] = blue[i] = i;
}
wxPalette palette(256, red, green, blue);
// 设置调色板和颜色深度
image.SetPalette(palette);
image.SetOption(wxIMAGE_OPTION_BMP_FORMAT, wxBMP_8BPP_PALETTE);
// 存储文件
image.SaveFile(wxT("image.bmp"), wxBITMAP_TYPE_BMP); 
```

降低颜色深度的更实用的方法请参考第十章，�在程序中使用图片�中的�降低颜色深度�小节，介绍怎样用 wxQuantize 类来作这件事。

wxWidgets 定义了一个空的调色板对象 wxNullPalette.

(译者注：这一小节翻译的太费劲了)

# 5.3 设备上下文中的绘画函数

# 5.3 设备上下文中的绘画函数

在这一小节里，我们来近距离的了解一下怎样在设备上下文中绘画。下表列出了设备上下文类主要的成员函数，在接下来的小节里，我们会举例介绍其中的大部分函数，你也可以从 wxWidgets 的手册中获取它们的详细信息。

| Blit | 把某个设备上下文的一部分拷贝到另外一个设备上下文上. 你可以指定的参数包括拷贝多大，拷贝到什么位置,拷贝的逻辑函数以及如果源设备上下文是内存设备上下文的时候，是不是使用透明遮罩等。 |
| --- | --- |
| Clear | 使用当前的背景刷刷新背景. |
| SetClippingRegion DestroyClippingRegion GetClippingBox | 用来设置和释放设备上下文使用的区域。区域是用来将设备上下文中所有的绘画操作限制在某个范围内的，它可以是一个矩形，也可以是由 wxRegion 指定的复杂区域，使用 GetClippingBox 函数来取得包含当前区域的一个矩形范围。 |
| DrawArc DrawEllipticArc | 使用当前的画笔和画刷画一段圆弧或者椭圆弧线. |
| DrawBitmap DrawIcon | 在指定位置画一副图片或者是一个图标.如果是图片可以指定一个透明遮罩. |
| DrawCircle | 使用当前的画笔和画刷画一个圆形. |
| DrawEllipse | 使用当前的画笔和画刷画一个椭圆形. |
| DrawLine DrawLines | 使用当前的画笔画一条线或者多条线。最后的那个点是不被显示的。 |
| DrawPoint | 使用当前设置的画笔画一个点. |
| DrawPolygon DrawPolyPolygon | DrawPolygon 函数通过指定一个点的数组或者指向点结构的指针的列表来绘制一个封闭的多边形，还可以指定一个可选的座标平移。wxWidgets 会自动封闭第一个点和最后一个点，所以你不必在最开始和最末端指定同一个点。 DrawPolyPolygon 函数则同时绘制多个多边形，在某些平台上这比多次调用 DrawPolygon 函数更有效率。 |
| DrawRectangle DrawRoundedRectangle | 使用当前的画笔和画刷绘制一个矩形或者圆角矩形 |
| DrawText DrawRotatedText | 使用当前字体，文本前景色和文本背景色在指定的位置绘制一段文本或者一段旋转的文本。 |
| DrawSpline | 使用当前的画笔在指定的控制点下使用云行规画一条平滑曲线. |
| FloodFill | 以 Flood 填充的方式使用当前的画刷对指定起始点的范围进行填充（比如：封闭相邻同颜色区域中的颜色将被替换）. |
| Ok | 如果设备已经准备好可以开始绘画则返回 true. |
| SetBackground GetBackground | 用来设置或者获取背景画刷设置。这些设置将在 Clear 函数或者其它一些设置了复杂的绘画逻辑参数的函数中被使用。默认设置为 wxtrANSPARENT_BRUSH。 |
| SetBackgroundMode GetBackgroundMode | 用来设置绘制文本时候的背景类型，取值为 wxSOLID 或者 wxTRANSPARENT。通常你希望设置为后者，以便保留绘制文本区域已经存在的背景。 |
| SetBrush GetBrush | 用来设置当前画刷，默认值未定义。 |
| SetPen GetPen | 用来设置当前画笔，默认值未定义。 |
| SetFont GetFont | 用来设置当前字体，默认值未定义。 |
| SetPalette GetPalette | 用来设置当前调色板。 |
| SetTextForeground GetTextForeground SetTextBackground GetTextBackground | 用来设置文本的前景颜色和背景颜色，默认值前景为黑色背景为白色。 |
| SetLogicalFunction GetLogicalFunction | 逻辑函数用来设置画笔或者画刷或者（在 Blit 函数中）设备上下文自己的象素的显示和传输方式。默认值 wxCOPY 表示直接显示或者拷贝当前颜色。 |
| GetPixel | 返回某个点的颜色.在 wxPostScriptDC 和 wxMetafileDC 中还没有实现这个功能. |
| GetTextExtent GetPartialTextExtents | 返回给定文本的大小。 |
| GetSize GetSizeMM | 返回以设备单位或者毫米指定的长宽。 |
| StartDoc EndDoc | 这两个函数只在打印设备上下文中使用，前者显示一条消息表明正在打印，后者则隐藏这个消息。 |
| StartPage EndPage | 在打印设备上下文中开始和结束一页。 |
| DeviceToLogicalX DeviceToLogicalXRel DeviceToLogicalY DeviceToLogicalYRel | 将设备座标转换成逻辑座标，可以是绝对值也可以是相对值。 |
| LogicalToDeviceX LogicalToDeviceXRel LogicalToDeviceY LogicalToDeviceYRel | 将逻辑座标转换成设备座标。 |
| SetMapMode GetMapMode | 象前面描述过的那样，MapMode 用来（和 SetUserScale 一起）指定逻辑单位到设备单位的映射。 |
| SetAxisOrientation | 用来指定 X 轴和 Y 轴的方向。默认 X 轴为从左到右（True），Y 轴为从上到下（False）。 |
| SetDeviceOrigin GetDeviceOrigin | 设置座标原点，可以用来实现平移。 |
| SetUserScale GetUserScale | 设置缩放值。该值用于逻辑单位到设备单位的转换。 |

绘制文本

设备上下文绘制文本的方式取决于以下几个参数：当前字体，字体背景模式，文本背景色和文本前景色。如果背景模式是 wxSOLID，文本背后的部分将会以当前的文本背景色擦除，如果是 wxTRANSPARENT，则文本的背景将保留原先的背景。

传递给 DrawText 的参数是一个字符串和一个点（或者两个整数）。其中点（或者两个整数）指定的位置将会是文本最左上角的位置。下面是一个例子：

```cpp
void DrawTextString(wxDC& dc, const wxString& text,
                    const wxPoint& pt)
{
    wxFont font(12, wxFONTFAMILY_SWISS, wxNORMAL, wxBOLD);
    dc.SetFont(font);
    dc.SetBackgroundMode(wxTRANSPARENT);
    dc.SetTextForeground(*wxBLACK);
    dc.SetTextBackground(*wxWHITE);
    dc.DrawText(text, pt);
} 
```

你也可以使用 DrawRotatedText 函数来绘制一段旋转的文本，其中角度的值由函数的最后一个参数指定，下面的代码演示了一段以 45 度角增加的文本:

```cpp
wxFont font(20, wxFONTFAMILY_SWISS, wxNORMAL, wxNORMAL);
dc.SetFont(font);
dc.SetTextForeground(wxBLACK);
dc.SetBackgroundMode(wxTRANSPARENT);
for (int angle = 0; angle &lt; 360; angle += 45)
    dc.DrawRotatedText(wxT("Rotated text..."), 300, 300, angle); 
```

运行结果如下图所示：

![](img/mht30B0%281%29.tmp)

在 Windows 平台上，只有 TrueType 类型的字体才可以实现旋转输出，要注意 wxNORMAL_FONT 指定的字体并不是 TrueType 字体。

通常情况下，你需要知道当前正要绘制的文本将占用设备上下文中多大的地方，你可以通过 GetTextExtent 函数来作到这一点，它的原型如下：

```cpp
void GetTextExtent(const wxString& string,
    wxCoord* width, wxCoord* height,
    wxCoord* descent = NULL, wxCoord* externalLeading = NULL,
    wxFont* font = NULL); 
```

从这个函数的原型中可以看出，后面的三个参数是可选的，其中 descent 参数和 externalLeading 参数（译者注：在汉字里面用处不大）的含义是：英文字符的基线通常不是字符的最底端，descent 用来获取某个字符的最底端到基线的距离，而 externalLeading 则用来获取 descent 到下一行顶端的距离。最后一个参数 font 可以用来指定以这个字体为基准（而不是设备上下文自己的字体）来获取测量值。

下面的代码把一段文本在窗口的中间位置显示：

```cpp
void CenterText(const wxString& text, wxDC& dc, wxWindow* win)
{
    dc.SetFont(*wxNORMAL_FONT);
    dc.SetBackgroundMode(wxTRANSPARENT);
    dc.SetTextForeground(*wxRED);
    // 获取窗口大小和文本大小
    wxSize sz = win->GetClientSize();
    wxCoord w, h;
    dc.GetTextExtent(text, & w, & h);
    // 计算为了居中显示需要的文本开始位置
    // 并保证其不为负数.
    int x = wxMax(0, (sz.x - w)/2);
    int y = wxMax(0, (sz.y - h)/2);
    dc.DrawText(msg, x, y);
} 
```

如果你需要知道每个字符的精确占用大小，你可以使用 GetPartialTextExtents 函数，它使用 wxString 和一个 wxArrayInt 的引用作为参数。这个函数的效率在某些平台上优于对每个单个的字符使用 GetTextExtent 函数。

绘制线段和形状

这里用到的函数原型包括那些用来画点，画线，矩形，圆形以及椭圆等的函数。它们都使用当前的画笔设置和画刷设置，画笔用来决定轮廓线的颜色和模式，画刷用来决定填充的颜色和方式：

下面演示了一段代码和这段代码产生的图形：

```cpp
void DrawSimpleShapes(wxDC& dc)
{
    // 设置黑色的轮廓线，绿色的填充色
    dc.SetPen(wxPen(*wxBLACK, 2, wxSOLID));
    dc.SetBrush(wxBrush(*wxGREEN, wxSOLID));

    // 画点
    dc.DrawPoint(5, 5);

    // 画线
    dc.DrawLine(10, 10, 100, 100);

    // 画矩形
    dc.SetBrush(wxBrush(*wxBLACK, wxCROSS_HATCH));
    dc.DrawRectangle(50, 50, 150, 100);

    // 改成红色画刷
    dc.SetBrush(*wxRED_BRUSH);

    // 画圆角矩形
    dc.DrawRoundedRectangle(150, 20, 100, 50, 10);

    // 没有轮廓线的圆角矩形
    dc.SetPen(*wxTRANSPARENT_PEN);
    dc.SetBrush(wxBrush(*wxBLUE));
    dc.DrawRoundedRectangle(250, 80, 100, 50, 10);

    // 改变颜色
    dc.SetPen(wxPen(*wxBLACK, 2, wxSOLID));
    dc.SetBrush(*wxBLACK);

    // 画圆
    dc.DrawCircle(100, 150, 60);

    // 再次改变画刷颜色
    dc.SetBrush(*wxWHITE);

    // 画一个椭圆
    dc.DrawEllipse(wxRect(120, 120, 150, 50));
} 
```

![](img/mht30B3%281%29.tmp)

注意一个约定俗成的规矩，线上的最后一个点将不会绘制。

要绘制一段圆弧，需要使用 DrawArc 函数，提供一个起点，一个终点和一个圆心的位置。这段圆弧将以逆时针方向从起点画至终点，举例如下：

```cpp
int x = 10, y = 200, radius = 20;
dc.DrawArc(xradius, y, x + radius, y, x, y); 
```

![](img/mht30C6%281%29.tmp)

绘制椭圆弧的函数 DrawEllipticArc 采用一个容纳这个椭圆的矩形的四个顶点，以及椭圆弧开始和结束的角度作为参数，如果开始和结束的角度相同，则将绘制一个完整的椭圆。

```cpp
// 绘制一个包含在顶点为(10, 100),
// 大小为 200x40\. 圆弧角度从 270 到 420 度的圆弧.
dc.DrawEllipticArc(10, 100, 200, 40, 270, 420); 
```

![](img/mht30C9%281%29.tmp)

如果你需要快速绘制很多条线段，那么 DrawLines 将会比多次调用 DrawLine 拥有更高的效率，下面的例子演示了快速绘制 10 个点之间的线段，并且指定了一个（100,100）的偏移量：

```cpp
wxPoint points[10];
for (size_t i = 0; i &lt; 10; i++)
{
  pt.x = i*10; pt.y = i*20;
}
int offsetX = 100;
int offsetY = 100;
dc.DrawLines(10, points, offsetX, offsetY); 
```

DrawLines 不会填充线段环绕的区域。如果你想在画线的同时填充其环绕区域，你需要使用 DrawPolygon 函数，它的参数包括点的个数，点的列表，可选的平移参数以及填充类型。填充类型默认为 wxODDEVEN_RULE，也可以使用 wxWINDING_RULE。而 DrawPolygonPolygon 用来同时绘制多个 Polygon，它的额外的一个参数是另外一个整数的数组，用来指定在前面的点的数组中每个 Polygon 中点的个数。

下面代码演示了怎样使用这两个函数绘制 polygons：

```cpp
void DrawPolygons(wxDC& dc)
{
    wxBrush brushHatch(*wxRED, wxFDIAGONAL_HATCH);
    dc.SetBrush(brushHatch);
    wxPoint star[5];
    star[0] = wxPoint(100, 60);
    star[1] = wxPoint(60, 150);
    star[2] = wxPoint(160, 100);
    star[3] = wxPoint(40, 100);
    star[4] = wxPoint(140, 150);
    dc.DrawPolygon(WXSIZEOF(star), star, 0, 30);
    dc.DrawPolygon(WXSIZEOF(star), star, 160, 30, wxWINDING_RULE);
    wxPoint star2[10];
    star2[0] = wxPoint(0, 100);
    star2[1] = wxPoint(-59, -81);
    star2[2] = wxPoint(95, 31);
    star2[3] = wxPoint(-95, 31);
    star2[4] = wxPoint(59, -81);
    star2[5] = wxPoint(0, 80);
    star2[6] = wxPoint(-47, -64);
    star2[7] = wxPoint(76, 24);
    star2[8] = wxPoint(-76, 24);
    star2[9] = wxPoint(47, -64);
    int count[2] = {5, 5};
    dc.DrawPolyPolygon(WXSIZEOF(count), count, star2, 450, 150);
} 
```

结果如下图所示：

![](img/mht30CC%281%29.tmp)

使用云行规画平滑曲线

DrawSpline 函数让你可以绘制一个称为�云行规�的平滑曲线.这个函数有三个点和多个点两个形式，下面的代码对两者都进行了演示：

```cpp
// 三点云行规曲线
dc.DrawSpline(10, 100, 200, 200, 50, 230);
// 五点云行规曲线
wxPoint star[5];
star[0] = wxPoint(100, 60);
star[1] = wxPoint(60, 150);
star[2] = wxPoint(160, 100);
star[3] = wxPoint(40, 100);
star[4] = wxPoint(140, 150);
dc.DrawSpline(WXSIZEOF(star), star); 
```

结果如下图所示：

![](img/mht30DE%281%29.tmp)

绘制位图

在设备上下文上绘制位图有两种主要的方法：DrawBitmap 和 Blit。DrawBitmap 其实是 Blit 的一种简写形式，它使用一个位图，一个位置和一个 bool 类型的透明标志参数。根据图像的制作和读取过程的不同，位图的透明绘制可以通过指定一个透明遮罩或者一个 Alpha 通道（通常用来实现半透明）来实现，下面的代码演示了在一段文本上显示的半透明的图片：

```cpp
wxString msg = wxT("Some text will appear mixed in the image's shadow...");
int y = 75;
for (size_t i = 0; i &lt; 10; i++)
{
    y += dc.GetCharHeight() + 5;
    dc.DrawText(msg, 200, y);
}
wxBitmap bmp(wxT("toucan.png"), wxBITMAP_TYPE_PNG);
dc.DrawBitmap(bmp, 250, 100, true); 
```

运行结果如下图所示：文本在图片下面以半透明的方式隐隐浮现。

![](img/mht30E1%281%29.tmp)

Blit 函数就略微显的复杂些，它允许你拷贝一个设备上下文的一部分到另外一个设备上下文，下面是这个函数的原型：

```cpp
bool Blit(wxCoord destX, wxCoord destY,
          wxCoord width, wxCoord height, wxDC* dcSource,
          wxCoord srcX, wxCoord srcY,
          int logicalFunc = wxCOPY,
          bool useMask = false,
          wxCoord srcMaskX = -1, wxCoord srcMaskY = -1); 
```

这个函数将 dcSource 参数指定的设备上下文中开始于 srcX，srcY 的位置，大小为 width，height 的区域拷贝到目标设备上下文（函数调用者自己）的 destX，destY 的位置，并且使用指定的逻辑函数进行拷贝。默认的逻辑函数是 wxCOPY，意味着直接把源中的象素原封不动的传输到目标去。其它的逻辑函数依照平台的不同有所不同，不是所有的逻辑函数都支持所有的平台。我们将在本节稍后对逻辑函数进行专门介绍。

最后三个参数仅在园设备上下文为透明位图的时候才有效。useMask 参数指定是否使用透明遮罩，而 srcMaskX 和 srcMaskY 则可以通过其设置不采用和主位图一致的遮罩位置。

下面的代码演示了怎样读取一个位图并将其平铺在另外一个更大的设备上下文上,并且保留图片本身的透明属性：

```cpp
wxMemoryDC dcDest;
wxMemoryDC dcSource;
int destWidth = 200, destHeight = 200;
// 创建目标位图
wxBitmap bitmapDest(destWidth, destHeight);
// 加载调色板位图
wxBitmap bitmapSource(wxT("pattern.png"), wxBITMAP_TYPE_PNG);
int sourceWidth = bitmapSource.GetWidth();
int sourceHeight = bitmapSource.GetHeight();
// 用白色清除目标背景
dcDest.SelectObject(bitmapDest);
dcDest.SetBackground(*wxWHITE_BRUSH);
dcDest.Clear();
dcSource.SelectObject(bitmapSource);
// 将小的位图平铺到大的位图
for (int i = 0; i &lt; destWidth; i += sourceWidth)
    for (int j = 0; j &lt; destHeight; j += sourceHeight)
    {
        dcDest.Blit(i, j, sourceWidth, sourceHeight,
                    & dcSource, 0, 0, wxCOPY, true);
    }
//释放内存设备上下文的位图部分
dcDest.SelectBitmap(wxNullBitmap);
dcSource.SelectBitmap(wxNullBitmap); 
```

你可以使用 DrawIcon 函数直接在设备上下文的某个位置显示图标，图标将总以透明方式显示：

```cpp
#include "file.xpm"
wxIcon icon(file_xpm);
dc.DrawIcon(icon, 20, 30); 
```

填充特定区域

FloodFill 函数采用三个参数来填充某个特定的区域。一个起始点参数，一个颜色参数用来确定填充的边界和一个填充类型参数，设备上下文将使用当前的画刷定义进行填充。

下面的例子演示了绘制一个绿色矩形区域，它的轮廓线是红色，先进行黑色填充，进行蓝色填充：

```cpp
// 画一个红边绿色的矩形
dc.SetPen(*wxRED_PEN);
dc.SetBrush(*wxGREEN_BRUSH);
dc.DrawRectangle(10, 10, 100, 100);
dc.SetBrush(*wxBLACK_BRUSH);
// 将绿色区域变成黑色
dc.FloodFill(50, 50, *wxGREEN, wxFLOOD_SURFACE);
dc.SetBrush(*wxBLUE_BRUSH);
// 开始填充蓝色(直到遇到红色)
dc.FloodFill(50, 50, *wxRED, wxFLOOD_BORDER); 
```

如果找不到指定的颜色，这个函数可能返回失败，如果指定的点在当前区域以外，这个函数也将返回失败。这个函数不支持打印设备上下文以及 wxMetafileDC。

逻辑函数

逻辑函数指定在绘画时，源象素怎样和目标象素进行合并操作，默认的 wxCOPY 只是使用源象素取代目标象素。其它的值则指定了一种逻辑操作。比如 wxINVERT 指定将目标象素的值取反作为新的值，这通常被用来绘制临时边框，因为使用这种方法进行绘图在第二次以同样的方法绘图的时候将会恢复原样。

下面的例子演示了怎样绘制一条点线段，然后再将相关区域恢复原来的样子。

```cpp
wxPen pen(*wxBLACK, 1, wxDOT);
dc.SetPen(pen);
// 以取反逻辑函数绘制
dc.SetLogicalFunction(wxINVERT);
dc.DrawLine(10, 10, 100, 100);
// 再次绘制
dc.DrawLine(10, 10, 100, 100);
// 恢复正常绘制方法
dc.SetLogicalFunction(wxCOPY); 
```

逻辑函数的另外一个用法是通过组合的方式创建新的图形。例如，我们可以用下面的方法来通过一个图片创建一组对应的拼图游戏需要的方块。首先在一幅白色背景的图片上使用固定但是随机的大小创建一个黑色轮廓线的网格作为拼图板的边界，然后对于每一个网格小块，使用 flood-fill 的方法将其填充成黑色以便创建一个白色背景上的黑色小方块，然后使用 wxAND_REVERSE 逻辑函数将原图 Blit 到这个模板，这样作的结果是使得方块内的部分变成原图，而白色的背景则变成黑色的背景，然后我们再指定黑色为透明色创建一个 wxImage，然后将其转换成透明的 wxBitmap 对象，就可以直接在拼图游戏中使用了。（注意这样的作法需要原图中没有黑色，否则在拼图小方块中就会出现没有颜色的窟窿了）。

下表列出了所有的逻辑函数以及它们的含义：

| 逻辑函数 | 含义 (src = 源, dst = 目的) |
| --- | --- |
| wxAND | src AND dst |
| wxAND_INVERT | (NOT src) AND dst |
| wxAND_REVERSE | src AND (NOT dst) |
| wxCLEAR | 0 |
| wxCOPY | src |
| wxEQUIV | (NOT src) XOR dst |
| wxINVERT | NOT dst |
| wxNAND | (NOT src) OR (NOT dst) |
| wxNOR | (NOT src) AND (NOT dst) |
| wxNO_OP | dst |
| wxOR | src OR dst |
| wxOR_INVERT | (NOT src) OR dst |
| wxOR_REVERSE | src OR (NOT dst) |
| wxSET | 1 |
| wxSRC_INVERT | NOT src |
| wxXOR | src XOR dst |

# 5.4 使用打印框架

# 5.4 使用打印框架

我们已经介绍过，可以直接使用 wxPrinterDC 来进行打印。不过，一个更灵活的方法是使用 wxWidgets 提供的打印框架来驱动打印机。要使用这个框架，最主要的任务就是要实现一个 wxPrintout 的派生类，重载其成员函数以便告诉 wxWidgets 怎样打印一页（OnPrintPage）, 总共有多少页（GetPageInfo）,进行页面设置(OnPreparePrinting)等等。而 wxWidgets 框架则负责显示打印对话框，创建打印设备上下文和调用适当的 wxPrintout 的函数。同一个 wxPrintout 类将被打印和预览功能一起使用。

当要开始打印的时候，一个 wxPrintout 对象实例被传递给 wxPrinter 对象，然后将调用 Print 函数开始打印过程，并且在准备打印用户指定的那些页面前显示一个打印对话框。如下面例子中的那样。

```cpp
// 一个全局变量用来存储打印相关的设置信息
wxPrintDialogData g_printDialogData;
// 用户从主菜单中选择打印命令以后
void MyFrame::OnPrint(wxCommandEvent& event)
{
    wxPrinter printer(& g_printDialogData);
    MyPrintout printout(wxT("My printout"));
    if (!printer.Print(this, &printout, true))
    {
        if (wxPrinter::GetLastError() == wxPRINTER_ERROR)
            wxMessageBox(wxT("There was a problem printing.\nPerhaps your current printer
 is not set correctly?"), wxT("Printing"), wxOK);
        else
            wxMessageBox(wxT("You cancelled printing"),
                         wxT("Printing"), wxOK);
    }
    else
    {
        (*g_printDialogData) = printer.GetPrintDialogData();
    }
} 
```

因为打印函数在所有的页面都已经被渲染和发送到打印机以后才会返回，因此 wxPrintout 对象可以以局部变量的方式创建。

wxPrintDialogData 对象用来存储所有打印有关的数据，比如用户选择的页面和份数等。将其保存在一个全局变量中并且在下次调用的时候传递给 wxPrinter 对象是一个不错的习惯，因此象上面代码中演示的那样，在创建 wxPrinter 的时候传递给它一份全局配置数据的指针，并在 Print 函数成功返回以后将当前的打印数据保存在全局变量里（当然，一个更专业的作法是将其保存在你的应用程序类中）。关于使用打印和页面设置对话框的详情请参考第八章，�使用标准对话框�。

如果要创建打印预览，你需要创建一个 wxPrintPreview 对象，给这个对象传递两个 wxPrintout 对象作为参数，一个用于预览，一个则用于在用户预览的时候直接请求打印，同样的，你可以传递一个全局的 wxPrintDialogData 对象给预览对象以便预览类使用用户以前选择的打印设置数据。然后将这个预览类传递给 wxPreviewFrame，然后调用 wxPreviewFrame 的 Initialize 函数和 Show 函数显示这个窗口，如下所示：

```cpp
// 用户选择打印预览菜单命令
void MyFrame::OnPreview(wxCommandEvent& event)
{
    wxPrintPreview *preview = new wxPrintPreview(
                             new MyPrintout, new MyPrintout,
                             & g_printDialogData);
    if (!preview->Ok())
    {
        delete preview;
        wxMessageBox(wxT("There was a problem previewing.\nPerhaps your current printer is
 not set correctly?"),
                     wxT("Previewing"), wxOK);
        return;
    }
    wxPreviewFrame *frame = new wxPreviewFrame(preview, this,
                             wxT("Demo Print Preview"));
    frame->Centre(wxBOTH);
    frame->Initialize();
    frame->Show(true);
} 
```

当打印预览窗口被初始化的时候，它将禁用所有其它的顶层窗口以确保任何可能导致正在预览或者打印的文档内容发生改变的可能。关闭这个窗口将会导致两个 wxPrintout 对象自动被释放。下图显示了预览窗口的样子，可以看到它内含一个工具条，上面提供了页面遍历，打印以及缩放控制等功能。

![](img/mht5D71%281%29.tmp)

关于 wxPrintout 的更多内容

当创建 wxPrintout 对象的时候，你可以传递一个可选的标题参数，在某些操作系统上，这个标题将显示在打印管理器上。另外，要重载一个 wxPrintout 对象，你至少需要重载 GetPageInfo, HasPage 和 OnPrintPage 函数，当然，下面介绍的这些函数你都可以视需要进行重载。

首先来介绍 GetPageInfo 函数，它用来返回最小页码，最大页码，开始打印页码和结束打印页码。前两个参数用来提供一个可以打印的范围，后两个参数被设计来反应用户选择的页码范围，不过目前没有用处。最小页面的默认值为 1,最大页码的默认值为 32000,不过当 HasPage 函数返回 False 的时候，wxPrintout 类也将停止打印。通常情况下，你的 OnPreparePrinting 应该计算当前打印内容在当前设置下的打印页数，将其保存在一个成员变量中，以便 GetPageInfo 函数返回正确的值，如下所示：

```cpp
void MyPrintout::GetPageInfo(int *minPage, int *maxPage,
                             int *pageFrom, int *pageTo)
{
    *minPage = 1; *maxPage = m_numPages;
    *pageFrom = 1; *pageTo = m_numPages;
} 
```

而 HasPage 函数则用来返回是否拥有某个页码，如果这个页码超出了最大页码的范围则必须返回 False。通常你的实现类似下面的样子：

```cpp
bool MyPrintout::HasPage(int pageNum)
{
    return (pageNum &gt;= 1 && pageNum &lt;= m_numPages);
} 
```

OnPreparePrinting 在预览或者打印过程刚开始的时候被调用，重载它使得应用程序可以进行各种设置工作，比如计算文档的总页数等。OnPreparePrinting 可以调用 wxPrintout 的 GetDC，GetPageSizeMM，IsPreview 等函数，因此方便获取这些信息。

OnBeginDocument 是在每次每篇文档即将开始打印的时候被调用的，如果这个函数被重载了，那么必须在重载的函数中调用 wxPrintout::OnBeginDocument。同样的 wxPrintout::OnEndDocument 也必须被它的重载函数调用。

OnBeginPrinting 和 OnEndPrinting 函数则是在整个打印过程开始和结束的时候被调用，和当前打印多少份没有关系。

OnPrintPage 会被传递一个页码参数，应用程序必须重载这个函数并且在成功打印这一页以后返回 true。这个函数应该使用 wxPrintout::GetDC 来取得打印设备上下文已经进行绘画（打印）工作。

下面这些函数作为一些工具函数，你可以在你的重载函数中使用它们，而无需重载它们。

IsPreview 函数用来检测当前正处于一个预览过程中还是一个真的打印过程中。

GetDC 函数为当前正在进行的工作返回一个合适的设备上下文。如果是在真实的打印过程中，则返回一个 wxPrinterDC，如果是预览，则返回一个 wxMemoryDC，因为预览其实是在一个位图上通过内存设备上下文来渲染的。

GetPageSizeMM 以毫米为单位返回当前打印页面的大小。GetPageSizePixels 则以象素为单位返回这个值（打印机的最大分辨率）。如果是在预览过程中，这个大小和 wxDC::GetSize 返回的值通常是不一样大的（参见下面的说明），wxDC::GetSize 返回用于预览的位图的大小。

GetPPIPrinter 返回当前设备上下文每一个英寸对应的象素的数目，而 GetPPIScreen 则返回当前屏幕上每英寸对应的象素的数目。

打印和预览过程中的缩放

当你在窗口上绘画的时候，你可能不大关心图片的缩放，因为显示器的分辨率大部分都是相同的。然后，在面对一个打印机的时候，有几方面的因素可能导致你必须关心这个问题：

你需要通过缩放和重新放置图片的位置来保证图片位于某个页面之内，在某种情况下，你甚至可能需要把一个图片分成两半。

字体都是基于屏幕分辨率的，因此在打印文本的时候，你需要设置一个合适的缩放的值，以便使打印设备上下文符合屏幕的分辨率。在打印文本的时候，通过应该设置通过用 GetPPIPrinter 的值除以 GetPPIScreen 的值计算而得的值作为缩放因子的值。

当渲染预览图案的时候，wxWidgets 使用了 wxMemoryDC 在一个位图上绘画。这个位图的大小（wxDC::GetSize）是基于当前预览的放大倍数的这就需要一个额外的缩放因子。通常这个缩放因子可以通过用 GetSize 返回的值除以 GetPageSizePixels 返回的实际页面的象素值来获得。通常这个值还应该乘以别的缩放因子定义的值。

你可以调用 wxDC::SetUserScale 来设置设备上下文的缩放因子，使用 wxDC::SetDeviceOrigin 来设置平移因子（例如，需要把一个图片放置在页面正中的时候）。如果有必要的话，你甚至可以在同一个页面绘画的时候反复使用不同的值来调用这两个函数。

wxWidgets 的例子中的 samples/printing 演示了怎样在打印过程中使用缩放，下面列出的代码是其中进行缩放的适配代码，它将演示了在打印过程中和预览过程中将一幅大小为 200x200 象素的图片进行了缩放和放置，如下所示：

```cpp
void MyPrintout::DrawPageOne(wxDC *dc)
{
    // 下面的代码可以这样写只是因为我们知道图片的大小是 200x200
    // 如果我们不知道的话，需要先计算图片的大小
    float maxX = 200;
    float maxY = 200;
    // 让我们先设置至少 50 个设备单位的边框
    float marginX = 50;
    float marginY = 50;
    // 将边框的大小增加到图片的周围
    maxX += (2*marginX);
    maxY += (2*marginY);
    // 获取象素单位的当前设备上下文的大小
    int w, h;
    dc->GetSize(&w, &h);
    //计算一个合适的缩放值
    float scaleX=(float)(w/maxX);
    float scaleY=(float)(h/maxY);
    // 选择 X 或者 Y 方向上较小的那个
    float actualScale = wxMin(scaleX,scaleY);
    // 计算图片在设备上的合适位置以便居中
    float posX = (float)((w - (200*actualScale))/2.0);
    float posY = (float)((h - (200*actualScale))/2.0);
    // 设置设备平移和缩放
    dc->SetUserScale(actualScale, actualScale);
    dc->SetDeviceOrigin( (long)posX, (long)posY );
    // ok,现在开始画画
    dc.SetBackground(*wxWHITE_BRUSH);
    dc.Clear();
    dc.SetFont(wxGetApp().m_testFont);
    dc.SetBackgroundMode(wxTRANSPARENT);
    dc.SetBrush(*wxCYAN_BRUSH);
    dc.SetPen(*wxRED_PEN);
    dc.DrawRectangle(0, 30, 200, 100);
    dc.DrawText( wxT("Rectangle 200 by 100"), 40, 40);
    dc.SetPen( wxPen(*wxBLACK,0,wxDOT_DASH) );
    dc.DrawEllipse(50, 140, 100, 50);
    dc.SetPen(*wxRED_PEN);
    dc.DrawText( wxT("Test message: this is in 10 point text"),
                 10, 180);
} 
```

在上面的例子中，我们只是简单的使用 wxDC::GetSize 来得到打印设备或者预览图像的分辨率，以便我们能够把要打印的图像放到合适的位置。我们没有关心类似每一个英寸多少个点这样的信息，因为我们不需要画精度很高的文本或者是线段。图片不需要很高的精度，因此我们只是进行简单的缩放以便它能够被合适的放置，不致于太大太小或者越界就可以了。

接下来，我们演示一下怎样进行精度很高的文本或者线段的打印以便看上去它和显示在屏幕上的是一致的。而不是只是进行简单的缩放：

```cpp
void MyPrintout::DrawPageTwo(wxDC *dc)
{
    // 你可以使用下面的代码来设置打印机以便其可以反应出文本在屏幕上的大小
    // 另外下面的代码还将打印一个 5cm 长的线段。
    // 首先获得屏幕和打印机上各自的 1 英寸的逻辑象素个数
    int ppiScreenX, ppiScreenY;
    GetPPIScreen(&ppiScreenX, &ppiScreenY);
    int ppiPrinterX, ppiPrinterY;
    GetPPIPrinter(&ppiPrinterX, &ppiPrinterY);
    // 这个缩放因子用来大概的反应屏幕到实际打印设备的一个缩放
    float scale = (float)((float)ppiPrinterX/(float)ppiScreenX);
    // 现在，我们还需要考虑页面缩放
    // (比如：我们正在作打印预览，用户选择了一个缩放级别)
    int pageWidth, pageHeight;
    int w, h;
    dc->GetSize(&w, &h);
    GetPageSizePixels(&pageWidth, &pageHeight);
    // 如果打印设备的页面大小 pageWidth == 当前 DC 的大小, 就不需要考虑这方面的缩放了
    // 但是它们有可能是不一样的
    // 因此，缩放吧.
    float overallScale = scale * (float)(w/(float)pageWidth);
    dc->SetUserScale(overallScale, overallScale);
    // 现在我们来计算每个逻辑单位有多少个毫米
    // 我们知道 1 英寸大概是 25.4 毫米.而 ppi
    // 代表的是以英寸为单位的. 因此 1 毫米就等于 ppi/25.4 个设备单位
    // 另外我们还需要再除以我们的缩放因子 scale
    // (译者注：为什么这里是 scale 而不是 overallScale？)
    // (其实 overallScale 比 scale 而言，多了一个预览的缩放)
    // （而在预览的时候，我们是希望线段的长度按照用户设置的比例变化的）
    // 因为我们的设备已经被设置了缩放因子
    // 现在让我们来画一个长度为 50mm 的 L 图案
    float logUnitsFactor = (float)(ppiPrinterX/(scale*25.4));
    float logUnits = (float)(50*logUnitsFactor);
    dc->SetPen(* wxBLACK_PEN);
    dc->DrawLine(50, 250, (long)(50.0 + logUnits), 250);
    dc->DrawLine(50, 250, 50, (long)(250.0 + logUnits));
    dc->SetBackgroundMode(wxTRANSPARENT);
    dc->SetBrush(*wxTRANSPARENT_BRUSH);
    dc->SetFont(wxGetApp().m_testFont);
    dc->DrawText(wxT("Some test text"), 200, 300 );
} 
```

在类 Unix 系统上的 GTK+版本上的打印

和 Mac OS X 以及 Windows 系统不同，Unix 系统没有提供一个标准的 API 同时支持在屏幕上显示文本图片和在打印机上打印文本和图片。实际上，在类 Unix 系统中，屏幕显示是通过 X11 库（被 GTK+封装又被 wxWidgets 封装）实现的，而打印要通过发送 PostScript 命令到打印机来完成。而这两种情况下使用不同的字体都是一个麻烦。直到最近，才有很少的程序在类 Unix 上提供了所见即所得的功能。以前 wxWidgets 提供自己的 PostScript 实现，但是它很难和屏幕显示的内容完全一致。

从版本 2.8 开始，Gnome 项目组开始通过 libgnomeprint 和 libgnomeprintui 库来提供打印支持，这样大多数的打印问题才算得以解决。从 wxWidgets 的版本 2.5.4 开始，GTK+的版本通过合适的配置以后可以支持这两个库。你需要使用--with- gnomeprint 来配置 wxWidgets，这将导致 wxWidgets 在运行期自动查找 GNOME 打印库。如果找的到，就使用它完成打印，否则就使用旧的 PostScript 打印的代码。需要说明的是，这并不需要用户的机器上一定要安装 gnome 的打印库程序才可以运行，因为程序本身并不依赖这些库。

# 5.5 使用 wxGLCanvas 绘制三维图形

# 5.5 使用 wxGLCanvas 绘制三维图形

感谢 OpenGL 和 wxGLCanvas，让 wxWidgets 拥有了绘制三维图形的能力。如果你的平台不支持 OpenGL，你仍然可以使用它的一个开放源码的实现 Mesa。

要让 wxWidgets 在 windows 平台上支持 wxGLCanvas，你需要编辑 include/wx/msw/setup.h，设置 wxUSE_GLCANVAS 为 1，然后编译的时候在命令行使用 USE_OPENGL=1，在连接的时候你也可能需要增加 opengl32.lib。而在 Unix 或者 Mac OS X 上，你只需要在配置 wxWidgets 的时候增加--with-opengl 参数来打开 OpenGL 或者 Mesa 的支持。

如果你已经是一个 OpenGL 的程序员，那么使用 wxGLCanvas 是非常简单的。你只需要在一个 frame 窗口或者其他任何容器窗口内创建一个 wxGLCanvas 对象，然后调用 wxGLCanvas::SetCurrent 函数将 OpenGL 的命令指向这个窗口，执行 OpenGL 命令，然后调用 wxGLCanvas::SwapBuffers 函数将当前的 OpenGL 缓冲区的内容绘制到窗口上。

下面的重绘事件处理函数演示了渲染一个三维立方体的一些基本代码书写原则。完整的例子可以在 wxWidgets 发行版本中的 samples/opengl/cube 目录中找到。

```cpp
void TestGLCanvas::OnPaint(wxPaintEvent& event)
{
    wxPaintDC dc(this);
    SetCurrent();
    glMatrixMode(GL_PROJECTION);
    glLoadIdentity();
    glFrustum(-0.5f, 0.5f, -0.5f, 0.5f, 1.0f, 3.0f);
    glMatrixMode(GL_MODELVIEW);
    /* 清除颜色和深度缓冲 */
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    /* 绘制一个立方体的六个面 */
    glBegin(GL_QUADS);
    glNormal3f( 0.0f, 0.0f, 1.0f);
    glVertex3f( 0.5f, 0.5f, 0.5f); glVertex3f(-0.5f, 0.5f, 0.5f);
    glVertex3f(-0.5f,-0.5f, 0.5f); glVertex3f( 0.5f,-0.5f, 0.5f);
    glNormal3f( 0.0f, 0.0f,-1.0f);
    glVertex3f(-0.5f,-0.5f,-0.5f); glVertex3f(-0.5f, 0.5f,-0.5f);
    glVertex3f( 0.5f, 0.5f,-0.5f); glVertex3f( 0.5f,-0.5f,-0.5f);
    glNormal3f( 0.0f, 1.0f, 0.0f);
    glVertex3f( 0.5f, 0.5f, 0.5f); glVertex3f( 0.5f, 0.5f,-0.5f);
    glVertex3f(-0.5f, 0.5f,-0.5f); glVertex3f(-0.5f, 0.5f, 0.5f);
    glNormal3f( 0.0f,-1.0f, 0.0f);
    glVertex3f(-0.5f,-0.5f,-0.5f); glVertex3f( 0.5f,-0.5f,-0.5f);
    glVertex3f( 0.5f,-0.5f, 0.5f); glVertex3f(-0.5f,-0.5f, 0.5f);
    glNormal3f( 1.0f, 0.0f, 0.0f);
    glVertex3f( 0.5f, 0.5f, 0.5f); glVertex3f( 0.5f,-0.5f, 0.5f);
    glVertex3f( 0.5f,-0.5f,-0.5f); glVertex3f( 0.5f, 0.5f,-0.5f);
    glNormal3f(-1.0f, 0.0f, 0.0f);
    glVertex3f(-0.5f,-0.5f,-0.5f); glVertex3f(-0.5f,-0.5f, 0.5f);
    glVertex3f(-0.5f, 0.5f, 0.5f); glVertex3f(-0.5f, 0.5f,-0.5f);
    glEnd();
    glFlush();
    SwapBuffers();
} 
```

下图演示了另外的一个 OpenGL 的例子，一个可爱的（当然，有点棱角的）企鹅，在例子程序中，你可以用鼠标来旋转它。完整的例子可以在光盘的 samples/opengl/penguin 目录中找到。

![](img/mht94CD%281%29.tmp)

# 第五章小节

# 第五章小节

在本章中，你学习了怎样使用设备上下文进行绘画，怎样使用 wxWidgets 提供的打印框架，还看到了一个非常简单的使用 wxGLCanvas 的介绍。wxWidgets 的发布包中有几个很本章内容相关的例子，列举如下以供参考。

*   samples/drawing
*   samples/font
*   samples/erase
*   samples/image
*   samples/scroll
*   samples/printing
*   src/html/htmprint.cpp
*   demos/bombs
*   demos/fractal
*   demos/life

对于更高级的二维绘画应用程序，你可能愿意考虑使用 wxArt2D 库，这个库提供了以 SVG 文件格式读取和保存图形对象，无闪烁更新，过渡色，矢量路径等等特性，附录 E，“wxWidgets 的第三方工具包”包含了获取 wxArt2D 的方法。

在下一章中，我们来看看 wxWidgets 怎样响应鼠标，键盘和游戏手柄输入。