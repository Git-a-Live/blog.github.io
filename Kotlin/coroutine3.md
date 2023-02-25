## Kotlin协程的挂起与恢复

上一节曾简单介绍协程只能在协程作用域，或是使用`suspend`关键字修饰的挂起函数中调用。事实上，`suspend`关键字实际上并不能起到直接的挂起、切换以及恢复作用，而仅仅是**用于提示JVM被修饰函数自身有耗时操作，需要以协程方式调用**。真正执行上述操作的，是Kotlin自带的一系列具有`Continuation`实例对象的挂起函数。

Kotlin协程的挂起与恢复能力本质上就是挂起函数的挂起和恢复。协程挂起就是**程序执行流程发生异步调用时，当前调用流程的执行状态进入等待状态**。

### 阻塞与非阻塞

Kotlin协程的挂起是“非阻塞式”的，从本质上来说，就是指代码采用同步的（看起来会阻塞主线程的）写法，但实际上不会阻塞主线程。原因很简单，主线程自身当然不可能被已经脱离出去、运行在其他线程的任务所阻塞（亦即所谓的“死道友不死贫道”）。即使不用协程，仅仅使用Java最底层的Thread进行线程切换，也同样能实现非阻塞式。从这个角度来讲，Kotlin协程的异步并不比Java底层Thread的异步更高级。任何耗时操作终归会有一个线程来承载，而承载耗时操作的线程自然就是处于阻塞状态的。

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

如果是`async`，情况就比较复杂了：当`async`**作为根协程**（也就是创建了一个`CoroutineScope`实例或是作为supervisorScope的直接子协程）时，异常不会被抛出，只有到调用`await`的时候才会抛出异常，因此`try-catch`应当包裹的是调用`await`的代码；而当`async`在coroutineScope构建器或其他协程创建的子协程中被调用时，它所抛出的异常**不仅无法捕获，而且还会传播到它所在的父协程，导致父协程以及同级的其他子协程都被影响，从而全部取消**。

### 异常传播机制

从前文可以了解到，`try-catch`并不是万无一失的，原因在于Kotlin协程的异常传播机制并不是像人们所想象的那般“循规蹈矩”地只将异常局限在抛出它的地方，而是逐级向上传播直到父协程，并最终导致父协程被取消。这在前面的[结构化并发](Kotlin/coroutine2?id=结构化并发)特性中有过简单描述。

换句话说，Kotlin协程的异常是会分发传播的，并最终牵连到其他兄弟协程以及父协程。

当协程因出现异常失败时，它会将异常传播到它的父级，父级会取消其余的子协程，同时取消自身的执行。最后将异常再传播给它的父级。当异常到达当前层次结构的根，在当前协程作用域启动的所有协程都将被取消。

一般情况下这样的异常传播是合理的，但是在应用中处理与用户的交互，当其中一个子协程出现异常，那就可能导致所在作用域被取消，也就无法开启新的协程，最后整个UI组件都无法响应。这种时候，Kotlin协程的异常传播机制就会成为一个比抛出异常更棘手的问题。通常采用的解决方案有两种：一种是使用全局的异常处理器`CoroutineExceptionHandler`，另一种则是使用`SupervisorScope`。

#### CoroutineExceptionHandler

`CoroutineExceptionHandler`是**处理无法捕获类型的异常**的一种手段。按照官方的说法，`CoroutineExceptionHandler`只用于处理未被捕获的异常，其他能够被正常捕获处理的异常是不会触发其作用的。

> Kotlin官方文档的说明为：“CoroutineExceptionHandler is invoked only on uncaught exceptions — exceptions that were not handled in any other way.”

值得注意的是，`CoroutineExceptionHandler`的使用范围很有限。首先，把它传给子协程是没用的，因为子协程最终会委托父协程去处理异常，所以只能传给父协程使用；其次，前面提到过的`async`构造器也不能用`CoroutineExceptionHandler`，因为`async`捕获的所有异常最后都会传给`Deferred`对象，并不会直接抛出，所以`CoroutineExceptionHandler`也不会起任何作用。

此外，即便用了`CoroutineExceptionHandler`，开发者也不能在里面恢复协程，因为当它捕获到异常的时候，说明协程已经挂掉了，不会再有任何响应。所以官方建议在`CoroutineExceptionHandler`进行的处理通常只有输出日志、提示错误信息、以及终止或重启应用等。

最后要提醒的是，由于`CoroutineExceptionHandler`是**全局异常处理手段**，它并不能保护一个父协程下面的所有子协程不被异常所牵连。所以开发者如果想确保某一子协程抛出的异常不会连累到其他的兄弟协程，要么在局部继续使用传统的`try-catch`，要么考虑使用下文要提到的`SupervisorScope`。

#### SupervisorScope

`SupervisorScope`主要是为解决`CoroutineExceptionHandler`无法保护兄弟协程不受异常牵连的问题而诞生的。当开发者在`SupervisorScope`中开启多个并行任务时，任何一个子协程的运行失败都不会影响到其他子协程，因为内部的异常不会向上传播，也就不会导致父协程被牵连取消，从而影响其他兄弟协程的运行。

#### 小结

通过简单探讨Kotlin协程的异常传播机制以及不同场景下的异常处理手段，可以得出以下结论：

+ 如果想要在代码的特定部分捕获并处理异常，可以使用传统的`try-catch`；

+ 如果是在某个并行任务异常便取消其他所有并行任务的场景，那就利用Kotlin协程的异常传播机制，顶多加个`CoroutineExceptionHandler`去处理；

+ 如果希望多个并行任务间互不干扰，任何一个并行任务失败都不影响其他任务的执行，那就使用`SupervisorScope`。

## Kotlin Flow

`Flow`是Kotlin协程与响应式编程模型结合的产物，它与[RxJava](/Android/rxjava)非常像，而且二者之间也有相互转换的API，使用起来非常方便。

`Flow`可以看作是介于[LiveData](/Android/livedata)与RxJava之间的一个解决方案，它有以下特点：

+ `Flow`支持线程切换和背压；
+ `Flow`入门的门槛很低，没有RxJava那么多的操作符；
+ 简单的数据转换与操作符，如`map`等等；
+ 冷数据流，**不消费则不生产数据**；
+ 属于Kotlin协程的一部分，可以很好地与协程基础设施结合。

> 注意，一般的`Flow`是冷数据流，但是有一部分`Flow`的具体实现，如`ChannelFlow`、`MutableStateFlow`以及`MutableSharedFlow`等却是热数据流。

现在Google官方也在Android开发领域推动`Flow`替换LiveData和RxJava，可以预见的是，只要项目是基于Kotlin开发的，那么迟早都会用到`Flow`。

### Flow的基本使用

`Flow`的创建有三种方式，分别如下列示例代码所示：

```
// 方式一：利用flow构造器
val flow1 = flow {
    // 使用emit“发射”（生产）数据
    emit(···)
}

// 方式二：从可迭代对象调用asFlow扩展函数
val flow2 = (···).asFlow

// 方式三：利用flowOf函数，传入单个值或vararg可变参数
val flow3 = flowOf(···)
```

注意，创建一个`Flow`对象是不需要在协程作用域里进行的。另外，无论是`asFlow`还是`flowOf`，它们的底层实现都是调用了`flow {}`构造器。

`Flow`对象生产的数据想要被消费，就需要调用`collect`函数，如下列示例代码所示：

```
flow {
    ···
}.collect { value ->
    // 消费者在这里消费数据
}
```

由于`collect`函数是一个挂起函数，因此必须在一个协程作用域中进行调用。如果想要取消一个`Flow`，只需要取消`collect`函数被调用的协程作用域即可——前文已经说过，`Flow`是冷数据流，只要没有消费就不会生产数据，因此不必担心会带来资源浪费的问题。后面要提到的`Channel`则跟`Flow`完全相反，不消费也照样生产数据，所以是热数据流。

+ **线程切换**

线程切换是一个在使用`Flow`的过程中值得高度关注的问题。官方在设计`Flow`的时候，就禁止开发者在创建`Flow`的阶段编写线程调度的业务逻辑，原因是为了保证`Flow`上下文的一致性。因此类似于下面的这种代码虽然在静态编译阶段不会出问题，但是只要运行起来就会报错：

```
// 下面这种代码运行时会提示Flow invariant is violated的错误
flow {
    for (i in 1..10) {
        delay(100)
        if (i % 2 == 0) {
            withContext(Dispatchers.IO) {
                emit(i)
            }
        } else {
            emit(i)
        }
    }
}.collect { println("value :$it") }
```

注意，官方只是禁止开发者在创建`Flow`的时候加入线程调度的逻辑，而不是禁止开发者在使用`Flow`的整个过程中进行线程调度，否则就没必要声称`Flow`跟RxJava相似，还白白在宣传上翻车。

最后要说明的是，如果`Flow`想进行线程调度，就得使用下面要介绍到的操作符`flowOn`，这里先不做展开。

### Flow常用操作符

`Flow`的操作符有很多，按照它们各自的调用位置，可以大致划分成起始、中间以及末端三个大类（虽然不是什么官方的划分方式）。

#### 起始操作符

就目前而言，`Flow`的起始操作符实际上就是一系列构造器。因为没有这些构造器的话，`Flow`对象就不知从何而来，更不必讨论后面的操作符了。

起始操作符前面已经介绍一部分了，这里仅仅简单列个表出来：

|操作符|作用描述|
|:-----:|:-----:|
|`flow`||
|`flowOf`||
|`asFlow`||
|`channelFlow`|热数据`Flow`构造器，允许内部切换线程|
|`callbackFlow`|将回调改成`Flow`，类似于`suspendCoroutine`|
|`emptyFlow`|构造一个空数据流|

#### 中间操作符

|操作符|作用描述|
|:-----:|:-----:|
|`onStart`|在上游`Flow`**启动之前**被调用|
|`onEach`|在上游`Flow`的每个值被下游**消费之前**调用|
|`onCompletion`|在流程完成（包括因异常被取消）后调用，传递执行结果和异常（如果有的话）|
|`map`|将上游所发送数据的**值**进行变换|
|`mapLatest`|当有上游发送新数据时，若上个变换还没结束就先将其取消掉|
|`mapNotNull`|仅发送经过变换后其值不为null的数据|
|`transform`|将上游发送数据的**值甚至类型**进行变换，可以执行或跳过变换，也可以重复发送数据，非常灵活|
|`transformLatest`|类似于`mapLatest`|
|`transformWhile`|执行该变换需要返回一个布尔值，若为false则不再进行后续变换|
|`filter`|对上游发送的数据进行过滤，返回符合筛选条件的数据|
|`filterNot`|对上游发送的数据进行过滤，返回**不符合**筛选条件的数据|
|`filterIsInstance<*>`|对上游发送的数据进行过滤，返回属于某一类型的数据|
|`filterNotNull`|对上游发送的数据进行过滤，返回非空数据|
|`zip`|将两个`Flow`发送的数据组合成一个，功能和RxJava中的`zip`一样|
|`take`|从上游数据流中截取前若干个数据，作为新的数据流发送到下游|
|`takeWhile`|类似于`filter`，但是当判断条件为false时就会中断后续操作|
|`drop`|和`take`相反，从上游数据流中丢弃掉前若干个数据，剩余部分作为新的数据流发送到下游|
|`onEmpty`|当上游发送的数据流为空时，对数据流进行处理|
|`catch`|捕获上游处理数据流时所发生的异常，每个有可能抛出异常的中间操作后面，都可以使用该操作符；还可以将异常通过`emit`继续发送出去|
|`flowOn`|切换当前数据流的上下文，仅对当前数据流生效，且不影响`collect`所用的协程上下文；可以在中间操作阶段重复组合使用|
|`buffer`|创建一个缓冲区接收上游（以较慢速度）发送的数据，避免下游消费数据速度也比较慢导致整个生产消费流程耗时极长，这样总时长大约只相当于上游发送第一个数据的耗时 + 下游消费所有数据的总耗时|
|`conflate`|上游发送数据的速度超过下游消费数据的速度时，直接舍弃旧数据，只处理下游来得及消费的新数据|

> `buffer`和`conflate`都是针对响应式编程所出现的背压问题的应对手段，前者是缓冲，后者则是取舍。所谓的背压（back pressure），是指生产者的生产速率**高于**消费者的消费速率，这种速率的不匹配就会造成数据积压。

#### 末端操作符

|操作符|作用描述|
|:-----:|:-----:|
|`collect`|触发`Flow`的运行，最基础常用的数据消费操作|
|`collectIndexed`|带有下标的数据消费操作|
|`collectLatest`|上游发送新数据时，若上一个数据尚未消费完毕，则取消上一个数据的消费操作|
|`toCollection`|将数据流转换成集合，不能在各类`collect`操作后面进行|
|`toList`|将数据流转换成列表，不能在各类`collect`操作后面进行|
|`toSet`|将数据流转换成数据集，不能在各类`collect`操作后面进行|
|`launchIn`|直接触发`Flow`的执行，需要传入协程作用域上下文，返回一个`Job`，通常跟`onEach`之类的操作符搭配使用|
|`first`|获取上游数据流的第一个数据，若遭遇空值则抛出异常|
|`firstOrNull`|在`first`的基础上提高了空安全性|
|`single`|仅尝试获取上游数据流的第一个数据，若为空或有多个值就抛出异常|
|`singleOrNull`|在`single`的基础上提高空安全性，仅保留第一个数据，多余的数据全部置为null|
|`count()`|计算数据流中所包含数据的数量，注意跟带有lambda入参的另一个`count{}`相区别|
|`fold`|接收一个初始值，然后需要在lambda表达式中，返回一个初始值与数据流内各个数据交互（比如数学运算）之后的最终结果|
|`reduce`|作用跟`fold`相似，只不过初始值变成了数据流的第一个数据|

## Kotlin Channel

`Channel`是一种**热数据通道**。如果要探究`Channel`的本质，就会发现它实际上就是一个**并发安全**的队列。通常情况下，`Channel`被用作两个协程之间进行通信的管道。

### Channel的基本使用

#### `Channel`的创建

要创建一个`Channel`，就目前的源码来看，一共有两种方式：一种是调用Kotlin协程库提供的`Channel<*>()`工厂函数，另一种就是创建一个子类或匿名类去实现`Channel`。前者是最为简单的，而后者相当麻烦，因为`Channel`作为一个接口，其内部要实现的接口函数实在是太多了。下面的示例代码就初步展示了两种创建方式麻烦程度的差异：

```
// 通过协程库内置的工厂函数直接创建
val channel = Channel<T>()

// 通过实现Channel接口来获得Channel对象
val channelImpl = object ::Channel<T> {
    ···
    override fun cancel(cause: Throwable?): Boolean {
        ···
    }

    override fun close(cause: Throwable?): Boolean {
        ···
    }

    override suspend fun send(element: T) {
        ···
    }

    override suspend fun receive(): T {
        ···
    }

    ···
}
```

注意，`Channel`本身是一个接口，而且还继承了`SendChannel`和 `ReceiveChannel`，因此`Channel`对象同时具备发送和接收数据的能力，这也是它能作为协程间通信管道的重要原因。另外，从上面的示例代码中还可以发现，`Channel`接口的`send`和`receive`全都是**挂起函数**，这进一步强化了`Channel`与Kotlin协程的绑定。最后要说明的是，除非确有强烈需求，否则通常情况下只要使用工厂函数直接创建一个`Channel`对象使用就行。

#### `Channel`数据生产与接收

在创建`Channel`对象之后，不同角色的协程就可以利用这个`Channel`来进行通信了，如下列代码所示。再次强调，`Channel`是并发安全的队列，开发者可以直接利用它来传递普通类型的数据，而不需要使用Atomic类进行额外封装。

```
// 创建一个传递指定类型数据的Channel
val channel = Channel<T>()

// 创建一个生产者协程
val producer = launch {
    while (true) {
        // 生产者发送数据
        channel.send(···)
    }
}

// 创建一个消费者协程
val consumer = launch {
    while (true) {
        // 消费者接收数据
        channel.receive()
    }
}

// 如果有必要就阻塞父协程，确保生产者和消费者可以一直发送和接收数据
producer.join()
consumer.join()
```

值得注意的是，`Channel`通常都用于**一对一**的通信场景。也就是说，一个`Channel`的两头只能是一个生产者和一个消费者。

如果有多个消费者调用了同一个`Channel`对象的`receive`函数，那么最先调用者将消费掉生产者发送的数据，而其他消费者只能空等。基于这种情况，如果想要确保逻辑执行的可预测性，一个`Channel`只应被一个消费者所持有。或者也可以这样认为：消费者通过一般的`Channel`对数据进行接收和消费的操作是**互斥**的。如果希望一个`Channel`可以支持多个消费者接收数据，那么就应该考虑使用`BroadcastChannel`，但是截止到Kotlin协程1.6.4版本，该功能依然处于实验性状态，未来还存在诸多变数。

反过来，如果一个`Channel`被多个生产者持有并发送数据，情况就不太一样了。先发送者的数据先被消费，但是只要有消费者继续接收，不同生产者通过相同`Channel`发送的数据，最终都会被消费掉。这听起来似乎跟之前说的“一对一”通信相互矛盾，实际上，如果仔细想想就会发现这样一件事：尽管有多个生产者通过同一个`Channel`发送数据，但是考虑到Kotlin协程结构化并发的特点，这些生产者之间终究不是绝对的并行操作；另外，`Channel`的并发安全特性也不会允许多个生产者在同一时刻对自己进行操作，必然有一个生产者能抢在其他所有生产者之前把数据先发送出去。因此，从某种角度来说，多个生产者通过同一个`Channel`发送数据，事实上可以视为一个生产者在不同时刻调用`Channel`的`send`函数。

#### `Channel`的关闭

和前文介绍的`Flow`不同，`Channel`有自己专门的关闭手段——`close`函数，也就是说通信双方都可以调用同一个`Channel`对象的`close`函数来直接关闭`Channel`，而不是像`Flow`那样只能取消`Flow`所处的协程。如下面代码所示：

```
// close函数可以传入Throwable，在发生异常需要关闭Channel的场景当中，
// 传入的Throwable可以为开发者分析排查问题带来一定的便利
channel.close()
```

当一个`Channel`被关闭的时候，它首先会**立即**停止接收新元素，如果这时候去调它的`isClosedForSend`字段，会马上返回`true`的结果；但是对于已经被发送出去的数据，如果消费者那一端还没有处理完，直接调`isClosedForReceive`字段可能会返回`false`，直到消费者处理完所有已发送数据之后，`isClosedForReceive`字段才会返回`true`。

产生这种差异的根本原因在于`Channel`内部存在**缓冲区**，更为详细的内容会在“`Channel`的背压问题”一节进行讨论。虽然创建`Channel`之后没有关闭的做法并不会造成系统资源的泄漏，但依然强烈建议在适当的时机关闭掉`Channel`，具体原因会在后面解释。

最后要讨论的一个问题是：谁来关闭`Channel`？在单向通信的情况下，显然应该由生产者主动关闭；而在双向通信的情况下，尽管通信双方在技术上是对等的，但是要从业务场景中去考虑谁是主导的一方。从实践来看，通常是由主导通信的一方去主动关闭`Channel`比较合适。

### Channel的背压问题

只要开发者在程序设计当中使用到了生产者-消费者模型，毫无疑问，就有机会遇到背压问题。`Channel`也是这样的，当生产者发送数据的速率超过了消费者接收处理的速率时就会不可避免地出现数据积压。前面已经提到过，`Channel`是一个并发安全的队列，当队列空间不足时，再往里面添加元素就会出现两种情况：要么阻塞，等待队列空出空间；要么直接拒绝添加元素。

既然已经知道了`Channel`采用的是队列的实现方式，那么就可以确定它一定存在缓冲区。这个缓冲区的大小可以通过协程库内置的`Channel`工厂函数设置：

```
// 默认的缓冲区容量为0，即默认传入Channel.RENDEZVOUS
val rendezvousChannel = Channel<T>()

// 无限容量的缓冲区传入Channel.UNLIMITED，实际大小为Int类型最大值
val unlimitedChannel = Channel<T>(Channel.UNLIMITED)

// 只保留最后一个元素的缓冲区传入Channel.CONFLATED
val conflatedChannel = Channel<T>(Channel.CONFLATED)

// 自定义缓冲区容量
val customCapacity = ···
val customChannel = Channel<T>(customCapacity)
```

上面示例代码所展示的就是几种最常用的`Channel`缓冲区容量配置。对于`Channel.RENDEZVOUS`，它表示的含义是只要没有消费者接收处理数据，生产者调用的`send`函数就会挂起等待；对于`Channel.UNLIMITED`，就表示`Channel`来者不拒，无论有没有消费者接收处理这些数据。

而对于`Channel.CONFLATED`来说，如果没有消费者接收处理，那么`Channel`就表现得跟配置了`Channel.UNLIMITED`一样；如果有消费者但是处理数据太慢，那么缓冲区里只会保留生产者发送的最后一个数据，其余的全部舍弃掉。

假如开发者为`Channel`配置了自定义缓冲区容量，那么`Channel`的表现是这样的：生产者发送数据，只要没填满缓冲区引起队列阻塞就会持续发送；一旦队列出现阻塞，`Channel`的`send`函数就会挂起等待。

刚刚讨论的是只有生产者生产数据而没有消费者消费数据的情形，如果倒过来，也就是只有消费者在尝试接收处理数据，而生产者没有数据可发送，会发生什么事情？答案也很简单，那就是`receive`函数直接挂起等待。

到这里已经基本上弄清楚一个事实：`Channel`在遭遇背压时所采取的流量控制的手段，就是让`send`或`receive`这两个挂起函数挂起等待。同时这也可以解答前文所提出的一个问题：即便不关闭`Channel`也不会引起系统资源泄漏，为什么还是要选择关闭？因为不关闭`Channel`就会导致接收端一直处于挂起等待的状态，很可能会影响到其他业务逻辑的执行。

## Kotlin协程的多路复用

Kotlin协程的多路复用概念，实际上来源于UNIX的IO多路复用。所谓IO多路复用，是指用户通过调用系统内核提供的select等多路复用API，在一个线程或进程内同时处理多个IO请求，从而提高资源利用率。Kotlin协程也提供有同样名为`select`的函数，来实现多路复用的操作。除了提供`select`函数之外，Kotlin协程库还提供了`SelectClause0`、`SelectClause1`以及`SelectClause2`这三种类型的回调。

+ `SelectClause0`：无返回值回调；
+ `SelectClause1`：有一个返回值的回调；
+ `SelectClause2`：有两个返回值的回调。

三种回调通常在`select`函数当中使用，并且常以`onXXX`的名称出现，其中`XXX`部分往往代表着Kotlin协程中的某些挂起函数。换句话说，如果一个挂起函数`XYZ`支持select多路复用，那么它一定有对应的SelectClause回调`onXYZ`，并且两边的返回值类型和个数都保持一致。

Kotlin协程基于`select`函数的多路复用，其最主要的作用就是同时监听多个协程任务，并返回最先获取到的结果。一方面提高了获取结果的速度，另一方面监听多个协程任务的返回值增加了获取结果的渠道数量，从而提高结果的可靠性。

### async-await的多路复用

`async-await`是Kotlin协程创建并行任务的常用方式，对其进行多路复用的基本操作，可以参考下面的示例代码：

```
// 在父协程中执行多路复用
launch {
    val deferred1 = async { ··· }
    ···
    val deferredN = async { ··· }

    // 在这里调用select函数进行多路复用操作，哪个协程任务先返回结果就用哪个
    val result = select<T> {
        deferred1.onAwait { ··· }
        ···
        deferredN.onAwait { ··· }
    }
}
```

### Channel的多路复用

对于多个`Channel`的复用，可以参考下面的示例代码：

```
// 模拟程序中有多个Channel存在
val channels = List(···) { Channel<T>() }

launch {
    // 模拟随机某个Channel发送数据
    while (true) {
        delay(1000)
        channels[Random.nextInt(10)].send(···)
    }
}

val result = select {
    // 模拟查找到最先发送数据的Channel，并接收处理其数据
    channels.forEach { c ->
        // 如果Channel被关闭，调用onReceive会抛出异常
        c.onReceive { it }
        // 如果不想让select抛出异常，就用onReceiveCatching，并做好判空
        c.onReceiveCatching { it.getOrNull() }
    }
}
```

### Flow的多路复用

`Flow`的多路复用操作跟前面所说的两种不太一样，不是使用`select`函数，而是使用一个中间操作符`merge`。鉴于`Channel`可以转换成`Flow`，这里继续沿用前面的示例代码进行部分改造：

```
// 模拟程序中有多个Channel存在
val channels = List(···) { Channel<T>() }

launch {
    // 模拟随机某个Channel发送数据
    while (true) {
        delay(1000)
        channels[Random.nextInt(10)].send(···)
    }
}

channels.map {
    // 将Channel诸逐个转换成Flow
    it.consumeAsFlow()
}
.merge() // 将若干个Flow整合成一个Flow进行处理
.collectIndexed { index, value ->
    ···
}
```

可以看到，使用`merge`操作符整合成一体的`Flow`，在进行多路复用操作上比使用`select`更为简洁优雅。如果遇到调用多个`Flow`但是只取其中一个结果的应用场景，采用`merge`进行多路复用无疑是比较好的选择。

## 并发安全

在JVM平台上，Kotlin协程是线程的一个良好封装方案，这在之前的内容中已经多次强调过。既然Kotlin协程的底层实现是线程，那么就不可避免地会遇到并发安全问题。说到并发安全，前面介绍的`Channel`就是一个典型的并发安全的数据结构，可以在不同协程间安全使用。而对于不使用`Channel`的场景而言，则需要采取另外的方式来确保并发安全。目前常用的保证Kotlin协程并发安全的操作有几种：1）避免协程间共享变量；2）使用原子类型封装再调用；3）使用协程框架提供的其他并发安全工具。这里主要简单介绍第三种操作。

Kotlin协程框架提供了`Mutex`锁和`Semaphore`信号量这两种并发安全工具。

`Mutex`是一种轻量级锁，它在使用上跟线程锁类似，但是由于用在Kotlin协程上，当它获取不到锁的时候只是挂起等待锁的释放，而不会像传统线程锁那样阻塞线程。下面的示例代码展示了`Mutex`的基本使用方式：

```
// 创建一个Mutex对象
val mutex = Mutex()

// 在协程中为原本不安全的并发操作加锁
val job1 = launch {
    mutex.withLock {
        ···
    }
}

···

val jobN = launch {
    mutex.withLock {
        ···
    }
}

```

`Semaphore`是一种轻量级的信号量，它可以有多个，协程在获取到信号量后即可执行并发操作。下面的示例代码展示里`Semaphore`的基本使用方式：

```
// 创建一个Semaphore对象
val semaphore = Semaphore(···)

// 在协程中为原本不安全的并发操作加锁
val job1 = launch {
    semaphore.withPermit {
        ···
    }
}

···

val jobN = launch {
    semaphore.withPermit {
        ···
    }
}
```

`Semaphore`有两个入参，一个名为permit，用于配置许可个数；另一个名为acquiredPermits，表示已经被分配使用的许可个数，不能超过permit的值，否则运行时会抛出异常。当permit减去acquiredPermits的值大于0时，`Semaphore`的效果等价于使用`Mutex`；若permit与acquiredPermits相等，被`Semaphore.withPermit{}`包裹起来的操作便不会执行，相当于获取不到`Mutex`锁只能挂起等待。