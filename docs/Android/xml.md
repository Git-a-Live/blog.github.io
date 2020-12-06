XML（eXtensible Markup Language，可扩展标记语言）也是非常常见的一种用于标记电子文件使其具有结构性的标记语言。和JSON一样，XML设计出来的主要用途也是**传输和存储数据**，因此它也是序列化手段的一种。常见的XML结构化数据有很多，最典型的例子就是Android项目中的AndroidManifest和各个布局文件。

XML中没有预定义的标签，这也就意味着创作者可以定义自己的标签和自己的文档结构。当然，Android Studio自带模板和标签提示功能是另外一回事，开发者在使用SharedPreferences时可以明显体会到XML的这一特点。

与JSON相比，XML不适合存储结构非常复杂的数据（比如多层嵌套结构），显而易见，随着结构复杂度的上升，标签数量也会急剧增长，数据量自然也会大幅增加。

对XML操作最常用的第三方库是[dom4j](https://search.maven.org/artifact/org.dom4j/dom4j)，它是一个非常优秀的Java XML API，支持DOM、SAX以及JAXP，具有性能优异、功能强大和极端易用的特点，同时也是一个开放源代码的软件。

## 创建XML文件

在Android Studio上，手动创建XML文件很少见，几乎都是依靠IDE自带的功能来创建，比如创建一个Activity，IDE通常会顺手创建一个.xml格式的文件，而且里面还会编写好标签内容。

利用代码生成XML文件，其实方式也有不少，最简单最常见的就是[SharedPreferences](Android/sp)，但是SharedPreferences生成的.xml文件局限性很大，不适合稍微复杂一些的数据结构的应用场景。

而如果使用dom4j配合输入输出流，那么就可以创建结构相对复杂的.xml文件。

首先在Android项目中通过以下语句导入dom4j：

```
implementation 'org.dom4j:dom4j:$latest_version'
```

然后按照下面几个步骤构建XML的内部结构：

1. **创建Document对象**

创建Document对象需要调用dom4j的DocumentHelper静态方法`createDocument()`：

```
val doc = DocumentHelper.createDocument()
```

2. **创建根节点**

根节点是从Document对象调用`addElement()`创建而来的：

```
val root = doc.addElement("rootName")
```

3. **添加子元素**

子元素又由根节点调用`addElement()`创建而来：

```
val ele = root.addElement("elementName")
```

4. **编辑子元素属性**

子元素属性通过调用诸如`add()`、`addAttribute()`和`addElement()`等方法可以设置属性以及添加其他标签内容。

最后需要依靠dom4j的XMLWriter类和Java自带的FileOutputStream类输出.xml格式文件：

```
val writer = XMLWriter()
val fos = FileOutputStream("XML.xml")
writer.setOutputStream(fos)
writer.write(doc)
writer.close()
```

## 解析XML文件

Android开发中，解析XML文件主要有三种方式，一种是DOM，一种是Pull，另一种则是SAX。注意，无论采用哪种方式解析XML，都要用`try`语句将执行逻辑包裹起来，以避免解析过程中抛出异常导致的应用崩溃。

### DOM

DOM解析是一次性将整个XML文档加载进内存，在内存中构建Document的对象树，通过Document对象，得到树上的节点对象，通过节点对象访问（操作）到XML文档的内容。**因此DOM方式占用内存大，解析慢，优点是可以任意遍历和修改树的节点**。但是考虑到Android设备性能的限制，在实际开发中并<font color=red>不推荐</font>使用DOM方式解析XML。

### SAX

相比于DOM，SAX是一种速度更快，更有效的方法。它逐行扫描文档，一边扫描一边解析。而且相比于DOM，SAX可以在解析文档的任意时刻停止解析。SAX解析可以立即开始，速度快，没有内存压力，但是不能对节点做任意修改。在正常情况下，如果没有修改节点的需求，优先选择使用SAX即可。

Android项目中使用SAX解析XML，主要分为两个步骤：

1. **创建DefaultHandler子类**

DefaultHandler类有许多方法可以重写，常用方法有以下5个：

```
class MyHandler: DefaultHandler() {
    override fun startDocument() {
        //XML解析开始时调用
    }

    override fun endDocument() {
        //XML解析完成时调用
    }

    override fun startElement(uri: String?, localName: String?, qName: String?, attributes: Attributes?) {
        //开始解析某个节点时调用
    }

    override fun endElement(uri: String?, localName: String?, qName: String?) {
        //完成解析某个节点时调用
    }

    override fun characters(ch: CharArray?, start: Int, length: Int) {
        //获取节点中的内容时调用
    }
}
```

2. **调用DefaultHandler子类**

在调用重写了方法的DefaultHandler子类的地方，需要编写以下代码来使用该子类以解析XML：

```
try {
    val xmlReader = SAXParseFactory.newInstance().newSAXParser().xmlReader
    xmlReader.contentHandler = MyHandler()
    xmlReader.parse(InputSource(StringReader(XML_Data_String)))
} catch (e: Exception) {
    //TODO
}
```

### Pull

Pull也是Android项目中常用的一种XML解析手段，在用法上跟SAX有一些类似，具体示例代码如下：

```
try {
    val xmlPullParser = XmlPullParserFactory.newInstance().newPullParser()
    xmlPullParser.setInput(StringReader(XML_Data_String))
    var eventType = xmlPullParser.eventType
    while (eventType != XmlPullParser.END_DOCUMENT) {
        when (eventType) {
            XmlPullParser.START_TAG -> {
                when (xmlPullParser.name) {
                    //TODO
                }
            }
            XmlPullParser.END_TAG -> {
                //TODO
            }
            ···
        }
        eventType = xmlPullParser.next()
    }
} catch (e: Exception) {
    //TODO
}
```

上述代码的含义是，获得一个XmlPullParser对象，传入String类型的XML数据，通过一个`while`循环进行解析。



