## 何为Gradle

[Gradle](https://docs.gradle.org/current/userguide/what_is_gradle.html)是一种项目构建工具，用于管理项目编译构建所需的依赖，并通过运行一系列构建任务，生成开发者需要的目标格式的最终产物，比如`.apk`或`.jar`文件。Gradle支持使用基于Java、[Groovy](https://groovy-lang.org/documentation.html)以及Kotlin语言开发的[DSL](/Kotlin/func?id=dsl)。如果使用的是Groovy，那么构建用的脚本文件以`.gradle`结尾；如果使用的是Kotlin DSL，那么脚本文件以`.kts`结尾。

目前Gradle在Android Studio和Intellij IDEA等IDE上都有集成使用。通常Java项目多会采用[Maven](https://maven.apache.org/)作为项目管理与构建工具，而Android项目则使用Gradle。一些早期的项目可能还会用[Ant](https://ant.apache.org/)。当然，随着时间的推移，越来越多的Java项目也开始使用Gradle。Gradle相对于Maven和Ant的优势主要在于以下几点：

1. 使用DSL而不是XML，来表述构建过程与依赖关系，定义简洁，使用简单；

2. 支持多种类型项目的使用，最典型的就是Android项目和Java项目都能支持；

3. 在灵活性和统一性上比较平衡，既能自定义业务逻辑，又能确保依赖和构建过程的统一和标准化；

4. Gradle支持增量构建，即可以只运行新增的构建任务，这样就能大幅缩减构建时间。

## 基本概念

Gradle中有两个非常重要的概念：**Project**和**Task**。

*Project* 是指**构建产物**（比如`.jar`包和`.apk`文件）或**实施产物**（将应用程序部署到生产环境），通常由一些组件组成，如一个Project可以代表一个`.jar`包、`.aar`包或者一个WEB应用程序，也可能包含其他项目生成的`.jar`包或`.aar`包。

*Task* 是指**不可分割的最小工作单元**，用于执行构建工作（比如编译项目或执行测试）。Task可以是编译一些Java类，或者创建一个`.jar`包，或者是生成JavaDoc，或者是发布文档到仓库。总之是作为<u><font color=red>原子操作</font></u>一类的存在。

由此可以得到如下关系：

· 每个gradle build由若干个*Project*组成；

· 每个*Project*由若干个*Task*组成。

具体到Android项目来说，每一个待构建的Module是一个Project，构建一个Project需要执行一系列Task，比如编译、打包这些构建过程的子过程都对应着一个Task。一个apk文件的构建包含以下Task：源码编译、资源文件编译、Lint检查、打包以生成最终的`.apk`文件等等。在通常情况下，IDE在项目创建时提供的Gradle构建脚本文件就能满足开发者基本的构建管理需要，但是如果想让Gradle在构建过程中执行一些开发者自定义的构建逻辑，就需要了解如何编写Task。

## 使用方式

如果使用Android Studio进行开发，那么在创建Android项目的时候IDE就会自动创建好Gradle相关的工具目录和构建脚本；或者在Intellij IDEA中创建项目时，选用Gradle作为构建工具，IDE也同样会准备好这些东西。下面就分别从Android项目和普通Java项目的角度，来了解Gradle的使用。

### Android项目中的Gradle

Android项目的Gradle工具由这几部分构成：1）`build.gradle`脚本；2）`settings.gradle`脚本；3）`gradle-wrapper.properties`等配置和执行文件。这里主要关注以下三个文件：

+ **`gradle.properties`配置文件**

这个文件中定义了一系列供`build.gradle`脚本使用的常量，比如keystore的存储路径、keyalias等等。这个文件有两份，在Android Studio的Android视图下，可以看到一份标注为“Global Properties”，另一份则标注为“Project Properties”。其中，标注为“Project Properties”的`gradle.properties`文件是比较常用的，通常定义配置也是在这份文件里进行。下面是某项目中的`gradle.properties`文件内容示例：

```
# Project-wide Gradle settings.
# IDE (e.g. Android Studio) users:
# Gradle settings configured through the IDE *will override*
# any settings specified in this file.
# For more details on how to configure your build environment visit
# http://www.gradle.org/docs/current/userguide/build_environment.html
# Specifies the JVM arguments used for the daemon process.
# The setting is particularly useful for tweaking memory settings.
org.gradle.jvmargs=-Xmx1536m
# When configured, Gradle will run in incubating parallel mode.
# This option should only be used with decoupled projects. More details, visit
# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
# org.gradle.parallel=true
android.enableD8.desugaring=true
android.useAndroidX=true
android.enableJetifier=true
```

可以看到，这份文件的配置就是简单的`key = value`，但是key具体有哪些，可以参考Gradle官网的[介绍](https://docs.gradle.org/current/userguide/build_environment.html)。另外要注意的是，除非开发者明确理解相关配置的含义以及修改它们所带来的影响，否则不要随意修改文件中的各项配置。

+ **`settings.gradle`脚本**

`settings.gradle`脚本主要用于指示项目中包含有哪些[Module](/Android/mod)，以便在构建的时候需要把包含的Module全部打包进去。下面就是某项目中的`settings.gradle`脚本内容示例：

```
include ':app'
include ':base'
include ':sub'
include ':config'
```



+ **`build.gradle`脚本**

### Java项目中的Gradle