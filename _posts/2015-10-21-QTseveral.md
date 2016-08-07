---
layout: post_code
title: QT杂谈
description: 
category: blogs
---

#####QT字符串

当QString和std::string互相转换时，容易出现很多问题
[Convert QString to string](http://stackoverflow.com/questions/4214369/how-to-convert-qstring-to-stdstring)
QString是使用utf-16编码的，但是std::string可能会是很多其他不同的编码格式。string自身并没有编码格式的概念，但是string里byte的来源会有很多种不同的编码格式。
我们常常使用的QString函数包括toUtf8,fromUtf8,toLocal8bit,fromLocal8bit,toStdString,fromStdString。

首先是编码格式的问题[coding](http://stackoverflow.com/questions/700187/unicode-utf-ascii-ansi-format-differences)，unicode并不是一种特定的编码格式，而是一个标准，在不同的地方unicode编码可能对应不同的编码格式，通常是utf-16，大家通常区分的utf-8和unicode实际上是utf-16和utf-8的竞争[utf-16&utf-8](http://stackoverflow.com/questions/4655250/difference-between-utf-8-and-utf-16)

你的计算机上可能会有ascii，utf-8，local8bit（就是你本地的编码格式，同样是8位，但是并不是统一的unicode编码格式），utf-16等等编码格式，要想保证QString的工作正常，最好的方法自然是保证QString的来源和输出都是使用了正确地对应的to，from方法，tostdstring和fromstdstring都是使用的ascii方法。但是如果你不知道自己的string来源是什么样的编码格式，那么也没有办法保证你的QString是正常工作的。

[QT国际化](http://www.kuqin.com/qtdocument/i18n.html)，QString的出现是为了满足QT跨平台跨地域的需要，因此使用了utf的编码格式，无论是从文件中读取还是其他方式获取的byte，都尽量使用QT的方法来处理（使用QT的文件流，QT的http方法），这样可以保证你的byte不会被莫名其妙地改变某些东西。



#####QT信号&槽
[signal&slot](http://doc.qt.io/qt-4.8/signalsandslots.html)

QT的slot和signal类似于callback，当对象状态发生改变时，可以释放自己定义的signal，然后由对应的slot进行处理

<pre class="brush: cpp">

#include &lt;QObject>

class Counter : public QObject
{
    Q_OBJECT

public:
    Counter() { m_value = 0; }

    int value() const { return m_value; }

public slots:
    void setValue(int value)
    {
        if (value != m_value) {
            m_value = value;
            emit valueChanged(value);
        }
    }
signals:
    void valueChanged(int newValue);

private:
    int m_value;
};

    Counter a, b;
    QObject::connect(&a, SIGNAL(valueChanged(int)),
                     &b, SLOT(setValue(int)));

    a.setValue(12);     // a.value() == 12, b.value() == 12
    b.setValue(48);     // a.value() == 12, b.value() == 48


</pre>



在QTCreator中使用signal和slot是非常方便的，基本上没有什么技术含量。但是实现原理也是非常复杂，我也是没有看懂。。
[concept](http://woboq.com/blog/how-qt-signals-slots-work.html)

#####QT NetworkManager

[networkmanager example](http://stackoverflow.com/questions/4509441/qt-http-post-issue-when-server-requires-cookies)

QTnetworkmanager也是一个很强大的类，使用官网上已经说得很详细了，如果需要像爬虫一样获取cookie模拟登录的话可以参照上面连接中的做法。
