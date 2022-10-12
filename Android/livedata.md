## 何为LiveData

按照Google官方的描述，LiveData“是一种具有生命周期感知能力、可观察的数据存储器类”。这个描述体现了LiveData最突出的几个特性：

+ **生命周期感知能力**

简单来说就是可以监听LifecycleOwner（通常是Activity/Fragment）的生命周期。当一个观察器被添加到LifecycleOwner之后，LiveData就会自动与LifecycleOwner关联，并且只有在LifecycleOwner处于活跃的生命周期才会更新观察器；一旦LifecycleOwner被销毁，LiveData和观察器也会自动销毁，不会引发内存泄漏。
+ **可观察**

表示LiveData对象存储的数据发生更改时，观察器会收到通知，View层可以根据观察器接收到的通知做出特定响应。前面曾提到过MVVM是数据驱动（Data-Driven）的架构，数据改变就会触发通知等一系列连锁动作。

+ **数据存储**

LiveData是一种可存储任何类型数据的封装容器，无论是基本类型还是引用类型，都可以被LiveData存储。

> LiveData通常跟`ViewModel`搭配使用，但是随着Kotlin协程Flow的出现，LiveData在使用Kotlin开发的Android项目中会被逐渐取代，而使用Java开发的Android项目还是可以继续保留。此外，`ViewModel`不能观察LiveData，否则会带来使用问题。

## LiveData的基本使用

Google官方提供了`LiveData`和`MutableLiveData`这两个泛型类（`MutableLiveData`继承于`LiveData`），其中前者只能读取数据不能修改数据，而后者跟其他名字中带有Mutable的实现类一样，里面的数据可读可写。按照Google的设计，`LiveData`类型会作为被观察的对象提供到外部使用，防止程序在`ViewModel`以外的地方修改`Livedata`的数据；而`MutableLiveData`则只局限在`ViewModel`内部读写，不会暴露给外部使用。如下列代码所示：

```
class MyViewModel: ViewModel() {
    // ViewModel内部使用MutableLiveData进行读写
    private val mutableLiveData = MutableLiveData<T>(···)

    // ViewModel对外提供LiveData用于观察
    val liveData: LiveData<T> get() = mutableLiveData

    // 修改MutableLiveData的业务逻辑
    fun foo() {
        // MutableLivedata在同步条件下（主线程之内）被赋值
        mutableLiveData.value = ···
        // MutableLivedata在异步条件下（主线程之外）被赋值
        mutableLiveData.postValue(···)
    }
}
```

在上面的代码中还注意到，`MutableLiveData`的数据修改不是直接赋值给mutableLiveData，而是赋值给它的属性值mutableLiveData.value。这就是`LiveData`作为“封装容器”的含义，类似于`List`等泛型容器一样，容器自身通常是不需要被修改的，只有容器里面的元素需要修改。所以通常情况下作为数据容器的mutableLiveData和liveData使用`val`进行修饰，表明它们是不可被重新赋值的。`value`这一属性值才是真正要被传递和使用的数据。

在LifecycleOwner中，`LiveData`要通过以下方式才能实现关联和被观察：

```
// 在Activity和Fragment中，observe()要传入的LifecycleOwner通常就是this
myViewModel.liveData.observe(this) { it: T ->
    // 这里编写观察到数据变化之后的动作，it的类型就是livedata.value的类型
}
```

`observe()`方法需要传两个参数，除了`LifecycleOwner`类型参数之外还有一个`Observer`回调（在上面代码中已经优化成lambda表达式），这就是之前提到的“观察器”。`Observer`是一个泛型接口，它会传递`LiveData`的`value`属性，供观察者使用。当数据发生变化的时候，`Observer`就会被调用，观察`LiveData`的LifecycleOwner也就自动接收到更新后的数据。跟 Presenter相比，ViewModel一方面不用持有View层接口，另一方面只要ViewModel在内部修改`MutableLiveData`，观察`LiveData`的View层就会自动响应。

到目前为止，MVVM在Android开发上的实现方案还是比较令人满意的。但是如果仔细观察就会发现，当`ViewModel`包含有太多`MutableLiveData`时，对应的`Livedata`也会非常多，这就给`LiveData`的使用带来比较大的麻烦，也促使了后面将要谈到的MVI架构的产生。除此之外，`LiveData`在设计上还存在一些缺陷（比如[这篇博客](https://juejin.cn/post/6844903846624362510)所提到的问题），在极为复杂的使用场景下可能会带来一些意想不到的问题，因此Google官方推荐使用Flow来替换`LiveData`。尽管如此，对于简单的业务而言，`LiveData`的使用也比[RxJava](Android/rxjava)要轻量得多，特别是在那些无法使用Kotlin Flow的纯Java开发的Android项目中，它还有用武之地。

### MediatorLiveData

`MediatorLiveData`顾名思义，就是发挥中介作用的一种`LivedData`。`MediatorLiveData`用于监听其他`LiveData`的数据变化，这些`LiveData`对于`MediatorLiveData`来说就是数据源。任意一个数据源的变化都会被`MediatorLiveData`监听到，然后触发通知，执行后续一系列操作。如果应用中存在多个数据源变化触发相同动作的场景，`MediatorLiveData`无疑是非常适合使用的。

`MediatorLiveData`继承于`MutableLiveData`，所以同样可读可写。它的基本使用方式如下：

```
class MyViewModel: ViewModel() {
    private val mutableLiveData = MutableLiveData<T>(···)
    private val mutableLiveData2 = MutableLiveData<T>(···)
    ···

    private val mediatorLiveData = MediatorLiveData<T>()
    val liveData: LiveData<T> get() = mediatorLiveData

    // MediatorLiveData监听其他数据源的初始化操作，通常放到ViewModel的init里面
    init {
        mediatorLiveData.apply {
            addSource(mutableLiveData) { it: T ->
                value = it
            }
            addSource(mutableLiveData2) { it: T ->
                value = it
            }
            ···
        }
    }
}
```

LifecycleOwner就像观察普通的`LiveData`那样观察`MediatorLiveData`即可。`MediatorLiveData`适合用于这样的场景：有多个**不同用途**的数据源（相同用途的数据源只要一个就够了），需要组合成一个统一的展示形式（所以`MediatorLiveData`的数据类型不一定跟数据源相同），并且每个数据源的变化都需要被监听和响应。从这个角度来说，`MediatorLiveData`更像是一个被复用了的消息通道，不同数据源的数据变化都要传递给`MediatorLiveData`，而LifecycleOwner观察`MediatorLiveData`，可能还要根据不同的数据内容进行业务逻辑上的“分用”。

> 除了`MediatorLiveData`之外，Google其实还有`CoroutineLiveData`以及`SavingStateLiveData`这两种`LiveData`，但是实际开发中基本不会用到。原因是这两种`LiveData`属于内部类，开发者通常调用不到它们。