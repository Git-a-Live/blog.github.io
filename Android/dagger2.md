[Dagger2](https://dagger.dev/)是一个在Android开发中广泛应用的依赖注入框架，前身是Square公司开发的Dagger（在Google介入共同开发维护之后才有的Dagger2）。通常情况下，为了区分Square旗下已停止维护的Dagger，就把前者称为Dagger2。鉴于原来由Square开发的Dagger已经停止维护，文中主要讨论的是新版Dagger，因此后续在不做特殊说明的情况下，Dagger和Dagger2均用于指代Google接手开发维护的那个依赖注入框架。

Google还在Dagger2的基础上进行二次封装，针对Android开发进行场景化定制，于是就有了下一节要介绍的Hilt框架。无论是Dagger2还是Hilt，都是采用<font color=red>性能更高</font>的**编译时注解**静态方案来实现依赖注入，旧版Dagger采用的是基于反射的动态方案（所以后来被淘汰了）。在了解Dagger2和Hilt框架如何使用之前，需要先简单了解什么是依赖注入。

## 控制反转与依赖注入

控制反转（Inversion of Control，IoC）是一种OOP程序设计思想，来源于设计模式中的[依赖倒置原则](DesignPattern/概述?id=三、面向对象软件设计的基本原则)，用于指导开发者设计出组件（模块）间低耦合、体系结构灵活的软件程序。

在传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制，比如一个组件要使用另一个组件，必须先知道如何正确地创建它（并且通常是在调用者组件内部创建被调用者对象），这样就会带来一个问题：类与类之间高度耦合。当一个系统有大量的组件，如果其生命周期和相互之间的依赖关系都由组件自身来维护，不但大大增加了系统的复杂度，而且会导致组件之间极为紧密的耦合，继而给测试和维护带来了极大的困难。在IoC模式下，控制权发生了反转，即从应用程序转移到了IoC容器，**所有组件不再由应用程序自己创建和配置，而是由IoC容器负责**，这样，应用程序只需要直接使用已经创建好并且配置好的组件。

IoC容器在解决了“谁创建和配置组件”的问题之后，还要解决一个问题：谁负责根据依赖关系组装组件？IoC容器通常采用**依赖注入**（Dependency Injection，DI）的方式，即从**外部**为组件提供运行所需的已完成初始化的资源（比如对象和数据等）。**DI实现的一个重要效果就是将组件的创建和配置与组件的使用相分离，在运行期动态地将某个依赖关系（实际上就是谁调用谁的关系）注入到组件当中**，调用者不再需要关心被调用者如何创建和配置，而被调用者也可以共享于多个调用者之间，从而提高组件复用性。如果只看依赖注入的定义和目的，会发现在很多时候开发者已经不知不觉地用到了依赖注入，比如AndroidViewModel通过构造方法把Application对象“注入”到ViewModel对象中，Glide通过一系列方法链式调用将展示的图片资源“注入”到ImageView对象中，等等。框架不过是把原本的手动实现（开发者充当了IoC容器）改成了自动实现，在基本原理上并没有什么不同。

## Dagger2基本使用

### 依赖导入

向Android项目中导入Dagger2的方式如下：

```
dependencies {
    ···
    implementation 'com.google.dagger:dagger:$specified_version'
    kapt 'com.google.dagger:dagger-compiler:$specified_version'
}
```

截至2022年6月，Dagger2最新版本为2.42，具体版本发布详见[google/dagger](https://github.com/google/dagger)和[sonatype](https://github.com/google/dagger)。


### `@Component`+`@Inject`构造方法注入

`@Component`和`@Inject`是Dagger2最基础的两个注解，仅使用这两个注解就可以实现最简单的依赖注入。前者用于创建一个Dagger容器（中间件），作为获取依赖项的入口，其中应包含满足其提供的类型所需的所有依赖项；后者用于指示Dagger如何实例化一个对象，需要修饰依赖项的构造方法——如果依赖项构造方法中还包含其他的依赖项，那么它们的构造方法也要用该注解修饰。

```
@Component
interface DemoComponent {
    // 定义接口方法以获取依赖项
    fun getDemo0(): Demo0
    // 定义注入外部依赖的方法
    fun injectObj(activity: AppCompatActivity)
    ···
}

class Demo0 @Inject constructor(
    private val demo1: Demo1,
    private val demo2: Demo2
) {
    fun bar() {
        demo1.foo1()
        demo2.foo2()
        ···
    }
    ···
}

class Demo1 @Inject constructor() {
    fun foo1() {
        // TODO
    }
}

class Demo2 @Inject constructor() {
    fun foo2() {
        // TODO
    }
}
```

在build项目之后，被`@Component`修饰的Dagger容器会自动生成一份以**Dagger前缀+容器类名**格式进行命名的`.java`文件（比如DemoComponent容器会生成DaggerDemoComponent.java文件），里面内容类似于下面的示例代码：

```
public final class DaggerDemoComponent implements DemoComponent {
    
    private DaggerDemoComponent() {
    }
    
    // 具体实现的接口，比如创建依赖项实例
    @Override
    public Demo0 foo() {
        return new Demo0(new Demo1(), new Demo2())
    }
    ···

    // 对外提供建造者
    public static DaggerDemoComponent.Builder builder() {
        return new DaggerDemoComponent.Builder();
    }

    // 生成Builder静态内部类
    public static final class Builder {
        ···
        private Builder() {
        }

        public DemoComponent build() {
            return new DaggerDemoComponent();
        }
        ···
    }
}
```

最后，要通过`DaggerDemoComponent.Builder().build()`方法（通常情况下需要进一步封装以便调用）来获取已经实例化了的`DemoComponent`对象，然后就可以执行预先定义好的业务逻辑了。

### `@Inject`字段注入

上面介绍的是基于构造器初始化实现的依赖注入，但是在实际开发当中，有些类并不是用构造器进行初始化的，比如Activity和Fragment，因此需要采用另一种注入方式——**字段注入**。

字段注入的操作类似于下列示例代码：

```
class MainActivity : AppCompatActivity() {
    @Inject
    lateinit var obj: Obj

    override fun onCreate(savedInstanceState: Bundle?) {
        // 手动调用注入方法
        DaggerDemoComponent.Builder().build().injectObj(this)
        super.onCreate(savedInstanceState)
        ...
    }
}

class Obj @Inject constructor(activity: AppCompatActivity) {
    ···
}
```

注意，在Activity或Fragment中使用时，手动注入必须考虑组件的生命周期。在Activity执行手动注入时，需要在`onCreate()`当中、`super.onCreate()`**之前**注入；在Fragment执行手动注入时，则需要在`onAttach()`当中完成注入，但不像Acitivity那样严格强调必须在`super`调用前执行。

### `@Singleton`/`@Scope`声明作用域

`@Singleton`和`@Scope`注解用于声明作用域，从而**约束依赖项的作用域周期**。作用域注解有两个约束：

+ 如果某个组件有作用域注解，那么该组件只能提供带有相同注解的类或者不带任何作用域注解的类；
+ 子组件（Subcomponent）不能使用和某个父组件的相同的作用域注解。

在不同组件上使用相同的作用域注解，就表明两者处于同一作用域周期，这意味着同一个组件多次提供的依赖项都是同一个实例，即实现了单例模式。直接使用内置的`@Singleton`是最简单的，当然开发者也可以基于`@Scope`声明自定义注解，效果和`@Singleton`一样，如下所示：

```
// 内置的@Singleton注解
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}

// 示例用自定义作用域注解
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomScope {}
```

只要满足上面提到的两条作用域约束，Dagger2并不严格限制自定义作用域的语义（划分标准），但是比较理想的做法是**按照声明周期而非业务周期**进行划分。在使用作用域注解之后，代码构建时会自动生成`Provider<T>`类型的对象，作为`T`类型单例依赖项的提供者，如下列代码所示：

```
public final class DaggerDemoComponent implements DemoComponent {
    private Provider<Demo0> demo0Provider;
    ···

    private DaggerDemoComponent(DaggerDemoComponent.Builder builder) {
        assert builder != null;
        this.initialize(builder);
    }

    private void initialize(final DaggerAppComponent.Builder builder) {
        this.demo0Provider = DoubleCheck.provider(/*工厂模式创建依赖项*/);
        ···
    }
    ···
}
```

### `@Module`+`@Providers`实例化对象

`@Module`和`@Providers`注解联合使用，可以用于指示Dagger如何实例化一个对象，但不是以构造器的方式。使用`@Module`修饰的组件，需要在内部使用`@Providers`，来修饰用于提供其它依赖项实例的方法。另外，`@Component`注解也要随之进行修改，以确保Dagger容器能够应用该模块，具体如下列代码所示：

```
@Module
class DemoModule {
    @Providers
    fun provideObj(params: T): Obj {
        ···
        return obj
    }
}

@Singleton
// 如果有多个模块，就使用@Component(modules = [DemoModule::class, ···])
@Component(modules = DemoModule::class)
interface DemoComponent {
    ···
}
```

### `@Subcomponent`声明子组件

子组件是继承并扩展父组件的对象图的组件。

## 基于Dagger2的单元测试

