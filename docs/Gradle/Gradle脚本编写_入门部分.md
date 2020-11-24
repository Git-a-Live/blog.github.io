## 一、概述

Gradle脚本是一种基于Groovy语言的脚本，常用在项目构建当中。

Gradle有两个重要的概念：*Project* 和 *Task*。

*Project* 是指**构建产物**（比如Jar包）或**实施产物**（将应用程序部署到生产环境），通常由一些组件组成，如一个Project可以代表一个JAR库或者一个WEB应用程序，也可能包含其他项目生成的JAR包。

*Task* 是指**不可分割的最小工作单元**，用于执行构建工作（比如编译项目或执行测试）。Task可以是编译一些Java类，或者创建一个JAR包，或者是生成JavaDoc，或者是发布文档到仓库。总之是作为<u><font color=red>原子操作</font></u>一类的存在。

由此可以得到如下关系：

· 每个gradle build由若干个Project组成；

· 每个Project由若干个Task组成。

在使用命令行工具通过`gradle`命令运行Gradle脚本时，该命令首先会在当前目录寻找`build.gradle`文件，这个文件是一个**构建配置脚本**（build configuration script），它定义了一个project和若干个task。

## 二、Task快速入门

### 声明与运行

#### 静态声明

常见的Task声明方式如下：

```
task taskName {
    //TODO
}
```

#### 动态声明

上述方式是一种静态的声明方式，事实上Gradle还支持task的动态声明：

```
task "${taskname}" {
    //TODO
}
```

将上面的语句包含在某个project或是task里面，在运行的时候，这个task就会被动态地声明（赋予task名称），同样也可以动态地调用（甚至在调用时这个task可能还没被声明）。

#### Task的运行

在IDEA和Android Studio中，当正确声明task之后，task的旁边会显示一个运行按钮，表示目前这个task可以直接运行。而编写完task里面的执行语句之后，在命令行工具中使用`gradle -q taskName`即可运行该task，执行里面的语句并输出结果。

> *-q* 用于屏蔽gradle命令运行过程中可能打印出的日志，减少信息干扰。

#### 内部语句

正如前面所述，Gradle是基于Groovy语言的脚本，因此task内部的执行语句通常会使用Groovy语言，一般只要编写正确的Groovy语句就可以运行。

许多教程会在task名称后面加上`<<`以替代`doLast{}`的使用，然而这是过时的做法（更不要说在Groovy语言中`<<`有自己的用法），<u><font color=red>Gradle 5.0以后已经废除了这种用法</font></u>，推荐用法如下：

```
task taskName {
    dependsOn(someTask)

    doLast{
        //TODO
    }

    doFirst {
        //TODO
    }
}
```

`doLast{}`表示被包裹的执行语句**最后**执行，`doFirst{}`则与之相反，表示被包裹的执行语句**首先**执行。如果使用了`dependsOn`语句，则先执行`dependsOn`<u>**所指定的task**</u>，然后再执行`doFirst{}`。注意：<u>若在一个task里面定义了多个`doLast{}`和`doFirst{}`，可能会出现不可预测的语句执行顺序，因此要合理使用。</u>

此外，无论是`doLast{}`、`doFirst{}`还是`dependsOn()`，它们都可以通过`taskName.doLast{}`、`taskName.doFirst{}`以及`taskName.dependsOn()`的形式被**覆写**，而且不会影响task内已经定义过的部分。

### 属性与方法

#### 本地属性与外部属性

在Gradle中，属性分为本地和外部两种类型，它们在Task里面的声明方式如下：

```
task taskName {
    //本地属性:
    def localProp = value

    //外部属性:
    ext.externalProp = value
}
```

本地属性类似于Java中被`protected`或`private`所修饰的字段，不能直接获取；而外部属性则类似于被`public`所修饰的Java字段，可以直接通过点运算符`.`获取。

#### 方法

由于使用了Groovy语言，因此在Gradle脚本中，声明方法的合法方式有很多种，比如省略了修饰符的Java方式：

```
ReturnType method(ParamType1 param1,···) {
    //TODO
}
```

Groovy默认方式：

```
def method(param1,···) {
    //TODO
    return value
}
```

混杂方式：

```
ReturnType method(param1,···) {
    //TODO
}
```

这些方法的调用也比较简单，直接输入方法名并传入参数即可。

### 默认Task

默认Task是指在命令行工具中执行`gradle`命令但不显示指定Task名称的情况下，Gradle可自行找到并执行的一类Task。它们的声明和内部语句编写与普通的Task没有区别，只是需要在脚本当中以下面的方式将其声明为默认Task：

```
defaulTaks('taskName1','taskName2',···)
```



