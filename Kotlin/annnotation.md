## 何为注解

注解（annotation）是一种可被编译器打包进字节码文件，起到标注作用的元数据代码标签。注解和注释类似，可以用在类和类成员上，只不过前者不会被编译器忽略，而后者会。Kotlin当中的注解概念，跟Java是一样的，并且Kotlin能完全兼容Java注解，因此在讨论注解的时候，除了部分语法差异需要说明外，一般不会区分到底是Kotlin注解还是Java注解。

注解可以大致分为以下三类：

+ **编译期注解**：诸如`@Override`这类注解只会在编译期使用，并不会进入到字节码文件当中。

+ **字节码文件注解**：有一些工具在处理字节码文件时会用到注解，以实现对字节码文件的动态修改，实现一些特殊功能。这类注解会被编译进字节码文件当中，但在加载结束之后就不会继续保留在内存里面。此外，这种注解只被一些底层库使用，开发者不必自己处理。

+ **运行时注解**：这类注解是最常用的，在加载后会一直存在于JVM当中，也能在程序运行期间被读取。代码读取到这类注解之后，就会执行其所实现的对应功能。



对于开发者而言，一些由语言标准库和第三方依赖库所提供的现成注解，使用起来通常并不会有什么问题，除非完全看不懂文档和示例。因此这里并不会详细介绍有什么常见的注解，以及这些注解应该怎么使用。相反，了解如何自定义注解，在很大程度上可以让开发者加深对于现成注解如何发挥作用的理解。

## 自定义注解

### 定义注解

#### 声明

在Kotlin语言中，声明一个自定义注解的方式如下：

```
annotation class Demo(···)
```

而Java当中的声明方式为：

```
@interface Demo {
    ···
}
```

可以发现，Kotlin声明注解的方式跟普通的class基本一样，只不过多了一个`annotation`修饰符。而Java声明注解的方式更接近于声明一个接口。

注解当中可以使用参数，注解支持的参数类型有：

+ 基本类型；
+ 字符串；
+ 类的Class；
+ 枚举；
+ 其他注解；
+ 上述类型的数组。

#### 元注解

元注解（meta annotation）是一类可以用来修饰其他注解的注解，通常由语言的标准库定义和内置，开发者一般不需要自己去编写元注解，只要使用现成的即可。下表展示的是Kotlin和Java当中可以使用的元注解，✔表示该语言支持使用该元注解，❌表示该语言不支持使用该元注解：

|元注解|用途说明|Kotlin|Java|
|:-----:|:-----:|:-----:|:-----:|
|`@Target`|限定注解的适用范围，如限定注解用于类/接口、字段、方法或者方法参数等|✔|✔|
|`@Retention`|用于定义注解的生命周期，如仅在编译期、运行期或字节码文件中存在|✔|✔|
|`@Repeatable`|用于设置注解能否重复使用，应用不是很广|✔|✔|
|`@MustBeDocumented`|用于指定该注解是公有API的一部分，应该包含在生成的API文档中所显示的类或方法签名中|✔|❌|
|`@Inherited`|用于设置子类能否集成父类定义的注解，仅对`ElementType.TYPE`类型的注解以及普通类（非接口）的继承有效|❌|✔|

Java和Kotlin在上表当中前三个元注解的参数方面也有一些差异，具体可见下表：

|元注解|Kotlin使用参数|Java使用参数|
|:-----:|:-----:|:-----:|
|`@Target`|类型（包括类、接口及枚举等）：`AnnotationTarget.TYPE`|类或接口：`ElementType.TYPE`|
||字段：`AnnotationTarget.FIELD`|字段：`ElementType.FIELD`|
||方法：`AnnotationTarget.FUNCTION`|方法：`ElementType.METHOD`|
||构造方法：`AnnotationTarget.CONSTRUCTOR`|构造方法：`ElementType.CONSTRUCTOR`|
||方法参数：`AnnotationTarget.VALUE_PARAMETER`|方法参数：`ElementType.PARAMETER`|
||数组写法：`@Target(XXX, XXX, ···)`|数组写法：`@Target({XXX, XXX, ···})`|
||注解类：`AnnotationTarget.ANNOTATION_CLASS`|注解类：`ElementType.ANNOTATION_TYPE`|
||泛型参数：`AnnotationTarget.TYPE_PARAMETER`|泛型参数：`ElementType.TYPE_PARAMETER`|
||局部变量：`AnnotationTarget.LOCAL_VARIABLE`|局部变量：`ElementType.LOCAL_VARIABLE`|
||普通类：`AnnotationTarget.CLASS`|| 
||文件：`AnnotationTarget.FILE`||
||类别名：`AnnotationTarget.TYPEALIAS`||
||表达式：`AnnotationTarget.EXPRESSION`||
||属性：`AnnotationTarget.PROPERTY`||
||属性getter：`AnnotationTarget.PROPERTY_GETTER`||
||属性setter：`AnnotationTarget.PROPERTY_SETTER`||
|||类用途：`ElementType.TYPE_USE`|
|||包：`ElementType.PACKAGE`|
|||模块（Java 9以上可用）：`ElementType.MODULE`|
|`@Retention`|编译期：`AnnotationRetention.SOURCE`|编译期：`RetentionPolicy.SOURCE`|
||运行期：`AnnotationRetention.RUNTIME`|运行期：`RetentionPolicy.RUNTIME`|
||字节码文件：`AnnotationRetention.BINARY`|字节码文件：`RetentionPolicy.CLASS`|
|`@Repeatable`|无参数|传入一个注解类的Class，并且该注解类的`value()`方法必须返回一个数组，数组类型就是被`@Repeatable`注解的类|

### 处理注解

由于开发者自定义的注解绝大多数情况下都是运行时注解，因此这里也主要介绍如何在代码中处理自定义的运行时注解。运行时注解发挥作用的主要方式就是利用[反射](Kotlin/reflection)机制。

下面就是一个基于自定义注解实现API调用时检查请求方法的例子，来源于这篇博客[https://juejin.cn/post/6959860379652456484#heading-12](https://juejin.cn/post/6959860379652456484#heading-12)：

```
public enum class Method {
    GET,
    POST
}

@Target(AnnotationTarget.CLASS)
@Retention(AnnotationRetention.RUNTIME)
annotation class HttpMethod(val method: Method)

interface Api {
    val name: String
    val version: String
        get() = "1.0"
}

@HttpMethod(Method.GET)
class ApiGetArticles : Api {
    override val name: String
        get() = "/api.articles"
}

fun fire(api: Api) {
    // 利用反射机制获取Api接口（包括其子类）用到的注解
    val annotations = api.javaClass.annotations

    // 从注解列表中筛选出指定类型的注解
    val method = annotations.find { it is HttpMethod } as? HttpMethod

    // 调用指定类型注解的字段或函数
    println("通过注解得知该接口需要通过：${method?.method} 方式请求")
}
```