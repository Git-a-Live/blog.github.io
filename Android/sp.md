在Android开发中，将数据本地持久化的一种方式，是以key-value的形式保存在特定文件中。这种键值对保存数据的方式适用于保存一些简单的配置信息，比如语言、主题或者设置开关等等。如果要保存极为复杂或者体量极为庞大的数据，正常情况下应该选择后面要提到的数据库，而非本章内容所介绍的键值对数据保存方式。

Android设备常用的键值对数据存储方案有Shared Preferences、MMKV和Data Store。其中Shared Preference和Data Store都是由Google官方先后开发推出，而MMKV则是由腾讯开源的。下面开始逐一介绍它们的适用场景和基本使用方式。

## SharedPreferences

SharedPreferences是Android设备上最常用也最古老的键值对数据存储方案，可实现简单的数据持久化保存（`.xml`文件），前提是用户没有特意删除数据，以及硬件设备没有被破坏。 

目前，它仅支持boolean、float、int、long和string等基本类型的存储，其他自定义的复合类型<font color=red>无法直接存储</font>（一般只能转换成字符串再保存）。 SharedPreferences主要用于存储一些配置信息，类似于Windows下常用的`.ini`文件。 

SHaredPreferences将数据保存在自己创建的文件当中，通过保存上一次用户所做的修改或者自定义参数设定，使得程序再次启动后能够依然保持原有设置。通过使用键值对的方式进行存储，SharedPreferences可以方便地管理写入和读取。

SharedPreferences对象的获取方式有两种：

```
// Activity直接获取：
val shp = getSharedPreferences(keyName, MODE_PRIVATE)

// 非Activity需要传入Context对象：
val shp = context.getSharedPreferences(keyName, MODE_PRIVATE)

```
如果只是读取数据，SharedPreferences对象只需要调用get类方法就够了：

```
val string = shp.getString(keyName, defValue)
val int = shp.getInt(keyName, defValue)
···
```

但是对数据进行写入以及持久化保存，还必须再创建一个SharedPreferences的内部类对象——Editor：

```
val editor = shp.edit()
editor.putInt(keyName, value)
editor.putString(keyName, value)
editor.apply()
// editor.commit
```

注意，在上面的过程中，实施写入操作之后，必须调用apply()或commit方法，否则内容不会写入。

apply()方法和commit()方法的区别在于：前者没有返回值，而且将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘；后者会返回布尔值以供判断写入是否成功，并且同步地提交到硬件磁盘（在多线程中并发使用commit()方法会发生阻塞进而降低执行效率）。

### SharedPreferences的缺陷

SharedPreferences虽然很轻量，使用方式比较简单，但也正因为如此，其定位就不是用来存储大体量的复杂数据，所以在存储大体量复杂数据的时候就会暴露出以下显著缺陷：

+ **内存占用**

每当用户获取到一个SharedPreferences对象时，该对象都会被系统作为**静态变量**缓存起来，这意味着该对象会一直保存在内存中，直至应用进程被终结。

另一个问题是，每个SharedPreferences对象对应一个`.xml`文件，在初始化后这些文件里的键值对数据也会全部加载到内存中，直至应用进程被终结。

在设备配置较高或键值对数据量较少的情况下，这种问题实际上并没有太大影响，但是如果碰到配置较低的设备和较大的数据量，这种内存占用问题就会显得尤为突出。

+ **线程阻塞**

SharedPreferences对象在初始化之后就要从磁盘加载`xml`文件，这个步骤是异步的，因此通常并不能保证执行时间。如果在初始化后立即调用getValue方法，那么就存在调用者线程因`.xml`文件尚未加载未完成而被阻塞的可能性。如果调用者线程是主线程，这种阻塞持续一定时间后就会引发ANR，因此在使用过程中还得注意从初始化到首次读取数据之间，可能需要预留一定的缓冲时间。

+ **类型安全**

Shared Preferences自身并没有相关机制，以确保对同一个key的读写都是相同的数据类型，只能完全依赖开发者自己的代码规范来进行限制。

也就是说，如果开发者在代码中对某个key写入的数据是类型A，但是读取的时候提供的默认值是类型B，那么在编译阶段IDE并不会报错，直到运行时才会抛出`java.lang.ClassCastException`的异常。这种问题本应在编译阶段就可以避免，但是SharedPreferences的类型安全机制缺陷使得该问题被轻易绕过。

+ **多进程支持**

SharedPreferences基本上不支持多进程数据共享。之所以说是“基本上”，是因为SharedPreferences（以前）确实提供有`MODE_MULTI_PROCESS`的标记位，但是作用不大且不保证多进程并发安全性，所以后来就直接被废弃了，Google官方也推荐使用Content Provider来实现跨进程数据共享。

+ **增量式更新**

SharedPreferences仅支持全量更新，而不支持增量式更新。也就是说用户哪怕只更新一个键值对，SharedPreferences也会把所有数据重新写进磁盘文件中，这就会带来不必要的资源消耗。


+ **可读性和可维护性**

SharedPreferences在较为复杂的数据持久化业务逻辑中，还会暴露出可读性和可维护性差的问题，该问题集中体现在key值上。在不甚规范的代码中，key值都是直接硬编码到对应的业务逻辑里面，换句话说可能会出现每次读取都要显式声明key值（而且还没有机制保证每次声明的key值一定都是相同的）。

此外，键值对的含义基本上只能依靠key值来表示，但是这又带来一个问题：如果开发者命名不规范甚至起到误导作用，那么必然会影响到键值对的读写，进而影响相应的执行逻辑。

## Jetpack DataStore

[Jetpack DataStore](https://developer.android.google.cn/jetpack/androidx/releases/datastore?hl=zh-cn)是Google于2020年推出的一个数据存储方案。按照Google官方的说法，DataStore“以异步、一致的事务方式存储数据，克服了SharedPreferences的一些缺点”，采用的异步方案不是传统的多线程，而是Kotlin协程和Flow。

DataStore提供了两种不同的实现：Preferences DataStore和Proto DataStore。其中前者和SharedPreferences一样也是键值对存储，不需要预定义的架构，但也不确保类型安全；后者则是将数据作为自定义数据类型的实例进行存储，但是需要开发者使用[协议缓冲区](https://developers.google.cn/protocol-buffers?hl=zh-cn)来定义架构，从而确保类型安全。由于Proto DataStore构建协议缓冲区的过程较为复杂，因此这里暂时只讨论Preferences DataStore，如果下文没有额外说明，DataStore均指代Preferences DataStore。

DataStore目前并没有内置在Android SDK中，因此需要以依赖库的方式引入：

```
dependencies {
    implementation "androidx.datastore:datastore-preferences:${specified_version}"

    // RxJava2支持，可选
    implementation "androidx.datastore:datastore-preferences-rxjava2:${specified_version}"

    // RxJava3支持，可选
    implementation "androidx.datastore:datastore-preferences-rxjava3:${specified_version}"
}

// 与 Android 无关的依赖项，可选
dependencies {
    implementation "androidx.datastore:datastore-preferences-core:${specified_version}"
}
```

### 创建实例对象

DataStore对象的创建方式跟以往的SharedPreferences有很大不同，参考下面由Google官方提供的示例代码：

```
val Context.myDataStore by preferencesDataStore("filename")

class SomeClass(val context: Context) {
   suspend fun update() = context.myDataStore.edit {...}
}
```

可以发现，DataStore对象**被创建为Context的扩展属性，使用代理方法完成创建，并且位于顶层**。Google在注释中的解释是：

> This should only be called once in a file (at the top level), and all usages of the DataStore should use a reference the same Instance. The receiver type for the property delegate must be an instance of Context.

也就是说，DataStore对象需要通过调用`preferencesDataStore`函数，采用代理方式创建实例，在一个文件中只有一次调用，并且是在顶层调用，以确保它是单例的。此外，还要将它设置为Context扩展属性，因为`preferencesDataStore`函数里有时需要传入一个以Context对象为入参的lambda表达式。Google的做法不是让`preferencesDataStore`函数接收一个Context入参，而是利用Kotlin的语法糖，让被代理的DataStore对象成为Context扩展属性，这样就可以获得一个Context类型的接收器。

如果想从原有SharedPreferences迁移到DataStore，那么还需要在`preferencesDataStore`函数中传入一个名为produceMigrations的lambda表达式，类似于下面的代码：

```
private val Context.dataStore by preferencesDataStore(
    name = USER_PREFERENCES_NAME,
    produceMigrations = { context ->
        // Since we're migrating from SharedPreferences, 
        // add a migration based on the SharedPreferences name
        listOf(SharedPreferencesMigration(context, USER_PREFERENCES_NAME))
    }
)
```

这个操作有点类似于Room迁移数据库，不过要方便得多。可以看到，迁移的核心部分，就是调用`SharedPreferencesMigration`函数，上面代码传入的context对象实际上就来源于接收器。

无论是直接创建还是迁移，最后DataStore对象会在应用的[内部存储空间](/Android/io?id=存储空间)下，也就是`内部存储空间目录/files/datastore`里面，创建`.preferences_pb`格式的存储文件（双击之后可以在IDE中直接查看内容），所有的键值对数据就保存在这个文件里。值得一提的是，迁移之后，SharedPreferences所创建的`.xml`文件会被移除。

### 基本使用

DataStore对象有两个重要函数和一个重要属性值得关注，分别是`edit`和`updateData`函数，以及一个`data`属性，下面开始对它们进行逐一介绍。

#### `edit`函数

`edit`函数的源码如下：

```
public suspend fun DataStore<Preferences>.edit(
    transform: suspend (MutablePreferences) -> Unit
): Preferences {
    return this.updateData {
        // It's safe to return MutablePreferences since we freeze it in
        // PreferencesDataStore.updateData()
        it.toMutablePreferences().apply { transform(this) }
    }
}
```

Google在该函数的注释中对该函数的主要用途做了以下说明：

> Edit the value in DataStore transactionally in an atomic read-modify-write operation. All operations are serialized.
>
> The coroutine completes when the data has been persisted durably to disk (after which DataStore.data will reflect the update). If the transform or write to disk fails, the transaction is aborted and an exception is thrown.

根据上面的源码和注释内容，首先可以明确一点，这个函数在lambda表达式中提供的`MutablePreferences`对象，就是完成读写操作的关键所在；其次，该函数是一个挂起函数，也就意味着读写操作需要在Kotlin协程中执行；此外，这些读写操作是以**事务**的形式封装成原子操作，以强制保证操作能够按顺序执行；最后，如果写入磁盘的时候失败，事务就会直接被终止并抛出异常，待写入的数据也不会被保存到磁盘文件上，避免造成文件损坏。

`MutablePreferences`对象通过`Preferences.Key<T>`类型的key来创建或查找键值对数据，语法上跟Map十分相似。`Preferences.Key<T>`类型key通常由专门的顶层函数`xxxPreferencesKey()`来创建，其中“xxx”前缀表示的是基本类型，比如整型、浮点型、字符串型或是布尔型等，具体可在`PreferencesKey.kt`文件中查看。

#### `updateData`函数

`updateData`函数在`edit`函数的源码中已经出现过，可以发现`edit`函数实际上就是以`updateData`函数作为其底层实现。`updateData`函数的源码如下：

```
public suspend fun updateData(transform: suspend (t: T) -> T): T
```

Google在该函数的注释中对该函数的主要用途做了以下说明：

> Updates the data transactionally in an atomic read-modify-write operation. All operations are serialized, and the transform itself is a coroutine so it can perform heavy work such as RPCs.
>
> The coroutine completes when the data has been persisted durably to disk (after which data will reflect the update). If the transform or write to disk fails, the transaction is aborted and an exception is thrown.

根据上面的源码和注释内容，`updateData`函数通过协程和事务封装成原子操作的形式来完成数据读写以及类似RPC（Remote Procedure Call，跨进程通信机制的一种）这样的重型任务，并且写入磁盘文件失败时就会终止事务并抛出异常。

#### `data`属性

通过查看源码可以知道，`data`属性是一个`Flow`类型的对象。关于`Flow`，这里只做一些简要说明，更详细的内容可以参考[冷数据流Flow](/Kotlin/coroutine3?id=冷数据流flow)。

`data`属性作为一个`Flow`对象，其主要用途是对最新的、持久保留的状态，为用户提供一个高效且可缓存（如果可能的话）的访问方式。当用户试图从磁盘文件中读取数据时，`data`属性要么返回一个值，要么就抛出一个异常。

## MMKV

[MMKV](https://github.com/Tencent/MMKV)是腾讯于2018年开源的一个键值对数据存储方案。MMKV名字来源于Memory Mapped Key-Value的缩写，采用Linux mmap（内存共享映射）原理，适用于**高频同步**读写的场景。MMKV同SharedPreferences和DataStore相比有一个最大的优势，就是支持跨进程调用。

### 工作原理

MMKV的工作原理就是内存映射文件，也就是将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而操作系统会自动回写数据到对应的文件磁盘上（这样就能避免应用崩溃导致数据丢失的问题），即完成了对文件的操作而不必再调用read,write等系统调用函数。内核空间对这段区域的修改也直接反映到用户空间，从而可以实现不同进程间的文件共享。

详细的设计原理可以参考MMKV开发团队的[介绍](https://github.com/Tencent/MMKV/wiki/design)。

### 基本使用

MMKV的导入方式如下：

```
dependencies {
    // 从1.2.8开始，MMKV迁移到Maven Center仓库
    implementation 'com.tencent:mmkv:${specified_version}'
}
```

在自定义Application类中初始化MMKV的根目录（默认就是`内部存储空间/files/mmkv/`）：

```
fun onCreate() {
    super.onCreate()
    val rootDir = MMKV.initialize(this)
    println("mmkv root: $rootDir");
    ...
}
```

使用MMKV的全局单例存取键值对数据：

```
val kv = MMKV.defaultMMKV()

kv.encode("bool", true)
val bValue = kv.decodeBool("bool", false)

kv.encode("int", 114514);
val iValue = kv.decodeInt("int", 1919810)

kv.encode("string", "Hello from mmkv")
val str = kv.decodeString("string")
```

MMKV使用`encode`方法写入键值对数据，使用`decodeValue`方法读取键值对数据。可以发现，MMKV最多就是通过不同的`decodeValue`方法来区分读取的数据是什么类型，而要确保类型安全，还得让开发者记住一个key对应什么类型的value。另外，每个类型的`decodeValue`方法都有两种形式，一种需要用户提供默认值，防止返回null引发异常，另一种则不需要提供默认值，但存在返回null的风险。

关于MMKV的进阶使用方式，可以参考MMKV开发团队的[介绍](https://github.com/Tencent/MMKV/wiki/android_advance_cn)；多进程调用的设计与实现，可以参考[这份说明](https://github.com/Tencent/MMKV/wiki/android_ipc)。