反射（reflection）是一种在运行时获取到一个对象所有信息（如类名、属性以及方法等）的功能。在Java当中，反射是很常见的，但也容易被滥用，从而导致对象的封装性被破坏。下面就从几个方面展开，介绍JVM语言的反射特性和实际用法。

Java的反射机制是JDK自带的，而Kotlin要想调用反射，还需额外添加一个依赖库，如下面所示：

```
dependencies {
    implementation("org.jetbrains.kotlin:kotlin-reflect:X.Y.Z")
}
```

需要注意的是，由于Kotlin兼容Java，后者的反射功能在Kotlin当中是可以调用的。Kotlin自己的反射功能一般通过`SomeType::class`来调用，而Java的反射功能则通过`SomeType::class.java`来调用，如下面代码所示：

```
// 获取一个类的KClass<*>对象调用Kotlin反射功能
String::class.run { 
    ···     
}

// 获取一个类的Class<*>对象调用JDK反射功能
String::class.java.run { 
    ···        
}
```

> 注意，在Java中，除基本类型外，其余的数据类型都是引用类型，换言之就是它们都属于`Class<*>`类型。所以Java反射实际上就是通过某个类的`Class<*>`类型实例来获取该类的所有信息。`Class<*>`类型实例是在JVM加载类/接口的时候创建的，里面已经保存了相应的对象信息。Kotlin也是类似的，只不过用的是`KClass<*>`类型实例。

## 类引用

类引用在Kotlin当中的含义是：通过`SomeType::class`语法来获得一个`KClass<*>`类型实例，通过`SomeType::class.java`语法来获得一个`Class<*>`类型实例。类引用是其他所有反射功能的基础，因此必须掌握。通过类引用，开发者可以获知一个对象实例的属性/字段、函数/方法、父类以及使用的注解等等信息。

## 访问类属性/字段

`KClass<*>`类型实例可以通过下表所示的方式来获取某个类的属性列表：

|访问方式|用途说明|
|:-----:|:-----:|
|`KClass<T>.declaredMemberProperties`|获取该类当中所有非扩展的属性，返回的属性元素类型为`KProperty1<T, *>`|
|`KClass<T>.memberProperties`|获取该类及其父类中所有非扩展的属性，返回的属性元素类型为`KProperty1<T, *>`|
|`KClass<T>.declaredMemberExtensionProperties`|获取该类声明的所有扩展属性，返回的属性元素类型为`KProperty2<T, *, *>`|
|`KClass<T>.memberExtensionProperties`|获取该类及其父类声明的所有扩展属性，返回的属性元素类型为`KProperty2<T, *, *>`|

`Class<*>`类型实例则通过下表所示的方式来获取某个类的特定字段或字段列表（注意返回的字段元素类型都是`Field`）：

|访问方法|用途说明|
|:-----:|:------:|
|`Class<T>.getField()`|获取该类及其父类当中指定名称的`public`字段|
|`Class<T>.getDeclaredField()`|获取该类当中自己声明的指定名称字段|
|`Class<T>.getFields()`|获取该类及其父类所有`public`字段|
|`Class<T>.getDeclaredFields()`|获取该类当中自己声明的所有字段|

### `KProperty*`的使用

前面的内容已经提到，`KClass<*>`实例访问类属性返回的都是`KProperty*`类型的属性元素（包括函数类型的属性也可以返回，并且有对应的操作API）。通过`KProperty*`，开发者可以进行以下操作：

|操作|使用API|
|:-----:|:-----:|
|获取属性名称|`KProperty*.name`|
|获取属性类型或函数类型属性的返回值类型|`KProperty*.returType`|
|获取属性可见性|`KProperty*.visibility`|
|获取属性可访问性|`KProperty*.isAccessible`|
|获取属性注解列表|`KProperty*.annotations`|
|获取属性当前赋值，若没有则返回null|`KProperty*.get()`|
|获取属性的代理赋值，若没有则返回则返回null|`KProperty*.getDelegate()`|
|调用函数类型的属性并获取返回值|`KProperty*.call()`/`KProperty*.callBy()`|
|获取函数类型属性的入参列表|`KProperty*.parameters`|
|判断属性类型是否为抽象类|`KProperty*.isAbstract`|
|判断属性是否为定值常量|`KProperty*.isConst`|
|判断属性类型是否为不可变|`KProperty*.isFinal`|
|判断属性类型是否可继承|`KProperty*.isOpen`|
|判断属性是否需要延迟初始化|`KProperty*.isLateinit`|
|判断函数类型属性是否为挂起函数|`KProperty*.isSuspend`|

### `Field`的使用

`Class<*>`实例可以返回`Field`类型的字段元素，通过`Field`对象，开发者可以进行以下操作：

|操作|使用API|
|:-----:|:-----:|
|获取字段名称|`Field.getName()`|
|获取字段类型|`Field.getType()`|
|获取字段的泛型类型|`Field.getGenericType()`|
|获取字段注解列表|`Field.getAnnotations()`|
|获取字段修饰符|`Field.getModifiers()`|
|获取字段值|`Field.get()`/`Field.getXXX()`|
|修改字段值|`Field.set()`/`Field.setXXX()`|
|允许无视字段可见性直接访问|`Field.setAccessible(true)`|
|判断字段是否可访问|`Field.isAccessible()`|
|判断字段类型是否属于枚举|`Field.isEnumConstant()`|
|判断字段类型是否属于编译器合成的|`Field.isSynthetic()`|

> 注意，Java的Synthetic类型是一种由Java编译器在编译阶段生成的类型，通常用于对`private`字段或类（比如内部类）的访问，从而绕开语言限制。在实际生产中基本不会让开发者用到。

最为值得关注的地方就是`Field`提供了修改字段可访问性以及读写字段值的方法。从开发规范的角度来说，这些操作会破坏掉对象的封装特性，如果滥用可能会给项目的可维护性带来负面影响。此外，`setAccessible(true)`也不是万能的，如果JVM运行期存在`SecurityManager`，那么就有可能阻止`setAccessible(true)`的操作。比如为保护JVM核心库的安全，某些`SecurityManager`可能会阻止开发者对以`java`或`javax`开头的包下面所辖的类调用`setAccessible(true)`。

## 访问类方法

`KClass<*>`类型实例可以通过下列方式来访问不同特性的类方法：

|访问方式|用途说明|
|:-----:|:-----:|
|`KClass<*>.functions`|获取该类及其父类中所有非静态方法，以及该类当中声明的静态方法|
|`KClass<*>.staticFunctions`|获取该类当中声明的静态方法|
|`KClass<*>.memberFunctions`|获取该类及其父类当中声明的所有非扩展函数和非静态方法|
|`KClass<*>.memberExtensionFunctions`|获取该类及其父类当中声明的所有扩展函数|
|`KClass<*>.declaredFunctions`|获取该类及其父类当中声明的所有非静态方法（包括扩展函数），以及该类当中声明的静态方法|
|`KClass<*>.declaredMemberFunctions`|获取该类当中声明的所有非扩展函数和非静态方法|
|`KClass<*>.declaredMemberExtensionFunctions`|获取该类当中声明的所有扩展函数|
|`KClass<*>.constructors`|获取该类所有构造函数|
|`KClass<*>.primaryConstructor`|获取该类最基础的构造函数，若该类没有这样的构造函数则返回null|

`Class<*>`类型实例则通过下列方法来获取类方法：

|访问方法|用途说明|
|:-----:|:-----:|
|`Class<*>.getMethods()`|获取该类及其父类中所有`public`方法|
|`Class<*>.getDeclaredMethods()`|获取该类当中声明的所有方法|
|`Class<*>.getMethod()`|获取该类及其父类中具备指定签名的`public`方法|
|`Class<*>.getDeclaredMethod()`|获取该类当中声明的具备指定签名的方法|
|`Class<*>.getConstructors()`|获取该类所有的`public`构造方法|
|`Class<*>.getDeclaredConstructors()`|获取该类声明的所有构造方法|
|`Class<*>.getConstructor()`|获取该类某个指定的`public`构造方法|
|`Class<*>.getDeclaredConstructor()`|获取该类某个构造方法|

### `KFunction<*>`的使用

`KFunction<*>`代表函数类型，它提供了以下API，方便开发者进行调用（由于有一部分API是从`KCallable`继承而来的（尤其是`call()`和`callBy()`），基本在介绍`KProperty*`的时候都已经提到过，这里不再重复）：

|使用API|用途说明|
|:-----:|:-----:|
|`KFunction<*>.isInline`|判断函数是否为内联函数|
|`KFunction<*>.isExternal`|判断函数是否为NDK调用|
|`KFunction<*>.isOperator`|判断函数是否运算符函数|
|`KFunction<*>.isInfix`|判断函数是否为中缀函数|

### `Method`和`Constructor<*>`的使用

`Method`代表的是非构造方法，Java提供了以下API，方便开发者进行调用：

|使用API|用途说明|
|:-----:|:-----:|
|`Method.getName()`|获取该方法的名称|
|`Method.getReturnType()`|获取方法返回值类型|
|`Method.getParameterTypes()`|获取方法入参类型的数组|
|`Method.getModifiers()`|获取该方法使用的修饰符|
|`Method.getDefaultValue()`|获取该方法的默认值，通常从注解类的方法上调用|
|`Method.isBridge()`|判断该方法是否为桥接方法|
|`Method.isDefault()`|判断该方法是否为接口当中的默认方法|
|`Method.isVarArgs()`|判断该方法是否能传入可变参数|
|`Method.getExceptionTypes()`|获取该方法抛出的异常的类型数组|
|`Method.setAccessible(true)`|允许无视方法可见性直接访问|
|`Method.invoke()`|调用该方法。普通方法需要传入对象实例和参数；静态方法传入的对象实例为null；保持多态特性|

> 注意，桥接方法和前文提到的Syhthetic一样，都是编译器生成的，通常情况下开发者不用太关注。

`Method.invoke()`是最为值得关注的，因为其高度灵活且位于底层位置，在后面介绍动态代理时会派上大用场。

`Constructor<*>`代表的是构造方法，它跟`Method`基本相同，只不过多了`Constructor<*>.newInstance()`这个API可供调用。`Constructor<*>.newInstance()`可以通过传入不同参数来调用类当中声明的**任意**构造方法，从而创建一个对象实例。事实上，`Class<*>`实例也有一个`newInstance()`可以用来创建对象，但它仅代表一个无参构造方法。

## 动态代理

动态代理（Dynamic Proxy）是Java标准库提供的一种机制，其主要作用就是在运行期创建某个接口实例。动态代理的“代理”思想源自[Proxy模式](DesignPattern/结构型设计模式?id=七、proxy)，而“动态”是指这种机制能够灵活实现被代理对象（也就是接口）的业务逻辑。Kotlin和Java在动态代理方面有各自的实现方案，前者通过`by`关键字来实现“委托”，而后者相对麻烦，需要通过专门方式才能实现动态代理。

### Kotlin的委托

#### 类委托

类委托的核心思想是：<font color=red>将一个类的具体实现交由另一个类去完成</font>，即一个类中定义的方法实际是调用另一个类的对象的方法来实现的。示例如下：

```
// 接口
interface Demo {
    fun foo()
}

// 被委托类
class DemoImp(): Demo {
    override fun foo() {
        ···
    } 
}

// 委托类
class Delegate(demo: Demo): Demo by demo {
    ···
}

// 调用
Delegate(DemoImp()).foo()
```

当接口只有较少的方法需要实现时，事实上并不需要委托模式，直接通过已经实现好的被委托类实例化对象去调用相关的方法即可。委托模式的适用场景为：接口定义了大量方法，但其中的**大部分都由被委托类实现，少部分在委托类中另外实现**，并且还可能再声明一些接口以外的独有方法以供调用。

这里再提供一个典型的例子——协程作用域的继承与实现，以便更好理解Kotlin的类委托。如下面代码所示，`Demo`类想要继承`CoroutineScope`的特性，但是如果直接继承`CoroutineScope`，就会发现不知道具体要实现哪些东西，或者实现出来了可能也没有官方的完善。在这种情况下，开发者可以通过`CoroutineScope by MainScope()`来继承`MainScope`对`CoroutineScope`的全部实现方案，不用自己手动实现任何内容，即可调用功能完善的协程API。

```
class Demo(): CoroutineScope by MainScope() {
    ···
}
```

#### 属性委托

属性委托的含义是：<font color=red>将一个属性（字段）的具体实现交由另一个类去实现</font>，即一个类的某个属性值不是在类中直接进行定义，而是将其托付给一个代理类，从而实现对该类的属性统一管理。示例如下：

```
//声明属性：
class Demo {
    var p by Delegate()
}

//管理属性：
class Delegate {
    var propValue: Any? = null

    operator fun getValue(demo: Demo, prop: KProperty<*>): Any? {
        return propValue
    }

    operator fun setValue(demo: Demo, prop: KProperty<*>, value: Any?) {
        propValue = value
    }
}
```

属性委托的意义在于：将属性的声明与读写控制相分离，这样就不必在声明属性的类中为它们一一编写具有相同业务逻辑的`get()`和`set()`方法（试想一下在一个声明了大量属性的类里面，要成对地编写`get()`和`set()`方法是何等恐怖）。

#### lazy函数

`lazy`函数的通用名称是“懒加载函数”，使用`lazy`函数的目的是直到第一次访问该属性的时候，才根据需要创建对象的一部分，以减少资源的消耗，确切地说就是某些逻辑只会在第一次访问的时候执行。其基本使用方式为：

```
val/var v: Type by lazy {
    ···
}
```

尽管在了解委托的基本原理后可以自定义一个`lazy`函数，但是Kotlin内置的`lazy`函数在实现上更为严谨，因而在实际开发中更推荐使用。Kotlin内置`lazy`函数还可以传入一些配置参数来改变具体的懒加载方式，如下表所示：

|参数|用途说明|
|:-----:|:-----:|
|`LazyThreadSafetyMode.SYNCHRONIZED`|保证线程安全的懒加载方式，在初始化实例对象时会加锁，默认方式|
|`LazyThreadSafetyMode.PUBLICATION`|允许并发初始化实例，但只有最先完成初始化操作的线程所返回的实例会被使用|
|`LazyThreadSafetyMode.NONE`|没有线程安全保障的懒加载方式，通常不推荐使用，或是只在单线程当中使用|

### Java动态代理

Java的动态代理方案是基于JDK提供的`Proxy.newProxyInstance()`来实现的。虽然没有写静态的实现类，但是开发在运行期间就可以利用`Proxy.newProxyInstance()`来动态创建接口实例对象。这一过程大致为：

1. 定义一个`InvocationHandler`类型实例，负责实现接口的方法调用；
2. 通过`Proxy.newProxyInstance()`创建接口实例；
3. 将返回的`Object`类型实例强制转换成对应的接口类型。

具体过程参考下面的示例代码：

```
// 接口
interface Computer {
    fun turnOn()
    ···
}

// 定义InvocationHandler实例
val handler = object: InvocationHandler {
    override fun invoke(proxy: Any?, method: Method, args: Array<out Any>?): Any? {

        // 过滤出指定名称的方法
        if ("turnOn" == method.name) {
            // TODO: 实现具体业务逻辑
        }

        // 根据方法返回值类型决定要返回什么结果
        return ···
    }
}

// 创建接口实例
val proxy = Proxy.newProxyInstance(
        Computer.javaClass.classLoader,     // 传入接口的ClassLoader
        arrayOf(Computer::class.java),      // 传入需要实现的接口Class<*>数组，至少包含一种接口 
        handler                             // 传入InvocationHandler实例
    ) as Computer                           // 类型强制转换

// 调用接口实例方法
proxy.turnOn()
···
```

由于Kotlin兼容Java，上面的代码是可以执行的。

动态代理实际上是JVM在运行期动态创建字节码文件并加载的过程，它并没有什么黑魔法，实际上也不存在任何可以直接实例化接口对象的途径——如果有的话，大概就是AI完全取代程序员的时候到了吧。最后再来展示一下JVM通过自动代理编写的出来的静态实现类大致是什么样的（非字节码形式）：

```
public class ComputerProxy implements Computer {
    InvocationHandler handler;

    public ComputerProxy(InvocationHandler handler) {
        this.handler = handler;
    }

    public void turnOn() {
        handler.invoke(
           this,
           Computer.class.getMethod("turnOn"),
           new Object[] {});
    }

    ···
}
```