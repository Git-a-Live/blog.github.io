## 协程的挂起与恢复

上一节曾简单介绍协程只能在协程作用域，或是使用`suspend`关键字修饰的挂起函数中调用。事实上，`suspend`关键字实际上并不能起到直接的挂起、切换以及恢复作用，而仅仅是**用于提示JVM被修饰函数自身有耗时操作，需要以协程方式调用**。真正执行上述操作的，是Kotlin自带的一系列具有`Continuation`实例对象的挂起函数。

### 阻塞与非阻塞

Kotlin协程的挂起是“非阻塞式”的，从本质上来说，就是指代码采用同步的（看起来会阻塞主线程的）写法，但实际上不会阻塞主线程。原因很简单，主线程自身当然不可能被已经脱离出去、运行在其他线程的任务所阻塞（亦即所谓的“死道友不死贫道”）。即使不用协程，仅仅使用Java最底层的Thread进行线程切换，也同样能实现非阻塞式。从这个角度来讲，Kotlin协程的异步并不比Java底层Thread的异步更高级。

最后要强调的是，任何耗时操作终归会有一个线程来承载，而承载耗时操作的线程自然是处于阻塞状态的，无非是处理耗时操作有时间长短，阻塞也就有久和不久的差异。

## 协程上下文与调度器

启动协程需要三大元素：上下文、启动模式和协程体。这三大元素在协程源码中大致是这样的：

```
public fun CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
): Job {
    val newContext = newCoroutineContext(context)
    val coroutine = if (start.isLazy)
        LazyStandaloneCoroutine(newContext, block) else
        StandaloneCoroutine(newContext, active = true)
    coroutine.start(start, coroutine, block)
    return coroutine
}
```

协程体就不用再多介绍了，它接受的是一个被`suspend`关键字修饰的函数对象，对应的就是开发者在构建器中编写的协程代码。

### 上下文

CoroutineContext类型的上下文是指完成某个任务所需要的前置资源和外部环境，其主要作用为<font color=red>携带参数、拦截协程执行以及实现线程切换</font>。在多数情况下，Kotlin提供的现成上下文已经可以满足开发需求，不用开发者自己实现。常见的上下文有两种：

+ CombinedContext，上下文组合，表示很多具体的上下文集合
+ EmptyCoroutineContext，什么都没有的空上下文，默认情况下就是这种

CoroutineContext是一个数据结构，可以理解为一个以key为索引的List。下面摘取了一部分源码用于分析：

```
@SinceKotlin("1.3")
public interface CoroutineContext {
    public operator fun <E : Element> get(key: Key<E>): E?
    public fun <R> fold(initial: R, operation: (R, Element) -> R): R
    public operator fun plus(context: CoroutineContext): CoroutineContext = ...
    public fun minusKey(key: Key<*>): CoroutineContext

    public interface Key<E : Element>

    public interface Element : CoroutineContext {
        public val key: Key<*>
        ...
    }
}
```

CoroutineContext作为一个集合，它的元素就是上述源码中看到的Element。每一个Element都有一个key，因此它可以作为元素出现；同时它也是CoroutineContext的子接口，因此也可以作为集合出现。

CombinedContext是一个CoroutineContext的子类，而且是一个位于CoroutineContextImpl.kt文件当中的内部类，开发者在协程体里面访问到的coroutineContext大多都是这个类型。如果想要找到某一个特别的上下文实现，就需要用对应的Key来查找，比如：

```
GlobalScope.launch {
    println(coroutineContext[Job.Key]) // 打印结果为StandaloneCoroutine{Active}@xxxxxxx
}
```

开发者还可以通过指定上下文为协程添加某些特性，最典型的例子就是给协程添加名称以便调试，比如：

```
launch(CoroutineName("Example")) {
    // TODO
}
```

如果有多个上下文需要添加，只要用`+`进行连接即可：

```
launch(Dispatchers.Main + CoroutineName("Example")) {
    // TODO
}
```

上下文里面还有两个值得注意的子类，一个是ContinuationInterceptor（协程拦截器），另一个是CoroutineDispatcher（协程调度器）。

>ContinuationInterceptor

从源码上看，ContinuationInterceptor是一个直接继承于CoroutineContext.Element的接口，但是CoroutineContext.Element本身也是CoroutineContext的子类，所以它们之间的关系就可以理清了。

ContinuationInterceptor也是一个上下文的实现方向，可以左右协程的执行，同时为了保证其功能的正确性，协程上下文集合永远将它放在最后面。ContinuationInterceptor的工作方式跟OkHttp的拦截器相似，它会拦截协程的Continuation以实现回调。而下面要提到的CoroutineDispatcher在本质上就是一种拦截器，只不过需要处理线程切换的问题。

>CoroutineDispatcher

CoroutineDispatcher是Kotlin协程中专门执行线程切换任务的重要角色，它是一个继承于ContinuationInterceptor的抽象类。

CoroutineDispatcher的`dispatch()`方法会在拦截器的`interceptContinuation()`方法中调用，进而实现协程的调度。所以如果开发者想要实现自己的调度器，继承这个类就可以了，不过通常情况下用现成的就可以了。

目前Kotlin提供有以下几种调度器：

|调度器|描述|
|:--|:--|
|`Dispatchers.Unconfined`|非受限调度器，适用于执行不消耗CPU时间的任务，以及不更新局限于特定线程的任何共享数据（如UI）的协程|
|`Dispatchers.Default`|默认调度器，在主线程之外执行占用大量CPU资源的工作，使用共享的后台线程池，也用于GlobalScope开启的协程|
|`Dispatchers.IO`|IO调度器，采用的是线程池，适合在主线程之外执行磁盘或网络I/O|
|`Dispatchers.Main`|主线程调度器，只能用于与界面交互和执行快速工作|
|`newSingleThreadContext()`|自定义线程调度器，实验性功能，使用结束后需要手动关闭线程或进行重用，以免发生线程泄漏等问题|

根据不同的需求，选用相应的调度器，可以有效提高程序执行的效率和效果。

>注意，非受限调度器是一种高级机制，可以在某些极端情况下提供帮助，而不需要调度协程以便稍后执行或是由此产生副作用，因为某些操作必须立即在协程中执行。非受限调度器不应该在通常的代码中使用。

如果开发者没有显式指明协程需要使用什么样的调度器（即直接调用构建器开启协程），那么默认就是<font color=red><u>直接运行在父协程的上下文中</u></font>。

最后需要注意的一个问题是，既然协程本身是基于线程的一种良好封装，那么通过调度器切换线程就依然不能摆脱由此带来的额外性能开销。频繁的线程切换势必会影响到程序的整体性能，因此，在实际开发工作中，通常只需要在一个线程中执行业务逻辑，只有一些耗时操作才需要切换到指定的线程去处理。

此外，自己通过线程池定义调度器的做法本身没什么问题，但最好只用一个线程，因为多线程除了线程切换开销外，还有后面会提到并发安全问题。

### 启动模式

CoroutineStart类型的参数为协程指定了启动模式，而在Kotlin协程当中，启动模式是一个枚举值，目前有一下四种：

|启动模式|描述|
|:--|:--|
|`DEFAULT`|立即执行协程体|
|`LAZY`|只有在需要的情况下运行|
|`ATOMIC`|立即执行协程体，但在开始运行之前无法取消。被注解`@ExperimentalCoroutinesApi`标记为实验性功能|
|`UNDISPATCHED`|立即在当前线程执行协程体，直到第一个挂起函数调用。被注解`@ExperimentalCoroutinesApi`标记为实验性功能|

最常用的启动模式实际上只有`DEFAULT`和`LAZY`两种。

`DEFAULT`是**饿汉式**启动，构建器调用后，协程就会立即进入待调度状态，一旦调度器准备就绪就可以开始执行。这是最为常用的启动模式。

`LAZY`是**懒汉式**启动，构建器调用后并不会有任何调度行为，协程体也自然不会进入执行状态，直到用户调用了Job对象的`start()`或`join()`方法，就跟Deffered对象调用`await()`方法后的效果类似。

`ATOMIC`这种启动模式的使用比较奇特，简单来说，它只有在Job对象调用`cancel()`方法时才会执行协程体。在Job对象调用`cancel()`之后，`ATOMIC`模式下启动的协程体会立即开始执行，并且在执行到第一个挂起函数前（挂起函数被挂起时会检查协程是否被取消）都不会被取消。换句话说，如果这个协程里没有一个挂起函数，那么调用`cancel()`的效果就等同于普通协程的Job对象调用`start()`。

`UNDISPATCHED`模式跟`ATOMIC`有一些相似，在遇到第一个挂起函数前会一直执行协程体，之后就会根据挂起点本身的逻辑以及调度器来决定后面要怎么执行。

## 协程异常处理

+ **异常处理逻辑**
+ **异常的传递**
+ **异常的监督**

## 通道

## 管道

## 协程间通信

## 并发安全

## 数据流Flow