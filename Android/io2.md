## Android存储空间

Android将设备的存储空间划分为内部存储空间和外部存储空间。

### 内部存储空间

内部存储空间（internal storage）是手机ROM当中的一块存储区域，主要用于<font color=red>存储应用程序的数据</font>。它跟“内存”（RAM）不是同一个东西，不能混淆。

在Android系统中，内部存储空间对应的是`data/data/package_name`，它是一个属于应用程序的私有目录，系统通常不会允许其他应用访问，但是可以通过Android Studio的Device File Explorer来查看。此外，从Android 10开始，系统会对这些私有目录实施加密，以确保只有对应的应用程序在不用申请任何权限的情况下访问里面的数据。

当应用程序被卸载，对应的内部存储空间目录也会跟着被删除，里面的数据也随之被清除掉。因此，如果不希望某些数据在应用卸载的时候跟着被清除，那就不要把它们保存在内部存储空间的目录下面。

### 外部存储空间

外部存储空间（external storage）是跟内部存储空间相对应的一个概念，它可以被各种应用程序和用户访问，里面的数据也不会随着应用程序卸载而被清除。在Android 4.4以前，外部存储空间专指SD卡，但随着机身存储容量的提升，除内部存储空间以外的部分也被称为外部存储空间。<font color=red>这里仅讨论机身存储部分的外部存储空间</font>。

外部存储空间又分为外部私有存储和外部公共存储，前者对应的目录是`/storage/emulated/0/Android/data/package_name`或`/storage/sdcard/Android/data/package_name`，和内部存储空间相似，后者对应的就是前者以外的目录。

#### 外部私有存储

外部私有存储和内部存储有些类似，除了对应程序不用申请权限即可访问、数据会跟着应用卸载一同被清除之外，它还有两个特点：1） 只有在应用中调用API访问外部私有存储目录时，才会创建以package_name命名的私有存储目录；2）Android 7.0之前，其他应用程序可以采用`file://`格式的Uri直接访问某个程序的外部私有存储目录下的数据，Android 7.0之后就必须使用FileProvider。

#### 外部公共存储

外部公共存储主要存放和应用无关的数据，这些数据在卸载App的时候不会被删除。它还具有这些特点：1）应用要申请权限才能访问，在Android 6.0以后，外部存储权限还要动态申请；2）任何应用只要有外部存储权限，都可以访问公共存储目录下的数据；3）外部公共存储和外部私有存储一样，只占机身存储的一部分，<font color=red>其余的受系统限制无法直接访问</font>。

综上，各种存储空间之间的关系可以用下图来概括：

![](pics/storage.png)

Google[官方文档](https://developer.android.com/training/data-storage)将<u>内部存储空间和外部私有存储</u>统称为<font color=green>“应用专属存储（App-specific storage）”</font>，而<u>外部公共存储</u>称为<font color=blue>“共享存储（Shared storage）”</font>，在后面的内容中也以此为依据进行分析介绍。

#### 判断外部存储空间状态

Android提供了`Environment.getExternalStorageState(···)`来判断指定外部存储目录（**如果不传参数就默认是外部私有存储**）的状态，它会以String的形式返回10种状态中的一种：

|状态|返回值|说明|
|:--|:--|:--|
|MEDIA_UNKNOWN|unknown|状态不明|
|MEDIA_REMOVED|removed|媒介已移除|
|MEDIA_UNMOUNTED|unmounted|媒介未挂载|
|MEDIA_CHECKING|checking|媒介检测中|
|MEDIA_NOFS|nofs|媒介为空或使用了不支持的文件系统格式|
|MEDIA_MOUNTED|mounted|媒介已挂载，可以读写|
|MEDIA_MOUNTED_READ_ONLY|mounted_ro|媒介已挂载，只读|
|MEDIA_SHARED|shared|媒介未挂载，而是通过USB进行共享|
|MEDIA_BAD_REMOVAL|bad_removal|媒介在未脱离挂载的情况下被移除|
|MEDIA_UNMOUNTABLE|unmountable|媒介无法挂载，通常是因为文件系统损坏|
|MEDIA_EJECTING|ejecting|媒介正在退出|

如果返回的是`MEDIA_MOUNTED`或`MEDIA_MOUNTED_READ_ONLY`，那么至少是可以访问的。

### 标准存储目录

系统预置了以下标准存储目录，用于存储对应类型的文件：

|目录|调用值|说明|
|:--|:--|:--|
|`···/Music`|DIRECTORY_MUSIC||
|`···/Podcasts`|DIRECTORY_PODCASTS||
|`···/Ringtones`|DIRECTORY_RINGTONES||
|`···/Alarms`|DIRECTORY_ALARMS|存储用于闹钟响铃的音频文件|
|`···/Notification`|DIRECTORY_NOTIFICATIONS|存储用于通知提示音的音频文件|
|`···/Pictures`|DIRECTORY_PICTURES||
|`···/Movies`|DIRECTORY_MOVIES||
|`···/Downloads`|DIRECTORY_DOWNLOADS||
|`···/DCIM`|DIRECTORY_DCIM||
|`···/Documents`|DIRECTORY_DOCUMENTS||
|`···/Screenshots`|DIRECTORY_SCREENSHOTS|存储截屏图片，Android Q（API = 29）以上使用|
|`···/Audiobooks`|DIRECTORY_AUDIOBOOKS|存储有声电子书，Android Q（API = 29）以上使用|

这些目录可以在应用专属存储和共享存储当中进行创建和文件存取操作。

## 在Android上访问数据文件

### 访问应用专属存储的数据文件

前面已经提到过，应用程序访问专属存储的数据文件是不需要申请权限的，而且Android为应用专属存储提供了以下方法用于获取文件所在的目录：

|方法|用途|
|:--|:--|
|getFilesDir()|获取`/data/data/package_name/files`目录|
|getCacheDir()|获取`/data/data/package_name/cache`目录|
|getExternalCacheDir()|获取`/storage/sdcard/Android/data/package_name/cache`目录|
|getExternalFilesDir()|获取`/storage/sdcard/Android/data/package_name/files`目录|

配合调用这些方法构建File对象，就可以开始操作文件和目录了：

```
val file = File(fileDir,fileName)
```

+ **存储文件**

```
/*使用File对象*/
file.writeBytes(···)
file.writeText(···)

/*不使用File对象*/
context.openFileOutput(fileName, MODE_PRIVATE).use { it:FileOutputStream!
    it.write(···)
    it.flush()
}
```

+ **读取文件**

```
/*使用File对象*/
val s: String = file.readText()
val l: List<String> = file.readLines()
val b: ByteArray = file.readBytes()


/*不使用File对象*/
context.openFileInput("test.txt").bufferedReader().useLines { lines ->
    lines.fold("") { some,text ->
        //TODO
        "$some\n${text}"
    }
}
```

+ **创建和删除缓存文件**

创建缓存文件有两种方式，一种通过调用File的扩展函数，另一种则是构建File对象：

```
/*方式一*/
File.createTempFile(fileName,fileExtension,cacheDir)

/*方式二*/
File(cacheDir,fileName).apply {
    if (!exists()) {
        createNewFile()
    }
}
```

方式一中，需要用户传入文件名、文件扩展名以及cache目录路径，如果文件扩展名为空，系统会自动为文件添加`.tmp`扩展名。此外，系统还会在文件名后面自行添加一串数字。如果介意的话就使用方式二，这样可以确保文件名和文件扩展名都是由用户自行决定的。

删除缓存文件也有对应的两种方式：

```
/*方式一*/
File(cacheDir,fileName).delete()

/*方式二*/
context.deleteFile(cacheFileName)
```

>注意，方式二不光能删除缓存文件，只要在应用专属存储范围内能搜索到的文件都可以删。

上面两种方式都是用于删除**单个**缓存文件，如果要清理cache目录下的所有文件，那么就得构建一个File对象遍历目录然后删除：

```
File(cacheDir.toString()).listFiles()?.let { 
    for (i in it) {
        i.delete()
    }
}
```

还有一种情况是存储空间不足，需要清理**所有**应用的缓存文件，那么就可以发送一条携带有action参数`ACTION_CLEAR_APP_CACHE`的隐式Intent广播。Google官方对此作出了一个提醒：

>The ACTION_CLEAR_APP_CACHE intent action can substantially affect device battery life and might remove a large number of files from the device.
>
>以`ACTION_CLEAR_APP_CACHE`作为action参数的Intent会对设备电量产生显著影响并移除大量文件。

### 访问共享存储的数据文件

共享存储的数据文件包括多媒体、文档以及其他可以被共享使用的文件（Shared Preferences和数据库不在此列），如果要访问它们，就必须在AndroidManifest当中声明以下权限：

|版本|申请权限|说明|
|:--|:--|:--|
|Android 6.0以下|`READ_EXTERNAL_STORAGE`<font color=red>和</font>`WRITE_EXTERNAL_STORAGE`|不需要动态申请|
|Android 6.0到Android 9|`READ_EXTERNAL_STORAGE`<font color=red>和</font>`WRITE_EXTERNAL_STORAGE`|需要动态申请|
|Android 10|`READ_EXTERNAL_STORAGE`<font color=blue>或</font>`WRITE_EXTERNAL_STORAGE`|同上|
|Android 11起|`READ_EXTERNAL_STORAGE`|同上|

### 存储访问框架的使用

[存储访问框架](https://developer.android.google.cn/guide/topics/providers/document-provider)（Storage Access Framework，SAF）是Google提供的一个跨应用文件访问方案。SAF在Android 4.4时期就已经引入，借助SAF，用户可轻松浏览和打开各种文档、图片及其他文件，而不用考虑这些文件来自其首选文档存储提供程序中的哪一个。用户可通过易用的**标准界面**，跨所有应用和提供程序以统一的方式浏览文件并访问最近用过的文件。

更多有关SAF的介绍，可以参考[Google官方文档](https://developer.android.google.cn/guide/topics/providers/document-provider)。

#### 创建新文件

SAF创建新文件时采用的是隐式Intent，action名为`Intent.ACTION_CREATE_DOCUMENT`。在配置Intent时，应指定文件的名称和MIME类型，并且还可以根据需要使用`EXTRA_INITIAL_URI`的Intent extra配置，来指定文件选择器在首次加载时应显示的文件或目录的URI。示例代码如下：

```
val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
    addCategory(Intent.CATEGORY_OPENABLE)
    // 设置文件MIME类型
    type = "application/pdf"
    // 设置文件名称
    putExtra(Intent.EXTRA_TITLE, "invoice.pdf")
    // Android 8.0开始才有DocumentsContract.EXTRA_INITIAL_URI
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        putExtra(DocumentsContract.EXTRA_INITIAL_URI, pickerInitialUri)
    }
}
```

> 常见的MIME类型，可参考[这里](ComputerNetwork/Chapter_6_应用层?id=通用互联网邮件扩充mime)。

在调用上述Intent之后，设备会弹出类似于下图的标准界面，指示要创建的文件需保存到何种目录下，同时也可以让用户自己选择要保存的位置：

<img src="./Android/pics/saf.png" width="360" height="720" /> <img src="./Android/pics/saf2.png" width="360" height="720" />

#### 打开文件

SAF打开文件也是使用隐式Intent，action名为`ACTION_OPEN_DOCUMENT`。此Intent会打开系统的文件选择器应用，若要仅显示应用支持的文件类型，还需指定MIME类型。此外，还可以根据需要使用`EXTRA_INITIAL_URI`的Intent extra配置，来指定文件选择器在首次加载时应显示的文件的URI。示例代码如下：

```
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
    addCategory(Intent.CATEGORY_OPENABLE)
    // 设置仅显示应用所支持的文件类型
    type = "application/pdf"
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
       putExtra(DocumentsContract.EXTRA_INITIAL_URI, Environment.DIRECTORY_DOCUMENTS.toUri())
    }
}
```

在调用上述Intent之后，设备会弹出类似于下图的标准界面，让用户自行选择在何种目录下打开何种文件：

<img src="./Android/pics/saf3.png" width="360" height="720" /> <img src="./Android/pics/saf4.png" width="360" height="720" />

注意到上面左图中PDF文件的右侧有一个图标，点击该图标之后，系统会自动为用户搜索合适的应用程序以打开该类型文件。

#### 修改文件

#### 删除文件

### MediaStore的使用

在更高版本的Android系统上，Google已经决意抛弃Java File的IO操作方案，改用一套统一的API为应用提供统一的访问共享存储媒体文件（媒体文件是指图片、音频和视频这三大类型文件）的方式。这个方案就是[MediaStore](https://developer.android.google.cn/reference/android/provider/MediaStore)。MediaStore在Android系统问世之初就已经存在，但彼时的Android系统远比今天要更开放（但功能也更少更不安全），因此它并没有得到许多开发者的关注。随之Android系统对存储空间的约束日益严格，以及高版本系统的市场占有率逐步提升，MediaStore将发挥越来越重要的作用。可以认为，（在高版本系统上）应用访问共享存储的媒体文件，就是围绕MediaStore的使用来展开的。

MediaStore通常会跟[ContentProvider](Android/contpro)一起搭配使用。也就是说，如果使用MediaStore对共享目录的数据文件实施操作，体验上跟使用数据库十分类似。基于这一特点，下面有关MediaStore的基本使用，会根据“增删查改”这四大类操作来展开。

#### 访问媒体文件

#### 添加媒体文件

#### 移除媒体文件

#### 修改媒体文件