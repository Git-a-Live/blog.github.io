协程（Coroutine），又称微线程或纤程，是一种并发设计模式。按照Google官方的说法，Kotlin协程的主要用处在于“**（以同步的写法）来简化异步执行的代码**”，从而避免不自觉地陷入“回调地狱（Callback Hell）”。因此在使用Kotlin协程的时候，开发者无需过多关心异步任务的线程切换问题，几乎只要按照同步的思路编写程序即可。Google官方对于协程十分看重，并且已经在Android R中替代了AsyncTask的使用。

尽管协程本身是一个很早提出的概念，但直到近些年才在某些语言中（如Go、Lua、Python、Kotlin甚至C++20等）广泛运用。值得注意的是，Kotlin协程与其他语言的协程存在一个本质上的差异：作为一种基于JVM的语言，Kotlin在底层还是要靠线程来实现协程——<font color=red>这意味着Kotlin协程在实质上，依然和Java的Executors、Future和ForkJoin，以及Android的Handler和AsyncTask一样是一种线程框架</font>。因此Kotlin协程和其他语言的协程并不是同一种东西，绝不可将它们混为一谈，更不能把其他语言的协程概念和特点套用到Kotlin协程上。

在了解协程之前，需要先掌握线程的用法。因为只有通过逐步接触**最原始**的线程用法，了解到基于原始线程用法执行并发任务的不便之后，才能够理解为什么会有Executor、AsyncTask乃至Kotlin协程这些线程框架的出现和应用。

## 线程

### 基础概念

与线程（thread）紧密相关的一个概念是进程（process）。所谓进程，在一些教科书中被定义为“程序的一次执行”，“系统进行资源分配和调度的一个独立单位”，简而言之就是系统的一个任务。

与线程相关的另一个重要概念是并发（concurrence）。所谓并发，是指若干事件在**同一时间间隔**内发生（同时发生则为<font color=red>并行</font>），宏观上表现为“同时进行”，微观上则是“交替执行”。

还有一个重要概念是异步（asynchronism）。所谓异步，是指发出一个功能调用后，即便调用者还没有得到结果，仍可以继续执行后续操作；而当这个调用完成后，就通过状态、通知和回调等方式来通知调用者。与异步概念相对应的就是同步（synchronism）。

最后一个重要概念是临界资源（critical resource）。临界资源是指一类只能采用**互斥**方式进行共享的资源，简单来说就是在一段时间内该类资源只能提供给一个访问者，比如硬件资源中的打印机。

现在再来介绍线程。线程是现代操作系统中调度和分派的基本单位，也是最小的任务执行单元。线程和进程之间属于被包含和包含的关系，简单地说，**进程是操作系统的子任务，而线程就是进程的子任务**。正如操作系统中通常有多个进程并发运行那样，进程中至少有一个线程（即主线程）在运行。

线程概念的出现与实现，跟进程并发执行的资源开销问题有十分密切的联系。在操作系统中，作为资源的拥有者，进程在创建、撤消以及切换方面都需要付出较大的资源开销，尤其是切换线程时，一方面需要保留当前进程的CPU环境，另一方面还要为新进程设置CPU环境。而单个线程由于占用资源较少，因此在创建、撤消和切换的过程中所要付出的代价也比进程小很多。

多线程几乎是所有现代操作系统的必备基础技术，同时也是程序应用最基本的并发模型。

### 基本使用

#### 创建线程

创建线程主要有两种方式：通过实例化Thread类或实现Runnable接口。某些专业书籍或博客将“实现Callable接口”列为第三种方式，然而<font color=red>其本质上依然是实现Runnable接口</font>，因此这里只介绍上述两种方式。

实例化Thread的方法为：

```
//Kotlin：
val thread = Thread {
    //TODO
}
thread.start()

//Java：
Thread thread = new Thread(() -> {
    //TODO
});
thread.start();
```

实现Runnable接口的方法为：

```
//Kotlin：
val thread = Thread(MyRunnable())
thread.start()

class MyRunnable: Runnable {
    override fun run() {
        //TODO
    }
}

//Java：
Thread thread = new Thread(new MyRunnable);
thread.start();

class MyRunnable implements Runnable {
    @Override
    public void run() {
        //TODO
    }
}
```

>注意，无论采用哪一种方式创建线程，必须<font color=red>调用`start()`方法来启动新线程</font>，直接调用`run()`方法等效于在当前线程执行代码（而这些代码原本需要在其他线程中执行），起不到多线程的效果。此外，一个线程对象只能调用一次`start()`方法。

线程对象可以设置名称和优先级，但是要注意，高优先级的线程不一定能先执行，因为线程的调度由操作系统决定，程序本身无法决定线程的调度顺序，而“程序调度”正是协程对线程所具备的优势。

#### 暂停和中断线程

要暂停某个线程，就需要在其内部的执行逻辑中调用`Thread.sleep()`方法，例如：

```
val thread = Thread {
    //TODO
    Thread.sleep(1000) //当前线程暂停1s
}
```

>注意，调用`Thread.sleep()`方法是有可能抛出异常的，因此通常需要使用`try-catch`语句进行拦截，或是在方法签名中向上抛出。

中断线程则通过调用线程对象的`interrupt()`方法，并且配合检测线程对象的`isInterrupted`标志位来确定线程是否已接收到中断指令，进而执行后续的操作，例如：

```
val thread = MyThread()
thread.start()
thread.interrupt()

class MyThread: Thread() {
    override fun run() {
        while (!isInterrupted) {
            //TODO
        }
    }
}
```

>注意，如果目标线程处于等待状态，在调用`interrupt()`方法时会抛出`InterruptedException`异常并被捕获。

只要目标线程在运行过程中接收到中断指令或捕获`InterruptedException`，都应该立即结束自身线程。

#### 线程状态

线程有以下6种状态：

```
New：新创建，创建线程对象之后、调用start()方法之前
Runnable：可运行，调用start()方法之后、被暂停、中断或终止之前
Blocked：被阻塞，执行某些操作（如网络通信、读写文件等）
Waiting：等待中，在调用Object.wait()或join()方法之后、执行notify()或notifyAll()方法之前
Timed waiting：计时等待，在调用Thread.sleep()等带有超时参数的方法之后
Terminated：被终止，run()方法退出（正常退出或异常终止）
```

大致的状态切换为：

![](pics/Screenshot%202020-12-18%20131529.png)

线程状态可以通过`getState()`方法获得。

#### 守护线程

守护线程是指程序运行时在后台提供某些通用服务的一种线程，比如Java的垃圾回收线程。守护线程具有**自动结束自己生命周期的特性**，亦即只要JVM退出，守护线程也一定会跟着结束退出。如果是非守护线程，尤其是<u>在内部开启了无限循环</u>的非守护线程，就很有可能会阻止JVM正常退出，影响程序的正常使用。

要使用守护线程，方法非常简单，只需要<font color=red>在调用`start()`方法之前</font>通过`isDaemon = true`或者`setDaemon(true)`的方式，将一个普通线程设置成守护线程即可。守护线程在使用上和普通线程几乎没有区别，但有两点需要注意：

>1. 在守护线程中开启的新线程也是守护线程；
>2. 守护线程不应该用于读写文件等访问固有资源的操作，否则可能会因为JVM退出发生数据丢失。

### 线程安全

#### 多线程同步问题

当多个线程同时运行时，线程的调度由操作系统决定，因此任何一个线程随时都可能被暂停，或是恢复执行。在这种情况下，一个单线程中不存在的问题便出现了：<font color=red>如果多个线程同时操作一个可共享的资源变量，那么势必会引发冲突</font>。

事实上，同步问题并非多线程独有，因为在多道程序环境下，多个进程间也同样存在这个问题，并由此衍生出诸如“生产者-消费者”、“哲学家进餐”以及“读者-写者”等一系列关于同步问题具有代表性的描述。

综上可知，同步问题的核心在于：**必须确保同一时刻只有一个线程/进程对共享资源进行操作**。妥善处理同步问题，是保证多线程运行安全的关键所在。

#### 线程同步

之前提到，同步问题的核心在于同一时刻只能有一个线程/进程对共享资源进行操作，亦即在多线程模型下，要保证逻辑正确，对共享资源进行读写时，必须保证一组指令以**原子**方式执行。在Java中，保证一段代码的原子性就是通过<u>加锁</u>和<u>解锁</u>实现的。

##### volatile关键字

`volatile`关键字用于修饰线程间的共享变量，目的是解决可见性问题：当一个线程修改了某个共享变量的值，其他线程能够立刻看到修改后的值。

在JVM中，变量的值保存在主内存，当线程访问变量时，它会先获取一个副本，并保存在自己的工作内存中。如果线程修改了变量的值，虚拟机会在某个<font color=red>不确定</font>的时刻把修改后的值回写到主内存。而这个过程中产生的时间差，就有可能造成多线程之间共享变量不一致的情况发生。

在使用`volatile`关键字修饰共享变量后，就能确保线程访问变量时，总是获取主内存的最新值，并在修改变量后立刻将其回写到主内存。

##### synchronized线程锁

+ **基本使用**

`synchronized`主要用于修饰线程代码块（将一组操作打包成原子操作），目的是解决多线程同步访问共享变量的正确性问题。它的基本使用方式如下：

```
1. 找出修改共享变量的线程代码块；
2. 选择一个共享实例作为锁；
3. 使用synchronized(lockObject) {}包裹代码块。
```

>注意，存在竞争关系的线程之间，使用的加锁对象必须是同一个共享实例。

在使用`synchronized`的时候，无论是否有异常，都会在`synchronized`结束处正确释放锁。

使用`synchronized`进行加锁和解锁，一方面限制了代码块的并发执行，另一方面需要消耗一定的时间，因此会降低程序的执行效率和性能。此外，`synchronized`锁是一种可重入锁，即**能被同一线程反复获取**。

当加锁对象是一个类的`this`实例时，可以将`synchronized`用在修饰类的实例方法上（然而Kotlin当中没有`synchronized`可用），比如：

```
//synchronized修饰方法：
class Demo {
    public synchronized ReturnType foo(param: ParamType) {
        //TODO
    }
}


//等效语法：
class Demo {
    public ReturnType foo(param: ParamType) {
        synchronized (this) {
             //TODO
        }
    }
}
```

对于Java中的静态方法，`synchronized`的加锁对象为该类的`Class`实例，比如：

```
//synchronized修饰静态方法：
class Demo {
    public synchronized static ReturnType foo(param: ParamType) {
        //TODO
    }
}


//等效语法：
class Demo {
    public static ReturnType foo(param: ParamType) {
        synchronized (Demo.class) {
             //TODO
        }
    }
}
```

+ **死锁**

“死锁”在计算机操作系统中可以大致定义为：一组进程中的每个进程都在等待仅由该组其他进程才能引发的事件触发后续操作。将进程换作线程，也能得到相似的定义。而无论是进程还是线程，死锁的产生都需要同时满足以下四个条件：

  1. <font color=red>互斥条件</font>：对所分配到的资源排他使用
  2. <font color=red>请求和保持条件</font>：在持有至少一个不可抢占资源的情况下又提出新的资源请求
  3. <font color=red>不可抢占条件</font>：已获得的资源只有在使用结束后才能被释放
  4. <font color=red>循环等待条件</font> ：持有其他访问者所请求的资源，并请求其他访问者所持有的资源，由此陷入死循环

只有破坏死锁产生的条件，才能解决死锁问题。由于互斥条件是强制要求，因此通常都是从破坏其他三个条件入手，但无论采取什么样的破坏手段，本质上都是为了实现“**不释放资源者不得请求资源**”。

对于多线程来说，各种线程锁就是一种不可抢占、互斥共享的资源。只要在使用中同时满足其他特定条件，就能够引起线程死锁。处于死锁状态的线程无法采取任何措施自行解除这种状态，只能强制结束JVM进程。一种行之有效的办法是：<font color=blue>确保各线程获取线程锁的顺序一致</font>。

+ **线程协调**

`synchronized`锁解决了系统调度下的多线程竞争问题，但是还需要一种机制以确保多线程在一定程度上能够为开发者所控制，进而能够协调地运行。最常见的就是在<u>`synchronized`代码块内</u>对<font color=red>锁对象</font>调用`wait()`、`notify()`或者`notifyAll()`方法。

调用了`wait()`方法的线程会进入等待状态，并释放持有的锁对象，以确保其他线程能获得锁。

在相同锁对象上调用`notify()`或者`notifyAll()`方法，会使处于等待状态的线程被唤醒恢复至运行状态，并在重新获取锁之后才能继续执行`wait()`后面的代码（如果有的话）。

>注意，`notify()`只能随机唤醒众多等待线程中的某一个，而`notifyAll()`可以一次性全部唤醒，因此通常情况下更推荐使用`notifyAll()`。

##### 其他线程锁

##### ThreadLocal

#### 线程安全类

在Java中，一个类如果不作特殊说明，通常都是默认为**非线程安全**的，需要通过`synchronized`等方式对共享变量和相关代码块进行加锁和解锁。事实上，<font color=red>线程安全最关键的地方在于“写”，仅仅是“读”其实并不太会引起线程安全问题</font>。

### 线程池

线程池（ThreadPool）是Java内置的一种线程复用机制，主要是为了解决频繁创建和销毁大量线程所带来的系统资源消耗问题。线程池内部维护了若干个线程（数量可设置），在没有任务的时候，这些线程处于等待状态，如果有任务则分配一个空闲线程去执行。当线程池满的时候，新来的任务要么放入队列等待，要么增加新的线程进行处理。

#### Executors

Java标准库提供了ExecutorService接口，以及FixedThreadPool、CachedThreadPool、SingleThreadExecutor以及ScheduledThreadPool等常见的实现类，这些实现类的创建都被封装到Executors类当中。四种常见实现类的创建和使用方法如下：

+ **FixedThreadPool**

FixedThreadPool是一种<font color=red>线程数固定</font>的线程池，其创建和使用方法为：

```
//创建线程池：
val tp = Executors.newFixedThreadPool(线程数) //传入的线程数决定线程池大小

//提交执行任务：
tp.submit(Task) //Task可以通过继承Runnable和Callable接口来实现，如无必要后面不再说明
tp.submit {
    //TODO
}

//关闭线程池：
tp.shutdown() //执行完当前任务之后再关闭
tp.shutdownNow() //立即关闭
tp.awaitTermination(等待时长,时间单位) //等待一段时间后自动关闭线程池
```

>注意，线程池使用完毕后必须关闭，以免发生内存泄漏。如无特别说明，所有线程池都应当如此。

+ **CachedThreadPool**

CachedThreadPool是一种<font color=red>可以根据任务数量动态调整线程数</font>的线程池，除了在创建上跟FixedThreadPool有所区别之外，其他的使用方式基本一样：

```
val tp = Executors.newCachedThreadPool()
```

>注意，如果想创建一个在某个范围内动态调整线程数的线程池，那么可以考虑实例化ThreadPoolExecutor类。

+ **SingleThreadExecutor**

SingleThreadExecutor是一种<font color=red>仅单线程执行</font>的线程池，通常用于执行顺序固定的任务（用线程做同步任务），遵循队列的FIFO原则：

```
val tp = Executors.newSingleThreadExecutor()
```

+ **ScheduledThreadPool**

ScheduledThreadPool是一种<font color=red>定长的、可执行定时或周期性任务</font>的线程池，但它实际上是继承于ScheduledExecutorService接口，和之前的三个实现类不同（Kotlin自动推导类型不需要特意声明）。其创建和使用方式如下：

```
//创建线程池：
val tp = Executors.newScheduledThreadPool(线程数)

//提交一次性定时任务：
tp.schedule(Task, 等待时长, 时间单位) //等待一段时间后执行任务

//提交周期性触发任务：
tp.scheduleAtFixedRate(Task, 等待时长, 时间间隔, 时间单位) //按照固定时间间隔触发执行任务

//提交固定时间间隔任务：
tp.scheduleWithFixedDelay((Task, 等待时长, 时间间隔, 时间单位) //任务执行完毕后等待固定时间间隔再执行下一个任务
```

#### Future / CompletableFuture

#### ForkJoin

## Kotlin协程

### 基本概念

#### 作用域

作用域（scope）是指**变量和函数执行代码可被访问和生效的区域范围**。例如在一个类中，函数以外定义的属性，其作用域为整个类；而函数以内定义的成员变量，其作用域仅为该函数。Java里面的使用`public static`修饰的类属性和方法，其作用域即为整个项目。

+ **CoroutineScope**

CoroutineScope是Kotlin官方提供的作用域<font color=red>接口</font>，它声明了一个名为`coroutineContext`（协程上下文）的属性。在此基础上，Kotlin又设计了`launch`、`async`以及`runBlocking`等扩展函数，作为协程构建器。注意，这些构建器里面执行的代码也被称作“协程”，因此从这里开始，协程实际上既可以指一种并发设计模型，也可以指一系列执行异步任务的代码，在后续的内容中需要联系上下文加以辨别。

常用构建器如下：

`launch`：用于创建一个不阻塞当前线程的协程，并返回一个Job对象用于管理该协程实例；

`async`：和`launch`类似，但是返回一个Deferred对象，里面包含一个异步代码块返回的结果；

`runBlocking`：创建一个可以阻塞当前线程的协程，官方强烈建议不要在正式开发中使用该构建器。

+ **GlobalScope**

GlobalScope是一个由Kotlin官方实现的最基础的<font color=red>自定义</font>作用域。它可以创建启动**顶级**协程，这些协程在整个应用程序生命周期内运行，不会被过早地被取消。

然而，GlobalScope通常也不与任何生命周期组件绑定，除非手动管理，否则很难满足实际开发中的需求。因此，程序代码应该使用自定义的协程作用域，<font color=red>强烈不建议直接使用GlobalScope</font>。

+ **MainScope**

MainScope是Android领域扩展出的自定义作用域（下面要谈到的lifecycleScope和viewModelScope也是一样），它用于在主线程中创建一个协程，但是需要注意在组件的生命周期结束时手动调用`cancel()`函数取消协程，以防内存泄露。

+ **lifecycleScope**

LifecycleCoroutineScope是一个抽象类，通常在Android中调用的是lifecycleScope这一实例对象，进而调用其内部的抽象方法。

lifecycleScope可以通过`launchWhenCreated`、`launchWhenStarted`以及`launchWhenResumed`等构建器，在合适的声明周期创建并启动协程。

+ **viewModelScope**

在采用了MVVM架构的项目当中，ViewModel内部无法通过lifecycleScope来创建协程，因而Android又为ViewModel提供了一个viewModelScope对象，用于创建管理只在ViewModel生命周期内运行的协程。

Kotlin协程的作用域是一个及其重要的概念，只有对它有足够了解，才能更好地实现结构化并发。

#### 挂起

目前任何的协程代码块，要么写在协程作用域里，要么写在挂起函数当中，绝无例外。同样地，挂起函数要么也是运行在协程作用域里，要么也是运行在另一个挂起函数中。之前已经对作用域进行了初步了解，下面要来谈一谈挂起函数。

挂起函数的“挂起”（suspend）是Kotlin协程中最为核心的概念。它的含义是：

1. **使协程从正在执行该协程的线程上脱离出来；**
2. **切换到挂起函数指定的线程（比如`withContext(Dispatcher.IO)`，IO处理线程）中继续执行；**
3. **在协程执行完毕之后自动切回1里面的原线程（Kotlin中称之为resume）。**

>注意：如果挂起函数没有指定要切换到哪个线程，那么协程就继续在当前线程上运行。

简而言之，挂起操作的本质就是<font color=red>线程切换</font>。这就是为什么说挂起是Kotlin协程这一线程框架当中最核心的概念，因为任何线程框架都必须处理线程间切换的问题，而这个问题也是最为关键的部分之一。

Kotlin协程的`suspend`关键字实际上并不起到直接的挂起作用，而是**用于提示被修饰函数自身有耗时操作，需要以协程方式调用**。因为真正执行这一操作的是Kotlin自带的一系列挂起函数，而这些挂起函数要求自身必须位于协程作用域或另一个挂起函数中。因此如果一个函数里面没有任何跟协程相关的代码，使用`suspend`关键字修饰时会被IDE提示要求删除。

#### 阻塞与非阻塞

Kotlin协程的挂起是“非阻塞式”的，从本质上来说，就是指代码采用同步的（看起来会阻塞线程的）写法，但实际上不会阻塞线程。原因很简单，线程自身当然不可能被已经脱离出去、运行在其他线程的协程所阻塞（亦即所谓的“死道友不死贫道”）。即使不用协程，仅仅使用Java最底层的Thread进行线程切换，也同样能实现非阻塞式。从这个角度来讲，Kotlin协程的异步并不比Java底层Thread的异步更高级。

最后要强调的是，任何耗时操作终归会有一个线程来承载，而承载耗时操作的线程自然是处于阻塞状态的，无非是处理耗时操作有时间长短，阻塞也就有久和不久的差异。

#### 通道

### 基本用法

#### 开启协程

#### 异步任务

#### 异常处理