从[Google官方文档](https://developer.android.google.cn/studio/build/shrink-code)的介绍来看，在项目中启用代码混淆只是应用代码和资源缩减工作中的一小部分，或者称之为“顺手”之举可能也不为过。事实上，代码混淆的出发点就是**缩短类和成员的名称**，从而压缩`.dex`文件的大小，下面就是Google提供的一个代码混淆的例子：

```
androidx.appcompat.app.ActionBarDrawerToggle$DelegateProvider -> a.a.a.b:
androidx.appcompat.app.AlertController -> androidx.appcompat.app.AlertController:
    android.content.Context mContext -> a
    int mListItemLayout -> O
    int mViewSpacingRight -> l
    android.widget.Button mButtonNeutral -> w
    int mMultiChoiceItemLayout -> M
    boolean mShowTitle -> P
    int mViewSpacingLeft -> j
    int mButtonPanelSideLayout -> K
```

按照Google官方的说法，虽然混淆处理不会从应用中移除代码，但如果应用的`.dex`文件将许多类、方法和字段编入索引，那么混淆处理将可以显著缩减应用的大小。不过，由于混淆处理会对代码的不同部分进行重命名，因此在执行某些任务（如检查堆栈轨迹）时需要使用额外的工具。

此外，如果项目中有代码依赖于应用的方法和类的可预测命名（例如使用反射时），开发者应该将相应的签名视为入口点，并为其指定保留规则。这些保留规则会告知`R8`编译器，除了要在应用最终的`.dex`文件中保留该代码之外，还要保留其原始命名。

## 启用代码混淆

代码混淆的启用实际上非常简单，如下面代码所示：

```
android {
    buildTypes {
        release {
            // 启用代码缩减、优化与混淆功能
            minifyEnabled true

            // 启用资源缩减，资源缩减开启时，代码缩减也必须开启
            shrinkResources true

            // 配置ProGuard规则文件，一般用默认的就行
            proguardFiles getDefaultProguardFile('proguard-android-optimizetxt'), 'proguard-rules.pro'
        }
    }
    ...
}
```

## 自定义保留代码

在大多数情况下，如要让`R8`编译器只移除用不到的代码，使用默认的ProGuard规则文件 (`proguard-android- optimize.txt`) 就已经足够。不过，在某些情况下，`R8`编译器可能会因为做出错误判断，因而从应用中移除那些实际上会被用到的代码。下面列举了几个示例，说明它在什么情况下可能会错误地移除代码：

+ 当应用通过JNI调用方法时；

+ 当应用在运行时查询代码时（如使用反射）。
  

通过测试应用应该可以发现哪些代码被错误移除而引发问题，但开发者也可以通过Android Studio生成的已移除代码报告，来检查在混淆和缩减过程中移除了哪些代码。如果需要告知`R8`编译器有哪些代码需要保留，可以采用以下两种方式：

1. 在`proguard-android- optimize.txt`文件中添加`-keep`代码行；

2. 在代码中为需要保留的类、方法或字段添加`@Keep`注解。

### 修改ProGuard规则

修改ProGuard规则通常适用于那些**开发者无法干预的代码**，比如第三方依赖库。一种最简单的保留规则如下面所示：

```
// 告诉R8编译器保留MyClass本身和它内部所有的成员
-keep public class MyClass
```

ProGuard规则常用的代码保留规则大致可分为**保留选项**、**配置选项**以及**目标**三大组成部分，通常格式为`-<保留选项>, <配置选项> <目标>`。各部分的含义和用途说明已在下面列出。更多详细内容，可以参考[ProGuard手册](https://www.guardsquare.com/manual/configuration/usage)。

+ **保留选项列表**

|保留选项|用途说明|
|:-----:|:-----:|
|`keep`|最底层的命令，可以通过灵活配置选项和目标，来指定代码中需要被保留的部分作为代码入口而不被混淆|
|`keepclassmembers`|指定不被混淆的类，同时指定其内部**所有**成员都不被混淆|
|`keepclasseswithmembers`|指定包含特定内部成员的类不被混淆，同时原样保留这些**特定的**内部成员|
|`keepnames`|`-keepnames <目标>`等效于`-keep, allowshrinking <目标>`指令，表示这些目标如果在应用缩减过程中没有被移除，那么就不进行混淆|
|`keepclassmembernames`|`-keepclassmembernames <目标>`等效于`-keepclasseswithmembers, allowshrinking <目标>`指令，表示这些类如果在应用缩减过程中没有被移除，那么就不混淆类本身和所有内部成员|
|`keepclasseswithmembernames`|`-keepclasseswithmembernames <目标>`等效于`-keepclasseswithmembernames, allowshrinking <目标>`，表示这些包含有特定内部成员的类如果在应用缩减过程中没有被移除，那么就不混淆类本身和特定的内部成员|
|`if`|使用方式为`-if <目标>`，相当于设置了一个全局通用的通配符，在该行指令之后的`-keepXXX`选项，都可以使用`<n>`来指代命令中的目标|
|`printseeds`|使用方式为`-printseeds <filename>`，用于列出所有通过`-keepXXX`选项被保留的类和类成员（包括使用通配符被包括进去的），可以将结果打印到控制台或者输出到指定目录文件当中|

+ **配置选项列表**

|配置选项|用途说明|
|:-----:|:-----:|
|`includedescriptorclasses`|用于指定任何包含有特定修饰符（如`native`）的类成员的类不被混淆|
|`includecode`|用于指定被保留方法的代码属性（code attributes）不会因为优化或混淆而发生改变，通常用于方法所属的类被优化或混淆的情形|
|`allowshrinking`|表示允许缩减那些在`-keep`选项中作为入口的代码，但是在必要的情况下也应予以保留，不进行优化或混淆|
|`allowoptimization`|表示允许**优化**那些在`-keep`选项中作为入口的代码，但是在必要的情况下也应予以保留。这个选项不常用|
|`allowobfuscation`|表示允许**混淆**那些在`-keep`选项中作为入口的代码，但是在必要的情况下也应予以保留。这个选项不常用|

+ **目标**

目标主要是指需要被保留的类和类成员。在ProGuard中，描述这些类和成员的模板比较复杂，如下面所示：

```
[@annotationtype] [[!]public|final|abstract|@ ...] [!]interface|class|enum classname
    [extends|implements [@annotationtype] classname]
[{
    [@annotationtype]
    [[!]public|private|protected|static|volatile|transient ...]
    <fields> | (fieldtype fieldname [= values]);

    [@annotationtype]
    [[!]public|private|protected|static|synchronized|native|abstract|strictfp ...]
    <methods> | <init>(argumenttype,...) | classname(argumenttype,...) | (returntype methodname(argumenttype,...) [return values]);
}]
```

在上面这份模板中，`[]`表示这些参数是可选的；`...`表示可能包含有多个参数；`|`用来分隔不同的可选参数；`()`用于突出强调紧密关联的部分。当然，如果熟悉类结构和类成员，这份模板很容易就能被理解。在ProGuard规则中，要描述类和类成员，实际上最关键的就是一些通配符。下面就是对Proguard当中一些重要的通配符进行整理汇总形成的一份表格。

|通配符|用途说明|
|:-----:|:-----:|
|`!`|严格匹配，比如ProGuard规则中class会包含interface和enum，用了`!`之后就只会匹配普通的class|
|`$`|用于标识内部类，ProGuard要求一般类名必须写全带有包名路径（比如`java.lang.String`），而内部类跟普通类又不能采用一样的格式以免混淆，因此就用`$`代替`.`|
|`?`|用于匹配类或类成员名里面的**单个字符**，**不可用于基本类型**|
|`*`|用于匹配类或类成员名中任何一部分，不包括`.`分隔符，也不包含下一级包目录里的类或类成员，**不可用于基本类型**|
|`**`|用于匹配类名中任何一部分，不包括`.`分隔符，但是包含了下一级包目录里的类或类成员，**不可用于基本类型**|
|`***`|匹配任意类型|
|`<n>`|在同一类或类成员名中，用于标识第n个被匹配到的通配符|
|`@`|用于限制类或方法的注解类型，缩小匹配范围|
|`<init>`|匹配任意构造方法|
|`<fields>`|匹配任意字段|
|`<methods>`|匹配任意方法|
|`%`|匹配任意基本类型，包括`void`|
|`...`|匹配任意数量和类型的参数|

更多官方例子，可以参考[https://www.guardsquare.com/manual/configuration/usage](https://www.guardsquare.com/manual/configuration/usage)的“Class Specifications”一节。

需要注意的是，不少大型第三方库，通常都会告知使用者在开启代码混淆的情况下使用何种指令保留它们的代码，因此一般按照这些第三方库的指导修改ProGuard规则文件即可。

### 使用`@Keep`注解

`@Keep`注解主要用在开发者可以高度掌控代码的项目中，其作用就如同它的字面意思一样，让类或类成员保持不变，如下面代码所示：

```
@Keep
public void foo() {
    ...
}
```

需要注意的是，`@Keep`注解的使用是有限制的。Google官方提醒开发者，只有在使用AndroidX注解库（某些第三方库也可能有这个注解，但作用完全不同）并且添加有AGP随附的ProGuard规则文件时，此注解才可用；另外，这个注解通常会用在只有通过反射才能访问的类或方法上，因为反射调用会让编译器误以为它们没有被用到；最后，不要在依赖库的代码中使用该注解。因为代码缩减往往需要移除依赖库中用不到的部分代码，而开发者并不总是能确定哪些代码会被移除，随意使用该注解可能会降低代码缩减的效果。Google建议开发者最好是修改ProGuard规则文件。

## 还原堆栈轨迹

经过`R8`编译器处理的代码会发生各种更改，这可能会使得应用运行时得堆栈轨迹难以理解，因为堆栈轨迹与源代码不完全一致。如果未保留调试信息，就可能会出现由于内联等代码优化手段导致行号更改和代码定位不准确的情况。但是，影响堆栈轨迹最大的因素是混淆处理，因为正如前面所说，进行混淆处理时，类和类成员的名称会被更改。

要还原堆栈轨迹，开发者可以使用`R8`编译器提供的`retrace`命令行工具，将编译时生成的映射文件传入命令当中并执行，系统会将经过轨迹还原的堆栈结果写入到标准输出文件当中。具体可以参考[Google官方文档](https://developer.android.google.cn/studio/command-line/retrace)。

如需支持对应用的堆栈轨迹进行还原，开发者应通过向模块级`proguard-rules.pro`文件添加以下规则，来确保项目构建时保留足够的信息以进行轨迹还原：

```
-keepattributes LineNumberTable,SourceFile
-renamesourcefileattribute SourceFile
```

`LineNumberTable`属性会在方法中保留位置信息，以便以堆栈轨迹的形式输出这些位置。`SourceFile`属性可确保所有可能的运行时都实际输出位置信息。`-renamesourcefileattribute`指令用于将堆栈轨迹中的源文件名称设置为仅包含 `SourceFile`。在轨迹还原过程中不需要实际的原始源文件名称，因为映射文件中已经包含有原始源文件。

`R8`编译器每次运行时都会创建一个`mapping.txt`文件，其中包含将堆栈轨迹重新映射为原始堆栈轨迹所需的信息。Android Studio会将该文件保存在`<module-name>/build/outputs/mapping/<build-type>/`目录中。此外，项目每次构建时都会覆盖掉Android Studio在上一次构建过程中生成的`mapping.txt`文件，因此每次发布新版本时都要注意保存一个该文件的副本，以便在面对旧版应用经过混淆处理的堆栈轨迹时，还能比较方便地进行轨迹还原。