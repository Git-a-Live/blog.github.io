ViewModel的出现使得应用架构由以往的MVC（Model-View-Controller）或MVP（Model-View-Presenter）转变成MVVM（Model-View-ViewModel）， 实现了数据操作与界面显示等功能的解耦。ViewModel通常和Live Data以及Data Binding搭配使用，以确保数据的存放， 不至于因为Activity的某些活动（如屏幕旋转导致Activity重新创建）使得数据总是被清空得不到保留。

## ViewModel

下面以一个简单的例子来说明ViewModel的使用。创建一个带有滑动条和文本框的简单应用，让滑动条滑动时， 显示框内的数字会随之变化，同时要实现应用在屏幕旋转或者切到后台再返回的情况下，文本框能继续显示当前滑动条所指示的数字。

首先在build.gradle（Module:app）文件中的dependencies部分添加如下内容：

```
implementation 'androidx.lifecycle:lifecycle-extensions: ‹the latest version on official website›'
```

修改gradle文件之后，需要重新同步Gradle， 然后在MainActivity文件所在的目录下，编写一个继承于ViewModel的类，比如本例中的MyViewModel：

```
class MyViewModel: ViewModel(){
    var number = 0;
}
```

之后在MainActivity的onCreate方法中添加一个类型为MyViewModel的变量，并用ViewModelProvider初始化：

```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val myViewModel: MyViewModel = ViewModelProvider(this,ViewModelProvider.NewInstanceFactory())[MyViewModel::class.java]
        ···
}
```

接着编写与控件相关的内容：

```
···
textView.text = myViewModel.number.toString()
seekBar.progress = myViewModel.number
seekBar.setOnSeekBarChangeListener(object: SeekBar.OnSeekBarChangeListener{
    override fun onProgressChanged(seekBar: SeekBar, progress: Int, fromUser: Boolean) {
        myViewModel.number = progress
            textView.text = myViewModel.number.toString()
   }
    override fun onStartTrackingTouch(seekBar: SeekBar) {}
    override fun onStopTrackingTouch(seekBar: SeekBar) {}
})
```

最后编译运行，应用在屏幕旋转以及切到后台再返回的情况下，可以继续显示文本框之前的内容。但是如果退出应用再返回，就会发现文本框已经被清空了，滑动条也被初始化为原来的状态。

利用Bundle和SavedState只能起到数据临时保存和复现的作用， 一旦发生应用闪退、手机关机等强制杀死应用的情况，保存在内存中的数据也会随之丢失，上述两种临时方法根本无法应对。 要彻底解决这个问题，就得等引入数据持久化技术，如SharedPreferences或是SQLite数据库等才行。

## AndroidViewModel

AndroidViewModel是ViewModel的一个子类，但是它和ViewModel相比有一个优点，即可以通过构造函数访问全局资源， 因此在配合LiveData，DataBinding以及SharedPreferences等基础上，能够发挥极大的作用。值得注意的是， SharedPreferences对于AndroidViewModel来说也是必需的，正如ViewModel必须搭配LiveData和DataBinding一样。

本次实践将构建一个同时使用AndroidViewModel和SharedPreferences的例子，该实例的界面和功能都很简单， 应用界面只包含一个文本框和按键，当用户按下按键时，文本框内的数字就会+1，同时在用户翻转屏幕以及退出应用重新进入的情况下， 显示的内容依然能够得到保留。

由于AndroidViewModel继承于ViewModel，因此添加的依赖和ViewModel完全一致； 而DataBinding的启用在[之前](Android/db.md)也有叙述，在此不再多做说明。

通过Android Studio新建一个项目，在MainActivity文件的同一目录下新建一个继承于AndroidViewModel的类文件MyAndroidViewModel：

```
class MyAndroidViewModel(application: Application): AndroidViewModel(application) {
    //TODO
}
```

注意，在使用AndroidViewModel的时候，IDE会提示添加一个构造函数以获取Application对象，从而可以访问全局资源：

```
//Activity
val myViewModel = ViewModelProvider(this, ViewModelProvider.AndroidViewModelFactory.getInstance(application))[MyAndroidViewModel::class.java]

//Fragment
val myViewModel = ViewModelProvider(requireActivity(), ViewModelProvider.AndroidViewModelFactory.getInstance(requireActivity().application))[MyAndroidViewModel::class.java]
```

>注意：尽管AndroidViewModel自带构造器，但是**强烈不建议**直接通过构造器的方式去创建AndroidViewModel实例。<font color=red>因为直接通过构造器创建实例对象，起不到绑定组件生命周期的作用，这样使用ViewModel和LiveData的意义就没有了。</font>

接着继续完成MyAndroidViewModel文件的编写，通过application对象调用get()方法，可以直接在MainActivity以外创建SharedPreferences对象：

```
private val shp = application.getSharedPreferences("KEY",MODE_PRIVATE)
private val editor = shp.edit()
private var number = MutableLiveData(0)

fun getNumber():MutableLiveData<Int>{
    number.value = shp.getInt("KEY",0)
    return number
}

fun add(){
    number.value = number.value!!.plus(1)
    editor.putInt("KEY", number.value!!)
    editor.apply()
}
```

DataBinding对象的创建以及在布局文件中操作和之前完全一致，不再重复。完成上述工作之后，编译运行，通常可以成功。 至此，采用AndroidViewModel、Data Binding以及SharedPreferences所开发的应用，已经比较好地实现了界面与代码解耦， 还有数据的简单保存和复现。