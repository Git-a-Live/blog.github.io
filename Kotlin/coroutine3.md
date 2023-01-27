## Kotlin协程的挂起与恢复

上一节曾简单介绍协程只能在协程作用域，或是使用`suspend`关键字修饰的挂起函数中调用。事实上，`suspend`关键字实际上并不能起到直接的挂起、切换以及恢复作用，而仅仅是**用于提示JVM被修饰函数自身有耗时操作，需要以协程方式调用**。真正执行上述操作的，是Kotlin自带的一系列具有`Continuation`实例对象的挂起函数。

Kotlin协程的挂起与恢复能力本质上就是挂起函数的挂起和恢复。协程挂起就是**程序执行流程发生异步调用时，当前调用流程的执行状态进入等待状态**。

### 阻塞与非阻塞

Kotlin协程的挂起是“非阻塞式”的，从本质上来说，就是指代码采用同步的（看起来会阻塞主线程的）写法，但实际上不会阻塞主线程。原因很简单，主线程自身当然不可能被已经脱离出去、运行在其他线程的任务所阻塞（亦即所谓的“死道友不死贫道”）。即使不用协程，仅仅使用Java最底层的Thread进行线程切换，也同样能实现非阻塞式。从这个角度来讲，Kotlin协程的异步并不比Java底层Thread的异步更高级。

最后要强调的是，任何耗时操作终归会有一个线程来承载，而承载耗时操作的线程自然是处于阻塞状态的，无非是处理耗时操作有时间长短上的差异。

### 协程上下文

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

启动模式在上一节已经介绍过，而协程体也不用再多介绍了，它接受的是一个被`suspend`关键字修饰的函数对象，对应的就是开发者在构建器中编写的协程代码。这里主要介绍协程上下文。

`CoroutineContext`类型的上下文是指**完成某个任务所需要的前置资源和外部环境**，其主要作用为<font color=red>携带参数、拦截协程执行以及实现线程切换</font>。在多数情况下，Kotlin提供的现成上下文已经可以满足开发需求，不用开发者自己实现。常见的上下文有两种：

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

CombinedContext是一个CoroutineContext的子类，而且是一个位于CoroutineContextImpl.kt文件当中的内部类，开发者在协程体里面访问到的CoroutineContext大多都是这个类型。如果想要找到某一个特别的上下文实现，就需要用对应的Key来查找，比如：

```
GlobalScope.launch {
    // 打印结果为StandaloneCoroutine{Active}@xxxxxxx
    println(coroutineContext[Job.Key])
}
```

开发者还可以通过指定上下文为协程添加某些特性，最典型的例子就是给协程添加名称以便调试，比如：

```
launch(CoroutineName("Example")) {
    ···
}
```

如果有多个上下文需要添加，只要用`+`进行连接即可，比如下面代码所示：

```
launch(Dispatchers.Main + CoroutineName("Example")) {
    ···
}
```

上下文里面还有两个值得注意的子类，一个是ContinuationInterceptor（协程拦截器），另一个是CoroutineDispatcher（协程调度器）。

> ContinuationInterceptor

从源码上看，ContinuationInterceptor是一个直接继承于CoroutineContext.Element的接口，但是CoroutineContext.Element本身也是CoroutineContext的子类，所以它们之间的关系就可以理清了。

ContinuationInterceptor也是一个上下文的实现方向，可以左右协程的执行，同时为了保证其功能的正确性，协程上下文集合永远将它放在最后面。ContinuationInterceptor的工作方式跟OkHttp的拦截器相似，它会拦截协程的Continuation以实现回调。而下面要提到的CoroutineDispatcher在本质上就是一种拦截器，只不过需要处理线程切换的问题。

> CoroutineDispatcher

CoroutineDispatcher是Kotlin协程中专门执行线程切换任务的重要角色，它是一个继承于ContinuationInterceptor的抽象类。

CoroutineDispatcher的`dispatch()`方法会在拦截器的`interceptContinuation()`方法中调用，进而实现协程的调度。所以如果开发者想要实现自己的调度器，继承这个类就可以了，不过通常情况下用现成的已经可以满足开发需要。

> 注意，非受限调度器是一种高级机制，可以在某些**极端**情况下提供帮助（通常情况下不会用到），而不需要调度协程以便稍后执行或是由此产生副作用，因为某些操作必须立即在协程中执行。

如果开发者没有显式指明协程需要使用什么样的调度器（即直接调用构建器开启协程），那么默认就是<font color=red><u>直接运行在父协程的上下文中</u></font>。

最后需要注意的一个问题是，既然协程（在JVM平台上）是基于线程的一种良好封装，那么通过调度器切换线程就依然不能摆脱由此带来的额外性能开销。频繁的线程切换势必会影响到程序的整体性能，因此，在实际开发工作中，通常只需要在一个线程中执行业务逻辑，只有一些耗时操作才需要切换到指定的线程去处理。

## 异常处理

和其他代码一样，协程运行时也可能会遭遇异常，这种情况下开发者必须考虑异常处理的问题。

### 异常捕获手段

除了传统的`try-catch`方式之外，Kotlin协程还提供了`runCatching`这种内建的工具方法（但是其内部实现还是基于`try-catch`）。但是，对于由不同构建器构造出来的协程，异常的捕获方式也有所不同。

如果是`launch`，异常会在它发生的第一时间被抛出，可以将抛出异常的代码包裹到`try-catch`当中直接捕获。

如果是`async`，情况就比较复杂了：当`async`**作为根协程**（也就是创建了一个`CoroutineScope`实例或是做为supervisorScope的直接子协程）时，异常不会被抛出，只有到调用`await`的时候才会抛出异常，因此`try-catch`应当包裹的是调用`await`的代码；而当`async`在coroutineScope构建器或其他协程创建的子协程中被调用时，它所抛出的异常不仅无法捕获，而且还会传播到它所在的父协程，导致父协程以及同级的其他子协程都被影响，从而全部取消。

#### CoroutineExceptionHandler

#### SupervisorJob

### 异常传播机制

协程的异常是会分发传播的，牵连到其他兄弟协程以及父协程。

当协程因出现异常失败时，它会将异常传播到它的父级，父级会取消其余的子协程，同时取消自身的执行。最后将异常再传播给它的父级。当异常到达当前层次结构的根，在当前协程作用域启动的所有协程都将被取消。

一般情况下这样的异常传播是合理的，但是在应用中处理与用户的交互，当其中一个子协程出现异常，那就可能导致所在作用域被取消，也就无法开启新的协程，最后整个UI组件都无法响应。

当协程出现异常时，会根据当前作用域触发异常传递：

+ coroutineScope
  
一般情况下，协程的取消操作会通过协程的层次结构来进行传播。如果取消父协程或者父协程抛出异常，那么子协程都会被取消。而如果子协程被取消，则不会影响同级协程和父协程，但如果子协程抛出异常则也会导致同级协程被取消和将异常传递给父协程，进而导致整个协程作用域失败。

+ supervisorScope

它的取消操作只会向下传播，一个子协程的运行失败不会影响到其他子协程，内部的异常不会向上传播，不会影响父协程和兄弟协程的运行。

## 并发安全

## Channel

## Flow

Flow是Kotlin协程与响应式编程模型结合的产物，它与[RxJava](/Android/rxjava)非常像，而且二者之间也有相互转换的API，使用起来非常方便。

Flow可以看作是介于[LiveData](/Android/livedata)与RxJava之间的一个解决方案，它有以下特点：

+ Flow支持线程切换和背压；
+ Flow入门的门槛很低，没有那么多傻傻分不清楚的操作符；
+ 简单的数据转换与操作符，如`map`等等；
+ 冷数据流，**不消费则不生产数据**；
+ 属于Kotlin协程的一部分，可以很好地与协程基础设施结合。

现在Google官方也在Android开发领域推动Flow替换LiveData和RxJava，可以预见的是，只要项目是基于Kotlin开发的，那么迟早都会用到Flow。

## Select

