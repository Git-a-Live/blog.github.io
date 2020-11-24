AndroidViewModel是ViewModel的一个子类，但是它和ViewModel相比有一个优点，即可以通过构造函数访问全局资源， 因此在配合LiveData，DataBinding以及SharedPreferences等基础上，能够发挥极大的作用。值得注意的是， SharedPreferences对于AndroidViewModel来说也是必需的，正如ViewModel必须搭配LiveData和DataBinding一样。

本次实践将构建一个同时使用AndroidViewModel和SharedPreferences的例子，该实例的界面和功能都很简单， 应用界面只包含一个文本框和按键，当用户按下按键时，文本框内的数字就会+1，同时在用户翻转屏幕以及退出应用重新进入的情况下， 显示的内容依然能够得到保留。

由于AndroidViewModel继承于ViewModel，因此添加的依赖和ViewModel完全一致； 而DataBinding的启用在[之前](Android/db)也有叙述，在此不再多做说明。

通过Android Studio新建一个项目，在MainActivity文件的同一目录下新建一个继承于AndroidViewModel的类文件MyAndroidViewModel：

```
class MyAndroidViewModel(application: Application): AndroidViewModel(application) {
    //TODO:
}
```

注意，在使用AndroidViewModel的时候，IDE会提示添加一个构造函数以获取Application对象，从而获得全局资源的访问权限， 比如调用Application对象的get()方法来添加SharedPreferences对象等等。因为使用了构造函数，所以在MainActivity文件中， 创建MyAndroidViewModel对象时，不再和之前的ViewModel一样使用ViewModelProvider，而是可以直接通过构造方法将Application对象传入：

```
val myAndroidViewModel = MyAndroidViewModel(application)
```

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