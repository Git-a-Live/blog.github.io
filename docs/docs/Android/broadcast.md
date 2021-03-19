广播机制被广泛应用于计算机领域，在Android开发中自然也有这样一套机制。Android开发中的广播机制比较灵活， 每个应用都可以针对自己感兴趣的广播内容进行注册，这样就只会接收到自己关心的广播信息。这些广播信息既可以由系统发出， 也可以由其他应用程序发出。<font color=red>发出广播采用的是Intent，接收广播则使用Broadcast Receiver（广播接收器）</font>。

广播有两种类型，分别是标准广播和有序广播。标准广播是一种**完全异步执行、无法截断**的广播，可以被所有广播接收器同时接收到， 效率很高；有序广播是一种同步执行、可被截断的广播，在**同一时刻只能有一个广播接收器接收该广播**，并且广播接收器会按照优先级从高到低的顺序， 依次接收广播信息。若上一个广播接收器把广播截断，则后续的广播接收器将不再接收到该广播。

## 广播注册

注册广播有两种方式：动态注册和静态注册。动态注册是指在代码中通过创建和配置intentFilter来实现注册，虽然灵活， 但是要在应用启动后才能接收广播；静态注册是指通过在AndroidManifest文件中添加设置\<receiver>和\<intent-filter>标签信息， 以实现应用即使未启动也能接收广播并响应。

### 动态注册

动态注册的方式如下：

```
//MainActivity：
class MainActivity : AppCompatActivity() {
    private lateinit var intentFilter: IntentFilter
    private lateinit var demoBroadcastReceiver: DemoBroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        //利用intentFilter指定action等信息，效果类似于在AndroidManifest文件中编写intent-filter信息
        intentFilter = IntentFilter()
        intentFilter.addAction("ActionTag")
        //设置广播接收器，注册广播
        demoBroadcastReceiver = DemoBroadcastReceiver()
        registerReceiver(demoBroadcastReceiver,intentFilter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(demoBroadcastReceiver)
        //Activity被销毁时注销广播接收器，如果是在异步任务中启用广播接收器，那么最后要调用finish()将其回收
    }
}

//DemoBroadcastReceiver：
class DemoBroadcastReceiver: BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        //TODO：重写该方法以指定接收到广播后的业务逻辑
    }
}
```
### 静态注册

Broadcast Receiver的静态注册和Activity类似，也是分手动和非手动两种。手动执行静态注册不仅需要自己继承BroadcastReceiver编写子类，还需要在AndroidManifest文件中注册，因此在实际开发中更推荐通过`File`➡`New`➡`Other`➡`Broadcast Receiver`的途径创建Broadcast Receiver并自动注册，如下图所示。

![](pics/Screenshot%202020-11-30%20110452.png)

Exported属性表示是否允许接收本程序以外的广播，Enabled属性表示是否启用这个Broadcast Receiver。完成创建之后，Android Studio会自动创建一个BroadcastReceiver的子类：

```
class MyReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // This method is called when the BroadcastReceiver is receiving an Intent broadcast.
        TODO("MyReceiver.onReceive() is not implemented")
    }
}
```

同时，AndroidManifest文件中也会自动新建\<receiver>标签，并完成初步注册。但要真正接收广播，还需要开发者自行设置\<intent-filter>标签的内容：

```
<uses-permission android:name="PERMISSION"/>
···
<application
   <receiver
            android:name=".MyReceiver"
            android:enabled="true"
            android:exported="true">
        <intent-filter>
            <action android:name="ActionTag"/>
       </intent-filter>
       <!--要接收广播，需要开发者自行设置intent-filter-->
    </receiver>
</application>
```

<font color=red>由于静态注册可以使应用在未启动的情况下也能接收到系统广播，许多恶意应用（尤其是某些在中国大陆市场具有垄断地位的应用）都会利用该机制实现后台自动唤醒</font>，因此Android也在削减静态注册Broadcast Receiver的功能。因此，如果要接收系统广播并作出响应， 必须要在AndroidManifest文件里面设置相关权限，否则引起应用会崩溃。

>注意，Broadcast Receiver不允许开启多线程，而且在重写onReceive()方法时，不要添加过多的业务逻辑甚至是耗时操作， 否则可能会因为满足BroadcastQueue Timeout条件而触发[ANR](Android/perf?id=anr)。

## 广播发送

在之前的内容中已经提到，广播分为标准广播和有序广播，标准广播不可被截断，能被所有广播接收器同时接收到， 有序广播只能从高优先级的广播接收器依次向低优先级的广播接收器传递，并且还有可能被中间的某个广播接收器截断。

### 标准广播

标准广播的发送方式为：

```
val intent = Intent("ActionTag")
intent.setPackage("PackageName")
sendBroadcast(intent)
```

和Activity一样，Broadcast的ActionTag不仅可以使用系统提供的标准内容，还可以使用自定义的内容。 由于使用了Intent，因此可以通过Intent携带一些数据传递给合适的Broadcast Receiver。

>注意，在<font color=red>Android 8.0</font>以后，静态注册的Broadcast Receiver无法接收到隐式Intent形式的隐式广播，因此必须调用`setPackage()`指定该广播所要发往的目标程序，使之成为显式广播。

### 有序广播

有序广播的发送方式如下：

```
val intent = Intent("ActionTag")
intent.setPackage("PackageName")
sendOrderedBroadcast(intent, "Permission")
```
Permission是一个与权限相关的字符串，如果在\<uses-permission>中设置过指定Permission，那么只有在`sendOrderedBroadcast()`指定了Permission的Broadcast Receiver才能接收到广播。 

而将其设置为null的话，就和标准广播没有多大区别， 这时就要在AndroidManifest文件中设置Broadcast Receiver的优先级，从而确保高优先级者先接收到广播：

```
<receiver android:name=".MyReceiver"
             android:enabled="true"
             android:exported="true">
       <intent-filter android:priority="Priority"> <!--Priority是整数，数字越大则优先级越高-->
            < action android:name="ActionTag"/>
       </intent-filter>
</receiver>
```

未设置优先级的Broadcast Receiver默认排在设置过优先级的Broadcast Receiver后面，而且只要广播不被在它之前的Broadcast Receiver截断，一般都是最后才接收到广播。 

如果给不同的Broadcast Receiver设置了同样的优先级会发生什么情况？答案是等效于标准广播 ——但是请注意，要在对应的场合使用合适的方法，尽量不要把这些方法用在不常见的用途上，这样只会带来困扰。

广播截断的实现方法非常简单，就是在重写`onReceive()`时直接调用`abortBroadcast()`即可。

### 其他

之前谈到的标准广播和有序广播都属于系统全局广播，使用系统全局广播虽然很方便， 但也因此存在一定的隐患：若广播中携带有重要数据，则可能会被某些应用截获，带来安全性问题；此外，某些应用会不停发送垃圾广播， 这可能也会影响到其他应用的正常使用。 在这种情况下，本地广播（Local Broadcast）就可以发挥重要作用了。

然而，本地广播的使用已经在2018年年底被Google 废弃了， 官方的建议是使用LiveData的观察模式或是其他方式来替代本地广播。