# 第十章使用图像编程

这一章来了解一下我们可以使用图片来作些什么事情.一幅图胜过千言万语,在 wxWidgets,工具条,树形控件,notebooks,按钮,Html 窗口和特定的绘画代码中,都会用到图片.有时候它们还会在不可见的地方发挥作用,比如我们可以用它来创建双缓冲区以避免闪烁.这一章里,我们会接触到各种各样的图片类,还会谈到怎样覆盖 wxWidgets 提供的默认图片和图标.

# 10.1 wxWidgets 中图片相关的类

wxWidgets 支持四种和位图相关的类:wxBitmap, wxIcon, wxCursor 和 wxImage.

wxBitmap 是一个平台有关的类,它拥有一个可选的 wxMask 属性以支持透明绘画.在 windows 系统上,wxBitmap 是通过设备无关位图 (DIBs)实现的,而在 GTK+和 X11 平台上,每个 wxBitmap 则包含一个 GDK 的 pixmap 对象或者 X11 的 pixmap 对象.而在 Mac 平台上,则使用的是 PICT.wxBitmap 可以和 wxImage 进行互相转换.

wxIcon 用来实现各个平台上的图标,一个图标指的是一个小的透明图片,可以用来代表不同的 frame 或者对话框窗口.在 GTK+, X11 和 Mac 平台上,icon 就是一个小的总含有 wxMask 的 wxBitmp,而在 windows 平台上,wxIcon 则是封装了 HICON 对象.

wxCursor 则是一个用来展示鼠标指针的图像,在 GTK+平台上是用的 GdkCursor,X11 和 Mac 平台上用的是各自的 Cursor,而在 windows 平台上则使用的是 HCURSOR.wxCursor 有一个热点的概念(所谓热点指的是图片中用来精确代表指针单击位置的那个点),也总是包含一个遮罩(mask).

wxImage 则是四个类中唯一的一个平台无关的实现,它支持 24bit 位图以及可选的 alpha 通道.wxImage 可以从 wxBitmap 类使用 wxBitmap::ConvertToImage 函数转换而来,也可以从各种各样的图片文件中加载,它所支持的图片格式也是可以通过图片格式处理器来扩展的.它拥有操作其图片上某些 bit 的能力,因此也可以用来对图片进行一个基本的操作.和 wxBitmap 不同,wxImage 不可以直接被设备上下文 wxDC 使用,如果要在 wxDC 上绘图,需要现将 wxImage 转换成 wxBitmap,然后就可以使用 wxDC 的 DrawBitmap 函数进行绘图了.wxImage 支持设置一个掩码颜色来实现透明的效果,也支持通过 alpha 通道实现非常复杂的透明效果.

你可以在这些图片类型之间进行相互转换,尽管某些转换操作是平台相关的.

注意图片类中大量使用引用记数器,因此对图片类进行赋值和拷贝的操作的系统开销是非常小的,不过这也意味着对一个图片的更改可能会影响到别的图片.

所有的图片类都使用下表列出的标准的 wxBitmapType 标识符来读取或者保存图片数据:

| wxBITMAP_TYPE_BMP | Windows 位图文件 (BMP). |
| --- | --- |
| wxBITMAP_TYPE_BMP_RESOURCE | 从 windows 可执行文件资源部分加载的 Windows 位图. |
| wxBITMAP_TYPE_ICO | Windows 图标文件(ICO). |
| wxBITMAP_TYPE_ICO_RESOURCE | 从 windows 可执行文件资源部分加载的 Windows 图标. |
| wxBITMAP_TYPE_CUR | Windows 光标文件(CUR). |
| wxBITMAP_TYPE_CUR_RESOURCE | 从 windows 可执行文件资源部分加载的 Windows 光标. |
| wxBITMAP_TYPE_XBM | Unix 平台上使用的 XBM 单色图片. |
| wxBITMAP_TYPE_XBM_DATA | 从 C++数据中构造的 XBM 单色位图. |
| wxBITMAP_TYPE_XPM | XPM 格式图片,最好的支持跨平台并且支持编译到应用程序中去的格式. |
| wxBITMAP_TYPE_XPM_DATA | 从 C++数据中构造的 XPM 图片. |
| wxBITMAP_TYPE_TIF | TIFF 格式位图,在大图片中使用比较普遍. |
| wxBITMAP_TYPE_GIF | GIF 格式图片,最多支持 256 中颜色,支持透明. |
| wxBITMAP_TYPE_PNG | PNG 位图格式, 一个使用广泛的图片格式,支持透明和 alpha 通道,没有版权问题. |
| wxBITMAP_TYPE_JPEG | JPEG 格式位图, 一个广泛使用的压缩图片格式,支持大图片,不过它的压缩算法是有损耗压缩,因此不适合对图片进行反复加载和压缩. |
| wxBITMAP_TYPE_PCX | PCX 图片格式. |
| wxBITMAP_TYPE_PICT | Mac PICT 位图. |
| wxBITMAP_TYPE_PICT_RESOURCE | 从可执行文件资源部分加载的 Mac PICT 位图. |
| wxBITMAP_TYPE_ICON_RESOURCE | 仅在 Mac OS X 平台上有效, 用来加载一个标准的图标(比如 wxICON_INFORMATION)或者一个图标资源. |
| wxBITMAP_TYPE_ANI | Windows 动画图标(ANI). |
| wxBITMAP_TYPE_IFF | IFF 位图文件. |
| wxBITMAP_TYPE_MACCURSOR | Mac 光标文件. |
| wxBITMAP_TYPE_MACCURSOR_RESOURCE | 从可执行文件资源部分加载的 Mac 光标. |
| wxBITMAP_TYPE_ANY | 让加载图片的代码自己确定图片的格式. |

# 10.2 使用 wxBitmap 编程

你可以使用 wxBitmap 来作下面的事情:

1.  通过设备上下文将其画在一个窗口上.
2.  在某些类(比如 wxBitmapButton, wxStaticBitmap, and wxToolBar)中将其作为一个图片标签.
3.  使用其作为双缓冲区(在将某个图形绘制到屏幕上之前先绘制在一块缓冲区上).

某些平台(特别是 windows 平台)上限制了系统中 bitmap 资源的数目,因此如果你需要使用很多的 bitmap,你最好使用 wxImage 类来保存它们,而只在使用的时候将其转化成 bitmap.

在讨论怎样创建 wxBitmap 之前,让我们先来讨论一下几个主要的函数(如下表所示)

| wxBitmap | 代表一个 bitmap,可以通过指定宽度和高度,或者指定另外一个 bitmap,或者指定一个 wxImage,XPM 数据(char**), 原始数据(char[]), 或者一个指定类型的文件名的方式来创建. |
| --- | --- |
| ConvertToImage | 转换成一个 wxImage,保留透明部分. |
| CopyFromIcon | 从一个 wxIcon 创建一个 wxBitmap. |
| Create | 从图片数据或者一个给定的大小创建一个 bitmap. |
| GetWidth, GetHeight | 返回图片大小. |
| Getdepth | 返回图片颜色深度. |
| GetMask, SetMask | 返回绑定的 wxMask 对象或者 NULL. |
| GetSubBitmap | 将位图其中的某一部分创建为一个新的位图. |
| LoadFile, SaveFile | 从某种支持格式的文件加载或者保存到文件里. |
| Ok | 如果 bitmap 的数据已经具备则返回 True. |

创建一个 wxBitmap

可以通过下面的几个途径来创建一个 wxBitmap 对象.

你可以直接通过默认的构造函数创建一个不包含数据的 bitmap,不过你几乎不能用这个 bitmap 作任何事情,除非你调用 Create 或者 LoadFile 或者用另外一个 bitmap 赋值以便使其拥有具体的 bitmap 数据.

你还可以通过指定宽度和高度的方法创建一个位图,这种情况下创建的位图将被填充以随机的颜色,你需要在其上绘画以便更好的使用它,下面的代码演示了怎样创建一个 200x100 的图片并且将其的背景刷新为白色.

```cpp
// 使用当前的颜色深度创建一个 200x100 的位图
wxBitmap bitmap(200, 100,  -1);
// 创建一个内存设备上下文
wxMemoryDC dc;
// 将创建的图片和这个内存设备上下文关联
dc.SelectObject(bitmap);
// 设置背景颜色
dc.SetBackground(*wxWHITE_BRUSH);
// 绘制位图背景
dc.Clear();
// 解除设备上下文和位图的关联
dc.SelectObject(wxNullBitmap); 
```

你也可以从一个 wxImage 对象创建一个位图,并且保留这个 image 对象的颜色遮罩或者 alpha 通道:

```cpp
// 加载一幅图像
wxImage image(wxT("image.png"), wxBITMAP_TYPE_PNG);
// 将其转换成 bitmap
wxBitmap bitmap(image); 
```

通过 CopyFromIcon 函数可以通过图标文件创建一个 bitmap:

```cpp
// 加载一个图标
wxIcon icon(wxT("image.xpm"), wxBITMAP_TYPE_XPM);
// 将其转换成位图
wxBitmap bitmap;
bitmap.CopyFromIcon(icon); 
```

或者你可以通过指定类型的方式从一个图片文件直接加载一个位图:

```cpp
// 从文件加载
wxBitmap bitmap(wxT("picture.png", wxBITMAP_TYPE_PNG);
if (!bitmap.Ok())
{
    wxMessageBox(wxT("Sorry, could not load file."));
} 
```

wxBitmap 可以加载所有的可以被 wxImage 加载的图片类型,使用的则是各个平台定义的针对特定类型的加载方法.最常用的图片格式比如"PNG, JPG,BMP 和 XPM"等在各个平台上都支持读取和保存操作,不过你需要确认你的 wxWidgets 在编译的时候已经打开了对应的支持.

目前支持的图形类型处理函数如下表所示:

| wxBMPHandler | 用来加载 windows 位图文件. |
| --- | --- |
| wxPNGHandler | 用来加载 PNG 类型的文件.这种文件支持透明背景以及 alpha 通道. |
| wxJPEGHandler | 用来支持 JPEG 文件 |
| wxGIFHandler | 因为版权方面的原因,仅支持 GIF 格式的加载. |
| wxPCXHandler | 用来支持 PCX. wxPCXHandler 会自己计算图片中颜色的数目,如果没有超过 256 色,则采用 8bits 颜色深度,否则就采用 24 bits 颜色深度. |
| wxPNMHandler | 用来支持 PNM 文件格式. 对于加载来说 PNM 格式可以支持 ASCII 和 raw RGB 两种格式.但是保存的时候仅支持 raw RGB 格式. |
| wxTIFFHandler | 用来支持 TIFF. |
| wxIFFHandler | 用来支持 IFF 格式. |
| wxXPMHandler | 用来支持 XPM 格式. |
| wxICOHandler | 用来支持 windows 平台图标文件. |
| wxCURHandler | 用来支持 windows 平台光标文件. |
| wxANIHandler | 用来支持 windows 平台动画光标文件. |

在 Mac OS X 平台上,还可以通过指定 wxBITMAP_TYPE_PICT_RESOURCE 来加载一个 PICT 资源.

如果你希望在不同的平台上从不同的位置加载图片,你可以使用 wxBITMAP 宏,如下所示:

```cpp
#if !defined(__WXMSW__) && !defined(__WXPM__)
#include "picture.xpm"
#endif
wxBitmap bitmap(wxBITMAP(picture)); 
```

这将使得程序在 windows 和 OS/2 平台上从资源文件中加载图片,而在别的平台上,则从一个 picture_xpm 变量中加载 xpm 格式的图片,因为 XPM 在所有的平台上都支持,所以这种使用方法并不常见.

设置一个 wxMask

每个 wxBitmap 对象都可以指定一个 wxMask,所谓 wxMask 指的是一个单色图片用来指示原图中的透明区域.如果你要加载的图片中包含透明区域信息(比如 XPM,PNG 或者 GIF 格式),那么 wxMask 将被自动创建,另外你也可以通过代码创建一个 wxMask 然后调用 SetMask 函数将其和对应的 wxBitmap 对象相关连.你还可以从 wxBitmap 对象创建一个 wxMask,或者通过给一个 wxBitmap 对象指定一种透明颜色来创建一个 wxMask 对象.

下面的代码创建了一个拥有透明色的灰阶位图 mainBitmap,它的大小是 32x32 象素,原始图形从 imageBits 数据创建,遮罩图形从 maskBits 创建,遮罩中 1 代表不透明,0 代表透明颜色.

```cpp
static char imageBits[] = { 255, 255, 255, 255, 31,
  255, 255, 255, 31, 255, 255, 255, 31, 255, 255, 255,
  31, 255, 255, 255, 31, 255, 255, 255, 31, 255, 255,
  255, 31, 255, 255, 255, 31, 255, 255, 255, 25, 243,
  255, 255, 19, 249, 255, 255, 7, 252, 255, 255, 15, 254,
  255, 255, 31, 255, 255, 255, 191, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255, 255, 255, 255, 255, 255, 255, 255, 255, 255, 255,
  255 };
static char maskBits[] = { 240, 1, 0, 0, 240, 1,
  0, 0, 240, 1, 0, 0, 240, 1, 0, 0, 240, 1, 0, 0, 240, 1,
  0, 0, 240, 1, 0, 0, 240, 1, 0, 0, 255, 31, 0, 0, 255,
  31, 0, 0, 254, 15, 0, 0, 252, 7, 0, 0, 248, 3, 0, 0,
  240, 1, 0, 0, 224, 0, 0, 0, 64, 0, 0, 0, 0, 0, 0, 0, 0,
  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,
  0, 0, 0, 0, 0 };
wxBitmap mainBitmap(imageBits, 32, 32);
wxBitmap maskBitmap(maskBits, 32, 32);
mainBitmap.SetMask(new wxMask(maskBitmap)); 
```

XPM 图形格式

在使用小的需要支持透明的图片(比如工具栏上的小图片或者 notebook 以及树状控件上的小图片)的时候,wxWidgets 的程序员通常偏爱使用 XPM 格式,它的最大的特点是采用 C/C++语言的语法,既可以被程序动态加载,也可以直接编译到可执行代码中去,下面是一个例子:

```cpp
// 你也可以用 #include "open.xpm"
static char *open_xpm[] = {
/* 列数 行数 颜色个数 每个象素的字符个数 */
"16 15 5 1",
"  c None",
". c Black",
"X c Yellow",
"o c Gray100",
"O c #bfbf00",
/* 象素 */
"                ",
"          ...   ",
"         .   . .",
"              ..",
"  ...        ...",
" .XoX.......    ",
" .oXoXoXoXo.    ",
" .XoXoXoXoX.    ",
" .oXoX..........",
" .XoX.OOOOOOOOO.",
" .oo.OOOOOOOOO. ",
" .X.OOOOOOOOO.  ",
" ..OOOOOOOOO.   ",
" ...........    ",
"                "
};
wxBitmap bitmap(open_xpm); 
```

正如你看到的那样,XPM 是使用字符编码的.在图片数据前面,有一个调色板区域,使用字符和颜色对应的方法,颜色既可以用标准标识符表示,也可以用 16 进制的 RGB 值表示,使用关键字 None 来表示透明区域.尽管在 windows 系统上,XPM 并不被大多数的图形处理工具支持,不过你还是可以通过一些工具把 PNG 格式转换成 XPM 格式,比如 DialogBlocks 自带的 ImageBlocks 工具,或者你可以直接使用 wxWidgets 编写一个你自己的转换工具.

使用位图绘画

使用位图绘画的方式有两种,你可以使用内存设备上下文绑定一个位图,然后使用 wxDC::Blit 函数,也可以直接使用 wxDC:: DrawBitmap 函数,前者允许你使用位图的一部分进行绘制.在两种方式下,如果这个图片支持透明或者 alpha 通道,你都可以通过将最后一个参数指定为 True 或者 False 来打开或者关闭透明支持.

这两种方法的用法如下:

```cpp
// Draw a bitmap using a wxMemoryDC
wxMemoryDC memDC;
memDC.SelectObject(bitmap);
// Draw the bitmap at 100, 100 on the destination DC
destDC.Blit(100, 100,                         // Draw at (100, 100)
    bitmap.GetWidth(), bitmap.GetHeight(),    // Draw full bitmap
    & memDC,                                  // Draw from memDC
    0, 0,                                     // Draw from bitmap origin
    wxCOPY,                                   // Logical operation
    true);                                    // Take mask into account
memDC.SelectObject(wxNullBitmap);
// Alternative method: use DrawBitmap
destDC.DrawBitmap(bitmap, 100, 100, true); 
```

第五章,"绘画和打印"中对使用 bitmap 绘画有更详细的描述.

打包位图资源

如果你曾是一个 windows 平台的程序员,你可能习惯从可执行文件的资源部分加载一幅图片,当然在 wxWidgets 中也可以这样作, 你只需要指定一个资源名称一个资源类型 wxBITMAP_TYPE_BMP_RESOURCE,不过这种作法是平台相关的.你可能更倾向于使用另外一种平台无关的解决方案.

一个可移植的方法是,你可以将你用到的所有数据文件,包括 HTML 网页,图片或者别的任何类型的文件压缩在一个 zip 文件里,然后你可以用 wxWidgets 提供的虚拟文件系统对加载这个 zip 文件的其中任何一个或几个文件,如下面的代码所示:

```cpp
// 创建一个文件系统
wxFileSystem*fileSystem = new wxFileSystem;
wxString archiveURL(wxT("myapp.bin"));
wxString filename(wxT("myimage.png"));
wxBitmapType bitmapType = wxBITMAP_TYPE_PNG;
// 创建一个 URL
wxString combinedURL(archiveURL + wxString(wxT("#zip:")) + filename);
wxImage image;
wxBitmap bitmap;
// 打开压缩包中的对应文件
wxFSFile* file = fileSystem->OpenFile(combinedURL);
if (file)
{
    wxInputStream* stream = file->GetStream();

    // Load and convert to a bitmap
    if (image.LoadFile(* stream, bitmapType))
        bitmap = wxBitmap(image);

    delete file;
}
delete fileSystem;
if (bitmap.Ok())
{
    ...
} 
```

更多关于虚拟文件系统的信息请参考第十四章:文件和流操作.

# 10.3 使用 wxIcon 编程

一个 wxIcon 代表一个小的位图,它总有一个透明遮罩,它的用途包括:

*   设置 frame 窗口或者对话框的图标
*   通过 wxImageList 类给 wxTreeCtrl, wxListCtrl 或者 wxNotebook 提供图标 (更多信息请参考最后一章)
*   使用 wxDC::DrawIcon 函数在设备上下文中绘制一个图标

下表列出了图标类的主要成员函数

| wxIcon | 图标类可以通过指定另外一个图标类的方式,指定 XPM 数据(char**)的方式, 原始数据(char[])的方式,或者文件名及文件类型的方式创建. |
| --- | --- |
| CopyFromBitmap | 从 wxBitmap 类创建一个图标. |
| GetWidth, GetHeight | 返回图标的大小. |
| Getdepth | 返回图标的颜色深度. |
| LoadFile | 从文件加载图标. |
| Ok | 在图标数据已经具备的时候返回 True. |

创建一个 wxIcon

wxIcon 可以使用 XPM 数据创建,或者从一个 wxBitmap 对象中创建,或者从文件(比如一个 Xpm 文件)中读取.wxWidgets 也提供了类似于前一小节提到的 wxBITMAP 类似的宏,用来从一个平台相关的资源中获取图标.

在 windows 平台上,LoadFile 以及同等性质的操作可以使用的文件类型包括 BMP 图片和 ICO 文件,如果你要从其它图片格式中创建图标,可以先将其读入一个 wxBitmap 对象中,然后再将其转换为一个图标.

而在 Mac OSX 和 Unix/Linux 的 GTK+版本中,wxIcon 可以识别的图片类型和 wxBitmap 可以识别的图片类型是一样的.

下面代码演示了创建一个 wxIcon 对象的几种方法:

```cpp
// 方法 1: 从 XPM 数据创建
#include "icon1.xpm"
wxIcon icon1(icon1_xpm);
// 方法 2: 从一个 ICO 资源中创建(Window and OS/2 only)
wxIcon icon2(wxT("icon2"));
// 方法 3: 从一个图片文件中 (Windows and OS/2 only)
// 如果你的图片包含多个图标你可以指定单个图标的宽度
wxIcon icon3(wxT("icon3.ico"), wxBITMAP_TYPE_ICO, 16, 16);
// 方法 4: 从位图创建
wxIcon icon4;
wxBitmap bitmap(wxT("icon4.png"), wxBITMAP_TYPE_PNG);
icon4.CopyFromBitmap(bitmap); 
```

使用 wxIcon

下面的代码演示了 wxIcon 的三种使用方法:设置窗口图标,增加到一个图片列表或者绘制在某个设备上下文上

```cpp
#include "myicon.xpm"
wxIcon icon(myicon_xpm);
// 1: 设置窗口图标
frame->SetIcon(icon);
// 2: 增加到 wxImageList
wxImageList* imageList = new wxImageList(16, 16);
imageList->Add(icon);
// 3: 在(10, 10)的位置绘制
wxClientDC dc(window);
dc.DrawIcon(icon, 10, 10); 
```

将某个图标绑定到应用程序

将某个图标绑定到应用程序,以便系统可以显示这个图标在合适的位置使得用户可以通过点击图标的方式打开应用程序,这个工作 wxWidgets 是做不到的.这是极少的你需要在不同的平台使用不同的技术的领域中的一个.

在 windows 平台上,你需要在 makefile 中增加一个资源文件(扩展名是.rc),并且在这个资源文件中指定一个图标区域,如下所示:

```cpp
aardvarkpro ICON aardvarkpro.ico
#include "wx/msw/wx.rc" 
```

在这里, aardvarkpro.ico 就是这个和应用程序绑定的图标的名称,它可以有多种分辨率和颜色深度(典型的大小包括 48x48,32x32 和 16x16).当 windows 的资源管理器需要显示某个图标的时候,它将使用子母顺序排在第一个的那个图标,因此你最好给确定要作为应用程序图标的那个图标的名称前面加几个 a 子母以便按照子母顺序它排在前面,否则你的应用程序可能绑定的是你不期望的图标.

在 Mac 系统上,你需要准备一个应用程序包,其中包含一些 ICNS 文件.参考第二十章"让你的程序更完美",来获得关于程序包更多的信息,其中的主要文件 Info.plist 文件看上去应该象下面的额样子:

```cpp
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
<key>CFBundleIconFile</key>
<string>dialogblocks-app.icns</string>
... 
```

应用程序图标和应用程序相关的文档类型图标是由 CFBundleIconFile 和 CFBundleTypeIconFile 属性指定的.你可以直接用 Apple 提供图标编辑器编辑 ICNS 文件,不过如果你希望所有的平台使用同样的图标,你最好现用 PNG 图片创建各种大小的图标,然后再将它粘贴到各个平台上的图标编辑器中,要确保 PNG 使用的透明遮罩颜色和各个工具使用的透明颜色相一致.

而在 linux 平台上,Gnome 桌面系统和 KDE 桌面系统则各自拥有自己的图标提供体系,我们将在第二十章进行简要的描述.

# 10.4 使用 wxCursor 编程

光标用来指示鼠标指针当前的位置.你可以给某个窗口指定不同的光标以便提示用户这个窗口期待某种类型的鼠标操作.和图标一样,光标也是一种始终带有透明遮罩的小图片,可以使用一般的构造函数或者是平台相关的构造函数来创建.其中的一些构造函数还需要相对于整个图片的左上角指定一个热点位置,当鼠标点击的时候,热点所在的位置将作为鼠标点击的位置.

下表列举了光标相关的函数

| wxCursor | 光标可以从 wxImage 对象,二进制数据,系统定义的光标标识符以及光标文件来创建. |
| --- | --- |
| Ok | 如果光标数据已经具备,则返回 True. |

创建一个光标

创建光标最简单的方法是通过系统提供的光标标识符,如下面的例子所示:

wxCursor cursor(wxCURSOR_WAIT);

下表列出了目前支持的光标标识符和它们的光标的样子(依照平台的不同会有些变化)

| wxCURSOR_ARROW | ![](img/mhtB973%281%29.tmp) | 标准光标. |
| --- | --- | --- |
| wxCURSOR_RIGHT_ARROW | ![](img/mhtB986%281%29.tmp) | 标准反向光标. |
| wxCURSOR_BLANK | 透明光标. |
| wxCURSOR_BULLSEYE | ![](img/mhtB989%281%29.tmp) | 近视眼. |
| wxCURSOR_CROSS | ![](img/mhtB99B%281%29.tmp) | 十字. |
| wxCURSOR_HAND | ![](img/mhtB99E%281%29.tmp) | 手. |
| wxCURSOR_IBEAM | ![](img/mhtB9B1%281%29.tmp) | I 字光标. |
| wxCURSOR_LEFT_BUTTON | ![](img/mhtB9B4%281%29.tmp) | 按左键(GTK+ only). |
| wxCURSOR_MAGNIFIER | ![](img/mhtB9B7%281%29.tmp) | 放大镜. |
| wxCURSOR_MIDDLE_BUTTON | ![](img/mhtB9B1%281%29.tmp) | 按中键(译者注:原书图片有误) (GTK+ only). |
| wxCURSOR_NO_ENTRY | ![](img/mhtB9CA%281%29.tmp) | 禁止通行. |
| wxCURSOR_PAINT_BRUSH | ![](img/mhtB9CD%281%29.tmp) | 画刷. |
| wxCURSOR_PENCIL | ![](img/mhtB9DF%281%29.tmp) | 铅笔. |
| wxCURSOR_POINT_LEFT | ![](img/mhtB9E2%281%29.tmp) | 向左. |
| wxCURSOR_POINT_RIGHT | ![](img/mhtB9F5%281%29.tmp) | 向右. |
| wxCURSOR_QUESTION_ARROW | ![](img/mhtB9F8%281%29.tmp) | 带问号的箭头. |
| wxCURSOR_RIGHT_BUTTON | ![](img/mhtB9B1%281%29.tmp) | 按右键(译者注:图片有误) (GTK+ only). |
| wxCURSOR_SIZENESW | ![](img/mhtBA0A%281%29.tmp) | 东北到西南伸缩. |
| wxCURSOR_SIZENS | ![](img/mhtBA0D%281%29.tmp) | 南北伸缩. |
| wxCURSOR_SIZENWSE | ![](img/mhtBA20%281%29.tmp) | 西北到东南伸缩. |
| wxCURSOR_SIZEWE | ![](img/mhtBA23%281%29.tmp) | 东西伸缩. |
| wxCURSOR_SIZING | ![](img/mhtBA26%281%29.tmp) | 一般伸缩. |
| wxCURSOR_SPRAYCAN | ![](img/mhtB9CD%281%29.tmp) | 画刷. |
| wxCURSOR_WAIT | ![](img/mhtBA39%281%29.tmp) | 等待. |
| wxCURSOR_WATCH | ![](img/mhtBA3C%281%29.tmp) | 查看. |
| wxCURSOR_ARROWWAIT | ![](img/mhtBA4E%281%29.tmp) | 后台忙. |

你还可以使用预定义光标指针 wxSTANDARD_CURSOR, wxHOURGLASS_CURSOR 和 wxCROSS_CURSOR.

另外在 windows 和 Mac 平台上还可以从对应的资源文件中加载光标:

```cpp
//windows 平台
wxCursor cursor(wxT("cursor_resource"), wxBITMAP_TYPE_CUR_RESOURCE,
                 hotSpotX, hotSpotY);
// Mac 平台
wxCursor cursor(wxT("cursor_resource"), wxBITMAP_TYPE_MACCUR_RESOURCE); 
```

你还可以通过 wxImage 对象创建光标,而"热点"则要通过 wxImage::SetOptionInt 函数设置.之所以要设置热点, 是因为很多光标不太适合使用默认的左上角作为热点,比如对于十字光标来说,你可能希望将其十字交叉的地方作为热点.下面的代码演示了怎样从一个 PNG 文件中产生设置了热点的光标:

```cpp
// 用 wxImage 创建光标
wxImage image(wxT("cursor.png"), wxBITMAP_TYPE_PNG);
image.SetOptionInt(wxIMAGE_OPTION_CUR_HOTSPOT_X, 5);
image.SetOptionInt(wxIMAGE_OPTION_CUR_HOTSPOT_Y, 5);
wxCursor cursor(image); 
```

使用 wxCursor

每个窗口都可以设置一个对应的光标,这个光标在鼠标进入这个窗口的时候显示,如果一个窗口没有设置光标,其父窗口的光标将被显示,如果所有的父窗口都没有设置光标,则系统默认光标被显示:

使用下面的代码给窗口设置一个光标:

```cpp
window->SetCursor(wxCursor(wxCURSOR_WAIT)); 
```

使用 wxSetCursorEvent

在 windows 系统或者是 Mac OS X 系统上,有一些小地方我们需要注意一下.举个例子,如果你自己实现了一个容器类,比方说是一个分割窗口,并且给它设置了一个特殊的光标(比如说 wxCURSOR_WE 用来表明某个分割条是可以被拉动的),然后你在这个分割窗口中放置了两个子窗口,如果你没有给这两个子窗口设置光标的话,当光标在子窗口上移动时,它们可能会不恰当的显示其父窗口,那个 wxCURSOR_WE 光标.而本来你是希望只有在鼠标移动到分割条上的时候才显示的.

要告诉 wxWidgets 某个光标只应该在某种情况下被显示,你可以增加一个 wxSetCursorEvent 事件的处理函数,这个事件在 Windows 和 Mac 平台上,当需要设置光标的时候(通常是鼠标在窗口间移动的时候)被产生.在这个事件处理函数中可以调用 wxSetCursorEvent::SetCursor 来设置一个特殊的光标.如下所示:

```cpp
BEGIN_EVENT_TABLE(wxSplitterWindow, wxWindow)
    EVT_SET_CURSOR(wxSplitterWindow::OnSetCursor)
END_EVENT_TABLE()
// 指示光标只应该被设置给分割条
void wxSplitterWindow::OnSetCursor(wxSetCursorEvent& event)
{
    if ( SashHitTest(event.GetX(), event.GetY(), 0) )
    {
        // 使用默认的处理
        event.Skip();
    }
    //else:什么也不作,换句话说,不调用 Skip.则事件表不会被继续搜索
} 
```

在这个例子中,当鼠标指针移过分割条的时候,SashHitTest 函数返回 True,因此 Skip 函数被调用,事件表调用失败,这和没有定义这个事件表的效果是一样的,导致 wxWidgets 象往常一样显示指定给窗口的光标(wxCURSOR_WE).而如果 SashHitTest 函数返回 False,则表明光标是在子窗口上移动,这时候应该不显示我们指定的光标,因此我们不调用 Skip 函数,让事件表匹配成功,则事件表将不会在继续匹配,这将使得 wxWidgets 认为这个窗口没有被指定光标,因此.在这种情况下,即使子窗口自己没有光标(象 wxTextCtrl 这种控件,一般系统会指定一个它自己的光标,不过 wxWidgets 对这个是不感知的),也将不会使用我们指定给父窗口的光标.

# 10.5 使用 wxImage 编程

你可以使用 wxImage 对图形进行一些平台无关的调整,或者将其作为图片加载和保存的中间步骤.图片在 wxImage 中是按照每一个象素使用一个分别代表红色,绿色和蓝色的字节的格式保存的,如果图片包含 alpha 通道,则还会占用额外的一个字节.

wxImage 主要的函数如下:

| wxImage | wxImage 的创建方法包括:指定宽度和高度, 从另外一幅图片创建, 使用 XPM 数据, 图片元数据(char[]) 和可选的 alpha 通道数据,文件名及其类型,以及通过输入流等多种方式创建. |
| --- | --- |
| ConvertAlphaToMask | 将 alpla 通道(如果有的话)转换成一个透明遮罩. |
| ConvertToMono | 转换成一个黑白图片. |
| Copy | 返回一个不使用引用记数器的完全一样的拷贝. |
| Create | 创建一个指定大小的图片,可选的参数指明是否初始化图片数据. |
| Destroy | 如果没有人再使用的话,释放内部数据. |
| GeTData, SetData | 获取和设置内部数据指针(unsigned char*). |
| GetImageCount | 返回一个文件或者流中的图片个数. |
| GetOption, GetOptionInt, SetOption, HasOption | 获取, 设置和测试某个选项是否设置. |
| GetSubImage | 将图片的一部分返回为一个新的图像. |
| GetWidth, GetHeight | 返回图片大小. |
| Getred, GetGreen, GetBlue, SetRGB, GetAlpha, SetAlpha | 获得和指定某个象素的 RGB 以及 Alpha 通道的值. |
| HasMask, GetMaskRed, GetMaskGreen, GetMaskBlue, SetMaskColour | 用来测试图像是否有一个遮罩,以及遮罩颜色的 RGB 值或者整个颜色的值. |
| LoadFile, SaveFile | 各种图片格式文件的读取和保存操作. |
| Mirror | 在各种方向上产生镜像,返回一个新图片. |
| Ok | 判断图片是否已初始化. |
| Paste | 将某个图片粘贴在这个图片的指定位置. |
| Rotate, Rotate90 | 旋转图片,返回一个新图片. |
| SetMaskFromImage | 通过指定的图片和透明颜色产生一个遮罩并且设置这个遮罩. |
| Scale, Rescale | 缩放产生一个新图片或者缩放本图片. |

加载和保存图像

wxImage 可以读取和保存各种各样的图片格式,并且使用图像处理过程来增加扩展的能力.其它的图像类(比如 wxBitmap)在某个平台不具备处理某种图形格式的能力的时候,也通常使用的都是 wxImage 的图象处理过程来加载特定格式的图形.

本章第二小节中展示了 wxWidgets 支持的各种图形处理过程.其中 wxBMPHandler 是默认支持的,而要支持其它的图形格式处理,就需要使用 wxImage::AddHandler 函数增加对应的图形处理过程或者使用 wxInitAllImageHandlers 增加所有支持的图形处理过程.

如果你只需要特定的图形格式支持,可以在 OnInit 函数中使用类似下面的代码:

```cpp
#include "wx/image.h"
wxImage::AddHandler( new wxPNGHandler );
wxImage::AddHandler( new wxJPEGHandler );
wxImage::AddHandler( new wxGIFHandler );
wxImage::AddHandler( new wxXPMHandler ); 
```

或者,你可以简单的调用:

```cpp
wxInitAllImageHandlers(); 
```

下面演示了几种从文件或者流读取图片的方式,注意在实际使用过程中,最好使用绝对路径以避免依赖于当前路径的设置:

```cpp
// 使用构造函数指定类型来读取图像
wxImage image(wxT("image.png"), wxBITMAP_TYPE_PNG);
if (image.Ok())
{
    ...
}
// 不指定图像类型一般也能正常工作
wxImage image(wxT("image.png"));
// 使用两步法创建图像
wxImage image;
if (image.LoadFile(wxT("image.png")))
{
    ...
}
// 如果一个文件包含两副图片 Two-step loading with an index into a multi-image file:
// 下面演示选择第 2 副加载
wxImage image;
int imageCount = wxImage::GetImageCount(wxT("image.tif"));
if (imageCount &gt; 2)
    image.LoadFile(wxT("image.tif"), wxBITMAP_TYPE_TIFF, 2);
// 从文件流加载图片
wxFileInputStream stream(wxT("image.tif"));
wxImage image;
image.LoadFile(stream, wxBITMAP_TYPE_TIF);
// 保存到一个文件
image.SaveFile(wxT("image.png")), wxBITMAP_TYPE_PNG);
// 保存到一个流
wxFileOutputStream stream(wxT("image.tif"));
image.SaveFile(stream, wxBITMAP_TYPE_TIF); 
```

除了 XPM 和 PCX 格式以外,其它的图片格式都将以 24 位颜色深度保存(译者注:GIF 格式因为版权方面的原因不支持保存到文件),这两种格式的图形处理过程将会计算实际的颜色个数从而选择相应的颜色深度.JPEG 格式还拥有一个质量选项可供设置.它的值的范围为从 0 到 100,0 代表最低的图片质量和最高的压缩比,100 则代表最高的图片质量和最低的压缩比.如下所示:

```cpp
// 设置一个合理的质量压缩比
image.SetOption(wxIMAGE_OPTION_QUALITY, 80);
image.SaveFile(wxT("picture.jpg"), wxBITMAP_TYPE_JPEG); 
```

另外如果以 XPM 格式保存到流输出中的时候,需要使用 wxImage::SetOption 函数设置一个名称否则,处理函数不知道该用什么名称命名对应的 C 变量.

```cpp
// 保存 XPM 到流格式
image.SetOption(wxIMAGE_OPTION_FILENAME, wxT("myimage"));
image.SaveFile(stream, wxBITMAP_TYPE_XPM); 
```

注意处理函数会自动在你设置的名称后增加"_xpm".

透明

有两种方式设置一个 wxImage 为透明的图像:使用颜色遮罩或者 alpha 通道.一种颜色可以被指定为透明颜色,通过这种方法在将 wxImage 转换成 wxBitmap 的时候可以很容易的制作一个透明遮罩.

wxImage 也支持 alpha 通道数据,在每一个象素的 RGB 颜色之外来由另外一个字节用来指示 alpha 通道的值,0 代表完全透明,255 则代表完全不透明.中间的值代表半透明.

不是所有的图片都用有 alpha 通道数据的,因此在使用 GetAlpha 函数之前,应该使用 HasAlpha 函数来判断图像是否拥有 alpha 通道数据.到目前为止,只有 PNG 文件或者调用 SetAlpha 设置了 alpha 通道的图像才拥有 alpha 通道数据.保存一个带有 alpha 通道的图像目前还不被支持.绘制一个拥有 alpha 通道的方法是先将其转换成 wxBitmap 然后使用 wxDC::DrawBitmap 或者 wxDC:: Blit 函数.

下面的代码演示了怎样使用颜色掩码创建一个透明的 wxImage,它是蓝色的,拥有一个透明的矩形区域:

```cpp
// 创建一个有颜色掩码的 wxBitmap
// 首先,在这个 wxBitmap 上绘画
wxBitmap bitmap(400, 400);
wxMemoryDC dc;
dc.SelectObject(bitmap);
dc.SetBackground(*wxBLUE_BRUSH);
dc.Clear();
dc.SetPen(*wxRED_PEN);
dc.SetBrush(*wxRED_BRUSH);
dc.DrawRectangle(50, 50, 200, 200);
dc.SelectObject(wxNullBitmap);
// 将其转换成 wxImage
wxImage image = bitmap.ConvertToImage();
// 设置掩码颜色
image.SetMaskColour(255, 0, 0); 
```

在下面的例子中,使用从一个图片创建颜色遮罩的方式,其中 image.bmp 是原始图像,而 mask.bmp 则是一个掩码图像,在后者中所有透明的部分都是黑色显示的.

```cpp
// 加载一副图片和它的掩码遮罩
wxImage image(wxT("image.bmp"), wxBITMAP_TYPE_BMP);
wxImage maskImage(wxT("mask.bmp"), wxBITMAP_TYPE_BMP);
// 从后者创建一个遮罩并且设置给前者.
image.SetMaskFromImage(maskImage, 0, 0, 0); 
```

如果你加载的图片本身含有透明颜色,你可以检测并且直接创建遮罩:

```cpp
// 加载透明图片
wxImage image(wxT("image.png"), wxBITMAP_TYPE_PNG);
// 获取掩码
if (image.HasMask())
{
    wxColour maskColour(image.GetMaskRed(),
    image.GetMaskGreen(),
    image.GetMaskBlue());
} 
```

变形

wxImage 支持缩放,旋转以及镜像等多种变形方式,下面各举一些例子:

```cpp
// 把原始图片缩放到 200x200,并保存在新的图片里
// 原图保持不变.
wxImage image2 = image1.Scale(200, 200);
// 将原图缩放到 200x200
image1.Rescale(200, 200);
// 旋转固定角度产生新图片.
// 原图片保持不变.
wxImage image2 = image1.Rotate(0.5);
// 顺时针旋转 90 度产生新图片.
// 原图保持不变.
wxImage image2 = image1.Rotate90(true);
// 水平镜像产生新图片.
// 原图保持不变.
wxImage image2 = image1.Mirror(true); 
```

颜色消减

如果你想对某个图像的颜色进行消减,你可以使用 wxQuantize 类的一些静态函数,其中最有趣的函数 Quantize 的参数为一个输入图片,一个输出图片,一个可选的 wxPalette**指针用来存放经过消减的颜色,以及一个你希望保留的颜色个数,你也可以传递一个 unsigned char**变量来获取一个 8-bit 颜色深度的输出图像.最后的一个参数 style(类型)用来对返回的图像进行一些更深入的控制,详情请参考 wxWidgets 的手册.

下面的代码演示了怎样将一幅图片的颜色消减到最多 256 色:

```cpp
#include "wx/image.h"
#include "wx/quantize.h"
wxImage image(wxT("image.png"));
int maxColorCount = 256;
int colors = image.CountColours();
wxPalette* palette = NULL;
if (colors &gt; maxColorCount )
{
    wxImage reducedImage;
    if (wxQuantize::Quantize(image, reducedImage,
                               & palette, maxColorCount))
    {
        colors = reducedImage.CountColours();
        image = reducedImage;
    }
} 
```

一个 wxImage 可以设置一个 wxPalette,例如加载 GIF 文件的时候. 然后,图片内部仍然是以 RGB 的方式存储数据的,调色板仅代表图片加载时候的颜色隐射关系.调色板的另外一个用途是某些图片处理函数用它来将图片保存为低颜色深度的图片,例如 windows 的 BMP 图片处理过程将检测是否设置了 wxBMP_8BPP_PALETTE 标记,如果设置了,则将使用调色板.而如果设置了 wxBMP_8BPP 标记(而不是 wxBMP_8BPP_PALETTE),它将使用自己的算法进行颜色消减.另外某些图片处理过程自己也进行颜色消减,比如 PCX 的处理过程,除非它认为剩余的颜色个数已经足够低了,否则它将对图片的颜色进行消减.

关于调色板更多的信息请参考第五章的"调色板"小节.

直接操作 wxImage 的元数据

你可以直接通过 GetData 函数访问 wxImage 的元数据以便以比 GeTRed, GetBlue, GetGreen 和 SetRGB 更快的方式对其进行操作,下面举了一个使用这种方法将一个图片转换成灰度图片的方法:

```cpp
void wxImage::ConvertToGrayScale(wxImage& image)
{
    double red2Gray   = 0.297;
    double green2Gray = 0.589;
    double blue2Gray  = 0.114;
    int w = image.GetWidth(), h = image.GetHeight();
    unsigned char *data = image.GetData();
    int x,y;
    for (y = 0; y &lt; h; y++)
        for (x = 0; x &lt; w; x++)
        {
            long pos = (y * w + x) * 3;
            char g = (char) (data[pos]*red2Gray +
                              data[pos+1]*green2Gray +
                              data[pos+2]*blue2Gray);
            data[pos] = data[pos+1] = data[pos+2] = g;
        }
} 
```

# 10.6 图片列表和图标集

有时候,使用一组图片是非常方便的.这时候,你可以直接在你的代码中使用 wxImageList,也可以和 wxWidgets 提供的一些控件一起使用 wxImageList,wxNotebook,wxtreeCtrl 和 wxListCtrl 都需要 wxImageList 来管理它们所需要使用的图标. 你也可使用 wxImageList 中的某个单独的图片在设备上下文上绘画.

创建一个 wxImageList 需要的参数包括单个图片的宽度和高度,一个 bool 值来指定是否需要指定图片遮罩,以及这个图片列表的初始大小(主要是为了内部优化代码),然后一个一个的增加 wxBitmap 对象或者 wxIcon 对象.wxImageList 不能直接使用 wxImage 对象,你需要先将其转换为 wxBitmap 对象.wxImageList::Add 函数返回一个整数的索引用来代表这个刚增加的图片,在 Add 函数成功返回以后,你就可以释放原始图片了,wxImageList 已经在内部创建了一个这个图片的拷贝.

下面是创建 wxImageList 以及在其中增加图片的一些例子:

```cpp
// 创建一个 wxImageList
wxImageList *imageList = new wxImageList(16, 16, true, 1);
// 增加一个透明的 PNG 文件
wxBitmap bitmap1(wxT("image.png"), wxBITMAP_TYPE_PNG);
imageList->Add(bitmap1);
// 增加一个透明的来自别的 bitmap 的图片
wxBitmap bitmap2(wxT("image.bmp"), wxBITMAP_TYPE_BMP);
wxBitmap maskBitmap(wxT("mask.bmp"), wxBITMAP_TYPE_BMP);
imageList->Add(bitmap2, maskBitmap);
// 增加一个指定透明颜色的透明图片
wxBitmap bitmap3(wxT("image.bmp"), wxBITMAP_TYPE_BMP);
imageList->Add(bitmap3, *wxRED);
// 增加一个图标
#include "folder.xpm"
wxIcon icon(folder_xpm);
imageList->Add(icon); 
```

你可以直接把 wxImageList 中的图片绘制在设备上下文上,通过指定 wxIMAGELIST_DRAW_TRANSPARENT 类型来指示绘制透明图片,你还可以指定的类型包括 wxIMAGELIST_DRAW_NORMAL, wxIMAGELIST_DRAW_SELECTED 或者 wxIMAGELIST_DRAW_FOCUSED,用来表征图片的状态,如下所示:

```cpp
// 绘制列表中所有的图片
wxClientDC dc(window);
size_t i;
for (i = 0; i &lt; imageList->GetImageCount(); i++)
{
    imageList->Draw(i, dc, i*16, 0, wxIMAGELIST_DRAW_NORMAL|
                                    wxIMAGELIST_DRAW_TRANSPARENT);
} 
```

要把图片列表和 notebook 的 TAB 页面绑定在一起,你需要创建一个包含大小为 16x16 的图片的列表,然后调用 wxNotebook::SetImageList 或者 wxNotebook::AssignImageList 将其和某个 wxNotebook 绑定,这两个函数的区别在于,前者在 wxNotebook 释放的时候不释放列表,而后者在自己被释放的时候,会同时释放图片列表.指定完图片列表以后,你就可以给某个页面指定图标索引以便在页面标签上显示图标了,下面的代码演示了这个过程:

```cpp
//创建一个 wxImageList
wxImageList *imageList = new wxImageList(16, 16, true, 1);
// 增加一些图标
wxBitmap bitmap1(wxT("folder.png"), wxBITMAP_TYPE_PNG);
wxBitmap bitmap2(wxT("file.png"), wxBITMAP_TYPE_PNG);
int folderIndex = imageList->Add(bitmap1);
int fileIndex = imageList->Add(bitmap2);
// 创建一个拥有两个页面的 notebook
wxNotebook* notebook = new wxNotebook(parent, wxID_ANY);
wxPanel* page1 = new wxPanel(notebook, wxID_ANY);
wxPanel* page2 = new wxPanel(notebook, wxID_ANY);
// 绑定图片列表
notebook->AssignImageList(imageList);
// Add the pages, with icons
notebook->AddPage(page1, wxT("Folder options"), true, folderIndex);
notebook->AddPage(page2, wxT("File options"), false, fileIndex); 
```

wxtreeCtrl 和 wxListCtrl 的使用方法和上面介绍的非常相似,也包含类似的两种绑定方法.

如果你拥有很多图标,有时候很难通过索引来对应到具体的图标,你可能想编写一个类以便通过字符串来找到某个图片索引.下面演示了基于这个目的的一个简单的实现:

```cpp
#include "wx/hashmap.h"
WX_DECLARE_STRING_HASH_MAP(int, IconNameToIndexHashMap);
// 通过名字引用图片的类
class IconNameToIndex
{
public:
    IconNameToIndex() {}
    // 在图片列表中增加一个已经命名的图片
    void Add(wxImageList* list, const wxBitmap& bitmap,
        const wxString& name) {
        m_hashMap[name] = list->Add(bitmap);
    }
    // 在图片列表中增加一个已命名的图标
    void Add(wxImageList* list, const wxIcon& icon,
        const wxString& name) {
        m_hashMap[name] = list->Add(icon);
    }
    // 通过名称找到索引
    int Find(const wxString& name) { return m_hashMap[name]; }
private:
    IconNameToIndexHashMap m_hashMap;
}; 
```

wxIconBundle 类同样也是一个图片列表,不过这个类的目的是为了将多个不同分辨率的图标保存在一个类中而不是多个类中,以便系统在合适的时候根据不同的使用目的选择一个特定的图标.比如,在资源管理器中的图标通常比在主窗口标题栏上显示的图标要大的多.下面的例子演示了其用法:

```cpp
// 创建一个只有单个 16x16 图标的图片集
#include "file16x16.xpm"
wxIconBundle iconBundle(wxIcon(file16x16_xpm));
// 在图片集中增加一个 32x32 的图片
iconBundle.Add(wxIcon(wxT("file32x32.png"), wxBITMAP_TYPE_PNG));
// 从一个包含多个图片的文件中创建一个图片集
wxIconBundle iconBundle2(wxT("multi-icons.tif"), wxBITMAP_TYPE_TIF);
// 从图片集中获取指定大小的图片,如果找不到则继续寻找
// wxSYS_ICON_X, wxSYS_ICON_Y 大小的图片
wxIcon icon = iconBundle.GetIcon(wxSize(16,16));
// 将图片集指定给某个主窗口
wxFrame* frame = new wxFrame(parent, wxID_ANY);
frame->SetIcons(iconBundle); 
```

在 windows 系统上,SetIcons 函数期待一个包含 16x16 和 32x32 大小的图标的图标集.

# 10.7 自定义 wxWidgets 提供的小图片

wxArtProvider 这个类允许你更改 wxWidgets 默认提供的那些小图片,比如 wxWidgets HTML 帮助阅读器中或者默认的 Log 对话框中使用的图片.

wxWidgets 提供了一个标准的 wxArtProvider 对象,并且体系内的一些需要使用图标和小图片的地方都调用了这个类的 wxArtProvider::GetBitmap 和 wxArtProvider::GetIcon 函数.

小图片是由两个标识符决定的:主标识符(wxArtID)和客户区标识符(wxArtClient).其中客户区标识符只在同一个主标识符在不同的窗口中需要不同的图片的时候才使用,比如,wxHTML 帮助窗口使用的图标使用下面的代码取得的:

```cpp
wxBitmap bmp = wxArtProvider::GetBitmap(wxART_GO_BACK,wxART_TOOLBAR); 
```

如果你想浏览所有 wxWidgets 提供的小图片以及它们的标识符,你可以编译和运行 wxWidgets 自带的 samples/artprov 中的例子,它的外观如下图所示:

![](img/mhtECAD%281%29.tmp)

要替换 wxWidgets 提供的这些小图片,你需要实现一个 wxArtProvider 的派生类,重载其中的 CreateBitmap 函数,然后在 OnInit 函数中调用 wxArtProvider::PushProvider 以便让 wxWidgets 知道.下面的这个例子替换了 wxHTML 帮助窗口中的大部分默认的图标:

```cpp
// 新的图标
#include "bitmaps/helpbook.xpm"
#include "bitmaps/helppage.xpm"
#include "bitmaps/helpback.xpm"
#include "bitmaps/helpdown.xpm"
#include "bitmaps/helpforward.xpm"
#include "bitmaps/helpoptions.xpm"
#include "bitmaps/helpsidepanel.xpm"
#include "bitmaps/helpup.xpm"
#include "bitmaps/helpuplevel.xpm"
#include "bitmaps/helpicon.xpm"
#include "wx/artprov.h"
class MyArtProvider : public wxArtProvider
{
protected:
    virtual wxBitmap CreateBitmap(const wxArtID& id,
                                  const wxArtClient& client,
                                  const wxSize& size);
};
// 新的 CreateBitmap 函数
wxBitmap MyArtProvider::CreateBitmap(const wxArtID& id,
                                     const wxArtClient& client,
                                     const wxSize& size)
{
    if (id == wxART_HELP_SIDE_PANEL)
        return wxBitmap(helpsidepanel_xpm);
    if (id == wxART_HELP_SETTINGS)
        return wxBitmap(helpoptions_xpm);
    if (id == wxART_HELP_BOOK)
        return wxBitmap(helpbook_xpm);
    if (id == wxART_HELP_FOLDER)
        return wxBitmap(helpbook_xpm);
    if (id == wxART_HELP_PAGE)
        return wxBitmap(helppage_xpm);
    if (id == wxART_GO_BACK)
        return wxBitmap(helpback_xpm);
    if (id == wxART_GO_FORWARD)
        return wxBitmap(helpforward_xpm);
    if (id == wxART_GO_UP)
        return wxBitmap(helpup_xpm);
    if (id == wxART_GO_DOWN)
        return wxBitmap(helpdown_xpm);
    if (id == wxART_GO_TO_PARENT)
        return wxBitmap(helpuplevel_xpm);
    if (id == wxART_FRAME_ICON)
        return wxBitmap(helpicon_xpm);
    if (id == wxART_HELP)
        return wxBitmap(helpicon_xpm);

    // Any wxWidgets icons not implemented here
    // will be provided by the default art provider.
    return wxNullBitmap;
}
// 你的初始化函数
bool MyApp::OnInit()
{
    ...
    wxArtProvider::PushProvider(new MyArtProvider);
    ...
    return true;
} 
```

# 第十章小结

在这一章里,我们学习了怎样使用 wxWidgets 中的图片相关的类 wxBitmap, wxIcon, wxCursor 和 wxImage,还学习了怎样使用 wxImageList 和 wxIconBundle,以及怎样定义 wxWidgets 默认使用的小图片.更多相关的例子请参考 wxWidgets 自带的 samples/image, samples/listctrl 和 samples/dragimag 目录中的例子.

在下一章里,我们将介绍一下怎样使用剪贴板来传输数据以及怎样实现拖放编程.