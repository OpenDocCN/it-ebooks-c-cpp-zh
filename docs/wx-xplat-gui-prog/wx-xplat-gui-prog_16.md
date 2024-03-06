# 第十六章编写国际化程序

# 第十六章编写国际化程序

如果你象要求你的程序支持多平台一样要求你的程序支持多语言,那么你的程序可能会有更多的用户,这将很大程度的提高你的程序成功的机会.本章我们就来谈一谈怎样实现这样的目标,要让你的应用程序成为国际化的程序,你需要用到的知识,也就是俗称的"i18n"(没错,i18n).

# 16.1 国际化介绍

# 16.1 国际化介绍

当把你的程序带到国际化舞态上时,想到的第一件事情就是翻译.你需要为你的应用程序中的每一个字符串针对每一种可能用到的语言准备一个翻译字符串.在 wxWidgets 中,这是通过使用 wxLocale 类来加载某个分类条目来实现的.这种技术也许和你以前用过的字符串表的方式不太一样,字符串表的方式是给程序中的每个字符串一个标识符,然后通过切换不同的字符串表来实现翻译的.而分类条目则是通过把字符串按照某一种分类进行翻译的作法(你当然可以使用你的本地系统支持的任何方法,不过要注意,在 wxWidgets 库内部,也使用的是分类条目的方法).

如果没有指定分类条目,源代码中的字符串将被显示在用户界面的菜单或者按钮等控件上.如果你的本地语言包含非 ASCII 字符,你就必须进行翻译(使用一个分类条目),因为源代码只支持 ASCII 码.

代表另外一种语言的文本可以采用各种各样的编码方式,这意味着同一个字节在不同的语言中可能代表不同的字符.你需要确定你的应用程序能够正确识别这些编码并且你的 GUI 控件可以正确显示这些编码.也就是说,你需要注意各种情况下使用的编码方式以及不同编码方式之间的互相转换的问题.

国际化的另外一个方面是格式化数字,日期和时间.注意即使是在同一种语言中,它们也有可能不同.比如英语中的数字 1,234.56 在德语中被写作 1.234,56,而在瑞士德语中则被写作 1'234.56.在美国 11 月 10 日的表示方法是 11/10,而同样的写法在 UK 则代表 10 月 11 日.我们很快会介绍 wxWidgets 如何处理这些复杂的情况.

翻译的字符串有时候会比代码中的英文字符串要长,这意味着窗口控件的布局需要调整为不同的大小.在第七章(使用布局控件进行布局)中介绍的布局控件可以解决这个问题.另外一个关于布局的问题是,英语是从左向右阅读的,而某些语言,比如阿拉伯语和希伯来语是从右到左的顺序读的(称为 RTL),目前 wxWidgets 中对于这个问题还没有一个很好的解决机制.

最后一个需要根据不同的语言进行调整的项目是你用到的图片和声音.比如说,你正在制作一个电话目录程序,它有一个特性是可以读出来电话号码,这个读音是语言有关的,另外你也可能需要根据国家的不同使用不同的图片.

# 16.2 从翻译说起

# 16.2 从翻译说起

wxWidgets 是通过 wxLocale 来提供语言翻译支持的.而且它自己也已经被完整的支持了很多种语言.请从 wxWidgets 的官方网站下载最新的翻译包.

wxWidgets 提供的国际化语言支持和 GNU 的 gettext 工具包非常的相似.wxWidgets 使用的分类条目机制和 gettext 的分类条目机制是二进制兼容的,意味着你可以使用所有的 gettext 工具.并且在运行时不需要别的任何附加的库支持因为 wxWidgets 内部实现了对条目文件的读取.

在程序开发过程中,你需要 gettext 工具包来操作分类条目(或者 poEdit 工具,参考下面的小节).有两种类型的消息分类条目文件,一种是源文件,它是文本格式文件,扩展名为.po,另外一种是二进制分类条目文件,它是通过分类条目源文件使用 gettext 工具包中的工具 msgfmt 创建的,扩展名是.mo. 在程序运行过程中只需要二进制的分类条目文件.针对每一种你想要支持的语言,你都需要单独提供种分类条目文件.

poEdit

你不需要使用 gettext 提供的命令行工具来维护你的分类条目.Vaclav Slavik 制作了一个叫做 poEdit 的工具,它是一个图形化的 gettext 前端工具,可以从 [`www.poedit.org`](http://www.poedit.org) 下载.运行界面如下图所示.它可以用来帮助你维护分类条目,产生.mo 文件以及随着你的代码的更改和增加将新的需要翻译的条目合入旧的分类条目文件.

![](img/mht3E25%281%29.tmp)

一步一步介绍创建消息翻译分类条目

遵循下面的这些步骤来进行某个消息条目的建立:

1.  在你的代码中,将需要翻译的字符串常量使用 wxGettranslation 宏或者它的简短替代品 _()宏括起来.对于那些不需要翻译的字符串,也请使用 wxT()或者等价的 _T()宏括起来,以便其可以实现 Unicode 兼容.
2.  将你在代码中标识为需要翻译的字符串整理到.po 文件中.当然,这么负责的步骤你不需要使用手工去作它,你可以使用 gettext 工具包中的 xgettext 工具,然后使用"-k"参数指定到底翻译 wxGettranslation 宏还是 _()宏指代的字符串. poEdit 工具也可以帮你完成这个工作,和 xgettext 的"-k"参数等价的功能需要在配置对话框中指定.
3.  将整理好的.po 文件中的字符串翻译为你希望使用的那种语言的字符串,每种语言对应一个.po 文件,并且要指定这种语言使用的编码方式.

    如果没有使用 poEdit 工具,你可以使用任何你喜欢的文本编辑器来完成这个工作,对应的.po 文件的文件头如下所示:

    ```cpp
    # SOME DESCRIPTIVE TITLE.
    # Copyright (C) YEAR Free Software Foundation, Inc.
    # FIRST AUTHOR &lt;EMAIL@ADDRESS&gt;, YEAR.
    #
    msgid ""
    msgstr ""
    "Project-Id-Version: PACKAGE VERSION\n"
    "POT-Creation-Date: 1999-02-19 16:03+0100\n"
    "PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
    "Last-Translator: FULL NAME &lt;EMAIL@ADDRESS&gt;\n"
    "Language-Team: LANGUAGE &lt;LL@li.org&gt;\n"
    "MIME-Version: 1.0\n"
    "Content-Type: text/plain; charset=iso8859-1\n"
    "Content-Transfer-Encoding: 8bit\n" 
    ```

    注意倒数第二行的 charset 属性,指定了这个分类条目文件使用的编码方式.本文件中所有的字符串都将采用这种编码方式,这是非常重要的,因为如果你使用了非 Unicode 的编码而没有指定其编码方式,那些 GUI 控件将不知道怎样显示你翻译的文本.

4.  将翻译好的.po 文件编译为二进制的.mo 文件.这要用到一个工具叫做 msgfmt,poEdit 也可以帮你完成这个工作. 使用 msgfmt 的命令行如下所示:

    ```cpp
    msgfmt -o myapp.mo myapp.po 
    ```

5.  在你的代码中设置合适的 locale 参数以便它使用对应的分类条目(我们将在接下来的小节,"使用 wxLocale"中介绍相关内容).

在 Mac OS X 系统上,你还需要更改一个叫做 Info.plist 的文件,这个文件是用来描述你的软件包的相关信息的.它是一个使用 UTF-8 编码的 Xml 文件,其中包含一个叫做 CFBundleDevelopmentRegion 的条目用来描述软件的开发语言(比如说:英语),而 Mac OSX 将会查询软件包中某些默认路径来找到这个软件支持的语言.例如,如果存在目录 German.lproj,则认为这个软件支持德语.因为 wxWidgets 并不使用这样的目录名称,你需要在这个文件中显式的指定你的应用程序支持的语言类型.这可以通过在其中增加 CFBundleLocalizations 条目来实现,如下所示:

```cpp
<key>CFBundleDevelopmentRegion</key>
<string>English</string>
<key>CFBundleLocalizations</key>
<array>
       <string>en</string>
       <string>de</string>
       <string>fr</string>
</array> 
```

使用 wxLocale

wxLocale 封装了所有和本地化相关的设置,类似于 C 语言中的 locale 的概念.通常你在你的应用程序类中定义一个 wxLocale 类型的成员,比如说:m_locale,然后在你的应用程序的 OnInit 函数中,象下面这样初始化这个变量:

```cpp
if (m_locale.Init(wxLANGUAGE_DEFAULT,
                    wxLOCALE_LOAD_DEFAULT | wxLOCALE_CONV_ENCODING))
{
    m_locale.AddCatalog(wxT("myapp"));
} 
```

注意 wxLocale::Init 函数将会查找 wxstd.mo 文件,这个文件代表 wxWidgets 自己的翻译分类条目.参数 wxLANGUAGE_DEFAULT 指定使用系统默认的语言,你也可以通过使用对应的 wxLANGUAGE_xxx 宏来强制指定某种特定的语言.

当 wxLocale 加载一个翻译条目的时候,这个条目被自动从它自己的编码转换成系统当前使用的编码,这是 wxLocale 的默认行为,如果你不希望使用这个行为,你可以在调用 wxLocale::Init 时不传递 wxLOCALE_CONV_ENCODING 标记.

wxWidgets 在哪些目录里寻找对应的.mo 文件呢?对于任何一个待查目录<DIR>,它的查找范围包括下面的这些目录:

```cpp
<DIR>/<LANG>/LC_MESSAGES
<DIR>/<LANG>
<DIR> 
```

到底哪些目录是待查目录,依系统的不同各不相同:

*   在所有的平台上,LC_PATH 环境变量指定的目录将成为待查目录.
*   在 Unix 或 Mac OS X 上,wxWidgets 的安装路径将成为待查目录,另外还有/share/locale, /usr/share/locale, /usr/lib/locale, /usr/locale /share/locale 以及当前目录.
*   在 Windows 平台上,应用程序所在的目录也将成为待查目录.

你还可以通过函数 wxLocale::AddCatalogLookupPathPrefix 增加你自己的待查目录,比如:

```cpp
wxString resDir = GetAppDir() + wxFILE_SEP_PATH + wxT("resources");
m_locale.AddCatalogLookupPathPrefix(resDir);
// 假设 resDir 是 c:\MyApp\resources, 假设当前使用法语,
// AddCatalog 将查找的待查目录除了前面介绍的所有待查目录外,
// 还将额外查找下面的路径:
//
// c:\MyApp\resources\fr\LC_MESSAGES\myapp.mo
// c:\MyApp\resources\fr\myapp.mo
// c:\MyApp\resources\myapp.mo
m_locale.AddCatalog(wxT("myapp")); 
```

通常在发行你的应用程序的时候,你应该为每个语言创建一个子目录,目录名称使用代码某种区域的标准的国际化名称,然后在其中放置对应的 <appname>.mo.比如,wxWidgets 自带的 internat 例子程序将 ISO639 格式编码的法语和德语翻译文件分别放置在 fr 和 de 目录中.

# 16.3 字符编码和 Unicode

# 16.3 字符编码和 Unicode

这个世界上有太多太多的字符,远超过了一个字节(8bit)可能容纳的 256 个数目.为了显式超过 256 个字符以外的其它字符,一个新的手段被增加进来,那就是字符编码和字符集(更新和更好的"Unicode"解决方案,我们也将很快谈到.).

因此,到底字节 161 代表什么字符,是由当前使用的字符集决定的.在 ISO 8859-1(Latin-1)字符集中,它代表的是一个倒写的感叹号,而在 ISO 8859-2 字符集中,则代表的是字母 a(Aogonek).

当你在一个窗口上绘制字符的时候,系统必须知道你使用的编码,这成为字体编码,也就是所谓的字符集.创建一个没有指定字符集的字体意味着使用默认编码,这在大多数系统上都是没有问题的,因为大多数人都在使用支持本国语言的系统.

但是,如果你确定某些字符使用的是不同的编码(比如 ISO 8859-2),在创建字体的时候,你应该指定这种编码,如下所示:

```cpp
wxFont myFont(10, wxFONTFAMILY_DEFAULT, wxNORMAL, wxNORMAL,
               false, wxT("Arial"), wxFONTENCODING_ISO8859_2); 
```

否则,在一个西文系统 ISO 8859-1 中,字符将不能被正确显式.

有时候可能我们无法找到一个合适的满足某种编码的字体,这种情况下,我们可以尝试使用一种代替字体,不过你需要将要显式的字体转换成那种代替字体对应的编码方式.下面的代码演示了应该怎样作.一个字符串 text 的编码为 enc,准备用字体 facename 显示.同时下面的代码也演示了 wxCSConv 的用法:

```cpp
// 我们有一段'enc'编码的文本,我们希望用字体
// 'facename'显示.
//
// 首先,我们必须确定这个字体可以显示这种编码
wxString text; // 编码方式为 'enc'
if (!wxFontMapper::Get()->IsEncodingAvailable(enc, facename))
{
   // 不能支持这种编码,需要查找替代编码.
   // 能支持某种替代编码吗?
   wxFontEncoding alternative;
   if (wxFontMapper::Get()->GetAltForEncoding(enc, &alternative,
                                              facename, false))
   {
       // 我们找到了替代编码方案'alternative',
       // 因此我们进行编码的转换,转换成 alternative.
       wxCSConv convFrom(wxFontMapper::GetEncodingName(enc));
       wxCSConv convTo(wxFontMapper::GetEncodingName(alternative));
       text = wxString(text.wc_str(convFrom), convTo) ;
       // 然后创建 alternative 编码的字体
       wxFont myFont(10, wxFONTFAMILY_DEFAULT, wxNORMAL, wxNORMAL,
               false, facename , alternative);
       dc.SetFont(myFont);
   }
   else
   {
      // 不能找到完美替代编码;尝试有损耗的编码方案
      // ISO 8859-1 (7-bit ASCII)
      wxFont myFont(10, wxFONTFAMILY_DEFAULT, wxNORMAL, wxNORMAL,
              false, facename, wxFONTENCODING_ISO8859_1);
      dc.SetFont(myFont);
    }
}
else
{
    // OK,这个字体可以支持这个编码.
     wxFont myFont(10, wxFONTFAMILY_DEFAULT, wxNORMAL, wxNORMAL,
               false, facename, enc);
     dc.SetFont(myFont);
}
// 最后,我们使用选择的字体绘制可能已经经过编码转换的字符串.
dc.DrawText(text, 100, 100); 
```

转换数据

前面的代码演示了将一组字节流从一种编码转换为另外一种编码的方法.这种转换可以有两种方法,第一种是使用 wxEncodingConverter 类,这种方法是不被推荐的(可能在后续版本种被淘汰的方法),你不应该在新的代码种使用这种方法,除非你的编译器不支持 wchar_t 结构. 推荐使用第二种方法,字符集转换(使用基于 wxMBConv 的 wxCSConv).

wxEncodingConverter

这种方法只能支持部分的字符集,但是如果你的编译器不支持 wchar_t 结构,这是你唯一的选择,转换方法如下:

```cpp
wxEncodingConverter converter(enc, alternative, wxCONVERT_SUBSTITUTE);
text = converter.Convert(text); 
```

wxCONVERT_SUBSTITUTE 标记表明允许转换过程中如果找不到严格对应的字符,允许存在信息损失, 这将导致带重音符号的字母变成普通的字母或者短破折号和长破折号统一用"-"来代替等.

wxCSConv (wxMBConv)

Unicode 的解决方案的核心是,它使用 16bit 或者甚至是 32bit 的 wchar_t 结构来代表一个字符,因此它可以把全世界所有的字符用一种编码表示.这意味着你不需要处理任何编码转换之类的问题除非你需要处理老的 8-bit 格式数据,前面我们已经说过,8bit 的数据必须和字符集一起使用才有意义.

即使你没有把 wxWidgets 编译成 Unicode 模式(这种模式下,所有的字符串都是 Unicode 编码格式),只要你的系统支持,你还是可以使用它进行编码转换.转换的方法是,先把你的字符串从它的编码转换成 Unicode 编码,然后再从 Unicode 编码转换成目标编码. wxString 类也使用这种方法来提供编码转换支持.要记住的是:非 Unicode 版本的 wxWidgets 中的 wxString 对象采用的是 8bit 的方法保存字符串,因此它自己并不知道其内部的数据使用的是什么编码方式.

如果想把 wxString 转换成 Unicode,你需要使用 wxString::wc_str 函数,这个函数采用一个多字节转换类作为它的参数,这个参数告诉非 Unicode 版本的 wxString 它内部的字符串是采用什么编码方式的,但是在 Unicode 版本的 wxWidgets 中, 这个参数被忽略,因为 wxString 内部的编码已经是 Unicode 了.

在 Unicode 版本中,我们可以直接使用 wx_str 返回的字符串了,但是在非 Unicode 版本中,我们还需要将其转换为我们可以支持的编码方式 convTo,因此在下面的代码中,在 Unicode 版本中,convTo 也将被忽略:

```cpp
text = wxString(text.wc_str(convFrom), convTo); 
```

可以看到字符集编码比字体字体编码更常使用,因此有时候你需要通过下面的代码将字体编码名字装换成字符集编码名字:

```cpp
wxFontMapper::GetEncodingName(fontencoding); 
```

这就是上面例子中下面这一部分代码的含义:

```cpp
wxCSConv convFrom(wxFontMapper::GetEncodingName(enc));
wxCSConv convTo(wxFontMapper::GetEncodingName(alternative));
text = wxString(text.wc_str(convFrom) , convTo) ; 
```

有时候你需要直接使用 8bit 的字节流而不是使用 wxString,这可以通过使用 wxCharBuffer 类获得,下面我们看看这一行代码:

```cpp
wxCharBuffer output = convTo.cWC2MB(text.wc_str(convFrom)); 
```

如果你的输入数据不是一个字符串而也是一个 8bit 的数据流(比如也是一个 wxCharBuffer),你可以使用下面的转换方式:

```cpp
wxCharBuffer output = convTo.cWC2MB(convFrom.cMB2WC(input)); 
```

wxWidgets 定义了一些全局的类用于实现字符转换,比如 wxConvISO8859_1 是一个对象,而 wxConvCurrent 是一个指针,指向当前标准 C 的 locale 指定的编码类.另外还有一些 wxMBConv 的子类用来优化特定的编码转换任务,比如 wxMBConvUTF7,wxMBConvUTF8, wxMBConvUTF16LE/BE 和 wxMBConvUTF32LE/BE.其中后两个被重定义为 wxMBConvUFT16/32,它使用机器本身的字节序.更多信息请参考 wxWidgets 手册中的"wxMBConv Classes Overview"小节.

转化来自外部的临时缓存数据

正如我们刚刚讨论的那样,转换类允许你很方便的把一种字符集转换为另外一种字符集.然而,大多数的转换结果为一个新创建的字符串或者一个临时缓存.有时候我们需要将转换的结果保存起来已备以后使用,这种情况下我们可以把转换的结果复制到一个独立的存储区.

假设我们想在两个电脑之间通过 socket 传递字符串.我们首先应该在字符串采用的编码上取得一致.否则,平台默认的编码可能把传递的字符串搞的一团糟.在我们的这个例子中,我们把发送出去的字符串先转换成 UTF-8 编码,在接收的部分,在将 UTF-8 编码的字符串转换成系统默认的字符串.

下面的代码演示了怎样将符合本地编码的字符串转换成 UTF-8,将转换结果存储在一个 char*指针中,然后通过 socket 发送出去,接收的电脑再将收到的字符串从 UTF-8 转换成它自己的电脑上的本地编码.

```cpp
// 将本地编码字符串转换成 UTF-8 编码
const wxCharBuffer ConvertToUTF8(wxString anyString)
{
    return wxConvUTF8.cWC2MB( anyString.wc_str(*wxConvCurrent) ) ;
}
// 将 UTF-8 编码的字符串转换成本地编码字符串
wxString ConvertFromUTF8(const char* rawUTF8)
{
    return wxString(wxConvUTF8.cMB2WC(rawUTF8), *wxConvCurrent);
}
// 测试以下这两个转换函数
void StringConversionTest(wxString anyString)
{
    // 转化成 UTF-8 编码并保存在 wxCharBuffer 中.
    const wxCharBuffer bUTF8 = ConvertToUTF8(anyString);
    // wxCharBuffer 可以隐式的转换成 char*.
    const char *cUTF8 = bUTF8 ;
    // 重建字符串
    wxString stringCopy = ConvertFromUTF8(cUTF8);
    // 因为是同一个电脑,这两个字符串应该是完全相同的.
    wxASSERT(anyString == stringCopy);
} 
```

帮助文件

你需要为每个支持的语言制作一份帮助文件.你的帮助文件控制器在初始化的时候将指定帮助文件的名称.你可以使用 wxLocale::GetName 来获取语言相关的名称,也可以直接使用前面介绍的 _()宏以便获得语言相关的名称.比如:

```cpp
m_helpController->Initialize(_("help_english")); 
```

如果你使用的是 wxHtmlHelpController,记住你需要给每一个帮助页面指定 META 标记,如下所示:

```cpp
<meta http-equiv="Content-Type" content="text/html; charset=iso8859 //2"> 
```

你还需要注意帮助工程文件(扩展名.hhp)也许要包含一个指定编码的选项行:

```cpp
Charset=iso8859-2 
```

这个额外的条目告诉 HTML 帮助控制器帮助内容和帮助索引使用什么编码格式编码的.

# 16.4 数字和日期

# 16.4 数字和日期

本地化程序中的另外一个方面是格式化数字和日期,对于数字,基于 printf 的 wxString 格式化函数已经在内部实现了针对不同地域的本地化,如下面的代码所示:

```cpp
wxString::Format(wxT("%.1f") , myDouble); 
```

这里,Format 函数将会根据你设置的 locale 帮你处理地域差异. 而下面的日期格式化代码:

```cpp
wxDateTime t = wxDateTime::Now();
wxString str = t.Format(); 
```

Format 函数也将根据你设置的 locale 进行合适的格式化操作.在 wxWidgets 手册中时间和日期函数格式化的相关部分详细的介绍了怎样根据自定义的格式进行时间和日期的格式化.在这种情况下,你只需要将格式化文本使用 _()宏包括起来,然后针对不同的语言翻译成对应的本地格式就可以了.

如果你想知道当前设置的 locale 对应的数字分割符或者别的一些本地化相关的值,可以使用 wxLocale 的 GetInfo 函数,比如下面的代码返回当前设置的 locale 下数字的 10 进制分割符:

```cpp
wxString info = m_locale.GetInfo(wxLOCALE_THOUSANDS_SEP,
                                   wxLOCALE_CAT_NUMBER) ; 
```

# 16.5 其它媒介

# 16.5 其它媒介

你可以使用和文本同样的机制来针对不同的语言加载不同的声音和图片,如下所示:

```cpp
wxBitmap bitmap(_("flag.png")); 
```

上面的代码将导致 flag.png 在运行期被翻译,因此你可以将其针对不同的语言翻译成不同的文件,比如 de/flag.png.当然,你需要保证这是一个存在的真实文件.你也可以使用第十四章,"文件和流"中介绍的技术将其翻译为一个虚拟文件系统路径.

# 16.6 一个小例子

# 16.6 一个小例子

为了演示本章介绍的这些内容,随书光盘上 examples/chap16 目录中举了一个小例子.它以三种语言显示了一些字符串和图片:英语,法语和德语. 你可以从文件菜单更改当前的语言,这将导致菜单字符串,静态文本控件和使用的图片作出相应的改变.为了演示 _()宏和 wxT()的区别,状态栏的字符串始 终保持英语不变.

![](img/mht6FD4%281%29.tmp)

这个例子的应用程序类包含一个指向 wxLocale 类型的指针和一个函数 SelectLanguage 用来更改当前的语言.主要的声明和实现如下:

```cpp
class MyApp : public wxApp
{
public:
    ~MyApp() ;
    // 初始化应用程序
    virtual bool OnInit();
    // 根据用户选择的语言重新创建 wxLocale 变量
    void SelectLanguage(int lang);
private:
    wxLocale* m_locale; // 'our' locale
};
IMPLEMENT_APP(MyApp)
bool MyApp::OnInit()
{
    wxImage::AddHandler( new wxPNGHandler );
    m_locale = NULL;
    SelectLanguage( wxLANGUAGE_DEFAULT );
    MyFrame *frame = new MyFrame(_("i18n wxWidgets App"));
    frame->Show(true);
    return true;
}
void MyApp::SelectLanguage(int lang)
{
    delete m_locale;
    m_locale = new wxLocale( lang );
    m_locale->AddCatalog( wxT("i18n") );
}
MyApp::~MyApp()
{
    delete m_locale;
} 
```

主窗口的两个函数 SetupStrings 和 OnChangeLanguage 可能是你最感兴趣的部分,SetupStrings 更改相关控件的字符串并且重新创建菜单条,以便演示更改 wxLocale 以后相关字符串的翻译:

```cpp
void MyFrame::SetupStrings()
{
    m_helloString->SetLabel(_("Welcome to International Sample"));
    m_todayString->SetLabel( wxString::Format(_("Now is %s") , wxDateTime::Now().Format()
.c_str() ) );
    m_thousandString->SetLabel( wxString::Format(_("12345 divided by 10 is written as %
.1f") , 1234.5 ) );
    m_flag->SetBitmap(wxBitmap( _("flag.png") , wxBITMAP_TYPE_PNG ));
    // 创建菜单条
    wxMenu *menuFile = new wxMenu;
    // About 菜单应该位于帮助菜单
    wxMenu *helpMenu = new wxMenu;
    helpMenu->Append(wxID_ABOUT, _("&About...\tF1"),
                     wxT("Show about dialog"));
    menuFile->Append(wxID_NEW, _("Change language..."),
                     wxT("Select a new language"));
    menuFile->AppendSeparator();
    menuFile->Append(wxID_EXIT, _("E&xit\tAlt-X"),
                     wxT("Quit this program"));
    wxMenuBar *menuBar = new wxMenuBar();
    menuBar->Append(menuFile, _("&File"));
    menuBar->Append(helpMenu, _("&Help"));
    wxMenuBar* formerMenuBar = GetMenuBar();
    SetMenuBar(menuBar);
    delete formerMenuBar;
    SetStatusText(_("Welcome to wxWidgets!"));
} 
```

OnChangeLanguage 在用户更改当前语言的时候被调用,它将用户的选择映射到某种语言标识(比如 wxLANGUAGE_GERMAN)上.这个标识被传递给 MyApp::SelectLanguage 以便设置当前的 locale,然后调用 SetupStrings 根据设置的 locale 更改当前的字符串和图片,如下所示:

```cpp
void MyFrame::OnChangeLanguage(wxCommandEvent& event)
{
    wxArrayInt languageCodes;
    wxArrayString languageNames;
    languageCodes.Add(wxLANGUAGE_GERMAN);
    languageNames.Add(_("German"));
    languageCodes.Add(wxLANGUAGE_FRENCH);
    languageNames.Add(_("French"));
    languageCodes.Add(wxLANGUAGE_ENGLISH);
    languageNames.Add(_("English"));
    int lang = wxGetSingleChoiceIndex( _("Select language:"),
                             _("Language"), languageNames );
    if ( lang != -1 )
    {
        wxGetApp().SelectLanguage(languageCodes[lang]);
        SetupStrings();
    }
} 
```

# 第十六章小结

# 第十六章小结

本章我们介绍了怎样处理字符串,时间,日期,货币等在不同语言之间的翻译.最后的忠告是:你应该和一个熟悉那个国家语言的人一起作这个事情,才能了解到不同语言之间一些细微的差别.

wxWidgets 自带的 samples/internat 目录中也有一个相关的例子,它将使用到的字符串翻译成了十种语言.

下一章里,我们将学习怎样让你的应用程序通过多线程的方法同时处理多个任务.