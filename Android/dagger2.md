[Dagger 2](https://dagger.dev/)是一个在Android开发中广泛应用的依赖注入框架，前身是Square公司开发的Dagger（在Google介入共同开发维护之后才有的Dagger 2）。通常情况下，为了区分Square旗下已停止维护的Dagger，就把前者称为Dagger 2。鉴于原来由Square开发的Dagger已经停止维护，文中主要讨论的是Dagger 2，因此后续在不做特殊说明的情况下，Dagger和Dagger 2均用于指代Google接手开发维护的那个新版依赖注入框架。

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

截至2022年10月，Dagger2最新版本为2.44，具体版本发布详见[google/dagger](https://github.com/google/dagger)。


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

// 如果有多个模块，就使用@Component(modules = [DemoModule::class, ···])
@Component(modules = DemoModule::class)
@Singleton
interface DemoComponent {
    ···
}
```

### `@Subcomponent`声明子组件

子组件是继承并扩展父组件的对象图的组件，因此，父组件中提供的所有对象也将在子组件中提供，这样子组件中的对象就可以依赖于父组件提供的对象(共有)，父组件向子组件提供的对象的作用域仍限定为父组件的生命周期。使用子组件可以定义**更加细致**的作用域。

如何理解“使用子组件可以定义更加细致的作用域”？设想一个包含某些业务逻辑的模块，只在特定场景下才会被调用，那么这个模块显然不适合放到全局容器（比如上面的DemoComponent）当中，否则就会出现这种情况：哪怕只是一个很局限的场景，开发者也不得不再创建一个全局容器实例，而且里面有不少东西是派不上用场的。此外，将一个只在特定场景下才被调用的模块设置为全局容器，显然也不是很符合全局容器的定位；再加上各个模块之间不一定处于相同的作用域，更不可能强行塞进一个已经限定作用域的容器当中。

声明子组件的方式类似下列代码所示：

```
// 定义子组件
@Subcomponent
interface SubDemoComponent {

    @Subcomponent.Factory
    interface Factory {
        fun create(): SubDemoComponent
    }

    fun inject(Activity activity)
}

// 创建子组件模块
@Module(subcomponents = SubDemoComponent::class)
class SubDemoModule {
}

// 将声明子组件的模块添加到全局容器中
@Component(modules = [DemoModule::class, SubDemoModule::class])
@Singleton
interface DemoComponent {
    ···
    fun getSubDemoComponent(): SubDemoComponent.Factory
}
```

为确保子组件只在特定场景使用，还需要使用**相同的自定义`@Scope`注解**来修饰子组件和持有该子组件引用的组件，这样它们才能保持相同的生命周期。在上面的例子中，获取子组件实例对象时，需要先从全局容器调用`getSubDemoComponent()`拿到工厂对象，然后再调用工厂方法获得真正的`SubDemoComponent`类型实例。

