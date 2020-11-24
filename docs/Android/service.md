服务（Service）是Android实现应用程序后台运行的方案，不过它依赖于应用的进程，而非独立运行， 当应用程序被系统终止的时候，服务也会跟着终止。使用服务通常需要开启多线程（不开启多线程就默认在主线程运行，但是存在隐患）， Android常用的多线程机制是AsyncTask， 并且从Android R开始由Kotlin协程替代。

## 基本用法

Android Studio创建服务的方式和其他基本组件一样，也是New - Service，然后在配置窗口中设置一些属性。 新创建的服务只包含一个onBind()方法，而通常使用的方法还包括onCreate()、onStartCommand()以及onDestroy()。 也就是说，一个服务的基本架构可以是这样的：

```
class DemoService : Service() {
    ···
    override fun onBind(intent: Intent): IBinder { /*Activity和Service进行通信时调用*/
        //TODO
    }

    override fun onCreate() { /*第一次创建Service时调用*/
        super.onCreate()
        //TODO
    }

    override fun onDestroy() { /*停止Service时调用*/
        super.onDestroy()
        //TODO
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int { /*启动Service时调用*/
        //TODO
        return super.onStartCommand(intent, flags, startId)
    }
    ···
}
```

当然，仅仅创建了Service还不够，还需要让Activity能够控制它的工作。一般而言， Activity控制Service的方法仍然是通过Intent，具体的使用方式如下：

```
/*启动Service*/
···
val intent = Intent(applicationContext,DemoService::class.java)
startService(intent)
···

/*停止Service*/
···
val intent = Intent(applicationContext,DemoService::class.java)
stopService(intent)
···
```

如果希望Service可以自己停下来，而不是依赖Activity的控制， 那么只要在DemoService中任意一个位置执行DemoService().stopSelf()即可。
但是到目前为止，也不过是在Activity中启动和终止Service的运行，似乎没有太多更有用的操作，以及更多的交互。 而Service一旦运行起来就比较独立了，如果没有特别的设置，Activity并不会知道Service具体在做什么以及做得怎么样。 因此，如果希望Activity和Service之间的联系更紧密一些（将二者绑定在一起），就要用到onBind()方法。

首先要在DemoService里面新建一个继承于Binder的内部类：

```
class DemoBinder: Binder(){
    ···
    fun Foo(){···}
    ···
}
```

然后创建一个DemoBinder实例，让onBinder()返回：

```
private val demoBinder = DemoBinder()
override fun onBind(intent: Intent): IBinder {
    ···
    return demoBinder
}
```

在Activity中要编写如下代码，以实现自身和Service的绑定以及解绑：

```
/*创建ServiceConnection匿名类，覆写onServiceDisconnected()和onServiceConnected()*/
val serviceConnection = object : ServiceConnection {
      override fun onServiceDisconnected(name: ComponentName?) {
           //TODO
      }

      override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
           ···
           val demoBinder = DemoService.DemoBinder()
           /*也可以使用val demoBinder = service as DemoService.DemoBinder进行强制的向下转型*/
           /*通过该实例去调用DemoBinder()中的方法，使得Activity对于Service的控制程度得到提高*/
           ···
      }
}
···
val intent = Intent(applicationContext,DemoService::class.java)
bindService(intent, serviceConnection, CODE)
/*绑定Activity和Service，CODE是一个标志位，用于设置绑定模式*/
···
unbindService(serviceConnection)
/*Activity和Service解绑*/
```

注意，任何一个Service在整个应用程序当中都是通用的，任何一个Activity都可以跟它绑定，而且都能获取到相同的DemoBinder实例。 此外，在上文中并没有谈到如何为Service开启多线程，因此如果仅按照上述步骤去创建和使用Service，那么所有的任务都是在主线程中执行。

## 前台服务

Service通常用于后台，那么有没有用于前台的？答案是有的。前台Service通常会以状态栏通知的形式出现， 它的作用是防止Service在系统回收内存时被杀掉，从而可以一直保持运行。最为典型的就是音乐播放器的状态栏通知， 还有某些清理软件强行监控等等。前台Service的使用方式跟通知非常相似， 具体代码如下：

```
override fun onCreate() {
    super.onCreate()
    val intent = Intent(applicationContext,SOME_ACTIVITY::class.java)
    val pendingIntent = PendingIntent.getActivity(applicationContext,REQUEST_CODE,intent,FLAGS)
    /*在合适的时机去执行某个动作，简而言之就是延迟执行的Intent*/
    /*REQUEST_CODE是自己设置的，通常设为0*/
    /*FLAGS用于指定PendingIntent的行为，也可以设置为0*/
    val notification = NotificationCompat.Builder(applicationContext,CHANNEL_ID)
            .setContentTitle(TITLE) /*设置状态栏通知标题*/
            .setContentText(TEXT) /*设置状态栏通知内容*/
            .setWhen(System.currentTimeMillis()) /*设置通知被创建的时间*/
            .setSmallIcon(ICON) /*设置状态栏图标*/
            .setLargeIcon(BitmapFactory.decodeResource(resources,ICON)) /*设置状态栏下拉后的图标*/
            .setContentIntent(pendingIntent) /*设置点击通知后的动作*/
            ··· /*其他设置*/
            .build()
    startForeground(ID,notification) /*启动前台Service，ID为每个通知的唯一标识符，是一个整数*/
}
```

上述代码的含义是：当Service被创建的时候就启动前台Service，并以状态栏通知的形式驻留。 如果希望应用程序一启动就显示状态栏通知，那么就在Activity的onCreate里面直接执行startService()。

注意，从Android 8.0开始，如果想使用前台Service， 除了要在AndroidManifest.xml文件中添加android.permission.FOREGROUND_SERVICE权限之外， 还需要使用一个NotificationChannel类管理通知。因此在创建和启动前台Service之前，要添加如下代码：

```
val CHANNEL_ID = "CHANNEL_ID" /*与NotificationCompat.Builder中的CHANNEL_ID保持一致*/
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){ /*判断系统是否为Android 8.0以上，提高兼容性*/
    val CHANNEL_NAME = "CHANNEL_NAME"
    val IMPORTANCE = NotificationManager.IMPORTANCE_XXX /*通知的重要程度，通常由系统提供使用*/
    val notificationChannel = NotificationChannel(CHANNEL_ID,CHANNEL_NAME,IMPORTANCE)
    manager.createNotificationChannel(notificationChannel)
}
```

NotificationChannel有三个参数：channelId用于指定通知渠道的ID，可以是任意的字符串，全局唯一就可以； channelName用于指定通知渠道的名称，这个是用户可见的，开发者需要认真命名；importance用于指定通知渠道的重要等级：

>NotificationManager.IMPORTANCE_HIGH：紧急，发出声音并提醒通知

>NotificationManager.IMPORTANCE_DEFAULT：默认的通知，发出声音

>NotificationManager.IMPORTANCE_LOW：没有声音

>NotificationManager.IMPORTANCE_MIN：低声

NotificationChannel的常用方法：

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

普通的Service要开启多线程是比较麻烦的，即便是使用AsyncTask也得写不少代码，因此，可执行异步任务且可以实现自动停止的Server类——IntentService应运而生。

IntentService同样可以通过和Service一样的方式从Android Studio中创建，但是Android Studio自带的模板有很多用不到的代码， 所以通常需要精简成下面这种格式：

```
class DemoIntentService : IntentService("DemoIntentService") {
    ···
    override fun onHandleIntent(intent: Intent?) {
        //TODO
    }
    ···
}
```

onHandleIntent()方法中编写要执行异步任务的代码，比如下载、网络操作等等。 Activity中启动IntentService的方式和Service完全相同，只不过要把Intent里面的Service::class.java改成IntentService::class.java。

IntentService能够执行异步任务的原因在于，它在调用onCreate()时就会自动创建一个HandlerThread类的实例。 HandlerThread类继承于Thread，内部封装了Looper，所以可以在这里新建线程并启动。

IntentService能够自动停止的原因在于，它使用了一个handleMessage()方法，这个方法是这样的：

```
@Override
public void handleMessage(Message msg) {
    onHandleIntent((Intent)msg.obj);
    stopSelf(msg.arg1);
}
```

可以看到，当onHandleIntent()方法里面的代码执行完毕之后，紧接着就会执行一个含参stopSelf()方法把IntentService终止。 那么为什么不调用不带参数的stopSelf()方法？因为不带参数的stopSelf()会立即结束服务，而含参stopSelf()则会等待所有的消息都处理完毕才会终止服务。 一般来说，含参stopSelf()在尝试停止服务之前，会先判断最近启动服务的次数是否和参数中的数值相等，如果相等则立刻停止服务，反之则继续执行。

在实际开发中，不建议通过绑定Service的方式启动IntentService，因为IntentService源码中的onBind()默认返回null， 不适合那么干。如果一定要使用onBind()，那么最好还是使用普通的Service。这一点在前文等其他地方中也有强调，对应的东西要在对应的场合去使用， 否则会给开发人员带来困扰。

IntentService中使用Handler、Looper以及MessageQueue机制把消息发送到线程中去执行， 所以多次启动IntentService虽然不会创建新的服务和新的线程，但是会把消息加入到消息队列中等待执行， 而服务一旦停止，就会清除消息队列中的消息，里面等待执行的事件也就得不到执行。 但是当前正在执行的事件（线程）要等到执行完毕之后才会结束，不受服务停止的影响。

## 通知

在介绍前台服务的时候，对于通知的使用已经有过初步的涉及，现在要稍微深入一下。 通知一般用于应用程序在后台运行的情形，可以在Activity、Broadcast以及Service中创建使用，比较灵活。 此外，不管是在哪里创建使用，方式都基本相同。

首先回顾一下Android 8.0及以上的前台服务创建和使用过程：

```
val CHANNEL_ID = "CHANNEL_ID"
val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){ //判断系统是否为Android 8.0以上，提高兼容性
    val CHANNEL_NAME = "CHANNEL_NAME"
    val IMPORTANCE = NotificationManager.IMPORTANCE_XXX //通知的重要程度，通常由系统提供使用
    val notificationChannel = NotificationChannel(CHANNEL_ID,CHANNEL_NAME,IMPORTANCE)
    manager.createNotificationChannel(notificationChannel)
}

val intent = Intent(applicationContext,SOME_ACTIVITY::class.java)
val pendingIntent = PendingIntent.getActivity(applicationContext,REQUEST_CODE,intent,FLAGS)
//在合适的时机去执行某个动作，简而言之就是延迟执行的Intent
//REQUEST_CODE是自己设置的，通常设为0
//FLAGS用于指定PendingIntent的行为，也可以设置为0
val notification = NotificationCompat.Builder(applicationContext,CHANNEL_ID) //与上面的CHANNEL_ID保持一致
            .setContentTitle(TITLE) //设置状态栏通知标题
            .setContentText(TEXT) //设置状态栏通知内容
            .setWhen(System.currentTimeMillis()) //设置通知被创建的时间
            .setSmallIcon(ICON) //设置状态栏图标
            .setLargeIcon(BitmapFactory.decodeResource(resources,ICON)) //设置状态栏下拉后的图标
            .setContentIntent(pendingIntent) //设置点击通知后的动作
            ··· //其他设置
            .build()
startForeground(ID,notification) //启动前台Service，ID为每个通知的唯一标识符，是一个整数
```

显示通知的方式非常简单，就是将startForeground(ID,notification)改成manager.notify(ID,notification)。 

如果希望通知在被用户点击之后消失，那么可以采用两种方式：一种是在创建NotificationCompat实例的时候， 在链式设置中使用.setAutoCancel(true)，另一种方法是调用用manager.cancel(ID)，ID的含义和上文相同。

前台服务部分已经介绍过NotificationChannel的常用方法，下面介绍NotificationCompat.Builder的常用方法：

>setSound() //提供一个Uri，用于在通知发布到此频道时播放声音

>setVibrate() //设置来通知时的手机振动，需要声明权限

>setLights() //设置消息通知灯

>setDefault() //使用默认的声音、振动和消息灯设置

>setStyle(NotificationCompat.···) //设置在通知中可使用长文字和大图片，可通过BigTextStyle().bigText()或者BigPictureStyle().bigPicture()分别传入长文字内容以及图片，下拉之后可查看，长文字和大图片同时使用时，只有最靠近.build()的setStyle()会被应用；长文字和.setContentText()同时使用时，前者会取代后者显示在通知中

>setPriority(NotificationCompat.PRIORITY_XXX) //设置通知优先级，有MAX、HIGH、LOW、MIN和DEFAULT五个等级，DEFAULT的效果等同于不设置优先级