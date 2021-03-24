## 基本概念

EventBus（事件总线）是[观察者/发布-订阅模式](DesignPattern/行为型设计模式?id=七、observer)的一种实现，也是一种集中式事件处理机制，它允许不同组件之间进行彼此通信而又不需要相互依赖，从而达到解耦的目的。EventBus同时也是一个开源库，其项目地址为：[https://github.com/greenrobot/EventBus](https://github.com/greenrobot/EventBus)

EventBus主要由以下几个部分组成：

+ <font color=red>Event</font>：事件，可以是任意类型
+ <font color=red>Subscriber</font>：订阅者，接收事件通知
+ <font color=red>Publisher</font>：发布者，发送事件通知

这三个部分之间的交互示意图如下：

![](pics/EventBus-Publish-Subscribe.png)

在Android项目中使用EventBus需要通过Gradle导入依赖：

```
implementation 'org.greenrobot:eventbus:$specific_version'
```

普通的Java/Kotlin项目则通过[Maven](/Maven/maven)导入：

```
<dependency>
    <groupId>org.greenrobot</groupId>
    <artifactId>eventbus</artifactId>
    <version>specific_version</version>
</dependency>
```

## 基本使用

### 线程模型

EventBus有四种线程模型：

+ **POSTING**

默认使用的线程模型，表示事件处理函数的线程跟发布事件的线程在同一个线程。

+ **MAIN**

表示事件处理函数的线程在主线程(UI线程)，因此在这里不能进行耗时操作。

+ **BACKGROUND**

表示事件处理函数的线程在后台线程，不能进行UI操作。如果发布事件的线程是主线程(UI线程)，那么事件处理函数将会开启一个后台线程；如果发布事件的线程是在后台线程，那么事件处理函数就使用该线程。

+ **ASYNC**

表示无论事件发布的线程是哪一个，事件处理函数始终会新建一个子线程运行，同样不能进行UI操作。

### 定义事件类

事件类在本质上就是一个POJO（Plain Ordinary Java Object），里面封装了真正需要进行通信和传递的对象，在Kotlin中可以用Data类来代替：

```
data class EventMessage(
    val realObject: SomeType
)
```

### 事件注册与注销

虽然官方建议EventBus的注册在Activity的`onStart()`中进行，而注销则在`onStop()`中进行，但是在实践当中发现，**如果不使用粘性事件的话**，只有在`onCreate()`和`onDestroy()`进行注册和注销才能确保EventBus发挥作用：

```
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    EventBus.getDefault().register(this) //注册EventBus
}

override fun onDestroy() {
    super.onDestroy()
    if (EventBus.getDefault().isRegistered(this)) { //注销EventBus
        EventBus.getDefault().unregister(this)
    }
}
```

在上面的示例代码中，EventBus对象是通过默认单例来创建的，如果要创建自定义EventBus对象，那么就需要调用`builder()`和`build()`等方法进行链式配置。

### 事件发送与处理

事件的发送是在Publisher一侧进行的：

```
EventBus.getDefault().post(Message(···))
```

而事件的处理则在Subscriber一侧进行：

```
@Subscribe(threadMode = ThreadMode.MAIN)
fun onMessageEvent(message: Message) {
    //TODO：调用传入的Message对象的get方法拿到订阅信息，之后再进行其他处理
}
```

注意到`onMessageEvent()`使用了一个注解，它是在EventBus 3.0阶段加入的一项功能——编译时注解处理，同时也是整个EventBus的<font color=red>核心</font>。在3.0之前，EventBus利用Java反射机制，采用的是运行时注解方案，而这就会在性能上带来损耗。

此外，被@Subscribe所注解的方法完全是由开发者编写的，EventBus并没有对此提出什么要求，只要能完成事件的处理即可。而且处理多少种不同类型的事件，就要跟着编写多少个类似的方法。

### 粘性事件

所谓粘性事件（sticky events），是指发送了之后再订阅依然能够接收到的事件。使用粘性事件需要在两处地方进行修改：

```
//Subscriber
@Subscribe(threadMode = ThreadMode.MAIN,sticky = true)
fun onMessageEvent(message: Message) {
    //TODO：调用传入的Message对象的get方法拿到订阅信息，之后再进行其他处理
}

//Publisher
EventBus.getDefault().postSticky(Message(···))
```

### 优先级

@Subscribe注解当中还有一个属性`priority`，用于指定订阅方法的优先级，它只接收整数，且数字越大，优先级越高。另外需要注意的是，只有当两个订阅方法使用相同的`ThreadMode`参数的时候，它们的优先级才会与`priority`指定的值一致，只有当某个订阅方法的`ThreadMode`参数为POSTING的时候，它才能停止该事件的继续分发。
