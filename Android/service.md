Service是Android实现应用程序后台运行的方案，不过它依赖于应用的进程，而非独立运行， 当应用程序被系统终止的时候，Service也会跟着终止。使用Service通常需要开启多线程（不开启多线程就默认在主线程运行，但是存在隐患），为了配合Service的使用，Android提供了多线程以外的异步消息处理机制。<font color=red>Android常用的异步消息处理机制有[Handler](#handler)、[AsyncTask](Android/room?id=asynctask)、HandlerThread以及[IntentService](#intentservice)</font>， 并且从Android R开始由[Kotlin协程](https://kotlinlang.org/docs/reference/coroutines/basics.html)替代AsyncTask。

## 基本用法

Android Studio创建Service的方式为`File`➡`New`➡`Service`，然后在配置窗口中设置一些属性，如下图所示。

![](pics/Screenshot%202020-12-02%20133642.png)

新创建的Service只包含一个`onBind()`方法，而通常使用的方法还包括`onCreate()`、`onStartCommand()`以及`onDestroy()`。 也就是说，一个Service的基本架构可以是这样的：

```
class DemoService : Service() {
    override fun onBind(intent: Intent): IBinder {
        //Activity和Service进行通信时调用
    }

    override fun onCreate() {
        super.onCreate()
        //第一次创建Service时调用
    }

    override fun onDestroy() {
        super.onDestroy()
        //停止Service时调用
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        //每次启动Service时调用
        return super.onStartCommand(intent, flags, startId)
    }
}
```

当然，仅仅创建了Service还不够，还需要让Activity能够控制它的工作。Activity控制Service的方法仍然是通过Intent，具体的使用方式如下：

```
//启动Service：
val intent = Intent(this, DemoService::class.java)
startService(intent)

//停止Service：
stopService(intent)
```

如果希望Service可以自己停下来，而不是依赖Activity的控制， 那么只要在DemoService中任意一个位置执行`stopSelf()`即可。

Service一旦运行起来就处于相对独立的状态，如果没有特别的设置，Activity并不会知道Service具体在做什么以及做得怎么样。 因此，如果希望Activity和Service之间的联系更紧密一些，让二者可以进行通信，那么就要用到`onBind()`方法。

首先要在DemoService里面新建一个继承于Binder的**内部类**：

```
class DemoBinder: Binder() {
    fun Foo(){
        //TODO
    }
}
```

然后创建一个DemoBinder实例，并让`onBinder()`返回该实例：

```
private val demoBinder = DemoBinder()
override fun onBind(intent: Intent): IBinder {
    return demoBinder
}
```

在Activity中要编写如下代码，以实现自身和Service的绑定以及解绑：

```
//创建ServiceConnection匿名类的实现：
val serviceConnection = object : ServiceConnection {
      override fun onServiceDisconnected(name: ComponentName?) {
           //TODO
      }

      override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
           val demoBinder = DemoService.DemoBinder() //也可以使用service as DemoService.DemoBinder进行向下转型
           //通过demoBinder去调用DemoBinder()中的方法，使得Activity对于Service的控制程度得到提高
           ···
      }
}

//绑定Service：
val intent = Intent(this, DemoService::class.java)
bindService(intent, serviceConnection, FLAG) //FLAG是一个标志位，用于设置绑定模式，如Context.BIND_AUTO_CREATE

//解绑Service：
unbindService(serviceConnection)
```

注意，任何一个Service在整个应用程序当中都是通用的（单例），任何一个Activity都可以跟它绑定，而且都能获取到相同的DemoBinder实例。 此外，在上文中并没有谈到如何为Service开启多线程，因此如果仅按照上述步骤去创建和使用Service，那么所有的任务都是在主线程中执行。

>注意，在绑定并启动Service之后，必须同时调用`stopService()`和`unbindService()`才能将其销毁。

## 前台Service

Service通常用于后台，那么有没有用于前台的？答案是有的。前台Service通常会以[状态栏通知](#通知)的形式出现， 它的作用是防止Service在系统回收内存时被杀掉，从而可以一直保持运行（事实上，如果想要长期稳定执行后台任务，还可以选择WorkManager）。最为典型的就是音乐播放器的状态栏通知。前台Service的使用方式跟通知非常相似， 具体代码如下：

```
//在Activity中创建前台Service：
val intent = Intent(this, someActivity::class.java)
val pendingIntent = PendingIntent.getActivity(this,REQUEST_CODE, intent, FLAGS)
/** PendingIntent的含义是延迟执行的Intent，其作用为在合适的时机去执行某个动作，
*REQUEST_CODE是自己设置的，通常设为0；FLAGS用于指定PendingIntent的行为，也可以设置为0*/
val notification = NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle(TITLE) //设置状态栏通知标题
            .setContentText(TEXT) //设置状态栏通知内容
            .setWhen(System.currentTimeMillis()) //设置通知被创建的时间
            .setSmallIcon(ICON) //设置状态栏图标
            .setLargeIcon(BitmapFactory.decodeResource(resources,ICON)) //设置状态栏下拉后的图标
            .setContentIntent(pendingIntent) //设置点击通知后的动作
            ···
            .build()
startForeground(ID, notification) //启动前台Service，ID为每个通知的唯一标识符，是一个整数
```

上述代码的含义是：当Service被创建的时候就启动前台Service，并以状态栏通知的形式驻留。 如果希望应用程序一启动就显示状态栏通知，那么就在Activity的onCreate里面直接执行`startService()`。

注意，从Android 8.0开始，如果想使用前台Service， 除了要在AndroidManifest文件中声明`android.permission.FOREGROUND_SERVICE`权限之外， 还需要使用一个NotificationChannel类管理通知。因此在创建和启动前台Service之前，要添加如下代码：

```
val CHANNEL_ID = "CHANNEL_ID" //与NotificationCompat.Builder中的CHANNEL_ID保持一致
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){ //判断系统是否为Android 8.0以上，提高兼容性
    val CHANNEL_NAME = "CHANNEL_NAME"
    val IMPORTANCE = NotificationManager.IMPORTANCE_XXX //通知的重要程度，使用系统提供的值
    val notificationChannel = NotificationChannel(CHANNEL_ID, CHANNEL_NAME, IMPORTANCE)
    manager.createNotificationChannel(notificationChannel)
}
```

NotificationChannel有三个参数：channelId用于指定通知渠道的ID，可以是任意的字符串，全局唯一就可以； channelName用于指定通知渠道的名称，这个是用户可见的，开发者需要认真命名；importance用于指定通知渠道的重要等级：

>NotificationManager.IMPORTANCE_HIGH：紧急，发出声音并提醒通知

>NotificationManager.IMPORTANCE_DEFAULT：默认的通知，发出声音

>NotificationManager.IMPORTANCE_LOW：没有声音

>NotificationManager.IMPORTANCE_MIN：低声

NotificationChannel的其他常用方法：

```
getId()：检索给定通道的ID

enablellights()：如果使用中的设备支持通知灯，则说明此通知通道是否应显示灯

setLightColor()：如果我们确定通道支持通知灯，则允许使用传递一个int值，该值定义通知灯使用的颜色

enablementVisuration()：在设备上显示时，说明来自此通道的通知是否应振动

getImportance()：检索给定通知通道的重要性值

setSound()：提供一个Uri，用于在通知发布到此频道时播放声音

getSound()：检索分配给此通知的声音

setGroup()：设置通知分配到的组

getGroup()：检索通知分配到的组

setBypassDnd()：设置通知是否应绕过“请勿打扰”模式(中断筛选器优先级值)

canBypassDnd()：检索通知是否可以绕过“请勿打扰”模式

getName()：检索指定频道的用户可见名称

setLockScreenVisibility()：设置是否应在锁定屏幕上显示来自此通道的通知

getlockscreendisibility()：检索来自此通道的通知是否将显示在锁定屏幕上

getAudioAttributes()：检索已分配给相应通知通道的声音的音频属性

canShowBadge()：检索来自此通道的通知是否能够在启动器应用程序中显示为徽章
```

## IntentService

### 基本用法

普通的Service要开启多线程是比较麻烦的，即便是使用AsyncTask也得写不少代码，因此，<font color=red>可执行异步任务且可以实现自动停止</font>的Server——IntentService应运而生。

IntentService同样可以通过和Service一样的方式从Android Studio中创建，但是Android Studio自带的模板有很多用不到的代码， 所以通常需要精简成下面这种格式：

```
class DemoIntentService : IntentService("DemoIntentService") {
    ···
    override fun onHandleIntent(intent: Intent?) {
        //此处执行耗时操作
    }
    ···
}
```

`onHandleIntent()`方法中编写执行异步任务的业务逻辑，比如下载、网络操作等等。 Activity中启动IntentService的方式和Service完全相同，只不过要把Intent里面的Service::class.java改成IntentService::class.java。

IntentService能够执行异步任务的原因在于，它在调用`onCreate()`时就会自动创建一个HandlerThread类的实例。 HandlerThread类继承于Thread，内部封装了Looper，所以可以在这里新建线程并启动。

IntentService能够自动停止的原因在于，它使用了一个handleMessage()方法，这个方法是这样的：

```
@Override
public void handleMessage(Message msg) {
    onHandleIntent((Intent)msg.obj);
    stopSelf(msg.arg1);
}
```

可以看到，当`onHandleIntent()`方法里面的代码执行完毕之后，紧接着就会执行一个含参`stopSelf()`方法把IntentService终止。 那么为什么不调用不带参数的stopSelf()方法？**因为不带参数的`stopSelf()`会立即结束服务，而含参`stopSelf()`则会等待所有的消息都处理完毕才会终止服务**。 一般来说，含参`stopSelf()`在尝试停止服务之前，会先判断最近启动服务的次数是否和参数中的数值相等，如果相等则立刻停止服务，反之则继续执行。

>注意，在实际开发中，不建议通过绑定的方式启动IntentService，因为IntentService源码中的`onBind()`默认返回null， 不适合这么干。

### Handler

从上面可以看出，IntentService中使用的是Handler进行处理异步消息。Handler机制包含四个部分：Message、Handler、MessageQueue以及Looper。下面简要介绍一下这四个部分以及Handler的基本使用方式。

+ Message

可以在内部携带少量信息，在不同线程间传递数据。

+ Handler

主要用于发送和处理消息。发送消息调用的是`sendMessage()`或`post()`等方法，处理消息则调用`handleMessage()`方法。

+ MessageQueue

主要用于存放所有通过Handler发送并等待处理的消息。每个线程只会有一个MessageQueue对象。

+ Looper

主要用于从MessageQueue中取出Message，并传递到Handler的`handleMessage()`中。每个线程只包含一个Looper对象，且Looper会调用`loop()`方法进入一个无限循环，以持续检测MessageQueue中是否有Message。

>注意，Looper调用`looper()`进入无限循环并不会引起ANR，因为`Looper.loop()`是主线程执行消息循环的必要组成部分，即便它可能引起主线程的阻塞，**只要它的消息循环没有被阻塞，能够一直处理事件，就不会产生ANR异常**。

Handler机制的一种典型用法如下：

1. 在主线程中创建Handler对象，并重写`handleMessage()`：
   ```
   val handler = object: Handler() {
       override fun handleMessage(msg: Message) {
           //在此处可进行UI操作
       }
   }
   ```
2. 在子线程中创建Message对象，并通过Handler实例发送出去：
   ```
   thread {
        val msg = Message()
        msg.what = value1
        msg.arg1 = value2 //整型数据
        msg.arg2 = value3 //整型数据
        msg.obj = object //Object对象
        handler.sendMessage(msg)
   }
   ```
3. Message被添加到子线程的MessageQueue中等待处理，Looper取出Message后将其发回主线程的`handleMessage()`进行后续处理。

经过上述步骤之后，就可以实现子线程更新UI线程的操作了。

## 通知

在介绍前台Service的时候，对于通知的使用已经有过初步的涉及，现在要稍微深入一下。 通知一般用于应用程序在后台运行的情形，可以在Activity、Broadcast以及Service中创建使用，比较灵活。 此外，不管是在哪里创建使用，方式都基本相同。

首先回顾一下Android 8.0及以上的前台服务创建和使用过程：

```
val CHANNEL_ID = "CHANNEL_ID"
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
    val CHANNEL_NAME = "CHANNEL_NAME"
    val IMPORTANCE = NotificationManager.IMPORTANCE_XXX
    val notificationChannel = NotificationChannel(CHANNEL_ID,CHANNEL_NAME,IMPORTANCE)
    manager.createNotificationChannel(notificationChannel)
}

val intent = Intent(applicationContext,SOME_ACTIVITY::class.java)
val pendingIntent = PendingIntent.getActivity(applicationContext,REQUEST_CODE,intent,FLAGS)
val notification = NotificationCompat.Builder(applicationContext,CHANNEL_ID)
            .setContentTitle(TITLE)
            .setContentText(TEXT)
            .setWhen(System.currentTimeMillis())建的时间
            .setSmallIcon(ICON)
            .setLargeIcon(BitmapFactory.decodeResource(resources,ICON))
            .setContentIntent(pendingIntent) 
            ···
            .build()
startForeground(ID,notification)
```

显示通知的方式非常简单，就是将`startForeground()`改成`manager.notify()`，其余参数保持不变。 

如果希望通知在被用户点击之后消失，那么可以采用两种方式：一种是在创建NotificationCompat实例的时候， 在链式设置中使用.setAutoCancel(true)，另一种方法是调用用manager.cancel(ID)，ID的含义和上文相同。

前台Service部分已经介绍过NotificationChannel的几个常用方法，下面介绍NotificationCompat.Builder的其他常用方法：

```
setSound()：提供一个Uri，用于在通知发布到此频道时播放声音

setVibrate()：设置来通知时的手机振动，需要声明权限

setLights()：设置消息通知灯

setDefault()：使用默认的声音、振动和消息灯设置

setStyle(NotificationCompat.···)：设置在通知中可使用长文字和大图片

setPriority(NotificationCompat.PRIORITY_XXX)：设置通知优先级
```

`setStyle()`可分别通过`BigTextStyle().bigText()`或者`BigPictureStyle().bigPicture()`传入长文字内容以及图片，下拉之后可查看。长文字和大图片同时使用时，只有最靠近`build()`的`setStyle()`会被应用；长文字和`setContentText()`同时使用时，前者会取代后者显示在通知中。

`setPriority()`的通知优先级有MAX、HIGH、LOW、MIN和DEFAULT五个等级，DEFAULT的效果等同于不设置优先级。