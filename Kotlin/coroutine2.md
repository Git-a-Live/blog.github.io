## 协程基本概念

### 作用域

作用域（scope）是指**变量和函数执行代码可被访问和生效的区域范围**。例如在一个类中，函数以外定义的属性，其作用域为整个类；而函数以内定义的成员变量，其作用域仅为该函数。Java里面的使用`public static`修饰的类属性和方法，其作用域即为整个项目。

+ **CoroutineScope**

CoroutineScope是Kotlin官方提供的作用域<font color=red>接口</font>，它声明了一个名为`coroutineContext`（协程上下文）的属性。在此基础上，Kotlin又设计了`launch`、`async`以及`runBlocking`等扩展函数，作为协程构建器。注意，这些构建器里面执行的代码也被称作“协程”，因此从这里开始，协程实际上既可以指一种并发设计模型，也可以指一系列执行异步任务的代码，在后续的内容中需要联系上下文加以辨别。

常用构建器如下：

`launch{}`：用于创建一个不阻塞当前线程的协程，并返回一个Job对象用于管理该协程实例；

`async{}`：和`launch`类似，但是返回一个Deferred对象，里面包含一个异步代码块返回的结果；

`runBlocking{}`：创建一个可以阻塞当前线程的协程，官方**强烈建议不要在正式开发中使用该构建器**。

+ **GlobalScope**

GlobalScope是一个由Kotlin官方实现的最基础的<font color=red>自定义</font>作用域。它可以创建启动**顶级**协程，这些协程在整个应用程序生命周期内运行，不会被过早地被取消。

然而，GlobalScope通常也不与任何生命周期组件绑定，除非手动管理，否则很难满足实际开发中的需求。因此，Android项目的程序代码应该使用自定义的协程作用域，<font color=red>强烈不建议直接使用GlobalScope</font>。

+ **MainScope**

MainScope是Android领域扩展出的自定义作用域（下面要谈到的lifecycleScope和viewModelScope也是一样），它用于在主线程中创建一个协程，但是需要注意在组件的生命周期结束时手动调用`cancel()`函数取消协程，以防内存泄漏。

+ **lifecycleScope**

LifecycleCoroutineScope是一个抽象类，通常在Android中调用的是lifecycleScope这一实例对象，进而调用其内部的抽象方法。

lifecycleScope可以通过`launchWhenCreated{}`、`launchWhenStarted{}`以及`launchWhenResumed{}`等构建器，在合适的声明周期创建并启动协程。

+ **viewModelScope**

在采用了MVVM架构的项目当中，ViewModel内部无法通过lifecycleScope来创建协程，因而Android又为ViewModel提供了一个viewModelScope对象，用于创建管理只在ViewModel生命周期内运行的协程。

Kotlin协程的作用域是一个及其重要的概念，只有对它有足够了解，才能更好地实现结构化并发。

### 挂起

目前任何的协程代码块，要么写在协程作用域里，要么写在挂起函数当中，绝无例外。同样地，挂起函数要么也是运行在协程作用域里，要么也是运行在另一个挂起函数中。之前已经对作用域进行了初步了解，下面要来谈一谈挂起函数。

挂起函数的“挂起”（suspend）是Kotlin协程中最为核心的概念。它的含义是：

1. **使协程从正在执行该协程的线程上脱离出来；**
2. **切换到挂起函数指定的线程中继续执行；**
3. **在协程执行完毕之后自动切回1里面的原线程（Kotlin中称之为resume）。**

>注意：如果挂起函数当中没有指定要切换到哪个线程，那么协程就继续在当前线程上运行。

简而言之，挂起操作的本质就是<font color=red>线程切换</font>。这就是为什么说挂起是Kotlin协程这一线程框架当中最核心的概念，因为任何线程框架都必须处理线程间切换的问题，而这个问题也是最为关键的部分之一。

Kotlin协程的`suspend`关键字实际上并不起到直接的挂起作用，而是**用于提示被修饰函数自身有耗时操作，需要以协程方式调用**。因为真正执行这一操作的是Kotlin自带的一系列挂起函数，而这些挂起函数要求自身必须位于协程作用域或另一个挂起函数中。

>注意，如果一个函数既没有包含任何跟协程相关的代码，也没有在协程作用域或其他挂起函数中调用，在使用`suspend`关键字修饰时会被IDE提示要求删除。

### 阻塞与非阻塞

Kotlin协程的挂起是“非阻塞式”的，从本质上来说，就是指代码采用同步的（看起来会阻塞线程的）写法，但实际上不会阻塞线程。原因很简单，线程自身当然不可能被已经脱离出去、运行在其他线程的协程所阻塞（亦即所谓的“死道友不死贫道”）。即使不用协程，仅仅使用Java最底层的Thread进行线程切换，也同样能实现非阻塞式。从这个角度来讲，Kotlin协程的异步并不比Java底层Thread的异步更高级。

最后要强调的是，任何耗时操作终归会有一个线程来承载，而承载耗时操作的线程自然是处于阻塞状态的，无非是处理耗时操作有时间长短，阻塞也就有久和不久的差异。

## 协程基本用法

### 前置工作

在Android项目中使用协程，首先要通过Gradle导入相关依赖：

```
// 基本使用
implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:$specific_version"

// 在ViewModel中使用viewModelScope
implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$specific_version"

// 使Room支持协程
implementation "androidx.room:room-ktx:$specific_version"

// 在生命周期组件中使用lifecycleScope
implementation "androidx.lifecycle:lifecycle-runtime-ktx:$specific_version"
```

### 开启协程

+ **GlobalScope**

要在程序中开启一个协程，最为简单的方式如下：

```
// 开启全局顶级协程
GlobalScope.launch { 
    // TODO       
}

// 取消全局顶级协程的一种方式
GlobalScope.cancel()

// 取消全局顶级协程的另一种方式
GlobalScope.cancel("canceled by developer","exceptions")
```

在`launch{}`当中编写需要异步执行的代码，然后编译运行即可。

但是前面已经提到过，GlobalScope由于不绑定任何生命周期组件，手动管理相对麻烦，因此通常并不建议在正式的Android项目中使用。如果想要在正式的Android项目中使用协程，一般会选用MainScope、lifecycleScope以及viewModelScope这三个作用域来开启协程并进行管理。

此外，GlobalScope开启的协程类似于[守护线程](#守护线程)，只要虚拟机结束运行，无论里面的协程代码是不是一个一直在执行的死循环，都会跟着被取消。

+ **MainScope**

MainScope在主线程中开启和取消协程的基本方式如下：

```
// MainScope实例化
val mainScope = MainScope()

// 开启MainScope协程
mainScope.launch {
    // TODO
}

// 取消MainScope协程的一种方式
mainScope.cancel("Canceled by developer","exceptions")

// 取消MainScope协程的另一种方式
mainScope.cancel()
```

>注意，必须对同一个MainScope对象进行开启和取消协程的操作，否则达不到管理协程的目的，还会引发内存泄漏问题。

+ **lifecycleScope**

lifecycleScope主要是在具有生命周期的组件（比如Activity和Fragment等）当中开启协程。它最主要的协程构建器在前面已经提到过，下面是简单用法：

```
// 在组件创建时开启协程
lifecycleScope.launchWhenCreated {
    // TODO
}

// 在组件启动时开启协程
lifecycleScope.launchWhenStarted {
    // TODO
}

// 在组件恢复时开启协程
lifecycleScope.launchWhenResumed {
    // TODO
}

// 以普通方式开启协程
lifecycleScope.launch {
    // TODO
}
```

lifecycleScope的存在意义，就在于调用它不需要额外编写任何取消协程的代码，只要组件的生命周期走到`onDestroy`这个阶段，所有属于lifecycleScope作用域的协程都会被**自行取消**，从根本上杜绝了开发者忘记手动取消协程引发内存泄漏的可能性。因此从这个角度来说，用lifecycleScope比MainScope更为方便和安全。

+ **viewModelScope**

viewModelScope用在MVVM架构的项目当中，特别是用在ViewModel里面。使用viewModelScope在ViewModel当中开启协程的基本方式如下：

```
viewModelScope.launch { 
    // TODO
}
```

viewModelScope和lifecycleScope一样，也不需要额外编写取消协程的代码。但是要注意，viewModelScope开启的协程跟ViewModel本身的生命周期是同步的。

### 结构化并发

每个并发操作都在处理一个任务，它可能属于某个父任务，也可能有自己的子任务。每个任务都拥有自己的生命周期，而不同长度生命周期的并发任务整合在一起时，它们之间的协调就成了一个十分重要的问题。一种典型的情况是：如何确保一个短周期父任务在嵌套一个长周期子任务时，还能确保这个子任务执行完毕，而不是跟着父任务被强行结束？另外一种典型情况是：如何确保两个有明显顺序执行关系的并发任务按照开发者的意图依次执行？

结构化并发这一概念的提出就是为了解决上述的两个问题。对于开发者而言，父任务不应该在其所有的子任务执行完毕（或任一子任务抛出异常）前就自行结束；两个需要按照顺序执行下来的并发任务也不应该完全听凭操作系统的随机调度。传统的线程对于这两个问题都没有很便捷的解决方案，而Kotlin协程则提供了相对优雅的方法。

Kotlin协程只需要用一种方式编写代码即可同时解决上述两个问题，示意如下：

```
// 在某个作用域协程中开启子协程
launch {
    // 按同步写法依次开启子协程执行任务
    launch {
        // TODO
    }

    launch {
        // TODO
    }

    // 一个挂起函数
    foo() 
    ···
    // TODO: 所有子协程执行结束之后才会执行父协程
}
```

在上面的示意当中，代码结构非常清晰，而且意图也比较明确，很好地实现了结构化并发。

### 异步任务

Kotlin协程既然在本质上是线程的一种良好封装，那么用来执行异步任务自然是理所应当的了。Kotlin协程提供的`async{}`构建器就是专门用来实现异步并发执行的。

`async{}`构建器的一个使用示例如下：

```
launch {
    val a = async {
        delay(1000L)
        (Math.random()*10).toInt()
    }

    val b = async {
        delay(1500L)
        (Math.random()*10).toInt()
    }

    Log.e("MyTag","a = ${a.await()} b = ${b.await()} c = ${a.await()+b.await()}")
}
```

上述示例代码的总执行时间通常接近1500毫秒，而非2500毫秒。这表明两个子协程的确是并发执行的，而且最后拿到的结果也是二者结果的汇总。

前面的内容已经提到过，`async{}`构建器返回的是一个Deffered对象，而Deffered是一个继承于Job的泛型接口，可以视为一个轻量级的非阻塞[Future](#future和completablefuture)。在`async{}`构建器当中，最后一行代码往往就是要返回的结果，如果不返回结果，那么通过`await()`方法拿到的结果类型就是Unit。

利用`async{}`构建器构建协程执行异步任务，<font color=red><u>在需要结果的地方</u></font>，调用`await()`方法获取异步任务产生的结果，这就是Kotlin协程基本的异步任务使用方式。

如果将示例代码改成下面的形式：

```
launch {
    val a = async {
        delay(1000L)
        (Math.random()*10).toInt()
    }.await()

    val b = async {
        delay(1500L)
        (Math.random()*10).toInt()
    }.await()

    Log.e("MyTag","a = $a b = $b c = ${a+b}")
}
```

执行后会发现所需时间接近2500毫秒，表明这两个协程变成**同步执行**的了，而且IDE会提示可以将`async{···}.await()`修改成`withContext(···){···}`形式的代码：

```
launch {
    val a = withContext(Dispatchers.Default) {
        delay(1000L)
        (Math.random() * 10).toInt()
    }

    val b = withContext(Dispatchers.Default) {
        delay(1500L)
        (Math.random() * 10).toInt()
    }

    Log.e("MyTag","a = $a b = $b c = ${a+b}")
}
```

其中`Dispatchers.Default`是一种线程调度器，后面会进行详细的介绍。`withContext(){}`是一个主要用于同步执行并等待返回任务结果的协程构建器，

在使用`async{}`构建器编写结构化并发代码时，如果任意一个子协程抛出异常，那么第一个子协程以及整个父协程都会被取消。此外，这种抛出异常和协程取消会按照父子协程的结构依次传递出去。