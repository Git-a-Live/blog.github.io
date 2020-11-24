如果在早些时候利用Java开发Android应用，要想在一个Activity里面引用对应的布局控件，一般采用下列方式：

```
<Controls Type> control_name = findViewById(R.id.control_id);
```
显然，布局上有多少个控件，类似的代码就要写多少行，这是非常麻烦的。而且在定义了变量之后不马上调用findViewById()进行赋值的话， 很容易出现漏写导致空引用异常，引发应用崩溃。

Kotlin通过`import kotlinx.android.synthetic.*`的方式简化了控件的引用， 使得开发者可以在任意一个Activity或Fragment里面直接通过控件id，来引用项目中存在的控件——即便那个控件并不属于引用它的Activity/Fragment。 这就带来一个隐患：如果开发者不小心引用了不属于本Activity/Fragment的控件，在编译时并不会报错，而应用一旦开始运行，就会由于空引用导致应用崩溃。

Google推出ViewBinding的初衷也在于消除这样的隐患。

如果要在项目中使用ViewBinding，需要在build.gradle(:app)文件中添加下面语句以启用该功能：

```
android {
    ···
    viewBinding.enabled = true
    defaultConfig {
        ···
    }
    ···
}
```

可以看到，ViewBinding的启用方式和DataBinding是一样的。

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

注意，如果是在已有项目上进行修改，修改完成后需要Rebuild项目，否则可能出现意想不到的报错。

ViewBinding进一步强化了布局控件与Activity/Fragment的关联，在调用binding对应的控件时， 可以确保这些控件全都属于当前Activity/Fragment所对应的布局，从根源上避免了出现错误引用其他布局控件的情况。