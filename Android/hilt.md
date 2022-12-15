[Hilt](https://developer.android.google.cn/training/dependency-injection/hilt-android)是Google基于[Dagger 2](https://dagger.dev/)进行二次开发的**依赖注入**框架。Hilt在Dagger 2的基础上针对Android开发进行场景化定制，为Android开发者提供了在使用上更为简单的依赖注入框架。按照Google官方的说法，Hilt通过为项目中的每个Android类提供容器并自动管理其生命周期，提供了一种在应用中使用依赖注入的**标准方法**。无论是Dagger 2还是Hilt，都是采用**性能更高**的编译时注解静态方案来实现依赖注入，而Dagger 2的前身，也就是Square公司（没错，又是它）开发的[Dagger](https://github.com/square/dagger)，采用的是基于反射的动态方案（所以后来被淘汰了）。在了解Hilt框架如何使用之前，需要先简单了解什么是依赖注入。

## 控制反转与依赖注入

控制反转（Inversion of Control，IoC）是一种OOP程序设计思想，来源于设计模式中的[依赖倒置原则](DesignPattern/概述?id=三、面向对象软件设计的基本原则)，用于指导开发者设计出组件（模块）间低耦合、体系结构灵活的软件程序。

在传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，比如一个组件要使用另一个组件，必须先知道如何正确地创建它（并且通常是在调用者组件内部创建被调用者对象），这样就会带来一个问题：类与类之间高度耦合。当一个系统有大量的组件，如果其生命周期和相互之间的依赖关系都由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，**所有组件不再由应用程序自己创建和配置，而是由IoC容器负责**，这样，应用程序只需要直接使用已经创建好并且配置好的组件。

IoC容器在解决了“谁创建和配置组件”的问题之后，还要解决一个问题：谁负责根据依赖关系组装组件？IoC容器通常采用**依赖注入**（Dependency Injection，DI）的方式，即从**外部**为组件提供运行所需的已完成初始化的资源（比如对象和数据等）。**DI实现的一个重要效果就是将组件的创建和配置与组件的使用相分离，在运行期动态地将某个依赖关系（实际上就是谁调用谁的关系）注入到组件当中**，调用者不再需要关心被调用者如何创建和配置，而被调用者也可以共享于多个调用者之间，从而提高组件复用性。如果只看依赖注入的定义和目的，会发现在很多时候开发者已经不知不觉地用到了依赖注入，比如`AndroidViewModel`通过工厂把`Application`对象“注入”到`ViewModel`对象中，Glide通过一系列方法链式调用将展示的图片资源“注入”到`ImageView`对象中，等等。框架不过是把原本的手动实现（开发者充当了IoC容器）改成了自动实现，在基本原理上并没有什么不同。Google还特意在其[开发文档](https://developer.android.google.cn/training/dependency-injection/manual)中为开发者介绍如何通过手动方式执行依赖注入的流程。

## Hilt的基本使用

### 前置工作

首先在项目级build.gradle文件中添加以下依赖：

```
plugins {
    ···
    id 'com.google.dagger.hilt.android' version '$specified_version' apply false
}
```

然后在模块级build.gradle文件中添加以下依赖：

```
plugins {
    id 'kotlin-kapt'
    id 'com.google.dagger.hilt.android'
}

···

dependencies {
    implementation 'com.google.dagger:hilt-android:$specified_version'
    kapt 'com.google.dagger:hilt-android-compiler:$specified_version'
}

// Allow references to generated code
kapt {
  correctErrorTypes = true
}
```

> 注意，Hilt与[Data Binding](Android/databinding)一起使用时，项目必须使用Android Studio 4.0或更高版本。此外，Hilt使用Java 8功能，所以还要在Gradle配置中设置兼容Java 8（不过默认都是自动配置的）。

### 设置Hilt Application类

所有使用Hilt的应用，都必须包含一个带有`@HiltAndroidApp`注解的`Application`类。该注解会触发Hilt的代码生成操作，生成的代码包括应用的一个基类，该基类充当**应用级依赖项容器**：

```
@HiltAndroidApp
class MyApplication: Application() { 
    ···
}
```

此外，这个生成出来的Hilt组件会附加到`Application`对象的生命周期，并为其提供依赖项。此外，它也是应用的**父组件**，这意味着其他组件可以访问它所提供的依赖项。

### 将依赖项注入Android类

在`Application`类中设置了Hilt且有了应用级组件后，Hilt就可以通过使用`@Inject`注解，为带有`@AndroidEntryPoint`或`@HiltViewModel`注解的其他Android类提供依赖项，如下列代码所示：

```
@AndroidEntryPoint
class MainActivity: AppCompatActivity() { 
    ···
}

@HiltViewModel
class MyViewModel: ViewModel() {
    ···
}
```

目前Hilt所支持的Android类有以下几种：

|Android类|使用注解|说明|
|:-----:|:-----:|:-----:|
|`Application`|`@HiltAndroidApp`||
|`ViewModel`|`@HiltViewModel`||
|`Activity`|`@AndroidEntryPoint`|仅支持继承`ComponentActivity`的activity，如`AppCompatActivity`|
|`Fragment`|同上|仅支持继承`androidx.Fragment`的`Fragment`|
|`View`|同上||
|`Service`|同上||
|`BroadcastReceiver`|同上||

入口注解的使用有以下几点要注意：

1. 如果使用`@AndroidEntryPoint`为某个Android类添加注解，则还必须为依赖于（或者说使用）该类的其他Android类也添加注解；
2. `@AndroidEntryPoint`会为项目中的每个Android类生成一个单独的Hilt组件，这些组件可以从它们各自的父类接收依赖项；
3. 如需从组件获取依赖项，要使用`@Inject`注解执行字段注入，并且由Hilt注入的字段不能为私有字段，否则会导致编译错误；
4. Hilt注入的类可以有同样使用注入的其他基类；如果这些类是**抽象类**，则它们不需要`@AndroidEntryPoint`注解。;
5. `ViewModel`实例需要开发者手动通过`ViewModelProvider`“注入”到指定组件，`ViewModel`内部的依赖项才是通过Hilt注入的。

> 关于第五点的解释：如果Google允许开发者使用`@Inject`直接注入`ViewModel`实例，那么就相当于直接在组件里new了一个普通的`ViewModel`对象，后果就是该对象完全起不到监听组件生命周期的作用。

Hilt的入口点和组件息息相关。而除了这些入口点外，还能自定义入口点，但是意义不大，因为Hilt定义的入口点基本已经覆盖了Android中比较重要的东西了。如果确实有需要，可以参考[Google官方文档](https://developer.android.google.cn/training/dependency-injection/hilt-android?hl=zh-cn#not-supported)。

### 使用`@Inject`注入

Hilt需要知道如何从相应组件，对外提供必要依赖项的实例。Hilt“绑定”这个操作，包含了将某个类型的实例作为依赖项提供所需的信息。

向Hilt提供绑定信息的一种方法是构造注入。在某个类的构造函数中使用`@Inject`注解，以告知Hilt如何提供该类的实例，如下列代码所示：

```
// 为一个类的构造函数添加@Inject注解，Hilt就会自动通过该构造函数new一个该类的实例
class SomeObject @Inject constructor() { 
    ···
}

class SomeObject2 @Inject constructor(
    // 一个类通过构造函数所传入的参数，即是该类的依赖项，依赖项也需要让Hilt知道如何创建实例
    private val obj: SomeObject
) {
    ···
}

@HiltViewModel
class MyViewModel @Inject constructor(
    private val obj: SomeObject2
) {
    ···
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    private lateinit var viewModel: MyViewModel 

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // ViewModel只能通过ViewModelProvider获取实例，不能直接使用Hilt注入。
        // 尽管工厂采用无参构造函数创建实例，但ViewModel内部依赖项并不是在此注入，
        // 因此在实际使用中，这套流程不会导致ViewModel依赖项未初始化引发NPE的问题
        viewModel = ViewModelProvider(this)[MyViewModel::class.java]
        ···
    }
}
```

除了构造注入，还有一种被称为字段注入的手段，适用于那些无法在构造函数上使用`@Inject`注解（简单来说就是用不了构造注入）的类，比如Android官方库提供的Activity——在实际开发中，几乎没有人能通过直接new一个Activity来获得Activity实例。对于这种不能动构造函数的类，通常就是采取类似下面示例代码当中的注入方式来获得依赖项实例：

```
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    
    // 将原本给构造函数用的@inject注解用到作为依赖项的字段上
    @Inject
    lateinit var obj: SomeObject

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 直接调用对象函数即可
        obj.foo()
    }
}
```

如果仔细探究就会发现，字段注入实际上是基于构造注入的。因为开发者无论如何都必须要让Hilt知道一个依赖项是怎么被创建出来的，不管是哪种形式的注入，依赖项的构造函数都得加上`@Inject`注解。只要会用`@Inject`注解，那么Hilt最基本的依赖注入就算是入门了。

### Hilt Module

Hilt Module是一个类文件，其主要作用为通过一系列函数来对外提供依赖项。Hilt Module比单纯的构造注入和字段注入更为复杂，但是适用范围比较广，可以提供接口以及第三方库类等无法修改构造函数（或无法直接创建）的对象作为依赖项。

#### Module声明

Module的声明要用到两个注解：`@Module`和`@InstallIn`。前者用来提示该类是一个Hilt Module，能够对外提供依赖项；后者则用于配置当前Module需要安装在哪个Component上（具体有哪些Component，详见Hilt Component一节）。Component代表着一个作用范围，安装在该Component上的Module所提供的依赖方法，只能在当前Component范围内才能进行注入；而且不同的Component对应着不同的生命周期，安装在它上面的Module只会在其生命周期内存在。

Module声明如下列示例代码所示：

```
@Module
@InstallIn(SingletonComponent::class)
object MyHiltModule {
    ···
}
```

Module可以是普通类、单例类、接口或者抽象类。单例类比较特殊，在Component当中是不会经历对象创建和销毁流程的，因此不管在哪里调用，对外提供依赖项的始终都是同一个对象；其他类型因为会随着Component的生命周期而创建销毁，所以不同地方调用的时候极有可能获取到不同的对象。

#### `@Provides`注解的使用

按照Google官方文档的说明，`@Provides`注解用在提供依赖实例的函数上，被标注函数的具体作用主要包括：

+ 告知Hilt自己返回何种类型，即函数会提供哪个类型的实例；
+ 告知Hilt相应类型的依赖项，即函数参数是何种类型；
+ 告知Hilt如何提供相应类型的实例，即执行函数主体当中创建实例的逻辑。

`@Provides`注解的典型使用方式如下面示例代码所示：

```
interface MyInterface {
    fun foo()
}

@Module
@InstallIn(SingletonComponent::class)
object MyHiltModule {
    @Provides
    fun providesMyInterface(): MyInterface = object :MyInterface {
        override fun foo() {
            ···
        }
    }
}

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var myInterface: MyInterface

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 直接调用对象函数即可
        myInterface.foo()
    }
}
```

在Module中，Module的类名以及函数名都可以随意定，Hilt只关心函数的返回值。只要能返回指定类型的实例（如果指定类型是接口或抽象类，开发者必须提供其具体实现的匿名类或子类），Hilt就会直接调用。需要注意的是，在没有其他配置的情况下，**同一种类型的依赖在Module内只能存在一个提供渠道**，否则会发生冲突，具体表现就是编译时会提示该类型“is bound multiple times”，导致项目无法通过编译。另外需要注意的是，Module一般用来提供**不可直接注入**的对象，如果使用`@Inject`就能满足需要，通常都用不到Module。

#### `@Binds`注解的使用

Google官方文档对于`@Binds`注解所发挥的作用有如下说明：

+ 函数返回类型会告知Hilt该函数提供哪个**接口**的实例；
+ 函数参数会告知Hilt要提供哪种**实现**。

在实际使用中，`@Binds`一般用在**抽象函数或者接口函数**——也就是说，如果想使用`@Binds`注解，Module的类型得是接口或者抽象类。具体的使用可以参考下面的示例代码：

```
@Module
@InstallIn(SingletonComponent::class)
interface MyHiltModule2 {
    @Binds
    fun providesMyInterfaces3(impl: MyInterfaceImpl): MyInterface
}

interface MyInterface {
    fun foo()
}

class MyInterfaceImpl @Inject constructor(): MyInterface {
    override fun foo() {
        ···
    }
}
```

由于接口和抽象类的特殊性，使用`@Binds`比`@Provides`要多一些限制：

+ 函数必须有且仅有一个入参；
+ 这个入参必须是返回值类型的具体实现（子类），否则Hilt没法创建实例；
+ 返回值类型必须是接口或抽象类，否则直接用`@Inject`就行了，根本不需要再用Module。

同样地，在没有其他配置的情况下，一种返回类型在Module中依然只能存在一个提供渠道。

#### `@Qualifier`注解的使用

现在考虑这样一个问题：一个接口或抽象类有若干不同实现的子类，出于“面向抽象编程”的考虑，调用这些子类的地方可能都会将实例对象的类型统一设置成相同的父类。在这种情况下，Hilt需要如何分辨出所调用的实例对象究竟属于哪个子类？

换一种问法就是：在有需要的情况下，如何在一个Module中，为同一种类型提供若干种不同的绑定方式？

前面提到过，**在没有其他配置的情况下**，一种返回类型在Module中只能存在一个提供渠道，否则就会发生冲突导致项目无法通过编译。由此可以得知，如果进行了恰当的配置，开发者是可以在一个Module中为同一种类型提供若干种不同绑定方式的，这就用到了`@Qualifier`注解。但是`@Qualifier`并不是用到Module或者函数上，而是用在**定义限定符**。

所谓限定符，本质上也是一些注解，它们的作用就是为Hilt标示出某个类型的子类所对应的绑定方式，说白了就是通过注解的方式提供额外信息，让Hilt知道某个接口或抽象类的实例对象，具体是属于哪个实现子类，从而调用Module中对应函数提供特定类型的依赖项。具体使用方式如下：

首先定义限定符：

```
@Qualifier
annotation class Impl

@Qualifier
annotation class Impl2

···
```

然后在Module中定义不同的提供函数，但是都返回相同类型，并且**为每个函数添加各自的限定符**，以标示出实际返回的子类类型：

```
@Module
@InstallIn(SingletonComponent::class)
interface MyHiltModule2 {
    @Impl
    @Binds
    fun providesMyInterfaces(impl: MyInterfaceImpl): MyInterface

    @Impl2
    @Binds
    fun providesMyInterfaces2(impl: MyInterfaceImpl2): MyInterface

    ···
}

interface MyInterface {
    fun foo()
}

class MyInterfaceImpl @Inject constructor(): MyInterface {
    override fun foo() {
        // 一种实现方式
    }
}

class MyInterfaceImpl2 @Inject constructor(): MyInterface {
    override fun foo() {
        // 另一种实现方式
    }
}

···
```

在需要注入的地方，实例对象也必须使用对应的限定符进行标示，否则调用方无法获得指定类型的实例对象进行调用：

```
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Impl
    @Inject
    lateinit var myInterfaceImpl: MyInterface

    @Impl2
    @Inject
    lateinit var myInterfaceImpl2: MyInterface

    ···

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        myInterfaceImpl.foo()
        myInterfaceImpl2.foo()
        ···
    }
}
```

需要注意的是，限定符并不关注函数到底使用的是`@Binds`还是`@Provides`，因此只要像上面的示例代码那样直接定义和使用即可。

> Hilt预定义有两种限定符，专门用于指定类型上下文的获取：一种是`@ApplicationContext`，另一种是`@ActivityContext`。在使用时，只要将对应限定符添加到Context对象上，Hilt就会提供相应的`Application`或`Activity`的上下文。

### Hilt Component

Hilt会为前文所述的Android类生成对应的Dagger Component，如下表所示：

|Android类|组件|生成时机|销毁时机|作用域|
|:-----:|:-----:|:-----:|:-----:|:-----:|
|`Application`|`SingletonComponent`|`Application.onCreate()`|`Application`销毁后|`@Singleton`|
|`Activity`|`ActivityRetainedComponent`|`Activity.onCreate()`|`Activity.onDestroy()`|`@ActivityRetainedScoped`|
||`ActivityComponent`|同上|同上|`@ActivityScoped`|
|`ViewModel`|`ViewModelComponent`|`ViewModel`创建时|`ViewModel`销毁后|`@ViewModelScoped`|
|`Fragment`|`FragmentComponent`|`Fragment.onAttach()`|`Fragment.onDestroy()`|`@FragmentScoped`|
|`View`|`ViewComponent`|`View.super()`|`View`销毁后|`@ViewScoped`|
||`ViewWithFragmentComponent`|同上|同上|同上|
|`Service`|`ServiceComponent`|`Service.onCreate()`|`Service.onDestroy()`|`@ServiceScoped`|

按照Google官方的说法，Hilt生成这些Component之后还会自动巡查代码，以便**构建并验证依赖关系图**，确保没有未满足的依赖关系且没有依赖循环，并且生成它在运行时用来创建实际对象及其依赖项的类。需要强调的是，由于生命周期和对象的创建销毁都是敏感问题，因此在实际开发中不要滥用长生命周期的Component来提供依赖项，避免造成内存泄漏等问题。下面简单介绍一下各个Component的作用。

#### `SingletonComponent`

`SingletonComponent`是针对`Application`的组件，安装在它上面的Module会与`Application`的生命周期保持一致，并且该Module所提供的依赖，**在整个程序中都是可以使用的**。

#### `ActivityRetainedComponent` / `ActivityComponent`

`ActivityRetainedComponent`是针对`Activity`的组件，其生命周期实际上跟`ViewModel`一致，安装在该组件上的Module提供的依赖，可以在`ViewModel`、`Activity`、`Fragment`以及`View`中注入。

`ActivityComponent`跟`ActivityRetainedComponent`的差别只有两点：一是`ActivityComponent`的生命周期与`Activity`完全一致，二是`ActivityComponent`不能在`ViewModel`中注入。

#### `ViewModelComponent`

`ViewModelComponent`和`ActivityRetainedComponent`在生命周期方面是一样的，唯一的区别是，安装在它上面的Module提供的依赖只能在`ViewModel`中使用。

#### `FragmentComponent`

`FragmentComponent`是针对于`Fragment`的，生命周期跟`Fragment`一致，安装在它上面的Module提供的依赖只能在`Fragment`中使用。

#### `ViewComponent` / `ViewWithFragmentComponent`

`ViewComponent`是针对于`View`的，生命周期与`View`一致，并且安装在它上面的Module提供的依赖只能在`View`中使用。

`ViewWithFragmentComponent`也是针对`View`的，但是注入的时候不仅要求在`View`上加入`@AndroidEntryPoint`，**还要加上`@WithFragmentBindings`注解**。安装在它上面的Module的生命周期也是与`ViewComponent`一样的。其中提供的依赖只能用在`View`上，**而且这个`View`还只能用在`Fragment`中，不能用在`Activity`中**。

#### `ServiceComponent`

`ServiceComponent`是针对`Service`的，生命周期与`Service`一致，并且安装在它上面的Module只能用在`Service`中。

