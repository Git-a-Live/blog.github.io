## ANR

ANR是Application Not Responding（程序无响应）的缩写，它是由于**负责更新界面的应用主线程无法处理用户输入事件或绘制操作**所引起的。ANR由消息处理机制保证，Android在系统层实现了一套精密的机制来发现ANR，核心原理就是消息调度和超时处理。发生ANR时，系统通常（有些定制过的Android系统可能会禁止提示ANR）会弹出类似于下面的提示框，十分影响用户体验。

![](pics/Screenshot%202020-12-12%20141622.png)

引起ANR的原因主要有四种：
1. **InputDispatching Timeout**：表示5s内无法响应屏幕触摸事件或键盘输入事件；
2. **BroadcastQueue Timeout**：表示在执行BroadcastReceiver的`onReceive()`函数时，前台10s内（后台60s内）没有处理完成；
3. **Service Timeout**：表示Service在前台20s内，后台200s内未将任务执行完毕；
4. **ContentProvider Timeout**：表示ContentProvider的publish在10s内未能执行完毕。

用于分析ANR的方法也有四种，包括：

+ 查看Logcat内容
+ 分析traces.txt文件
+ 利用`jps`和`jstack`指令分析线程情况
+ 使用Android Studio的Android Profiler分析应用运行情况 

规避ANR的核心其实只有一条原则：**不要让主线程被长时间阻塞**。具体来说，就是可以采取诸如避免在主线程里做耗时操作、避免CPU满负荷运行以及优化内存使用等措施。

## 布局优化

Android系统在渲染UI界面的时候会消耗大量资源，而布局优化的出发点就是减少这个过程中的系统资源消耗。简单来说，布局优化的主要措施就是**减少布局层级和控件数量**，避免在短时间内做大量渲染工作，从而避免UI绘制过程发生阻塞造成卡帧等问题。在实际开发中，可以采取以下措施：

### 优化布局层级

优化布局层级的核心措施就是**避免无用嵌套**，在传统的Android开发中，开发者往往大量嵌套使用LinearLayout，这就容易导致View Tree高度急剧增长，渲染层级也随之加深，性能自然会大受影响。Google官方将Android Studio的项目默认布局从LinearLayout变更为ConstraintLayout，使得布局层级**扁平化**，其主要原因也在这里。

而近年来Android原生开发逐步转向H5开发（如小程序等Web应用），可谓是釜底抽薪之计，既然连原生应用都不需要开发了，自然也就没有App的布局优化问题。

### 布局代码复用

布局代码复用不仅仅是为了提高程序可维护性、减低程序冗余度，还为了

+ **使用\<include>**

使用\<include>引入自定义的布局控件组合在[先前](Android/controller)有过提及，用法也比较简单，这里不再赘述。

+ **使用\<merge>**

\<merge>通常和\<include>配合使用，按照Google官方的说法，在一个布局中添加另一个**相同类型**的布局时，使用\<merge> 有助于消除View层次结构中的冗余ViewGroup。

\<merge>使用在被复用的布局中，用于**替换被复用布局的布局标签**：

```
<!--替换前示例标签-->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/app_bg"
        android:gravity="center_horizontal">

<!--替换后示例标签-->
<merge xmlns:android="http://schemas.android.com/apk/res/android">
```

>注意，\<merge>本身不是什么布局类型，因此需要<font color=red>先排布好控件，再替换标签</font>，否则控件是无法直接拖进去的。

可以看到，使用\<merge>之后，原本被复用布局的一些属性会被移除失效。而通过\<include>将这个被复用布局添加到别的布局里面时，系统会忽略\<merge>元素并直接在布局中按照已经排布好的位置放置控件，以代替 <include/>标记（效果就如同直接在该布局中添加排列控件）。

+ **使用\<ViewStub>**

\<ViewStub>专门用于**按需加载布局文件**（比如有些控件不需要在界面初始化的时候加载进来），是一种没有大小，不占用布局的View。\<ViewStub>在XML中的使用方式跟\<include>有些类似，可以通过`android:layout`来引用布局，但是\<ViewStub>还需要在代码中进行一些操作，比如用两种方式让布局可见：

```
//方式一：
viewStub.visibility = View.VISIBLE

//方式二：
viewStub.inflate().findViewById<View>(R.id.xxx)
```

方式一就是简单的让引用布局可见，方式二还可以返回被引用控件的ID。

被\<ViewStub>所引用的布局在加载阶段是看不到的，只有在代码中调用相关的方法使之可见。而且在布局处于曝光可见的状态之后，\<ViewStub>本身就会被布局所替换，不能再次调用。

### 绘制优化

绘制优化是指在自定义View的时候，**避免在`onDraw()`方法中执行耗时操作和繁复操作**。因为该方法可能会被频繁调用，复杂度过高、耗时过长的操作会引起绘制超时（大于16ms），影响界面的渲染效果。

## 内存优化

由于Android应用的沙箱机制，每个应用所分配到的内存大小是有限度的。如果应用所占用的内存资源过高，就会触发LMK（Low Memory Killer）机制来杀死进程以释放内存。应用在内存使用上所出现的常见问题有两大类，一类是内存溢出，另一类是内存泄漏。


### 内存溢出

内存溢出（Out Of Memory，OOM）是指应用程序申请内存时，没有足够的内存供申请者使用，于是抛出异常。造成内存溢出的常见原因有以下几种：

+ **内存中短时间内加载的数据量过于庞大**
+ **内存泄漏导致被占用的内存资源无法回收造成堆积**
+ **应用程序短时间内产生过多对象实体，将内存消耗殆尽**
+ **启动参数内存值设定过小**

在实际的开发工作中，还存在一种被称为“内存抖动”的情况。所谓内存抖动，是指应用程序在运行过程中，短时间内出现内存使用量急剧增长并触发GC机制，之后再重复，形成具有一定周期性规律的“急剧增长-GC-再急剧增长-再GC”的现象。内存抖动的主要成因在于应用程序在短时间内创建了大量对象消耗内存，也就是上述第三个原因的情况，触发了GC机制，宏观上表现为内存使用量的周期性涨跌。

避免内存溢出的方法有很多，但基本原则总结起来只有四条，即**分配更多内存、及时释放内存、使用已有缓存**以及**减少代码缺陷**（最关键）。

### 内存泄漏

内存泄漏（Memory Leak）是指程序在申请内存后，无法释放已申请的内存空间，对于JVM来说，就是GC机制无法从内存中删除不再使用的对象，造成内存的无端占用和垃圾堆积。内存泄漏在到达临界点后会引起OOM和应用程序崩溃。造成内存泄漏的常见原因主要有：

+ **被引用对象无法释放**
+ **大量创建重复对象实例**
+ **调用系统资源未释放**

内存泄漏根据发生频率可以分为常发性、偶发性、一次性以及隐式四类。常发性表示发生内存泄漏的代码会被多次执行到，一执行就内存泄露；偶发性表示代码只有在某些特定环境或操作过程下才会发生内存泄漏；一次性表示代码只会被执行一次，或者由于算法上的缺陷，导致总会有一块仅且一块内存发生泄漏；隐式则表示程序在运行过程中不停分配内存，结束后才释放内存，在这个过程中因为没有及时释放内存，因而有内存泄露的隐患。

解决内存泄漏的根本办法在于减少代码缺陷，如避免大量使用静态/成员变量、避免大量创建重复对象、资源的注册和注销必须成对使用、避免多线程长期持有对象引用以及避免大量递归调用等等。

Android Studio上检查内存泄漏的基本工具就是Android Profiler，可以通过它来监测应用程序在一段时间内运行过程中的CPU负荷、内存使用量、网络使用量以及电量消耗状况。当然，Android Studio还提供了一个由Gradle封装好的[Lint](https://developer.android.google.cn/studio/write/lint?hl=zh-cn)工具，在开发UI界面的时候各种下划线提示就是Lint在工作的证明，限于篇幅，这里不做展开。

### LeakCanary

LeakCanary是一个非常流行的内存泄漏分析工具，它是Square公司（没错，就是那个开发了OkHttp和Retrofit的Square公司）基于MAT开发的[开源项目](https://github.com/square/leakcanary)，集成方便，使用便捷，配置简单，而且功能强大。

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

根据官方的说法，LeakCanary会在以下几种场景自动检测是否存在内存泄露的情况：

+ Activity被销毁
+ Fragment被销毁
+ Fragment视图被销毁
+ ViewModel被清理

LeakCanary通过四个步骤自动监测和报告内存泄漏情况：①检测残余对象➡②堆转储➡③堆分析➡④泄漏类型分类。

>检测残余对象

LeakCanary利用Android lifecycle回调，在Activity、Fragment等组件的`onDestroyed()`方法中自动监测这些实例对象，并将实例传递给ObjectWatcher，使用WeakReference来持有它们。

如果`onDestroyed()`调用，等待5秒并执行GC（Garbage Collection）后，被观察的对象依然保留，就表明有可能存在内存泄漏。LeadCanary会在保留对象达到5个时执行堆转储，并展示一个通知。

>堆转储

当保留对象达到5个时，LeakCanary执行dump，将Java堆栈信息存入`.hprof`文件。由于执行dump需要花费一些时间，LeakCanary会显示一个Toast。

LeakCanary会在Download目录下生成一个leakcanary-package_name文件夹来存储`.hprof`文件，当然，这需要先获得访问存储的权限，否则LeakCanary会弹出通知提示用户进行授权。

>堆分析

LeakCanary使用Shark库来分析`.hprof`文件并定位这些被保留对象的位置。

对于每个被保留对象，LeakCanary将分析其泄漏轨迹，即阻止该对象被GC的引用链。在分析结束之后，LeakCanary会展示一个通知概要，同时在Logcat中打印分析结果。这些分析出来的泄漏轨迹会被创建签名，而具有同样签名的泄漏轨迹将被划分一起，表示它们都是由相同的bug所引发的。

上述信息在点击进入LeakCanary之后可以看到更为详细的内容，并且它们都会以可视化的形式呈现给用户。

>泄漏类型分类

LeakCanary把内存泄漏分成两类，一类是由应用本身引起的（Application Leaks），另一类是由引入的第三方库引起的（Library Leaks）。前者通常由应用开发者自行处理，后者则可能需要联系第三方库的开发人员协助处理。

对于应用本身引起的内存泄漏，LeakCanary有一个专门的数据库用于存储、匹配和识别目前已知的引起内存泄露的原因，其工作方式是在数据库中查找匹配引用的名称，更多细节可以查看[AndroidReferenceMatchers](https://github.com/square/leakcanary/blob/main/shark-android/src/main/java/shark/AndroidReferenceMatchers.kt#L49)这个枚举类。

基于LeakCanary的分析结果，可以参考官方的[指导说明](https://square.github.io/leakcanary/fundamentals-fixing-a-memory-leak/)，进行下一步的内存泄漏排查和处理工作。

## 电量优化

电量优化是一个容易被忽视的性能优化点，但是一个应用耗电量的多少，往往也是用户评价该应用好坏的重点之一，因此有必要了解Android开发中的电量优化手段。其实电量优化的原则总结起来只有一条：**应用按需运行**。

按照这个原则，可以采取以下措施减少应用的耗电量：

+ **避免频繁唤醒**
+ **避免长时间使用联网、蓝牙以及定位功能**
+ **避免长时间执行后台任务**
+ **尽量在充电和连接Wifi的条件下进行数据处理**
+ **集中发送网络请求，精简传输数据**