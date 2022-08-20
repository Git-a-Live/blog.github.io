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

DataStore提供了两种不同的实现：Preferences DataStore和Proto DataStore。其中前者和SharedPreferences一样也是键值对存储，不需要预定义的架构，但也不确保类型安全；后者则是将数据作为自定义数据类型的实例进行存储，但是需要开发者使用[协议缓冲区](https://developers.google.cn/protocol-buffers?hl=zh-cn)来定义架构，从而确保类型安全。

DataStore目前并没有内置在Android SDK中，因此需要以依赖库的方式引入：

+ **Preferences DataStore依赖方式**

```
dependencies {
    implementation "androidx.datastore:datastore-preferences:${specified_version}"

    // RxJava2支持，可选
    implementation "androidx.datastore:datastore-preferences-rxjava2:${specified_version}"

    // RxJava3支持，可选
    implementation "androidx.datastore:datastore-preferences-rxjava3:${specified_version}"
}

// Alternatively - use the following artifact without an Android dependency.
dependencies {
    implementation "androidx.datastore:datastore-preferences-core:${specified_version}"
}
```

+ **Proto DataStore依赖方式**

```
dependencies {
    implementation "androidx.datastore:datastore:${specified_version}"

    // RxJava2支持，可选
    implementation "androidx.datastore:datastore-rxjava2:${specified_version}"

    // RxJava3支持，可选
    implementation "androidx.datastore:datastore-rxjava3:${specified_version}"
}

// Alternatively - use the following artifact without an Android dependency.
dependencies {
    implementation "androidx.datastore:datastore-core:${specified_version}"
}
```

### Preferences DataStore



### Proto DataStore

## MMKV

[MMKV](https://github.com/Tencent/MMKV)是腾讯于2018年开源的一个键值对数据存储方案。MMKV名字来源于Memory Mapped Key-Value的缩写，采用Linux mmap（内存共享映射）原理，适用于**高频同步**读写的场景。MMKV同Shared Preferences和Data Store相比有一个最大的优势，就是支持多进程调用。