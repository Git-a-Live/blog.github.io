## 何为ViewModel

`ViewModel`是Google推出的Android Jepack组件当中对MVVM模式的具体实现方案。按照Google官方的说法，`ViewModel`类旨在以注重生命周期的方式存储和管理界面相关数据，并且可以让这些数据在发生屏幕旋转等配置更改的情形后继续留存。`ViewModel`类的生命周期如下图所示：

![](pics/viewmodel1.png)

可以看到，`ViewModel`的生存周期覆盖了Activity/Fragment的大部分生命周期，这就是为什么当Activity因旋转等情形发生重建时，数据还能继续留存的重要原因。`ViewModel`会在页面彻底销毁（也就是其作用域`ViewModelStoreOwner`永久消失）的时候被自动销毁，所以并不需要开发者额外增加手动解除持有的业务逻辑，也能避免发生内存泄漏问题。

> `ViewModel`类作为MVVM在Android开发领域的具体实现方案，自然要符合MVVM的要求，最关键的一点就是不能像Presenter那样持有View层，否则会引发内存泄漏。

注意，如果下文不做特殊说明，ViewModel默认表示MVVM中的ViewModel层概念，而`ViewModel`或者`ViewModel`类用于描述Google Jetpack组件库中MVVM的ViewModel具体实现。

### ViewModel的恢复

前面的内容已经提到，`ViewModel`的生命周期可以一直持续到Activity/Fragment被彻底销毁为止，一个有意思的问题出现了：为什么在页面旋转这种情形下，Activity/Fragment已经执行过`onDestroy()`了，`ViewModel`还能继续保留？

这个问题的答案是：页面旋转属于**配置更改**进行销毁重建，而非结束页面。在这种情况下，Activity/Fragment的行为会发生很大改变，虽然还是正常走销毁重建的生命周期，但是中间已经动了不少手脚。

在屏幕发生旋转前销毁Activity时，`ComponentActivity`调用`onRetainNonConfigurationInstance()`方法，将要销毁的Activity的`mViewModelStore`转化为`NonConfigurationInstances`对象；接着调用Activity的`retainNonConfigurationInstances()`方法，最终在`ActivityThread`的`performDestroyActivity()`方法里面，将数据保存到`ActivityClientRecord`当中。

在Activity重建并启动时，`ActivityThread`调用`performLaunchActivity()`方法，将存储在`ActivityClientRecord`中的`lastNonConfigurationInstances`通过Activity的`attach()`方法传递到对应的Activity中；然后通过`getLastNonConfigurationInstance()`恢复`mViewModelStore`实例对象，最后根据对应的key拿到销毁前对应的`ViewModel`实例。

`ViewModel`的`onCleared()`方法，并不是每次Activity执行`onDestroy()`的时候都一定会触发。因为`ViewModel`实例保存在`ViewModelStore`里面，而只有`ViewModelStore`调用`clear()`之后，才会真正触发`ViewModel`的`onCleared()`。`ViewModelStore`调用`clear()`的时候，还会先判断本次销毁是否属于配置变更，这就是为什么`ViewModel`不一定会执行`onCleared()`的根本原因，如下列源码所示：

```
// 在ComponentActivity的构造方法中执行清理逻辑
getLifecycle().addObserver(new LifecycleEventObserver() {
    @Override
    public void onStateChanged(@NonNull LifecycleOwner source,
            @NonNull Lifecycle.Event event) {
        if (event == Lifecycle.Event.ON_DESTROY) {
            // Clear out the available context
            mContextAwareHelper.clearAvailableContext();
            // And clear the ViewModelStore
            if (!isChangingConfigurations()) {
                // 判断是否因为配置变化而发生销毁重建，如果是的话就不执行清理操作，重建后可以正常恢复ViewModel实例
                getViewModelStore().clear();
            }
        }
    }
});
```

## ViewModel的基本使用

### 继承抽象类

Google通过`androidx.lifecycle`为开发者提供了`ViewModel`的抽象类，用于继承和重写函数——事实上，继承`ViewModel`抽象类唯一需要重写的函数只有一个`onCleared()`。这个函数会在`ViewModel`被销毁时调用，因此继承`ViewModel`的子类如果要统一释放资源，通常会在`onCleared()`当中执行。

`ViewModel`的继承比较简单，类似于下列代码所示：

```
class MyViewModel: ViewModel() {
    
    // 重写onCleared()函数
    override fun onCleared() {
        super.onCleared()
        ···
    }

    // 其他业务逻辑
    fun foo() {
        ···
    }
    ···
}
```

### 工厂模式实例化

`ViewModel`的实例化比较特殊，不是像普通类一样直接通过构造器去创建，而是要通过工厂模式创建一个单例出来才能使用。这种设计的考量在于，通过特定的工厂方法调用，可以将`ViewModel`实现类与生命周期组件（Google将其称作LifecycleOwner）的生命周期绑定在一起，起到监听生命周期从而自动执行销毁步骤的作用。如果通过构造器直接创建一个实例出来，那么LifecycleOwner顶多就是持有`ViewModel`实现类的引用，除此之外没有任何联系，更不用说让`ViewModel`实现类去监听这些组件的生命周期了。在这种情况下，ViewModel就退化成了一个不持有View层引用的Presenter，其优势也无法发挥，自然也就失去了使用MVVM架构的意义。

+ **无构造参数的`ViewModel`实例创建**

无构造参数的`ViewModel`通过工厂模式创建实例的流程非常简单，类似于下列代码所示：

```
// Activity的lifecycleOwner传入this即可
val myViewModel: MyViewModel = ViewModelProvider(this)[MyViewModel::class.java]

// 归属于某个Activity的Fragment，lifecycleOwner通常传入requireActivity()，否则拿不到同一个实例
val myViewModel = ViewModelProvider(requireActivity())[MyViewModel::class.java]
```

当然，在添加AndroidX的`activity-ktx`和`fragment-ktx`依赖库之后，无参构造器的`ViewModel`实例还可以通过更为简单的方式创建：

```
dependencies {
    // 注意从1.6.0版本开始，activity-ktx要求targetSDK和compileSDK的最低版本达到33
    implementation 'androidx.activity:activity-ktx:$specified_version'
    // Fragment可以通过这个依赖库调用viewModels代理方法
    implementation 'androidx.fragment:fragment-ktx:$specified_version'
}
```

```
class MainActivity : AppCompatActivity() {
    // 采用委托代理的方式直接创建实例，不需要修饰为延迟初始化
    private val myViewModel by viewModels<MyViewModel>()
    ···
}

class MyFragment : Fragment() {
    // Fragment虽然也可以采用委托代理的方式创建ViewModel实例，但是获取到的实例跟上层Activity创建的不是同一个
    private val myViewModel by viewModels<MyViewModel>()
    ···
}
```

+ **有构造参数的`ViewModel`实例创建**

这里新建一个带有构造器参数的`ViewModel`实现类：

```
// 注意ViewModel的构造器参数不能包含Activity/Fragment的上下文
class MyViewModel(private val repository: Repository): ViewModel() {
    ···
}
```

构建一个实现了`ViewModelProvider.Factory`的自定义工厂类：

```
class MyFactory(private val repository: Repository): ViewModelProvider.Factory {
    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        // 判断传进来的类型是否属于ViewModel的目标实现类
        if (modelClass.isAssignableFrom(MyViewModel::class.java)) {
            return MyViewModel(repository) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}
```

在Activity/Fragment中通过普通方式创建带构造参数的`ViewModel`实例：

```
// 创建工厂实例
val factory = MyFactory(Repository())

// 将工厂实例传入ViewModelProvider中，即可创建出带构造参数的ViewModel实例
val myViewModel = ViewModelProvider(lifecycleOwner, factory)[MyViewModel::class.java]
```

通过委托代理方式创建带构造参数的`ViewModel`实例：

```
// 创建工厂实例
val factory = MyFactory(Repository())

// 通过委托代理创建实例
val myViewModel by viewModels<MyViewModel> { factory }
```

### AndroidViewModel

`AndroidViewModel`是`ViewModel`的一个子类，带有一个`Application`类型的构造参数。通过这个参数，`AndroidViewModel`可以在不持有View层上下文的情况下全局访问相关资源。

这里新建一个继承`AndroidViewModel`的实现类：

```
class MyAndroidViewModel(application: Application): AndroidViewModel(application) {
    ···
}
```

`AndroidViewModel`作为一个带构造参数的`ViewModel`类，在创建实例的时候自然要传入工厂实例，不过Google已经提供了现成的。下列代码展示的是普通方式如何创建`AndroidViewModel`实例：

```
val androidViewModel = ViewModelProvider(lifecycleOwner, ViewModelProvider.AndroidViewModelFactory(application))[MyAndroidViewModel::class.java]
```

通过委托代理方式创建`AndroidViewModel`实例：

```
val androidViewModel by viewModels<MyAndroidViewModel> { ViewModelProvider.AndroidViewModelFactory(application) }
```

> 注意，在Fragment中使用委托代理创建`AndroidViewModel`实例时，传入的`Application`参数通常是`requireActivity().application`。<font color=red>尽管如此，这样创建出来的实例跟上层Activity的依然不是同一个</font>，因为lifecycleOwner默认就是不同的。