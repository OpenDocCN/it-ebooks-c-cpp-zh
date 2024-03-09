# 第十四章文件和流操作

在这一章里,我们来看看 wxWidgets 提供的底层文件操作对象以及流操作.wxWidgets 的流对象不但能使得你的应用程序可以和各种标准的 C++ 库打交道,而且还提供完整的压缩,写 zip 文档以及 socket 读写等操作.我们还将描述一下 wxWidgets 提供的虚拟文件系统,它让你的应用程序可以很容易的从非常规文件中获取各种资源.

# 14.1 文件类和函数

wxWidgets 提供了一系列的平台无关的文件处理功能.在概览所有文件函数之前,我们先来看看几个主要的类.

wxFile 和 wxFFile

wxFile 类可以用来处理底层的文件输入输出.它封装了常用的用于操作整数文件标识符的标准 C 操作(打开关闭,读写,移动游标等),但是和标准 C 不同的是,它使用 wxLog 类来报告错误并且在析构函数中自动关闭文件.而 wxFFile 则提供缓冲的输入输出操作,内部使用的是 FILE 类型的指针.

你可以通过下面的方法创建一个 wxFile 对象:使用默认构造函数然后调用 Create 或者 Open 函数;或者直接在构造函数中指明文件名和打开模式(wxFile::read, wxFile::write 或 wxFile::read_write);或者直接使用已经存在的整数文件描述符(相关构造函数或者默认构造函数加 Attach 函数).Close 函数关闭当前使用的文件,文件也将在析构函数中视需要进行关闭.

从文件中读取数据使用 Read 函数,参数为一个 void*和一个缓冲区大小,返回实际读取的数值大小或者 wxInvalidOffset 如果读取过程发生错误.使用 Write 函数将指定大小的 void*类型的缓冲区写入文件,如果你希望写操作立即写到物理文件,你需要使用 Flush 函数.

Eof 函数用来检测当前的文件指针是否位于文件结尾的位置(而 wxFFile 的 Eof 函数只有在其读操作越过了文件结尾的时候才返回 True).你可以用 Length 函数返回文件的长度.

Seek 和 SeekEnd 函数将文件指针移动到相对于文件开始或者文件结尾的一个偏移位置.Tell 函数返回 wxFileOffset 类型的当前文件指针位置,这个类型在支持 64 位操作系统上是 64 位整数,否则是 32 位整数.

Access 函数是一个静态函数,用来判断某个文件是否可以以指定的模式打开,而 Exists 函数则用来判断指定的文件是否存在.

下面的代码演示了怎样使用 wxFile 打开一个文件并且将其内容读取到一个数组中:

```cpp
#include "wx/file.h"
if (!wxFile::Exists(wxT("data.dat")))
    return false;
wxFile file(wxT("data.dat"));
if ( !file.IsOpened() )
    return false;
//文件大小
wxFileOffset nSize = file.Length();
if ( nSize == wxInvalidOffset )
    return false;
// 将所有内容读取到一个数组中
wxUint* data = new wxUint8[nSize];
if ( fileMsg.Read(data, (size_t) nSize) != nSize )
{
    delete[] data;
    return false;
}
file.Close(); 
```

下面的代码则演示了怎样将一个文本框的所有内容写入到文件中:

```cpp
bool WriteTextCtrlContents(wxTextCtrl* textCtrl,
                           const wxString& filename)
{
    wxFile file;
    if (!file.Open(filename, wxFile::write))
        return false
    int nLines = textCtrl->GetNumberOfLines();
    bool ok = true;
    for ( int nLine = 0; ok && nLine &lt; nLines; nLine++ )
    {
        ok = file.Write(textCtrl->GetLineText(nLine) +
                        wxTextFile::GetEOL());
    }
    file.Close();
    return ok;
} 
```

wxTextFile

wxTextFile 提供了一种非常直接的方式来以行为单位读取和写入小型的文本文件.

使用 Open 函数将这个文本文件读取到内存中并且以行为单位进行分割,使用 Write 函数写回到文本文件.你可以使用 GetLine 函数或者直接按照数组的方式操作某个特定的行.或者使用 GetFirstLine,GetNextLine 和 GetPrevLine 进行遍历.AddLine 和 InsertLine 用来增加新行,RemoveLine 用来移除特定的行,Clear 函数则用来清空所有的行.

下面的例子演示了将文本文件中的每一行都增加一个前导文本的方法:

```cpp
#include "wx/textfile.h"
void FilePrepend(const wxString& filename, const wxString& text)
{
    wxTextFile file;
    if (file.Open(filename))
    {
        size_t i;
        for (i = 0; i &lt; file.GetLineCount(); i++)
        {
            file[i] = text + file[i];
        }
        file.Write(filename);
    }
} 
```

wxTempFile

wxTempFile 是 wxFile 的一个派生类,它使用临时文件来写入数据,数据在 Commit 函数被调用之前不会被写入.如果你需要写一些用户数据,你可以将其写在临时文件里,它的好处是:如果遇到突然的断电或者应用程序不可知错误或者其它大的错误时,临时文件不会对磁盘上的文件系统造成任何伤害.

提示:文档/视图框架在创建一个输出流然后调用 SaveObject 的时候没有使用临时文件.你可以尝试重载其 DoSaveDocument 函数,在其中构建一个 wxFileOutputStream 并且让其使用一个临时文件 wxTempFile 对象,在全部数据写完以后,调用 Sync 函数和 Commit 函数将其写入临时文件.

wxDir

wxDir 是一个轻便的等价于 Unix 上的 open/read/closedir 函数的类,它支持枚举目录中的所有文件.wxDir 既支持枚举目录中的文件,也支持枚举目录中的子目录.它还提供了一个灵活的递归枚举文件的方法 Traverse 函数,和另外一种简单的方法: GetAllFiles 函数.

首先,你需要调用 Open 函数打开一个目录(或者通过构造函数直接赋值),然后调用 GetFirst 函数,参数为一个指向字符串类型的指针用来接收找到的文件名,一个可选的文件通配符(默认为所有文件)和一个可选的选项.然后调用 GetNext 函数直到其返回 False.文件通配符可以是固定的文件名以及包含"*(匹配任意字符)"和"?"(匹配单个字符)的通配符.选项参数可选的值为:wxDIR_FILES(所有文件), wxDIR_DIRS(所有文件夹),wxDIR_HIDDEN(隐藏文件)以及 wxDIR_DOTDOT("."和"..")以及它们的组合,默认值为除了最后一项的所有文件.

下面是一个例子:

```cpp
#include "wx/dir.h"
wxDir dir(wxGetCwd());
if ( !dir.IsOpened() )
{
    // 如果遇到这个情况,wxDir 已经显示了一个出错信息.
    // 所以直接返回就可以了
    return;
}
puts("Enumerating object files in current directory:");
wxString filename;
wxString filespec = wxT("*.*");
int flags = wxDIR_FILES|wxDIR_DIRS;
bool cont = dir.GetFirst(&filename, filespec, flags);
while ( cont )
{
    wxLogMessage(wxT("%s\n"), filename.c_str());
    cont = dir.GetNext(&filename);
} 
```

如同上面注释中说的那样,如果 wxDir 打开的时候出现错误,将会弹出一个错误消息,如果想禁止这个消息,你可以临时通过设置 wxLogNull 的方法,如下所示:

```cpp
{
    wxLogNull logNull;
    wxDir dir(badDir);
    if ( !dir.IsOpened() )
    {
        return;
    }
} 
```

wxFileName

wxFileName 用来处理文件名.它可以分解和组合文件名,还提供了很多额外的操作,其中某些为静态函数.下面演示了一些例子,更多的功能请参考 wxWidgets 的手册:

```cpp
#include "wx/filename.h"
// 使用字符串创建文件名
wxFileName fname(wxT("MyFile.txt"));
// Normalize,在 windows 平台上这个函数的动作包括
// 确保文件名为长文件名格式
fname.Normalize(wxPATH_NORM_LONG|wxPATH_NORM_DOTS|wxPATH_NORM_TILDE|
                wxPATH_NORM_ABSOLUTE);
// 返回全路径
wxString filename = fname.GetFullPath();
// 返回相对于当前目录的路径
fname.MakeRelativeTo(wxFileName::GetCwd());
// 文件存在吗?
bool exists = fname.FileExists();
// 另外一个文件存在吗?
bool exists2 = wxFileName::FileExists(wxT("c:\\temp.txt"));
// 返回文件名的名称部分
wxString name = fname.GetName();
// 返回路径部分
wxString path = fname.GetPath();
// 在 windows 系统上返回相应的短路径文件名,其它平台上
// 返回本身.
wxString shortForm = fname.GetShortPath();
// 创建一个文件夹
bool ok = wxFileName::Mkdir(wxT("c:\\thing")); 
```

File Functions

下表列出了一些有用的静态文件操作函数,它们定义在头文件 wx/filefn.h 中.请同时参考 wxFileName 类,尤其是其中的静态函数部分,比如 wxFileName::FileExists 函数.

| wxDirExists(dir) | 是否目录存在. 参考 wxFileName::DirExists |
| --- | --- |
| wxConcatFiles(f1, f2, f3) | 将 f1 和 f2 合并为 f3, 成功时返回 True. |
| wxCopyFile(f1, f2, overwrite) | 拷贝 f1 到 f2,可选择是否覆盖已存在的 f2.返回 Bool 型 |
| wxFileExists(file) | 测试是否文件存在. 参考 wxFileName::FileExists |
| wxFileModificationTime(file) | 返回文件修改时间(time_t 类型). 参考 wxFileName::GetModificationTime,它返回 wxDateTime 类型 |
| wxFileNameFromPath(file) | 返回文件全路径的文件名部分. 推荐使用 wxFileName::SplitPath 函数 |
| wxGetCwd() | 返回当前工作目录. 参考 wxFileName::GetCwd |
| wxGetdiskSpace (path, total, free) | 返回指定路径所在的磁盘的全部空间和空闲空间,空间为 wxLongLong 类型. |
| wxIsAbsolutePath(path) | 测试指定的路径是否为绝对路径. |
| wxMkdir(dir, permission=777) | 创建一个目录,其父目录必须存在,可选指定目录访问掩码.返回 bool 型 |
| wxPathOnly(path) | 返回给定全路径的目录部分. |
| wxRemoveFile(file) | 删除文件,成功时返回 True. |
| wxRenameFile(file1, file2) | 重命名文件,成功时返回 True.如果需要进行文件拷贝,则直接返回 False. |
| wxRmdir(file) | 移除空目录,成功时返回 True. |
| wxSetWorkingDirectory(file) | 设置当前工作目录,返回 bool 型. |

wxWidgets 同样提供了许多函数来封装标准 C 函数,比如: wxFopen, wxFputc 和 wxSscanf,这些函数没有在手册中记录,不过你应该可以在 include/wx/wxchar.h 中找到它们.

另外一个有用的宏是 wxFILE_SEP_PATH,它代表了不同平台上的路径分割符,比如在 windows 平台上它代表"\",而在 Unix 平台上则代表"/".

# 14.2 流操作相关类

所谓流模型，指的是一种用于提供相对于文件读写更高层的数据读写的模型.使用流模型,你的代码不需要关心当前操作的是文件,内存还是 socket(参考第十八章,"使用 wxSocket 编程",其中演示了用流方式使用 socket 的方法).某些 wxWidgets 标准类同时支持文件读写操作和流读写操作, 比如 wxImage 类.

wxStreamBase 类是所有流类的基类,它声明的函数包括类似 OnSysRead 和 OnSysWrite 等需要继承类实现的函数.其子类 wxInputStream 和 wxOutputStream 则提供了更具体的流类(比如 wxFileInputStream 和 wxFileOutputStream 子类)共同需要的用于读写操作的基本函数.让我们来具体来看一下 wxWidgets 提供的各种流操作相关类.

文件流

wxFileInputStream 和 wxFileOutputStream 是基于 wxFile 类实现的,可以通过文件名,wxFile 对象或者整数文件描述符的方式来进行初始化.下面的例子演示了使用 wxFileInputStream 来进行文件读取,文件指针移位并读取当前位置数据等.

```cpp
#include "wx/wfstream.h"
// 构造函数初始化流缓冲区并且打开文件名参数对应的文件.
// wxFileInputStream 将在析构函数中自动关闭对应的文件描述符.
wxFileInputStream inStream(filename);
// 读 100 个字节
int byteCount = 100;
char data[100];
if (inStream.Read((void*) data, byteCount).
             LastError() != wxSTREAM_NOERROR)
{
    // 发生了异常
    // 异常列表请参考 wxStreamBase 的相关文档.
}
// 你可以通过下面的方法判断到底成功读取了多少个字节.
size_t reallyRead = inStream.LastRead();
// 将文件游标移动到文件开始处.
// SeekI 函数的返回值为移动游标以前的游标相对于文件起始处的位置
off_t oldPosition = inStream.SeekI(0, wxFromBeginning);
// 获得当前的文件游标位置
off_t position = inStream.TellI(); 
```

使用 wxFileOutputStream 的方法也很直观.下面的代码演示了使用 wxFileInputStream 和 wxFileOutputStream 实现文件拷贝的方法.每次拷贝 1024 个字节.为了使代码更简介,这里没有显示错误处理的代码.

```cpp
// 下面的代码实现固定单位大小的流拷贝.
// 缓冲区的使用是为了加快拷贝的速度.
void BufferedCopy(wxInputStream& inStream, wxOutputStream& outStream,
                  size_t size)
{
    static unsigned char buf[1024];
    size_t bytesLeft = size;
    while (bytesLeft &gt; 0)
    {
        size_t bytesToRead = wxMin((size_t) sizeof(buf), bytesLeft);
        inStream.Read((void*) buf, bytesToRead);
        outStream.Write((void*) buf, bytesToRead);
        bytesLeft -= bytesToRead;
    }
}
void CopyFile(const wxString& from, const wxString& to)
{
    wxFileInputStream inStream(from);
    wxFileOutputStream outStream(to);
    BufferedCopy(inStream, outStream, inStream.GetSize());
} 
```

wxFFileInputStream 和 wxFFileOutputStream 跟 wxFileInputStream 和 wxFileOutputStream 的用法几乎完全一样, 不同之处在于前者是基于 wxFFile 类而不是 wxFile 类的. 因此它们的初始化方法也不同,相应的,前者可以使用 FILE 指针或 wxFFile 对象来初始化.文件结束处理相应的有所不同: wxFileInputStream 在最后一个字节被读取的时候报告 wxSTREAM_EOF, 而 wxFFileInputStream 则在最后一个字节以后进行读操作的时候返回 wxSTREAM_EOF.

内存和字符串流

wxMemoryInputStream 和 wxMemoryOutputStream 使用内部缓冲区来处理流数据.默认的构造函数都采用 char*类型的缓冲区指针和缓冲区大小作为参数.如果没有这些参数,则表明要求该类的事例自己进行动态缓冲区管理.我们很快会看到一个相关的例子.

wxStringInputStream 则采用一个 wxString 引用作为构造参数来进行数据读取. wxStringOutputStream 采用一个开选的 wxString 指针作为参数来进行数据写操作;如果构造参数没有指示 wxString 指针,则将构造一个内部的 wxString 对象,这个对象可以通过 GetString 函数来访问.

读写数据类型

到目前为止,我们描述的流类型都处理的是原始的字节流数据.在实际的应用程序中,这些字节流必须被赋予特定的函数.为了帮助你实现这一点,你可以使用下面四个类来以一个更高的层级处理数据.这四个类分别是:wxTextInputStream, wxTextOutputStream, wxDataInputStream 和 wxDataOutputStream.这些类通过别的流类型类构造,它们提供了操作更高级的 C++数据类型的方法.

wxTextInputStream 从一段人类可读的文本中获取数据.如果你使用的构造类为文件相关类,你需要自己进行文件是否读完的判断.即使这样,读到空数据项(长度为 0 的字符串或者数字 0)还是无法避免,因为很多文本文件都用空白符(比如换行符)来结束.下面的例子演示了怎样使用由 wxFileInputStream 构造的 wxTextInputStream:

```cpp
wxFileInputStream input( wxT("mytext.txt") );
wxTextInputStream text( input );
wxUint8 i1;
float f2;
wxString line;
text &gt;&gt; i1;       // 读一个 8bit 整数.
text &gt;&gt; i1 &gt;&gt; f2; // 先读一个 8bit 整数再读一个浮点数.
text &gt;&gt; line;     // 读一行文本 
```

wxTextOutputStream 则将文本数据写至输出流,换行符自动使用当前平台的换行符.下面的例子演示了将文本数据输出到标准错误输出流:

```cpp
#include "wx/wfstream.h"
#include "wx/txtstrm.h"
wxFFileOutputStream output( stderr );
wxTextOutputStream cout( output );
cout &lt;&lt; wxT("This is a text line") &lt;&lt; endl;
cout &lt;&lt; 1234;
cout &lt;&lt; 1.23456; 
```

wxDataInputStream 和 wxDataOutputStream 使用发放类似,但是它使用二进制的方法处理数据.数据使用可移植的方式存储因此能够作到平台无关.下面的例子分别演示了以这种方式从数据文件读取以及写入数据文件.

```cpp
#include "wx/wfstream.h"
#include "wx/datstrm.h"
wxFileInputStream input( wxT("mytext.dat") );
wxDataInputStream store( input );
wxUint8 i1;
float f2;
wxString line;
store &gt;&gt; i1;       // 读取一个 8bit 整数
store &gt;&gt; i1 &gt;&gt; f2; // 读取一个 8bit 整数,然后读取一个浮点数.
store &gt;&gt; line;     // 读取一行文本

#include "wx/wfstream.h"
#include "wx/datstrm.h"
wxFileOutputStream output(wxT("mytext.dat") );
wxDataOutputStream store( output );
store &lt;&lt; 2 &lt;&lt; 8 &lt;&lt; 1.2;
store &lt;&lt; wxT("This is a text line") ; 
```

Socket 流

wxSocketOutputStream 和 wxSocketInputStream 是通过 wxSocket 对象构造的,详情参见第十八章.

过滤器流对象

wxFilterInputStream 和 wxFilterOutputStream 是过滤器流对象的基类,过滤器流对象是一种特殊的流对象,它用来将过滤后的数据输入到其它的流对象.wxZlibInputStream 是一个典型的过滤器流对象.如果你在其构造函数中指定一个文件源为一个 zlib 格式的压缩文件的文件流对象,你可以直接从 wxZlibInputStream 中读取数据而不需要关心解压缩的机制.类似的,你可以使用一个 wxFileOutputStream 来构造一个 wxZlibOutputStream 对象,如果你将数据写入 wxZlibOutputStream 对象,压缩后的数据将被写入对应的文件中.

下面的例子演示了怎样将一段文本压缩以后存放入另外一个缓冲区中:

```cpp
#include "wx/mstream.h"
#include "wx/zstream.h"
const char* buf =
    "01234567890123456789012345678901234567890123456789";
// 创建一个写入 wxMemoryOutputStream 对象的 wxZlibOutputStream 类
wxMemoryOutputStream memStreamOut;
wxZlibOutputStream zStreamOut(memStreamOut);
// 压缩 buf 以后写入 wxMemoryOutputStream
zStreamOut.Write(buf, strlen(buf));
// 获取写入的大小
int sz = memStreamOut.GetSize();
// 分配合适大小的缓冲区
// 拷贝数据
unsigned char* data = new unsigned char[sz];
memStreamOut.CopyTo(data, sz); 
```

Zip 流对象

wxZipInputStream 是一个更复杂一点的流对象,因为它是以文档的方式而不是线性的二进制数据的方式工作的.事实上,文档是通过另外的类 wxArchiveClassFactory 和 wxArchiveEntry 来处理的,但是你可以不用关心这些细节.要使用 wxZipInputStream 类,你可以有两种构造方法,一种是直接使用一个指向 zip 文件的文件流对象,另外一种方法则是通过一个 zip 文件路径和一个 zip 文件中文档的路径来指定一个 zip 数据流.下面的例子演示了这两种方法:

```cpp
#include "wx/wfstream.h"
#include "wx/zipstrm.h"
#include "wx/txtstrm.h"
// 方法一: 以两步方式创建 zip 输入流.
wxZipEntry* entry;
wxFFileInputStream in(wxT("test.zip"));
wxZipInputStream zip(in);
wxTextInputStream txt(zip);
wxString data;
while (entry = zip.GetNextEntry())
{
    wxString name = entry->GetName();    // 访问元数据
    txt &gt;&gt; data;                         // 访问数据
    delete entry;
}
// 方法二: 直接指定源文档路径和内部文件路径.
wxZipInputStream in(wxT("test.zip"), wxT("text.txt"));
wxTextInputStream txt(zip);
wxString data;
txt &gt;&gt; data;                             // 访问数据 
```

wxZipOutputStream 用来写 zip 压缩文件. PutNextEntry 或 PutNextDirEntry 函数用来在压缩文件中创建一个新的文件(目录),然后就可以写相应的数据了.例如:

```cpp
#include "wx/wfstream.h"
#include "wx/zipstrm.h"
#include "wx/txtstrm.h"
wxFFileOutputStream out(wxT("test.zip"));
wxZipOutputStream zip(out);
wxTextOutputStream txt(zip);
zip.PutNextEntry(wxT("entry1.txt"));
txt &lt;&lt; wxT("Some text for entry1\n");
zip.PutNextEntry(wxT("entry2.txt"));
txt &lt;&lt; wxT("Some text for entry2\n"); 
```

虚拟文件系统

wxWidgets 提供了一套虚拟文件系统机制,让你的应用程序可以象使用普通文件那样使用包括 zip 文件中的文件,内存文件以及 HTTP 或 FTP 协议这样的特殊数据.不过,这种虚拟文件机制通常是只读的,意味着你不可以修改其中的内容.wxWidgets 提供的 wxHtmlWindow 类(用于提供 wxWidgets 内部的 HTML 帮助文件的显示)和 wxWidgets 的 XRC 资源文件机制都可以识别虚拟文件系统路径格式.虚拟文件系统的使用比起前面介绍的 zip 文件流要简单,但是后者可以更改 zip 文档的内容.除了内部都使用了流机制以外,这两者其实没有任何其它的联系.

各种不同的虚拟文件系统类都继承自 wxFileSystemHandler 类,要在应用程序中使用某个特定实现,需要在程序的某个地方 (通常是 OnInit 函数中)调用 wxFileSystem::AddHandler 函数.使用虚拟文件系统通常只需要使用那些定义在 wxFileSystem 对象中的函数,但是有些虚拟文件系统的实现也提供了直接给用于使用的函数,比如 wxMemoryFSHandler's 的 AddFile 和 RemoveFile 函数.

在我们介绍怎样通过 C++函数访问虚拟文件系统之前,我们先看看怎样在 wxWidgets 提供的其它子系统中使用虚拟文件系统.下面的例子演示了怎样在用于在 wxHtmlWindow 中显示的 HTML 文件中使用指定虚拟文件系统中的路径:

```cpp
<img src="file:myapp.bin#zip:images/logo.png"> 
```

"#"号前面的部分是文件名,后面的部分则是虚拟文件系统类型以及文件在虚拟文件系统中的路径.

类似的,我们也可以在 XRC 资源文件中使用虚拟文件系统:

```cpp
<object class="wxBitmapButton">
    <bitmap>file:myapp.bin#zip:images/fuzzy.gif</bitmap>
</object> 
```

在上面的这些用法中,操作虚拟文件系统的代码被隐藏在 wxHtmlWindow 和 XRC 系统的实现中.如果你希望直接使用虚拟文件系统, 通常你需要通过 wxFileSystem 和 wxFSFile 类.下面的代码演示了怎样从虚拟文件系统中加载一幅图片,当应用程序初始化的时候,增加一个 wxZipFSHandler 类型的虚拟文件系统处理器.然后创建一个 wxFileSystem 的实例,这个实例可以是临时使用也可以存在于整个应用程序的生命周期,这个实例用来从 zip 文件 myapp.bin 中获取 logo.png 图片.wxFSFile 对象用于返回这个文件对应的数据流,然后通过流的方式创建 wxImage 对象.在这个对象被转换成 wxBitmap 格式以后,wxFSFile 和 wxFileSystem 对象就可以被释放了.

```cpp
#include "wx/fs_zip.h"
#include "wx/filesys.h"
#include "wx/wfstream.h"
// 这一行代码只应该被执行依次,最好是在应用程序初始化的时候
wxFileSystem::AddHandler(new wxZipFSHandler);
wxFileSystem* fileSystem = new wxFileSystem;
wxString archive = wxT("file:///c:/myapp/myapp.bin");
wxString filename = wxT("images/logo.png");
wxFSFile* file = fileSystem->OpenFile(
              archive + wxString(wxT("#zip:")) + filename);
if (file)
{
    wxInputStream* stream = file->GetStream();
    wxImage image(* stream, bitmapType);
    wxBitmap bitmap = wxBitmap(image);
    delete file;
}
delete fileSystem; 
```

注意要使用 wxFileSystem:: OpenFile 函数,其参数必须是一个 URL 而不能是一个绝对路径,其格式应该为"file:/<主机名>//<文件名>", 如果主机名为空,则使用三个"/"符号.你可以使用 wxFileSystem::FileNameToURL 函数获取某个文件对应的 URL,也可以用 wxFileSystem::URLToFileName 函数将某个 URL 转换成对应的文件名.

下面的例子演示了怎样获取虚拟文件系统中获取一个文本文件的内容,并将其存放在某个 wxString 对象中,所需参数为 zip 文件路径以及 zip 文件中的虚拟文件的路径:

```cpp
// 从 zip 文件中加载一个文本文件
bool LoadTextResource(wxString& text, const wxString& archive,
                      const wxString& filename)
{
    wxString archiveURL(wxFileSystem::FileNameToURL(archive));
    wxFileSystem* fileSystem = new wxFileSystem;
    wxFSFile* file = fileSystem->OpenFile(
                 archiveURL + wxString(wxT("#zip:")) + filename);
    if (file)
    {
        wxInputStream* stream = file->GetStream();
        size_t sz = stream->GetSize();
        char* buf = new char[sz + 1];
        stream->Read((void*) buf, sz);
        buf[sz] = 0;
        text = wxString::FromAscii(buf);
        delete[] buf;
        delete file;
        delete fileSystem;
        return true;
    }
    else
        return false;
} 
```

wxMemoryFSHandler 允许你将数据保存在内存中并且通过内存协议在虚拟文件系统中使用它.显然在内存中存放大量数据并不是一个值得推荐的作法,不过有时候却可以给应用程序提供一定程序的灵活性,比如你正在使用只读的文件系统或者说使用磁盘文件存在性能上的问题的时候.在 DialogBlocks 软件中,如果用户提供的自定义图标文件还不具备,DialogBlocks 仍然可以通过下面的 XRC 文件显示一个内存中的图片, 这个图片并不存在于任何的物理磁盘中.

```cpp
<object class="wxBitmapButton">
    <bitmap&gt;memory:default.png</bitmap>
</object> 
```

wxMemoryFSHandler 的 AddFile 函数可以使用的参数包括一个虚拟文件名和一个 wxImage, wxBitmap, wxString 或 void*数据. 如果你不再使用某个内存虚拟文件了,可以通过 RemoveFile 函数将其删除.如下所示:

```cpp
#include "wx/fs_mem.h"
#include "wx/filesys.h"
#include "wx/wfstream.h"
#include "csquery.xpm"
wxFileSystem::AddHandler(new wxMemoryFSHandler);
wxBitmap bitmap(csquery_xpm);
wxMemoryFSHandler::AddFile(wxT("csquery.xpm"), bitmap,
                           wxBITMAP_TYPE_XPM);
...
wxMemoryFSHandler::RemoveFile(wxT("csquery.xpm")); 
```

wxWidgets 支持的第三种虚拟文件系统是 wxInternetFSHandler,它支持 FTP 和 HTTP 协议.

# 第十四章小结

本章介绍了怎样使用 wxWidgets 提供的跨平台的文件和流操作相关的类.还介绍了 wxWidgets 中的虚拟文件系统机制,它可以让你很方便的访问位于 zip 文件,内存或者 Internet 上的文件.

在下一章里,我们将介绍一些你的最终用户可能不会直接看到但是却仍然非常重要的话题:内存管理,调试和错误检测.