单元测试（Unit Test）用于来对一个模块、一个函数或者一个类等来进行正确性检验，它是TDD（Test Driven Development，测试驱动开发）的一个重要组成部分。单元测试的普遍好处在于：

1. 便于后期重构；
2. 反馈快速；
3. 缩小关注范围，降低开发负担。


在Android开发方面，按照Google官方的说法：

>通过针对代码创建和运行单元测试，您可以轻松验证各个单元的逻辑是否正确。在每次构建后运行单元测试可帮助您快速捕捉和修复由应用的代码更改导致的软件回归。（注：“回归”指未有效修复原有bug，甚至还引入了新bug）

同时Google官方也强调：

>单元测试不适用于测试复杂的界面互动事件。

鉴于单元测试在开发中的重要地位，Android Studio提供了一整套相对完善的单元测试工具，使用了JUnit、Robolectric、AndroidJUnitRunner以及Espresso等单元测试框架，基本能够满足开发者的需要。当然，Google还推荐使用[Mockito框架](https://github.com/mockito/mockito)，但是限于篇幅，这里不作展开。

但是要注意，在开发中切忌本末倒置，绝不能纯粹地为了测试而测试（比如一行简单代码也要编写单元测试），这样就违背了单元测试的初衷。


## 本地测试

几乎每个在Android Studio上创建的项目，都会包含一个test目录，里面存放着一份ExampleUnitTest文件，其内容大概如下：

```
class ExampleUnitTest {
    @Test
    fun addition_isCorrect() {
        assertEquals(4, 2 + 2)
    }
}
```

这就是Android Studio基于JUnit 4所提供的本地测试类。所谓本地测试，是指仅在本地计算机上运行的单元测试，测试代码编译为在 Java 虚拟机 (JVM) 本地运行，以最大限度地缩短执行时间。也就是说，如果要测试的代码只要有JVM就能跑，那么可以直接放到本地测试类里去执行，就像上面的示例代码。

使用Android Studio的JUnit进行基本的单元测试有几个地方需要注意：

+ 核心测试方法必须加上`@Test`注解；
+ 核心测试方法没有参数传入和结果返回；
+ 测试代码不能依赖于Android框架（比如尝试获取Context对象）。

前两条是JUnit本身的要求，关于最后一条注意事项，Google官方建议使用AndroidX Test提供的Robolectric工件，以执行真实的Android框架代码和原生框架代码的虚假对象，这就涉及到接下来要介绍的插桩测试。

## 插桩测试

Google官方对于插桩测试的解释是这样的：

>插桩单元测试是在实体设备和模拟器上运行的测试，此类测试可以利用 Android 框架 API 和辅助性 API，如 AndroidX Test。

同时强调：

>我们建议只有在必须针对真实设备的行为进行测试时才使用插桩单元测试。（注：比如获取一个Context对象）

和本地测试一样，插桩测试的示例在创建项目时就已经存在，位于androidTest目录，文件名为ExampleInstrumentedTest。插桩测试类的用法和本地测试类有许多相似的地方，主要区别就在于插桩测试类可以通过AndroidX Test运行JUnit 4 测试运行程序 (AndroidJUnitRunner) 和用于功能界面测试的 API（[Espresso](https://developer.android.google.cn/training/testing/espresso)和[UI Automator](https://developer.android.google.cn/training/testing/ui-automator)），在功能上比本地测试更为丰富和复杂。
