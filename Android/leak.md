LeakCanary是一个非常流行的内存泄漏分析工具，它是Square公司（没错，就是那个开发了OkHttp和Retrofit的Square公司）基于MAT开发的[开源项目](https://github.com/square/leakcanary)，集成方便，使用便捷，配置简单，而且功能强大。

## LeakCanary的导入

把LeakCanary导入到Android项目中非常简单：

```
dependencies {
  // debugImplementation because LeakCanary should only run in debug builds.
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:$specific_version'
}
```

编译运行程序，过滤Logcat日志，查看是否有以下这条日志：

```
D/LeakCanary: LeakCanary is running and ready to detect leaks
```

如果有的话就表明LeakCanary已经成功导入并正常工作。

从2.0开始，LeakCanary通过[ContentProvider](Android/contpro)的方式完成自行安装和初始化，废除了以往必须在Application的`onCreate()`中调用`install()`方法的方式，使得这一工具使用起来更为“无感化”。

## 检测机制

根据官方的说法，LeakCanary会在以下几种场景自动检测是否存在内存泄露的情况：

+ Activity被销毁
+ Fragment被销毁
+ Fragment视图被销毁
+ ViewModel被清理

LeakCanary通过四个步骤自动监测和报告内存泄漏情况：①检测残余对象➡②堆转储➡③堆分析➡④泄漏类型分类。

### 检测残余对象

LeakCanary利用Android lifecycle回调，在Activity、Fragment等组件的`onDestroyed()`方法中自动监测这些实例对象，并将实例传递给ObjectWatcher，使用WeakReference来持有它们。

如果`onDestroyed()`调用，等待5秒并执行GC（Garbage Collection）后，被观察的对象依然保留，就表明有可能存在内存泄漏。LeadCanary会在保留对象达到5个时执行堆转储，并展示一个通知。

### 堆转储

当保留对象达到5个时，LeakCanary执行dump，将Java堆栈信息存入`.hprof`文件。由于执行dump需要花费一些时间，LeakCanary会显示一个Toast。

LeakCanary会在Download目录下生成一个leakcanary-package_name文件夹来存储`.hprof`文件，当然，这需要先获得访问存储的权限，否则LeakCanary会弹出通知提示用户进行授权。

### 堆分析

LeakCanary使用Shark库来分析`.hprof`文件并定位这些被保留对象的位置。

对于每个被保留对象，LeakCanary将分析其泄漏轨迹，即阻止该对象被GC的引用链。在分析结束之后，LeakCanary会展示一个通知概要，同时在Logcat中打印分析结果。这些分析出来的泄漏轨迹会被创建签名，而具有同样签名的泄漏轨迹将被划分一起，表示它们都是由相同的bug所引发的。

上述信息在点击进入LeakCanary之后可以看到更为详细的内容，并且它们都会以可视化的形式呈现给用户。

### 泄漏类型分类

LeakCanary把内存泄漏分成两类，一类是由应用本身引起的（Application Leaks），另一类是由引入的第三方库引起的（Library Leaks）。前者通常由应用开发者自行处理，后者则可能需要联系第三方库的开发人员协助处理。

对于应用本身引起的内存泄漏，LeakCanary有一个专门的数据库用于存储、匹配和识别目前已知的引起内存泄露的原因，其工作方式是在数据库中查找匹配引用的名称，更多细节可以查看[AndroidReferenceMatchers](https://github.com/square/leakcanary/blob/main/shark-android/src/main/java/shark/AndroidReferenceMatchers.kt#L49)这个枚举类。

基于LeakCanary的分析结果，可以参考官方的[指导说明](https://square.github.io/leakcanary/fundamentals-fixing-a-memory-leak/)，进行下一步的内存泄漏排查和处理工作。