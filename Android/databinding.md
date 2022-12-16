Data Binding是Google推出的Android Jetpack的重要组件，它和`ViewModel`一样都是Android MVVM架构的具体实现方案。按照Google的说法，Data Binding采用“声明式”（declartive）的编程方法将布局中的控件绑定到应用中的数据源。

最直白的解释就是，借助Data Binding，`ViewModel`可以直接跟`xml`文件进行数据和动作交互，而不必像以往那样在Activity/Fragment当中为控件编写大量的赋值和监听回调语句。这种方式减少了Activity/Fragment的工作负担，更加提高了ViewModel层和View层的解耦程度。总体来看，Data Binding在MVVM架构当中发挥的作用主要有两个：1）<font color=red>简化控件引用方式；</font>2）<font color=red>实现数据单向或双向绑定到布局文件。</font>如果只是想简化控件引用，那么应该考虑选用[View Binding](Android/viewbinding)。

> Data Binding的使用对AGP版本有要求，即项目使用的AGP版本不能低于**1.5.0**。当然，现在新项目是不需要考虑这个问题的，只有那些在Android Studio 1.0刚出来的时期就开发的老旧项目，想要接入Data Binding功能时才要考虑AGP的版本兼容问题。

## Data Binding入门

### 功能启用

Data Binding的启用不需要导入任何依赖库，只需在模块级build.gradle文件中添加如下语句即可开启这个功能：

```
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
```

开启功能之后还要同步Gradle。同步成功之后，布局的`xml`文件的左上角会出现系统提示（如下图所示的黄色灯泡图标），点进去之后选择"Convert to data binding layout"，布局文件就会自动为原有的布局文件最外层再加上一对\<layout>标签，如下图以及下面的示例代码所示：

![](pics/db_1.png)

```
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>

    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">
        ···
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

### 设置绑定变量

在上面的示例代码中，启用Data Binding布局除了新增一对\<layout>标签之外，还出现了一对\<data>标签，这里面要填的内容在Google官方文档中被称为“绑定变量”，具体大致如下：

```

<data>
    <variable 
        name="viewmodel" 
        type="com.example.app.MyViewModel" />
    <variable 
        name="parent" 
        type="com.example.app.ParentEntity" />
    ···
</data>
```

这些由\<variable>标签封装的绑定变量乍一看很令人迷惑，但是如果把`xml`文件视为一个“类”，把绑定变量看作是这个“类”的字段属性，比如`MyViewModel viewModel`和`val viewModel: MyViewModel`，就能意识到这些绑定变量大概会起到什么作用——获取属性、设置属性以或是调用业务逻辑函数。事实上，Data Binding就是这么干的，只不过增加了一个把`xml`文件映射成类的环节，这在下一步的内容中会介绍。

### 设置Binding类

现在暂时把目光从`xml`文件当中移开，放到需要调用`xml`文件渲染界面的那些对象（Activity、Fragment或是RecyclerView Adapter等）上。前面提到，如果把`xml`文件视为一个“类”，把绑定变量看作是这个“类”的字段属性，那么就很容易理解绑定变量是用来干什么的。Data Binding确实将`xml`文件映射成了专门的类，它们的名字都是`XXXBinding`。比如MainActivity的`xml`文件名为`activity_main.xml`，那么对应的Binding类名称为“ActivityMainBinding”，也就是把`xml`文件名去掉`.xml`后缀，然后将剩下的部分用大驼峰命名法重新写出来，再加上一个“Binding”后缀。

在Activity中，绑定`xml`文件的方式类似于下列代码：

```
class MainActivity : AppCompatActivity() {
    // 定义一个Binding类型的对象
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 通过DataBindingUtil.setContentView()取代Activity原有的setContentView()
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        

        // 从现在开始，Activity可以直接调用布局文件中定义的绑定变量了
        binding.viewModel = ···
        // 绑定变量中有ViewModel的，必须设置LifecycleOwner，否则LiveData无法被监听
        binding.lifecycleOwner = this
        binding.parent = ParentEntity(···)
        ···
    }
}
```

> 注意在使用Data Binding的情况下，如果Activity通过`XXXBinding.inflate()`去渲染界面，可能会出现找不到视图引发应用崩溃的问题。

如果是在Fragment或者ListView/RecyclerView Adapter当中，要采用下面方式设置Binding类：

```
// 直接从Binding类渲染界面
val binding: XXXBinding = XXXBinding.inflate(layoutInflater, viewGroup, false)

// 调用Binding类的bind()渲染界面
val view = inflater.inflate(R.layout.xxx, container, false)
val binding: XXXBinding = XXXBinding.bind(view)

// 通过DataBindingUtil.inflate()间接渲染界面
val binding: XXXBinding = DataBindingUtil.inflate(layoutInflater, R.layout.xxx, viewGroup, false)
```

### 绑定表达式

Data Binding最重要的一步，就是将数据绑定到`xml`文件当中的目标控件上，这一步就是通过所谓的“表达式语言”（expression language）来实现。Google官方在文档中提供了一个简单示例，来初步展现如何使用表达式语言：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

注意示例代码中`android:text="@{···}"`的部分，`@{}`里面包含的内容就是被绑定到目标控件上的数据以及函数等“表达式”。以上面代码为例，`android:text="@{user.firstName}"`就等价于开发者Activity等View层载体中，使用“命令式”风格编写的`textView.text = user.firstName`这行代码。换句话说，采用Data Bindng的表达式语言后，就几乎可以不用在Activity、Fragment以及RecyclerView Adapter里面编写诸如`textView.text = user.firstName`或是`button.setOnClickListener{···}`这类命令式的框架调用代码了。

当然，Google官方也强调称“Layout expressions should be kept small and simple, as they can't be unit tested and have limited IDE support”，布局表达式要短小精炼，因为它们没法进行单元测试，并且IDE的支持也很有限，太复杂的表达式不仅容易出错，而且有可能会超出Data Binding所能支持的用法，反而给开发者自己带来不便。

实际上，表达式语言的使用并非只有这么点儿东西，这里仅仅是介绍了它的格式以及含义，更多具体的使用方法会在下一节进行深入介绍。

## Data Binding进阶

### 表达式语言

表达式语言跟Java代码很相似，在某种程度上甚至可以将其视为Java的一个子集。表达式语言支持使用以下运算符和关键字：

|运算符/关键字|说明|
|:-----:|:-----:|
|算术运算符`+ - / * %`||
|字符串连接运算符`+`||
|逻辑运算符`&& \|\|`||
|二元运算符`& \| ^`||
|一元运算符`+ - ! ~`||
|移位运算符`>> >>> <<`|注意`<`需要转义为`&lt;`|
|比较运算符`== > < >= <=`|同上|
|`instanceof`|返回boolean值|
|分组运算符`()`||
|字面量值|字符、字符串、数字、`null`|
|类型转换|一般调用`XXX.valueOf()`方法，也可以使用在Java所允许的强制转换方式|
|方法调用|一般用在`android:onClick`这类动作属性上面|
|属性访问|比如将一个`LiveData`绑定到`android:text`属性上，`android:text`实际访问的是`LiveData.value`|
|集合访问`[]`|通过`[]`直接访问集合中指定下标或者指定key的元素|
|三元运算符`?:`||

> 注意，属性访问**不需要手动添加判空逻辑**，因为Data Binding生成的代码会自动检查有没有`null`值并采取一定措施避免出现NPE。例如，在表达式`@{user.name}`中，如果`user`为 Null，则为`user.name`分配默认值`null`；如果引用`user.age`，其类型为`int`，则数据绑定使用默认值0。

下面给出了一些常见用法。

+ **`Null`合并运算符`??`**

运算符`??`的含义为：若左边运算数不是`null`，就选择左边运算数，否则选右边运算数。使用方式为：

```
android:text="@{user.displayName ?? user.lastName}"

// 等效语法
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

+ **视图引用**

通过以下语法可以按ID引用同一布局中的其他视图：

```
<EditText
    android:id="@+id/example_text"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"/>

<TextView
     android:id="@+id/example_output"
     android:layout_width="wrap_content"
     android:layout_height="wrap_content"
     android:text="@{exampleText.text}"/>
```

> 注意，Binding类会将控件ID转换为驼峰式大小写。

+ **字符串字面量**

可以使用单引号`''`括住特性值，这样就可以在表达式中使用双引号`""`，如以下示例所示：

```
android:text='@{map["firstName"]}'
```
 
也可以使用双引号括住特性值。如果这样做，则应使用反单引号“`”将字符串字面量括起来：

```
android:text="@{map[`firstName`]}"
```

#### 资源引用

表达式可以使用以下语法引用资源：

```
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```
    
也可以通过提供参数来评估格式字符串和复数形式：

```
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```
    
还可以将属性引用和视图引用作为资源参数进行传递：

```
android:text="@{@string/example_resource(user.lastName, exampleText.text)}"
```
    
当一个复数带有多个参数时，必须传递所有参数：

```
Have an orange
Have %d oranges

android:text="@{@plurals/orange(orangeCount, orangeCount)}"
```

某些资源需要显式类型求值，如下表所示：

|类型|常规引用|表达式引用|
|:---:|:-----:|:-----:|
|`String[]`|@array|@stringArray|
|`int[]`|@array|@intArray|
|`TypedArray`|@array|@typedArray|
|`Animator`|@animator|@animator|
|`StateListAnimator`|@animator|@stateListAnimator|
|`color int`|@color|@color|
|`ColorStateList`|@color|@colorStateList|  

#### 事件处理

这里的“事件”主要是指用户点击控件时所发生的交互事件。许多控件会提供诸如`setOnClickListener()`这类设置事件监听器的方法，供开发者在Activity等处调用。在`xml`文件中，这些监听器会以`android:onXXX`形式的属性出现，其名称通常为设置监听器方法签名去掉“set”前缀和“Listener”后缀剩余的部分。比如`setOnClickListener()`对应的属性名称就是`android:onClick`。

事件处理的表达式支持**方法引用**和**监听器绑定**两种机制，前者传入的是一个函数类型对象的引用，后者传进去的是一个lambda表达式。方法引用和监听器绑定之间的主要区别在于：当传入一个方法引用，Data Binding会将方法引用和所有者对象封装到监听器中，并在<font color=red>绑定数据时</font>为目标控件设置该监听器（传入null就不会创建监听器）；当传入一个lambda表达式，就会在<font color=blue>事件触发时</font>创建监听器并设置给目标控件。

方法引用大致类似于下列代码所示：

```
class MyViewModel: ViewModel() {
    // 采用方法引用时，函数入参类型和数量必须和监听器回调方法完全一致   
    fun foo(view: View) {
        ···
    }
}

// xml文件设置监听器
android:onClick="@{viewModel::foo}"/>
```

监听器绑定大致类似于下列代码所示：

```
class MyViewModel: ViewModel() {
    // 采用监听器绑定时，函数入参类型和数量不一定和监听器回调方法完全一致   
    fun bar(param: Int，param2: String, ···) {
        ···
    }
}

// xml文件设置监听器
android:onClick="@{(p1, p2, ···) -> viewModel.bar(p1, p2, ···)}"/>
```

> 注意，无论是方法引用还是监听器绑定，如果监听器回调方法本身有非空返回值，传入的函数类型对象也必须具有<font color=red>相同类型</font>的返回值。

#### 导入、变量与包含

+ **导入**

在\<data>标签里面，除了\<variable>之外，还有一个\<import>标签。这个标签的作用跟源码里面`import`关键字的作用是基本一致的。在导入以后，开发者就可以像在Java/Kotlin源码当中一样调用这些类，包括静态方法等方面的使用都跟平时没有什么两样。此外，导入指定类以后，在定义绑定变量时，对应的`type`属性就可以直接写类名，而不用再加上package的部分。

当导入的类发生类名冲突时，还可以为其中一个类设置别名`alias`属性，其他地方在调用这个类时只要使用别名即可避免冲突，如下列代码所示：

```
<import type="android.view.View"/>
<import type="com.example.real.estate.View" alias="Vista"/>
```

最后要提醒的是，如果绑定变量的`type`是**泛型类**，在设置的时候要将`<`转义成`&lt;`，如下列代码所示：

```
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>
```

+ **变量**

变量类型在编译时进行检查，因此，如果变量实现`Observable`或者是可观察集合，则应反映在类型中。如果该变量是不实现`Observable`接口的基类或接口，则变量是“不可观察的”。

如果不同配置（例如横向或纵向）有不同的布局文件，则变量会合并在一起。这些布局文件之间**不得存在有冲突的变量定义**。

在生成的Binding类中，每个描述的变量都有一个对应的setter和getter。在调用 setter之前，这些变量会被赋予默认值，例如引用类型采用`null`，`int`采用0，`boolean`采用false等等。

系统会根据需要生成名为context的特殊变量，用于绑定表达式。context的值是根视图`getContext()`方法提供的`Context`对象。context变量会被具有该名称的显式变量声明替换。

+ **包含**

通过使用应用命名空间和特性中的变量名称，变量可以从上层布局传递到被包含在里面的下一层布局，并实现绑定。以下示例展示了来自name.xml和contact.xml布局文件的被包含变量——user：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
    <data>
        <variable name="user" type="com.example.User"/>
    </data>
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <include layout="@layout/name"
            bind:user="@{user}"/>
        <include layout="@layout/contact"
            bind:user="@{user}"/>
    </LinearLayout>
</layout>
```
    
数据绑定不支持`include`作为`merge`元素的直接子元素。例如以下这种布局就是不受支持的：

```
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
    <data>
        <variable name="user" type="com.example.User"/>
    </data>
    <!-- Doesn't work -->
    <merge>
        <include layout="@layout/name"
            bind:user="@{user}"/>
        <include layout="@layout/contact"
            bind:user="@{user}"/>
    </merge>
</layout>
```

### 双向数据绑定

#### 双向绑定快速入门

上文所提到的绑定数据到布局文件的方式属于“单向数据绑定”，即布局文件可以通过绑定变量来感知数据源的变化并刷新界面，但是不能将用户与界面交互时产生的数据（比如通过文本框输入一串字符）直接传递给数据源，而是要借助事件处理机制，或者在View层载体中另外调用控件的交互事件监听器，来间接地修改数据源。比如下列代码就是在单向数据绑定情况下，控件感知数据变化和传递交互数据需要分开执行的一个典型例子：

```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{viewmodel.rememberMeChanged}" />
```

可以看到，如果一个控件同时具备展示数据和产生数据的功能，那么至少要在布局文件中写两种执行逻辑。在控件数量较多的情况下，Data Binding的简洁优势很快就会被这种重复工作所抵消。于是Data Binding由提供了一种被称作“双向数据绑定”的工作方式，如下列示例代码所示：

```
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}" />
```

注意到示例代码中使用的表达式格式跟之前相比有些许差异。**单向数据绑定采用的是`@{}`表示法，而双向数据绑定采用的是`@={}`表示法（多了一个`=`符号）**。通过双向数据绑定，控件可以在一行代码中实现两种功能：接收对应属性的数据更改，同时监听用户与控件交互所产生的数据更新。需要注意的是，双向数据绑定不是什么地方都能用，在一般情况下，只有具备“双向特性”的控件属性才能像上面示例代码那样使用这个功能。官方已经预定义了一些的支持双向特性的控件属性，具体如下表所示：

|类|预置的双向特性属性|
|:-----:|:-----:|
|`AdapterView`|`android:selectedItemPosition`、`android:selection`|
|`CalendarView`|`android:date`|
|`CompoundButton`|`android:checked`|
|`DatePicker`|`android:year`、`android:month`、`android:day`|
|`NumberPicker`|`android:value`|
|`RadioButton`|`android:checkedButton`|
|`RatingBar`|`android:rating`|
|`SeekBar`|`android:progress`|
|`TabHost`|`android:currentTab`|
|`TextView`|`android:text`|
|`TimePicker`|`android:hour`、`android:minute`|

这些属性之所以能够支持双向特性，最关键的一点在于配置了Binding适配器。事实上，如果知道了怎么编写Binding适配器，原本不支持双向特性的控件属性，还有开发者自定义的控件属性，都可以具备双向特性，从而用上双向数据绑定功能，而这些内容将会在下文进行更为详细的介绍。

#### 可观察数据对象

“可观察性”是指**一个对象将其数据变化告知其他对象**的能力。Data Binding库提供了可让开发者轻松观察数据更改情况的类和方法，因此开发者不需要操心在底层数据源发生更改时应该如何刷新界面。借助该库，开发者可以将对象、字段、控件属性或集合设为可观察。

按照Google官方的说法，任何普通对象（plain-old object，一个典型的例子就是Java当中未继承实现任何父类，并且未被其他框架入侵过的POJO）都能够用于数据绑定，但是它们自身是不能自动通知UI去刷新界面的。Data Binding就为这些普通对象提供了在数据变化时通知控件监听器的能力。当然，还有一种途径，就是让这些普通对象自己成为可观察对象。像`LiveData`这类可观察对象被绑定到UI界面之后，就可以自动刷新UI界面了。Google在Data Binding库中提供了一系列继承`BaseObservable`的可观察类，供开发者使用。但是Google后来在[开发文档](https://developer.android.google.cn/topic/libraries/data-binding/observability#kotlin)中增加了一条提示，表示从Android Studio 3.1开始，`LiveData`已经可以取代可观察类，承担可观察类原有的功能。换句话说，现在Data Binding通常是跟`LiveData`搭配使用，原来这些基于`BaseObservable`类的可观察类基本上已经没有用武之地了。

#### Binding适配器

每一个布局表达式都有一个对应的Binding适配器，用于进行框架调用来设置相应的属性或监听器。例如，Binding适配器可以通过调用`setText()`方法来设置文本属性，或者通过调用`setOnClickListener()`方法向点击事件添加监听器。常用的Bidning适配器可以在[Google官方文档](https://developer.android.google.cn/topic/libraries/data-binding/two-way#two-way-attributes)中查看了解，开发者在`android.databinding.adapters`依赖库中就可以调用它们。事实上，官方提供的这些Binding适配器在大多数情况下都已经够用了，因为它们覆盖了大多数的控件类型。如果要为某一现成类型的控件增加适配器方法，利用Kotlin的[扩展函数](Kotlin/late?id=扩展函数)就基本可以解决问题。当然，如果是开发者自定义的控件类型，可能确实需要自定义的Binding适配器来满足开发需要。限于篇幅，这里并不深入介绍如何自定义Binding适配器，如有需要可以访问相应的[Google开发文档](https://developer.android.google.cn/topic/libraries/data-binding/binding-adapters)。

