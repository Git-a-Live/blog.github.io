广播机制被广泛应用于计算机领域，在Android开发中自然也有这样一套机制。Android开发中的广播机制比较灵活， 每个应用都可以针对自己感兴趣的广播内容进行注册，这样就只会接收到自己关心的广播信息。这些广播信息既可以由系统发出， 也可以由其他应用程序发出。发出广播采用的是Intent，接收广播则使用Broadcast Receiver（广播接收器）。

## 标准广播与有序广播

广播有两种类型，分别是标准广播和有序广播。标准广播是一种完全异步执行、无法截断的广播，可以被所有广播接收器同时接收到， 效率很高；有序广播是一种同步执行、可被截断的广播，在同一时刻只能有一个广播接收器接收该广播，并且广播接收器会按照优先级从高到低的顺序， 依次接收广播信息。若上一个广播接收器把广播截断，则后续的广播接收器将不再接收到该广播。

## 广播注册

注册广播有两种方式：动态注册和静态注册。动态注册是指在代码中通过创建和配置intentFilter来实现注册，虽然灵活， 但是要在应用启动后才能接收广播；静态注册是指在AndroidManifest.xml文件中添加< receiver>标签信息， 这样就可以确保应用即使未启动也能接收广播并响应。注意，无论是动态注册还是静态注册，如果要接收系统广播并作出响应， 必须要在AndroidManifest.xml文件里面设置相关权限，否则应用会崩溃。

### 动态注册

动态注册的方式如下：

```
//MainActivity文件
class MainActivity : AppCompatActivity() {
    private lateinit var intentFilter: IntentFilter
    private lateinit var demoBroadcastReceiver: DemoBroadcastReceiver

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        /*利用intentFilter指定action等信息，效果类似于在AndroidManifest.xml文件中编写intent-filter信息*/
        intentFilter = IntentFilter()
        intentFilter.addAction("ACTION")
        ...

        /*设置广播接收器，注册广播*/
        demoBroadcastReceiver = DemoBroadcastReceiver()
        registerReceiver(demoBroadcastReceiver,intentFilter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(demoBroadcastReceiver)
        /*Activity被销毁时注销广播接收器*/
        /*如果是在异步任务中启用广播接收器，那么最后要调用.finish()将其回收*/
    }
}

//DemoBroadcastReceiver文件
class DemoBroadcastReceiver: BroadcastReceiver() {
    override fun onReceive(context: Context?, intent: Intent?) {
        //TODO：覆写该方法以指定接收到广播后的业务逻辑
    }
}
```
### 静态注册

在Manifest文件中静态注册广播的方式如下：

```
<uses-permission android:name="PERMISSION"/>
···
<application
   ···
   <receiver android:name=".BROADCASTRECEIVER" /*BROADCASTRECEIVER是开发者自己创建的广播接收器*/
             android:enabled="true" /*true表示启用该广播接收器*/
             android:exported="true"> /*true表示允许该广播接收器接收本应用以外的广播*/
       <intent-filter>
            <action android:name="ACTION"/>
       </intent-filter>
   </receiver>
</application>
```
注意，广播接收器不允许开启线程，而且在覆写onReceive()方法时，不要添加过多的业务逻辑甚至是耗时操作， 否则可能会因为满足BroadcastQueue Timeout条件而触发ANR（Application Not Responding，程序无响应）。

## 广播发送

在之前的内容中已经提到，广播分为标准广播和有序广播，标准广播不可被截断，能被所有广播接收器同时接收到， 有序广播只能从高优先级的广播接收器依次向低优先级的广播接收器传递，并且还有可能被中间的某个广播接收器截断。

### 标准广播

标准广播的发送方式为：

```
val intent = Intent("ACTION")
sendBroadcast(intent)
```
没错，就是这么简单，要执行标准广播发送，最少两行代码足矣。ACTION不仅可以使用系统提供的标准内容，还可以使用自定义的内容。 由于使用了Intent，因此可以通过Intent携带一些数据传递给广播接收器。

### 有序广播

有序广播的发送方式如下：

```
val intent = Intent("ACTION")
sendOrderedBroadcast(intent,PERMISSION)
```
PERMISSION是一个字符串，只有在\<uses-permission>中设置过指定PERMISSION的广播接收器才能接收到广播。 如果将其设置为null，那么就和标准广播没有多大区别（或者换个说法，如果有多个广播接收器都设置了同样的PERMISSION，从实际效果上看跟标准广播相差无几）， 这时就要通过在AndroidManifest.xml文件中设置广播接收器的优先级，从而确保高优先级的广播接收器先接收到广播：

```
···
<receiver android:name=".BROADCASTRECEIVER"
             android:enabled="true"
             android:exported="true">
       <intent-filter android:priority="PRIORITY"> <--PRIORITY是整数，数字越大则优先级越高-->
            < action android:name="ACTION"/>
            ···
       </intent-filter>
</receiver>
···
```

未设置优先级的广播接收器通常要排在设置过优先级的接收器后面，只要广播不被截断，一般都是最后才接收到广播。 如果给不同的广播接收器设置了同样的优先级会发生什么情况？相同的优先级和直接使用sendBroadcast()效果类似 ——当然，不同的方法要出现在对应的场合，尽量不要把这些方法用在不常见的用途上，这样只会给开发带来困扰。

广播截断的实现方法非常简单，就是在覆写onReceive()时直接调用abortBroadcast()即可。

### 其他

之前谈到的标准广播和有序广播都属于系统全局广播，使用系统全局广播虽然很方便， 但也因此存在一定的隐患：若广播中携带有重要数据，则可能会被某些应用截获，带来安全性问题；此外，某些应用会不停发送垃圾广播， 这可能也会影响到其他应用的正常使用（如果这种不停发送垃圾广播的应用是开发者自己写出来的，那么就更加不能容忍了）。 在这种情况下，本地广播（Local Broadcast）就可以发挥重要作用了。

然而，本地广播的使用已经在2018年年底被Google 废弃了， 官方的建议是使用Live Data的观察模式或是其他方式来替代本地广播。