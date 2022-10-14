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

此外，这个生成出来的Hilt组件会附加到`Application`对象的生命周期，并为其提供依赖项。此外，它也是应用的**父组件**，这意味着其他组件可以访问它提供的依赖项。

### 将依赖项注入Android类

在`Application`类中设置了Hilt且有了应用级组件后，Hilt就可以为带有`@AndroidEntryPoint`或`@HiltViewModel`注解的其他Android类提供依赖项，如下列代码所示：

```
@AndroidEntryPoint
class MainActivity : AppCompatActivity() { 
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

注解的使用有以下几点要注意：

1. 如果使用`@AndroidEntryPoint`为某个Android类添加注解，则还必须为依赖于（或者说使用）该类的其他Android类也添加注解；
2. `@AndroidEntryPoint`会为项目中的每个Android类生成一个单独的Hilt组件，这些组件可以从它们各自的父类接收依赖项；
3. 如需从组件获取依赖项，要使用`@Inject`注解执行字段注入，并且由Hilt注入的字段不能为私有字段，否则会导致编译错误；
4. Hilt注入的类可以有同样使用注入的其他基类；如果这些类是**抽象类**，则它们不需要`@AndroidEntryPoint`注解。

Hilt会为上述Android类生成对应的Dagger组件，如下表所示：

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

### 定义Hilt绑定

为了执行字段注入，Hilt需要知道如何从相应组件对外提供必要依赖项的实例。Hilt“绑定”这个操作，包含了将某个类型的实例作为依赖项提供所需的信息。

向Hilt提供绑定信息的一种方法是**构造注入**。在某个类的构造函数中使用`@Inject`注解，以告知Hilt如何提供该类的实例，如下列代码所示：

```
class AnalyticsAdapter @Inject constructor(
    // AnalyticsService就是AnalyticsAdapter的一个依赖项
    private val service: AnalyticsService
) { 
    ···
}
```

在一个类中，带注解构造函数所传入的参数**即是该类的依赖项**，因此Hilt还必须知道如何提供依赖项的实例。

### Hilt模块



