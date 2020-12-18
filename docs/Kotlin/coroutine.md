协程（Coroutine），又称微线程或纤程，是一种并发设计模式，按照Google官方的说法，在Android开发中使用协程，可以“简化异步执行的代码”。

协程本身是一个很早提出的概念，但直到近些年才在某些语言中（如Go、Lua、Python以及Kotlin等）广泛运用。**简单地说，协程是一种轻量级的线程**，它在编程语言层面就能进行调度控制，如协程切换等。一个单一线程（比如主线程）可以开启的协程数量是非常惊人的，且运行时消耗的资源也相对少很多，这种特性在高并发应用的环境中具有极大的优势。但是在了解协程之前，需要先掌握线程的用法。

## 线程

### 基础概念

与线程（thread）紧密相关的一个概念是进程（process）。所谓进程，在一些教科书中被定义为“程序的一次执行”，“系统进行资源分配和调度的一个独立单位”，简而言之就是系统的一个任务。

与线程相关的另一个重要概念是并发（concurrence）。所谓并发，是指若干事件在**同一时间间隔**内发生（同时发生则为<font color=red>并行</font>），宏观上表现为“同时进行”，微观上则是“交替执行”。

还有一个重要概念是异步（asynchronism）。所谓异步，是指发出一个功能调用后，即便调用者还没有得到结果，仍可以继续执行后续操作；而当这个调用完成后，就通过状态、通知和回调等方式来通知调用者。与异步概念相对应的，就是按照事件预定顺序一步一步执行操作的同步（synchronism）。

线程是现代操作系统中调度和分派的基本单位，也是最小的任务执行单元。线程和进程之间属于被包含和包含的关系，简单地说，**进程是操作系统的子任务，而线程就是进程的子任务**。正如操作系统中通常有多个进程并发运行那样，进程中至少有一个线程（即主线程）在运行。

线程概念的出现与实现，跟进程并发执行的资源开销问题有十分密切的联系。在操作系统中，作为资源的拥有者，进程在创建、撤消以及切换方面都需要付出较大的资源开销，尤其是切换线程时，一方面需要保留当前进程的CPU环境，另一方面还要为新进程设置CPU环境。而单个线程由于占用资源较少，因此在创建、撤消和切换的过程中所要付出的代价也比进程小很多。

多线程几乎是所有现代操作系统的必备基础技术，同时也是程序应用最基本的并发模型。

### 基本使用方法

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

#### 使用线程池

线程池（ThreadPool）是Java内置的一种线程复用机制，主要是为了解决频繁创建和销毁大量线程所带来的系统资源消耗问题。线程池内部维护了若干个线程（数量可设置），在没有任务的时候，这些线程处于等待状态，如果有任务则分配一个空闲线程去执行。当线程池满的时候，新来的任务要么放入队列等待，要么增加新的线程进行处理。

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

### 资源协调与同步

### 线程间通信

## 协程

### 基本概念

#### 作用域

#### 挂起

#### 通道

#### 阻塞与非阻塞

### 基本用法

#### 开启协程

#### 异步任务

#### 异常处理