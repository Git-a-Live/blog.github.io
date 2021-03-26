不管是SharedPreferences，还是Room/SQLite，它们都为本地数据持久化提供了行之有效的途径。然而新的问题也随之出现了，如果要保存的数据是图片，或是一大段极长的文字，甚至是一段音视频，单单用SharedPreferences或Room/SQLite就会让开发者感到十分为难。

因为把这类数据往往体积较大，把它们转化成二进制之后再以键值对之类的形式进行存储显然是不合适的，尤其是进行大数据量的下载任务时，SharedPreferences或Room/SQLite更是无法胜任。因此在这种情况下，IO就派上用场了。

## 基本概念

### 什么是IO

IO是指Input/Output，即输入和输出。IO以内存为中心：

+ Input指**从外部读入数据到内存**，例如，把文件从磁盘读取到内存，从网络读取数据到内存等等。

+ Output指**把数据从内存输出到外部**，例如，把数据从内存写入到文件，把数据从内存输出到网络等等。

IO流是一种顺序读写数据的模式，它的特点是单向流动。IO流以byte（字节）为最小单位，因此也称为字节流。如果需要读写的是字符，并且字符不全是单字节表示的ASCII字符，那么，按照char来读写显然更方便，这种流称为字符流。

IO功能由Java的`java.io`包提供API支持。

### 存储空间

Android将设备的存储空间划分为内部存储空间和外部存储空间。

+ **内部存储空间**

内部存储空间（internal storage）是手机ROM当中的一块存储区域，主要用于<font color=red>存储应用程序的数据</font>。它跟“内存”（RAM，memory）不是同一个东西，不能混淆。

在Android系统中，内部存储空间对应的是`data/data/package_name`，它是一个属于应用程序的私有目录，系统通常不会允许其他应用访问，但是可以通过Android Studio的Device File Explorer来查看。此外，从Android 10开始，系统会对这些私有目录实施加密，以确保只有对应的应用程序在不用申请任何权限的情况下访问里面的数据。

当应用程序被卸载，对应的内部存储空间目录也会跟着被删除，里面的数据也随之被清除掉。因此，如果不希望某些数据在应用卸载的时候跟着被清除，那就不要把它们保存在内部存储空间的目录下面。

+ **外部存储空间**

外部存储空间（external storage）是跟内部存储空间相对应的一个概念，它可以被各种应用程序和用户访问，里面的数据也不会随着应用程序卸载而被清除。在Android 4.4以前，外部存储空间专指SD卡，但随着机身存储容量的提升，除内部存储空间以外的部分也被称为外部存储空间。<font color=red>这里仅讨论机身存储部分的外部存储空间</font>。

外部存储空间又分为外部私有存储和外部公共存储，前者对应的目录是`/storage/emulated/0/Android/data/package_name`，和内部存储空间相似，后者对应的就是前者以外的目录。

>外部私有存储

外部私有存储和内部存储有些类似，除了对应程序不用申请权限即可访问、数据会跟着应用卸载一同被清除之外，它还有两个特点：1） 只有在应用中调用API访问外部私有存储目录时，才会创建以package_name命名的私有存储目录；2）Android 7.0之前，其他应用程序可以采用`file://`格式的Uri直接访问某个程序的外部私有存储目录下的数据，Android 7.0之后就必须使用FileProvider。

>外部公共存储

外部公共存储主要存放和应用无关的数据，这些数据在卸载App的时候不会被删除。它还具有这些特点：1）应用要申请外部存储权限才能访问，在Android 6.0以后，外部存储权限还要动态申请；2）任何应用只要有外部存储权限，都可以访问公共存储目录下的数据。

综上，各种存储空间之间的关系可以用下图来概括：

![](pics/storage.png)

## 基本使用

### File对象

File是`java.io`包提供的一个核心类，用于操作文件和目录。在Android设备上访问和存取文件，都必须经过File对象。

+ **对象构建与路径**

最简单的File对象构建方式如下：

```
val file = File("file_path") //传入一个文件路径字符串
```

File的构造器可接收绝对路径和相对路径，所谓绝对路径，就是以根目录为开头的完整路径，比如Windows上的`C:\Program Files\Google\Chrome`，Linux上的`/usr/bin/javac`（Android基于Linux内核自然使用Linux格式的路径）。

相对路径就是省略了根目录的路径，此时根目录就默认为是应用程序所在的当前目录，比如`/`。相对路径的表示方法有`mnt`、`./mnt`以及`../mnt`，其中`.`表示当前目录，`..`表示上级目录。

通过File对象还能获取到路径。File对象提供了三种形式的路径：1）从`getPath()`方法拿到传入构造器的路径；2）从`getAbsolutePath()`方法拿到绝对路径；3）从`getCanonicalPath()`拿到把相对路径转换成绝对路径的规范路径。这三种路径可以根据需要选用。

值得注意的是，构造一个File对象，即使传入的文件或目录不存在，代码也不会出错。因为构造一个File对象，并不会导致任何磁盘操作。只有在调用File对象的某些方法的时候，才会真正进行磁盘操作。

+ **文件和目录**

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

+ **File的扩展函数**

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

### InputStream/OutputStream

InputStream和OutputStream都是Java标准库提供的基本输入输出流，位于`java.io`内。它们都是抽象类，也是所有输入输出流的超类。

二者通常在try-catch-finally代码块中使用，并在finally处调用`close()`方法释放资源，或是利用Java 7推出的try-with-resource工具进行自动管理。然而用try还是比较麻烦，用Kotlin开发的话，还可以选择专门为IO流封装设计的扩展函数`use`。

>注意，IO流是以同步的方式进行的，这意味着会阻塞主线程。

#### InputStream

InputStream顾名思义就是输入流，它提供了一个非常重要的方法`read()`。这个方法有三种重载形式：一种是`int read()`，每次只读取一个字节，返回的是这个字节的整型值；第二种是`int read(byte b[])`，接收一个Byte类型的数组作为缓冲区，返回的是每次读取到的字节数；第三种是`int read(byte b[], int off, int len)`，功能上跟第二种类似。三种重载形式都会以返回`-1`作为输入流读取完毕的标志。

InputStream作为一个抽象类，其功能只能由具体的子类来实现。常用的子类有FileInputStream、ByteArrayInputStream以及FilterInputStream。

+ **FileInputStream**

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

+ **ByteArrayInputStream**

ByteArrayInputStream用于在内存中模拟字节流的输入。当创建真实文件比较麻烦的时候，就可以考虑使用ByteArrayInputStream，它的用法跟FileInputStream基本一样，只不过需要把传入的参数从路径字符串换成<font color=red>Byte数组</font>：

```
ByteArrayInputStream(···).use {
    //TODO
}
```

+ **FilterInputStream**

FilterInputStream是一种在直接提供数据的基础上增添额外功能的输入流，但是很少被直接使用。因为它仅仅只是简单地重写了那些将所有请求传递给所包含输入流的InputStream的所有方法，而更复杂的实现是交由子类去做的。FilterInputStream的常用子类包括：BufferedInputStream、InflaterInputStream以及DataInputStream等。

先来说说BufferedInputStream。这个子类提供的是缓冲输入流的功能，它在内部维护了一个Byte类型的缓冲数组。在每次调用`read()`方法的时候，它会首先尝试从缓冲区里读取数据，若缓冲区内无数据可读，则选择从物理数据源（譬如文件）读取新数据放入到缓冲区中，最后再将缓冲区中的内容部分或全部返回给用户。这样读取效率就相对高了不少。BufferedInputStream对象的创建需要向构造器传入一个InputStream类型的参数，例如：

```
val bis = BufferedInputStream(ByteArrayInputStream(byteArrayOf(1,2,3,4,5,6)))
```

接着是InflaterInputStream。这个子类和FilterInputStream一样也是很少直接使用，更常用的是由它派生出来的ZipInputStream及其子类JarInputStream，还有GZIPInputStream这几个子类。从名字上看就知道，它们都是负责将数据文件打包成特定格式压缩包的。

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

最后介绍一下DataInputStream。它从输入流中读取Java基本数据类型，且采用的是与机器无关方式——这句话的含义是，DataInputStream可以从输入流中识别出Java的基本数据类型——但这是有前提的，**输入流的来源必须是那些通过DataOutputStream所输出的做过特殊标记的文件**，否则就无法识别，而且会引起EOFException。DataInputStream除了提供一般的`read()`方法外，还提供了`readBoolean()`、`readByte()`、`readChar()`、`readDouble()`、`readFloat()`、`readInt()`、`readLong()`以及`readShort()`等方法来读取特定的基本类型数据。

#### OutputStream

和InputStream相反，OutputStream是输出流，提供的是`write()`方法。这个方法同样也有三种重载形式：一种是`void write(int b)`，每次只会写入一个字节；第二种是`void write(byte b[])`，接收Byte数组作为缓冲区；最后一个是`void write(byte b[], int off, int len)`，用法上跟输入流差不多，只不过是写入文件的。

除了`write()`方法，OutputStream还提供了一个`flush()`方法，用于<font color=red>强制输出缓冲区的内容</font>。一般情况下，缓冲区写满了就会自动调用该方法，要是缓冲区没满但是又得输出内容的话就得手动调用了。

同InputStream一样，OutputStream作为一个抽象类也是不能直接使用的，其功能同样得交由子类来实现。常用的子类包括：FileOutputStream、ByteArrayOutputStream以及FilterOutputStream等。

+ **FileOutputStream**
+ **ByteArrayOutputStream**
+ **FilterOutputStream**

### Reader/Writer

#### Reader

#### Writer

### 访问内部存储空间的数据文件

### 访问外部存储空间的数据文件