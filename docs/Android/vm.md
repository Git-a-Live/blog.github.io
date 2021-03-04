ViewModel的出现使得应用架构由以往的MVC（Model-View-Controller）转变成MVVM（Model-View-ViewModel）， 实现了数据操作与界面显示等功能的解耦。ViewModel通常和Live Data以及Data Binding搭配使用，以确保数据的存放， 不至于因为Activity的某些活动（如屏幕旋转导致Activity重新创建）使得数据总是被清空得不到保留。

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