## ViewBinding

如果在早些时候利用Java开发Android应用，要想在一个Activity里面引用对应的布局控件，一般采用下列方式：

```
<Controls Type> control_name = findViewById(R.id.control_id);
```
显然，布局上有多少个控件，类似的代码就要写多少行，这是非常麻烦的。而且在定义了变量之后不马上调用findViewById()进行赋值的话， 很容易出现漏写导致空引用异常，引发应用崩溃。

Kotlin在早期曾通过`import kotlinx.android.synthetic.*`的方式简化了控件的引用， 使得开发者可以在任意一个Activity或Fragment里面直接通过控件id，来引用项目中存在的控件——即便那个控件并不属于引用它的Activity/Fragment。 这就带来一个隐患：如果开发者不小心引用了不属于本Activity/Fragment的控件，在编译时并不会报错，而应用一旦开始运行，就会由于空引用导致应用崩溃。

Google推出ViewBinding的初衷也在于消除这样的隐患，而现在ViewBinding也确实成为替代ButterKnife的重要开发技术。

如果要在项目中使用ViewBinding，需要Android Studio使用`3.6`版本以上的Gradle插件（可通过File-Project Structure-Project-Android Gradle Plugin Version查看），并在build.gradle(:app)文件中添加以下三种形式的语句启用该功能：

```
android {
    ···
    viewBinding.enabled = true //Android Studio 5.0中弃用

    android.buildFeatures.viewBinding = true //允许

    buildFeatures { //允许
        viewBinding true
    }

    defaultConfig {
        ···
    }
    ···
}
```

可以看到，ViewBinding的启用方式和DataBinding是类似的。

然后在Activity中编写下列代码：

```
//MainActivity文件
···
private lateinit var binding: ActivityMainBinding
···
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        ···
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        ···
        binding.controls.··· = ···
        ···
}

//Fragment文件
···
private lateinit var binding: Layout_Name_Binding
···
override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        ···
        binding = Layout_Name_Binding.inflate(inflater)
        ···
        return binding.root
}
···
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        ···
        binding.controls.··· = ···
        ···
}
```

注意，在某些情况下，如果是在已有项目上进行修改，修改完成后需要Rebuild项目清除缓存，否则可能出现意想不到的报错。

ViewBinding进一步强化了布局控件与Activity/Fragment的关联，在调用binding对应的控件时， 可以确保这些控件全都属于当前Activity/Fragment所对应的布局，从根源上避免了出现错误引用其他布局控件的情况。

## DataBinding

DataBinding可以实现界面与代码逻辑的进一步解耦，不仅能够简化代码， 而且当开发者对应用界面进行大规模改动时，代码逻辑只需要进行很少改动甚至不用改动， 极大提高了项目的可维护性。

这里接着使用[LiveData](Android/ld)中的开发例子， 但需要稍作修改：UI部分将滑动条换成按钮，代码部分的修改在下面进行详细说明。本实例要实现的功能是， 当用户按下按钮时，应用界面上的文本框内的数字会+1，同时继续保持屏幕旋转后内容不变。

首先要在build.gradle(Module:app)文件中找到defaultConfig代码块，然后加入如下语句并同步Gradle文件：

```
dataBinding.enabled = true
```

同步成功之后，activity_main.xml文件的左上角会出现系统提示（如下图所示的黄色灯泡图标）， 点进去之后选择"Convert to data binding layout"，布局文件就会自动转换为DataBinding的布局。

![](pics/db_1.png)


布局转换完成后，就会出现一对data标签，里面具体要填入什么内容会在稍后介绍。现在先往myViewModel文件里面添加一个执行函数， 这个函数将负责响应按钮点击事件，使得文本框内的数字+1：

```
fun add(){
        number.value = number.value!!.plus(1)
}
```

执行函数写完以后，再转到MainActivity文件，把onCreate()函数中ViewModel以外的语句都删掉，只剩下这个部分：

```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val myViewModel = ViewModelProvider(this)[MyViewModel::class.java]
}
```

接着在myViewModel语句后面添加如下内容，MainActivity文件的修改就完成了：

```
val binding: ActivityMainBinding = DataBindingUtil.setContentView(this,R.layout.activity_main)
binding.data = myViewModel
binding.lifecycleOwner = this
```

注意，binding的类型是ActivityMainBinding， 其中，ActivityMain是当前界面布局文件的名称，如果之后使用到带有DataBinding的Fragment， 就要在Binding前添加对应Fragment的布局文件的名称。

此外，由于binding已经通过自己的setContentView()， 将MainActivity文件和当前对应的界面布局文件绑定，所以可以把原先onCreate()中的setContentView()注释掉或删除掉。 通过binding.data，应用可以把MyViewModel中的数据变化传递给布局文件； 而binding.lifecycleOwner则取代了先前LiveData的Observer，以更为简洁的语法实现了Live Data的自我监听，以及数据更新-界面刷新功能。

现在要继续完成activity_main.xml文件的修改工作。之前已经提到，这个文件进行布局转换之后，就出现了一对data标签，里面要填上一些内容， 而这些内容与MainActivity文件里面的binding息息相关。首先给出data标签当中要填写的内容如下：

```
<variable
        name="data"
        type="com.example.databinding.MyViewModel" />
```

这个部分的主要作用是实现布局文件和MyViewModel文件的关联，在关联之后，就可以进行这样的修改： 文本框（TextView）部分，将android:text修改为“@{data.number.toString()}”，表示文本框内容由MyViewModel文件中的number负责提供和展示； 按钮（Button）部分，需要添加android:onClick="@{()->data.add()}"，以实现按钮的点击事件响应，执行MyViewModel文件中的add()函数， 其中()->是一种匿名函数的写法：

```
<TextView
        ···
        android:text="@{data.number.toString()}"
        ···
        />
<Button
        ···
        android:onClick="@{()->data.add()}"
        ···
         />
```
在完成上述所有步骤之后，可以编译运行，检查效果。