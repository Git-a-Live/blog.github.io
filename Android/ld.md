LiveData是一个可观察的数据持有者类。与常规的可观察性不同，LiveData是生命周期感知的，意味着它遵循其他应用程序组件（例如Activity，Fragment或Service）的生命周期。这种意识确保LiveData只更新处于活动生命周期状态的应用程序组件观察者。

使用LiveData具有以下优点：

+ **保证用户界面与数据状态匹配**

LiveData遵循观察者模式。在生命周期状态更改时LiveData会通知Observer对象。您可以升级代码在observer对象中更新UI，不必每次数据改变时手动更新UI，观察者可以在每次更改时更新UI。

+ **没有内存泄漏**

观察者与Lifecycle对象绑定，并在具有生命周期的对象被destroyed后自行清理。

+ **停止activities不会导致崩溃**

如果观察者的生命周期处于非活动状态，例如在后退堆栈中的活动，则不会收到任何LiveData事件。

+ **无需手动处理生命周期**
  
UI组件只是观察相关数据，不会停止或恢复观察。LiveData自动管理所有这些操作，因为在观察时它实时响应相关的生命周期状态的变化。

+ **始终保持最新的数据**

如果生命周期变为非活动状态，它将在再次变为活动状态时接收到最新的数据。例如，后台Activity在返回到前台后立即收到最新数据。

+ **正确的配置更改**

如果由于配置更改（如设备旋转）而重新创建Activity或Fragment，则会立即收到最新的可用数据。

+ **共享资源**

可以使用单例模式来扩展LiveData对象来包装系统服务，以便它们可以在应用程序中共享。LiveData对象一旦连接到系统服务，然后其他任何需要系统资源的观察者只需要观看该LiveData 对象就可以。

这里继续使用[View Model](Android/vm)的例子。首先对MyViewModel中的变量number进行修改， 将其类型改为MutableLiveData\<Int>，并且用private修饰以避免其被随意修改， 之后再编写一个get函数来获取number的值。

```
//Before:
var number: Int = 0;

//After:
private var number: MutableLiveData<Int> = MutableLiveData(0);
fun getNumber(): MutableLiveData<Int>{
    return number
}
```

接着在MainActivity中修改如下两行，为Live Data添加一个Observer：

```
//Before:
textView.text = myViewModel.number.toString()
seekBar.progress = myViewModel.number

//After:
myViewModel.getNumber().observe(this, Observer {
textView.text = myViewModel.getNumber().value.toString()
    seekBar.progress = myViewModel.getNumber().value!!
})
```
还有seekBar里面的两处：

```
//Before:
myViewModel.number = progress
textView.text = myViewModel.number.toString()

//After:
myViewModel.getNumber().value = progress
textView.text = myViewModel.getNumber().value.toString()
```

最后编译运行，通常可以实现和[View Model](Android/vm)例子相同的效果。