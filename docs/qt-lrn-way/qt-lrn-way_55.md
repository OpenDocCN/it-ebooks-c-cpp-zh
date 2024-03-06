# 五十五、自定义拖放数据对象

前面的例子都是使用的系统提供的拖放对象 QMimeData 进行拖放数据的存储，比如使用 QMimeData::setText() 创建文本，使用 QMimeData::urls() 创建 URL 对象。但是，如果你希望使用一些自定义的对象作为拖放数据，比如自定义类等等，单纯使用 QMimeData 可能就没有那么容易了。为了实现这种操作，我们可以从下面三种实现方式中选择一个：

*   将自定义数据作为 QByteArray 对象，使用 QMimeData::setData() 函数作为二进制数据存储到 QMimeData 中，然后使用 QMimeData::Data() 读取；
*   继承 QMimeData，重写其中的 formats() 和 retrieveData() 函数操作自定义数据；
*   如果拖放操作仅仅发生在同一个应用程序，可以直接继承 QMimeData，然后使用任意合适的数据结构进行存储。

第一种方法不需要继承任何类，但是有一些局限：即是拖放不会发生，我们也必须将自定义的数据对象转换成 QByteArray 对象；如果你希望支持很多种拖放的数据，那么每种类型的数据都必须使用一个 QMimeData 类，这可能会导致类爆炸；如果数据很大的话，这种方式可能会降低系统的可维护性。然而，后两种实现方式就不会有这些问题，或者说是能够减小这种问题，并且能够让我们有完全控制权。 我们先来看一个应用，使用 QTableWidget 来进行拖放操作，拖放的类型包括 plain/text，plain/html 和 plain/csv。如果使用第一种实现方法，我们的代码将会如下所示：

```cpp

void MyTableWidget::mouseMoveEvent(QMouseEvent *event)  
{  
    if (event->buttons() & Qt::LeftButton) {  
        int distance = (event->pos() - startPos).manhattanLength();  
        if (distance >= QApplication::startDragDistance())  
            performDrag();  
    }  
    QTableWidget::mouseMoveEvent(event);  
}  

void MyTableWidget::performDrag()  
{  
    QString plainText = selectionAsPlainText();  
    if (plainText.isEmpty())  
        return;  

    QMimeData *mimeData = new QMimeData;  
    mimeData->setText(plainText);  
    mimeData->setHtml(toHtml(plainText));  
    mimeData->setData("text/csv", toCsv(plainText).toUtf8());  

    QDrag *drag = new QDrag(this);  
    drag->setMimeData(mimeData);  
    if (drag->exec(Qt::CopyAction | Qt::MoveAction) == Qt::MoveAction)  
        deleteSelection();  
}
```

对于这段代码，我们应该已经很容易的理解：在 performDrag() 函数中，我们调用 QMimeData 的 setText() 和 setHTML() 函数存储 plain/text 和 plain/html 数据，使用 setData() 将 text/csv 类型的数据作为二进制 QByteArray 类型存储。

```cpp

QString MyTableWidget::toCsv(const QString &plainText)  
{  
    QString result = plainText;  
    result.replace("\\", "\\\\");  
    result.replace("\"", "\\\"");  
    result.replace("\t", "\", \"");  
    result.replace("\n", "\"\n\"");  
    result.prepend("\"");  
    result.append("\"");  
    return result;  
}  

QString MyTableWidget::toHtml(const QString &plainText)  
{  
    QString result = Qt::escape(plainText);  
    result.replace("\t", "<td>");  
    result.replace("\n", "\n<tr><td>");  
    result.prepend("<table>\n<tr><td>");  
    result.append("\n</table>");  
    return result;  
}
```

toCsv() 和 toHtml() 函数将数据取出并转换成我们需要的 csv 和 html 类型的数据。例如，下面的数据

```cpp

Red   Green   Blue
Cyan  Yellow  Magenta
```

转换成 csv 格式为：

```cpp

"Red", "Green", "Blue"
"Cyan", "Yellow", "Magenta"
```

转换成 html 格式为：

| Red | Green | Blue |
| Cyan | Yellow | Magenta |

在放置的函数中我们像以前一样使用：

```cpp

void MyTableWidget::dropEvent(QDropEvent *event)  
{  
    if (event->mimeData()->hasFormat("text/csv")) {  
        QByteArray csvData = event->mimeData()->data("text/csv");  
        QString csvText = QString::fromUtf8(csvData);  
        // ...  
        event->acceptProposedAction();  
    } else if (event->mimeData()->hasFormat("text/plain")) {  
        QString plainText = event->mimeData()->text();  
        // ...  
        event->acceptProposedAction();  
    }  
}
```

虽然我们接受三种数据类型，但是在这个函数中我们只接受两种类型。至于 html 类型，我们希望如果用户将 QTableWidget 的数据拖到一个 HTML 编辑器，那么它就会自动转换成 html 代码，但是我们不计划支持将外部的 html 代码拖放到 QTableWidget 上。为了让这段代码能够工作，我们需要在构造函数中设置 setAcceptDrops(true) 和 setSelectionMode(ContiguousSelection)。

好了，上面就是我们所说的第一种方式的实现。这里并没有给出完整的实现代码，大家可以根据需要自己实现一下试试。下面我们将按照第二种方法重新实现这个需求。

```cpp

class TableMimeData : public QMimeData  
{  
    Q_OBJECT  

public:  
    TableMimeData(const QTableWidget *tableWidget,  
                  const QTableWidgetSelectionRange &range);  

    const QTableWidget *tableWidget() const { return myTableWidget; }  
    QTableWidgetSelectionRange range() const { return myRange; }  
    QStringList formats() const;  

protected:  
    QVariant retrieveData(const QString &format,  
                          QVariant::Type preferredType) const;  

private:  
    static QString toHtml(const QString &plainText);  
    static QString toCsv(const QString &plainText);  

    QString text(int row, int column) const;  
    QString rangeAsPlainText() const;  

    const QTableWidget *myTableWidget;  
    QTableWidgetSelectionRange myRange;  
    QStringList myFormats;  
};
```

为了避免存储具体的数据，我们存储 table 和选择区域的坐标的指针。

```cpp

TableMimeData::TableMimeData(const QTableWidget *tableWidget,  
                             const QTableWidgetSelectionRange &range)  
{  
    myTableWidget = tableWidget;  
    myRange = range;  
    myFormats << "text/csv" << "text/html" << "text/plain";  
}  

QStringList TableMimeData::formats() const 
{  
    return myFormats;  
}
```

构造函数中，我们对私有变量进行初始化。formats() 函数返回的是被 MIME 数据对象支持的数据类型列表。这个列表是没有先后顺序的，但是最佳实践是将“最适合”的类型放在第一位。对于支持多种类型的应用程序而言，有时候会直接选用第一个符合的类型存储。

```cpp

QVariant TableMimeData::retrieveData(const QString &format,  
                                     QVariant::Type preferredType) const 
{  
    if (format == "text/plain") {  
        return rangeAsPlainText();  
    } else if (format == "text/csv") {  
        return toCsv(rangeAsPlainText());  
    } else if (format == "text/html") {  
        return toHtml(rangeAsPlainText());  
    } else {  
        return QMimeData::retrieveData(format, preferredType);  
    }  
}
```

函数 retrieveData() 将给出的 MIME 类型作为 QVariant 返回。参数 format 的值通常是 formats() 函数返回值之一，但是我们并不能假定一定是这个值之一，因为并不是所有的应用程序都会通过 formats() 函数检查 MIME 类型。一些返回函数，比如 text(), html(), urls(), imageData(), colorData() 和 data() 实际上都是在 QMimeData 的 retrieveData() 函数中实现的。第二个参数 preferredType 给出我们应该在 QVariant 中存储哪种类型的数据。在这里，我们简单的将其忽略了，并且在 else 语句中，我们假定 QMimeData 会自动将其转换成所需要的类型。

```cpp

void MyTableWidget::dropEvent(QDropEvent *event)  
{  
    const TableMimeData *tableData =  
            qobject_cast<const TableMimeData *>(event->mimeData());  

    if (tableData) {  
        const QTableWidget *otherTable = tableData->tableWidget();  
        QTableWidgetSelectionRange otherRange = tableData->range();  
        // ...  
        event->acceptProposedAction();  
    } else if (event->mimeData()->hasFormat("text/csv")) {  
        QByteArray csvData = event->mimeData()->data("text/csv");  
        QString csvText = QString::fromUtf8(csvData);  
        // ...  
        event->acceptProposedAction();  
    } else if (event->mimeData()->hasFormat("text/plain")) {  
        QString plainText = event->mimeData()->text();  
        // ...  
        event->acceptProposedAction();  
    }  
    QTableWidget::mouseMoveEvent(event);  
}
```

在放置的函数中，我们需要按照我们自己定义的数据类型进行选择。我们使用 qobject_cast 宏进行类型转换。如果成功，说明数据来自同一应用程序，因此我们直接设置 QTableWidget 相关 数据，如果转换失败，我们则使用一般的处理方式。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)