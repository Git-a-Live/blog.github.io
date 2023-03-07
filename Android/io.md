不管是SharedPreferences，还是Room/SQLite，它们都为本地数据持久化提供了行之有效的途径。然而新的问题也随之出现了，如果要保存的数据是图片，或是一大段极长的文字，甚至是一段音视频，单单用SharedPreferences或Room/SQLite就会让开发者感到十分为难。

因为把这类数据往往体积较大，把它们转化成二进制之后再以键值对之类的形式进行存储显然是不合适的，尤其是进行大数据量的下载任务时，SharedPreferences或Room/SQLite更是无法胜任。因此在这种情况下，IO就派上用场了。

## 基本概念

IO是指Input/Output，即输入和输出。IO以内存为中心：

+ Input指**从外部读入数据到内存**，例如，把文件从磁盘读取到内存，从网络读取数据到内存等等。

+ Output指**把数据从内存输出到外部**，例如，把数据从内存写入到文件，把数据从内存输出到网络等等。

IO流是一种顺序读写数据的模式，它的特点是单向流动。IO流以byte（字节）为最小单位，因此也称为字节流。如果需要读写的是字符，并且字符不全是单字节表示的ASCII字符，那么，按照char来读写显然更方便，这种流称为字符流。

IO功能由Java的`java.io`包提供相应API支持。

## File类

File是`java.io`包提供的一个核心类，用于操作文件和目录。在Android设备上访问和存取文件，都必须经过File对象。

### 对象构建与路径

最简单的File对象构建方式如下：

```
val file = File("file_path") //传入一个文件路径字符串
```

File的构造器可接收绝对路径和相对路径，所谓绝对路径，就是以根目录为开头的完整路径，比如Windows上的`C:\Program Files\Google\Chrome`，Linux上的`/usr/bin/javac`（Android基于Linux内核自然使用Linux格式的路径）。

相对路径就是省略了根目录的路径，此时根目录就默认为是应用程序所在的当前目录，比如`/`。相对路径的表示方法有`mnt`、`./mnt`以及`../mnt`，其中`.`表示当前目录，`..`表示上级目录。

通过File对象还能获取到路径。File对象提供了三种形式的路径：1）从`getPath()`方法拿到传入构造器的路径；2）从`getAbsolutePath()`方法拿到绝对路径；3）从`getCanonicalPath()`拿到把相对路径转换成绝对路径的规范路径。这三种路径可以根据需要选用。

值得注意的是，构造一个File对象，即使传入的文件或目录不存在，代码也不会出错。因为构造一个File对象，并不会导致任何磁盘操作。只有在调用File对象的某些方法的时候，才会真正进行磁盘操作。

### 文件和目录

File对象操作文件和目录，主要通过以下方法来执行：

|方法名|用途|方法名|用途|
|:--|:--|:--|:--|
|list()|遍历指定路径下的目录和文件，返回一个String列表|listFiles()|遍历指定路径下的目录和文件，返回一个File列表|
|getName()|获取文件或目录的名称|exists()|判断文件或目录是否存在|
|isAbsolute()|判断是否为绝对路径|isDirectory()|判断是否为目录|
|isFile()|判断是否为文件|isHidden()|判断文件是否被隐藏|
|canExecute()|判断文件能否执行|canRead()|判断文件能否读取|
|canWrite()|判断文件能否写入|length()|获取目录和文件的字节大小|
|createNewFile()|在指定路径下创建一个文件|delete()|删除File对象所表示的文件或目录|
|deleteOnExit()|应用程序退出时删除File对象所表示的文件或目录|mkdir()|在指定路径下创建一个目录|
|mkdirs()|根据传入的路径创建一系列多级目录，包括可能并不存在的父目录|setExecutable()|设置文件是否可执行|
|setReadable()|设置文件或目录是否可读|setWritable()|设置文件或目录是否可写|
|setReadOnly()|设置文件或目录为只读|getFreeSpace()|获取文件或目录所在磁盘分区的未分配字节数，不保证准确|
|getUsableSpace()|获取文件或目录所在磁盘分区的可用字节数，不保证准确|getTotalSpace()|获取文件或目录所在磁盘分区的总字节数|

### File的扩展函数

Kotlin对File实施了扩展，为File设计了以下[扩展函数](Kotlin/late?id=扩展函数)：

|函数名|用途|
|:--|:--|
|readBytes()|将文件里的所有数据以Byte数组形式读出，只适用于小文件|
|writeBytes(array: ByteArray)|将数据以Byte数组形式写入文件，只适用于小文件，若文件已存在会被覆盖|
|appendBytes(array: ByteArray)|将数据以Byte数组形式添加到文件末尾|
|forEachBlock(action: (buffer: ByteArray, bytesRead: Int) -> Unit)|将文件以指定buffer大小分块读出，适用于大文件|
|readText(charset: Charset = Charsets.UTF_8)|将文件内容以String形式一次性读入，可指定字符编码格式，只适用于小文件|
|writeText(text: String, charset: Charset = Charsets.UTF_8)|将内容以String形式写入文件，可指定字符编码格式，若文件已存在会被覆盖|
|forEachLine(charset: Charset = Charsets.UTF_8, action: (line: String) -> Unit)|将文件内容以String形式逐行读入，可指定字符编码格式，适用于大文件|
|readLines(charset: Charset = Charsets.UTF_8)|将文件内容逐字读进一个String列表中，可指定字符编码格式，仅适用于小文件|
|useLines(charset: Charset = Charsets.UTF_8, block: (Sequence\<String>) -> T)|将文件内容以Sequence\<String>形式读入，可指定字符编码格式，会引起阻塞|

这些扩展函数都是间接调用了Kotlin专门为IO流封装设计的扩展函数`use`，因此不需要过多关注异常处理和释放资源的问题，可以放心使用。至于`use`函数怎么用，会在下面的内容中进行介绍。

### Files和Paths

Files和Paths都是Java 7开始由`java.nio`包提供的两个工具类，用于简单的文件读写操作，但是只有在<font color=red>Android O（API = 26）</font>以上才能使用，<font color=red>且仅适用于操作小文件和包含少量小文件的目录</font>。

Files主要封装了`copy()`、`delete()`、`exists()`、`move()`、`find()`以及`readAllBytes()`等静态方法以供调用；而Paths提供了两种重载形式的`get()`方法。

在简单读写小文件的场景下，使用Files和Paths可以有效简化代码。

## 输入输出流

InputStream和OutputStream都是Java标准库提供的基本输入输出流，位于`java.io`内。它们都是抽象类，也是所有输入输出流的超类。

二者通常在try-catch-finally代码块中使用，并在finally处调用`close()`方法释放资源，或是利用Java 7推出的try-with-resource工具进行自动管理。然而用try还是比较麻烦，用Kotlin开发的话，还可以选择专门为IO流封装设计的扩展函数`use`。

>注意，IO流是以同步的方式进行的，这意味着会阻塞主线程。

### InputStream

InputStream顾名思义就是输入流，它提供了一个非常重要的方法`read()`。这个方法有三种重载形式：一种是`int read()`，每次只读取一个字节，返回的是这个字节的整型值；第二种是`int read(byte b[])`，接收一个Byte类型的数组作为缓冲区，返回的是每次读取到的字节数；第三种是`int read(byte b[], int off, int len)`，功能上跟第二种类似。三种重载形式都会以返回`-1`作为输入流读取完毕的标志。

InputStream作为一个抽象类，其功能只能由具体的子类来实现。常用的子类有FileInputStream、ByteArrayInputStream以及FilterInputStream。

#### FileInputStream

FileInputStream用于从<font color=red>文件流</font>当中读取数据。它的典型用法如下：

```
/*使用try-catch-finally*/
var input: InputStream? = null
try {
    input = FileInputStream(···)

    //第一种read()
    while(input.read() != -1) { 
        //TODO
    }

    //第二种read()
    val byteArray = ByteArray(···)
    while(input.read(byteArray) != -1) {
        //TODO
    }

    //第三种read()
    val byteArray = ByteArray(···)
    while(input.read(byteArray,offSet,length) != -1) {
        //TODO
    }

} catch(e: Exception) {
    //TODO
} finally {
    input?.close()
}

/*在Java中使用try-with-resource*/
try(InputStream input = new FileInputStream(···)) {
    //TODO
} //对于实现了AutoCloseable接口的类型，编译器会自动加入finally并调用close()
```

原始的try用起来非常繁琐，而Kotlin又不支持try-with-resource的用法，这种时候扩展函数`use`就排上用场了：

```
FileInputStream(···).use { it:FileInputStream
    //第一种read()
    var r = it.read()
    while(r != -1) { 
        //TODO
        r = it.read
    }

    //第二种read()
    val byteArray = ByteArray(···)
    var r = it.read(byteArray)
    while(r != -1) {
        //TODO
        r = it.read
    }

    //第三种read()
    val byteArray = ByteArray(···)
    var r = it.read(byteArray,offSet,length)
    while(r != -1) {
        //TODO
        r = it.read(byteArray,offSet,length)
    }
}
```

`use`函数发挥的作用和try-with-resource类似，只能由实现了AutoCloseable接口的类型对象来调用，而且会自动关闭IO流——无论在此过程中FileInputStream是否会读取失败——其他的输入输出实现类也是如此。

#### ByteArrayInputStream

ByteArrayInputStream用于在内存中模拟字节流的输入。当创建真实文件比较麻烦的时候，就可以考虑使用ByteArrayInputStream，它的用法跟FileInputStream基本一样，只不过需要把传入的参数从路径字符串换成<font color=red>Byte数组</font>：

```
ByteArrayInputStream(···).use {
    //TODO
}
```

#### FilterInputStream

FilterInputStream是一种在直接提供数据的基础上增添额外功能的输入流，但是很少被直接使用。因为它仅仅只是简单地重写了那些将所有请求传递给所包含输入流的InputStream的所有方法，而更复杂的实现是交由子类去做的。FilterInputStream的常用子类包括：BufferedInputStream、InflaterInputStream以及DataInputStream等。

> *BufferedInputStream*

BufferedInputStream提供的是缓冲输入流的功能，它在内部维护了一个Byte类型的缓冲数组。在每次调用`read()`方法的时候，它会首先尝试从缓冲区里读取数据，若缓冲区内无数据可读，则选择从物理数据源（譬如文件）读取新数据放入到缓冲区中，最后再将缓冲区中的内容部分或全部返回给用户。这样读取效率就相对高了不少。BufferedInputStream对象的创建需要向构造器传入一个InputStream类型的参数，例如：

```
val bis = BufferedInputStream(ByteArrayInputStream(byteArrayOf(1,2,3,4,5,6)))
```

> *InflaterInputStream*

InflaterInputStream和FilterInputStream一样也是很少直接使用，更常用的是由它派生出来的ZipInputStream及其子类JarInputStream，还有GZIPInputStream这几个子类。从名字上看就知道，它们都是负责将数据从特定格式压缩包里面解压出来，然后再进行读取的。

要创建一个ZipInputStream，通常是传入一个FileInputStream作为数据源，然后循环调用`getNextEntry()`方法，直到返回null，表示zip流结束：

```
val zis = ZipInputStream(FileInputStream(···))
zis.use {
    var zip = it.nextEntry
    if (!zip.isDirectory) {
        while (zip != null) {
            //TODO
            zip = it.nextEntry
        }
    }
}
```

其他的InflaterInputStream子类在使用上与ZipInputStream基本一样，这里不再做赘述。

> *DataInputStream*

DataInputStream用于从输入流中读取Java基本数据类型，且采用的是与机器无关方式——这句话的含义是，DataInputStream可以从输入流中识别出Java的基本数据类型——但这是有前提的，**输入流的来源必须是那些通过DataOutputStream所输出的做过特殊标记的文件**，否则就无法识别，而且会引起EOFException。DataInputStream除了提供一般的`read()`方法外，还提供了`readBoolean()`、`readByte()`、`readChar()`、`readDouble()`、`readFloat()`、`readInt()`、`readLong()`以及`readShort()`等方法来读取特定的基本类型数据。在后面介绍DataOutputStream的时候会展示其典型用法。

### OutputStream

和InputStream相反，OutputStream是输出流，提供的是`write()`方法。这个方法同样也有三种重载形式：一种是`void write(int b)`，每次只会写入一个字节；第二种是`void write(byte b[])`，接收Byte数组作为缓冲区；最后一个是`void write(byte b[], int off, int len)`，用法上跟输入流差不多，只不过是写入文件的。

除了`write()`方法，OutputStream还提供了一个`flush()`方法，用于<font color=red>强制输出缓冲区的内容</font>。一般情况下，缓冲区写满了就会自动调用该方法，要是缓冲区没满但是又得输出内容的话就得手动调用了。

同InputStream一样，OutputStream作为一个抽象类也是不能直接使用的，其功能同样得交由子类来实现。常用的子类包括：FileOutputStream、ByteArrayOutputStream以及FilterOutputStream等。

#### FileOutputStream

FileOutputStream用于将输出流写入文件当中，它的典型用法如下：

```
val fos = FileOutputStream(File(···))
fos.use {
    it.write(···)
}
```

通常使用比较多的`write()`方法是第二种重载形式，即接收一个Byte数组。但是手动构建一个Byte数组并不总是可行的，因而也可以选择将字符串转换成Byte数组：

```
val fos = FileOutputStream(File(···))
fos.use {
    it.write("Some String".toByteArray())
}
```

#### ByteArrayOutputStream

ByteArrayOutputStream和ByteArrayInputStream相反，是将字节流写入内存中：

```
val bos = ByteArrayOutputStream()
bos.use {
    it.write(···)
}
Toast.makeText(context, String(bos.toByteArray()),Toast.LENGTH_SHORT).show()
```

#### FilterOutputStream

FilterOutputStream的常用子类有：BufferedOutputStream、DataOutputStream、DeflaterOutputStream、InflaterOutputStream以及PrintStream等。

> *BufferedOutputStream*

BufferedOutputStream是一个缓存输出流，当缓冲区写满的时候就会自动调用`flush()`方法将里面的数据全部输出，如果没满的话就要手动调用以防有数据残留在缓冲区未输出。一个比较好的做法就是，任何情况下，在BufferedOutputStream关闭之前都要手动调用`flush()`：

```
val bufferedos = BufferedOutputStream(···)
bufferedos.use {
    it.write(···)
    it.flush()
}
```

> *DataOutputStream*

DataOutputStream用于将数据流附上特殊标记再写入文件，这样当用户使用对应的DataInputStream时，就可以从文件中读出特定类型的数据：

```
//写入
val dos = DataOutputStream(···)
dos.use {
    it.writeBoolean(true)
    it.writeInt(1)
    it.writeUTF("Fuck you")
    it.writeFloat(1.2F)
    ···
    it.flush()
}

//读取
val dis = DataInputStream(fis)
dis.use {
    val b: Boolean = it.readBoolean()
    val i: Int = it.readInt()
    val u: String = it.readUTF()
    val f: Float = it.readFloat()
    ···
}
```

> *DeflaterOutputStream*

DeflaterOutputStream用于将数据输出并打包成指定格式的压缩包。同样地，它的具体实现也是由ZipOutputStream、GZIPOutputStream以及JarOutputStream等子类来完成，这里仍然以Zip格式压缩包为例：

```
val zos = ZipOutputStream(···)
val z = ZipEntry(···)
zos.use {
    it.putNextEntry(z)
}
```

值得注意的是，ZipOutputStream不是用`write()`方法，而是用`putNextEntry()`将数据写入压缩包。

> *InflaterOutputStream*

InflaterOutputStream用于将数据从通过DeflaterOutputStream压缩好的特定格式压缩包里解压出来，再输出到内存或文件当中。

> *PrintStream*

PrintStream在实现OutputStream接口的基础上又额外添加一些写入各种数据类型的方法，比如`print()`和`println()`。在Java中，`System.out`跟`System.err`都是系统默认提供的PrintStream对象，分别表示标准输出和标准错误输出，它们在使用的时候不会抛出`IOException`。

## Reader/Writer

Reader和Writer都是`java.io`包提供的另一种输入输出流接口，它们跟InputStream及OutputStream的主要区别在于：后者都是处理**字节流**，而前者专门处理**字符流**。

### Reader

Reader作为处理输入字符流的超类，其功能主要由FileReader、CharArrayReader、StringReader以及InputStreamReader等常用子类来实现。Reader对象主要通过`read()`方法来处理输入字符流。

#### FileReader

FileReader负责从文件读取字符流，其典型用法如下：

```
val reader: Reader = FileReader(···)

/*逐个字符读取*/
while (true) {
    val s = reader.read()
    if (s == -1) {
        break
    }
    //TODO
}

/*带有缓冲的字符读取*/
val buffer = CharArray(1000)
while (true) {
    val s = reader.read(buffer)
    if (s == -1) {
        break
    }
    //TODO
}
```

如果读取正常文件出现了乱码，则可以在FileReader构造方法中传入字符编码格式，比如`StandardCharsets.UTF_8`——经验证，在Java 8环境下没有这个构造方法，但是Java 15可以用。

#### CharArrayReader

CharArrayReader的作用和ByteArrayInputStream相似，都是在内存中利用一个Char数组模拟数据源进行读入，这样就不用在磁盘上创建真实存在的文件了：

```
val reader: Reader = CharArrayReader(charArrayOf('A','B','C','D'))
```

其他用法跟FileReader基本一样，此处不做赘述。

#### StringReader

StringReader把字符串作为输入的数据源，在用法上跟CharArrayReader也是基本一样：

```
val reader: Reader = StringReader("Surprise, motherfucker")
```

#### InputStreamReader

Reader在本质上是一个基于InputStream的把Byte类型数据转换为Char的转换器。InputStreamReader要做的事情就是如此，把一个InputStream对象转换成Reader对象，比如：

```
val reader: Reader = InputStreamReader(FileInputStream(···))
```

上述例子可以视为FileReader的一种实现方式。

>注意，尽管传进去一个InputStream对象，但只要确保InputStreamReader对象能够关闭，InputStream对象也会跟着关闭释放资源，不需要再手动进行管理。

### Writer

Writer作为处理输出字符流的超类，其功能主要由FileWriter、CharArrayWriter、StringWriter以及OutputStreamWriter等常用子类来实现。Writer对象主要通过`write()`方法来处理输出字符流。

#### FileWriter

FileWriter用于将字符流写进文件：

```
val writer: Writer = FileWriter(···)
writer.use {
    it.write(···)
    it.flush()
}
```

注意到FileWriter也可以调用`flush()`方法，其原因是父类Writer实现了Flushable接口，而非继承了OutputStream。

#### CharArrayWriter

CharArrayWriter可以在内存中创建一个Writer，它的作用实际上是构造一个缓冲区，可以写入Char类型数据，最后得到写入的Char数组，这和之前的ByteArrayOutputStream非常类似：

```
val writer = CharArrayWriter() //注意，如果将writer类型设置为父类Writer，就无法调用toCharArray()方法
writer.use {
    it.write(66)
    it.write(67)
    it.write(68)
    it.write("ABC".toCharArray())
    ···
    //TODO
}
```

#### StringWriter

StringWriter也是一个基于内存的Writer，它和CharArrayWriter类似。实际上，StringWriter在内部维护了一个StringBuffer，并对外提供了Writer接口：

```
val writer = StringWriter(···) //可以传入一个缓冲区大小的值
writer.use {
    it.write(66)
    it.write(67)
    it.write(68)
    it.write("Fuck you")
    ···
    //TODO
}
```

#### OutputStreamWriter

和InputStreamReader相反，OutputStreamWriter是一个将任意的OutputStream转换为Writer的转换器，比如：

```
val writer = OutputStreamWriter(FileOutputStream(···))
```

#### PrintWriter

PrintWriter在用法上跟PrintStream几乎是完全一样的，只不过前者输出的是Char类型的数据，后者输出的是Byte类型的数据。