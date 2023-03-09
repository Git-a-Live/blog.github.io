## 何为Gradle

[Gradle](https://docs.gradle.org/current/userguide/what_is_gradle.html)是一种项目构建工具，用于管理项目编译构建所需的依赖，并通过运行一系列构建任务，生成开发者需要的目标格式的最终产物，比如`.apk`或`.jar`文件。Gradle支持使用基于Java、[Groovy](https://groovy-lang.org/documentation.html)以及Kotlin语言开发的[DSL](/Kotlin/func?id=dsl)。如果使用的是Groovy，那么构建用的脚本文件以`.gradle`结尾；如果使用的是Kotlin DSL，那么脚本文件以`.kts`结尾。事实上，无论采用何种语言，Gradle的基本功能都是一样的，这里出于习惯都用Groovy来描述。

> 如果想了解Groovy和Kotlin DSL编写的Gradle文件在语法上有哪些差异，可以查看
[Google官方文档](https://developer.android.google.cn/studio/build/migrate-to-kts#apply-plugins)以及[Gradle官方文档](https://docs.gradle.org/current/userguide/migrating_from_groovy_to_kotlin_dsl.html)。

目前Gradle在Android Studio和Intellij IDEA等IDE上都有集成使用。通常Java项目多会采用[Maven](https://maven.apache.org/)作为项目管理与构建工具，而Android项目则使用Gradle。一些早期的项目可能还会用[Ant](https://ant.apache.org/)。当然，随着时间的推移，越来越多的Java项目也开始使用Gradle。Gradle相对于Maven和Ant的优势主要在于以下几点：

1. 使用DSL而不是XML，来表述构建过程与依赖关系，定义简洁，使用简单；

2. 支持多种类型项目的使用，最典型的就是Android项目和Java项目都能支持；

3. 在灵活性和统一性上比较平衡，既能自定义业务逻辑，又能确保依赖和构建过程的统一和标准化；

4. Gradle支持增量构建，即可以只运行新增的构建任务，这样就能大幅缩减构建时间。

Gradle中有两个非常重要的概念：**Project**和**Task**。

Project是指**构建产物**（比如`.jar`包和`.apk`文件）或**实施产物**（将应用程序部署到生产环境），通常由一些组件组成，如一个Project可以代表一个`.jar`包、`.aar`包或者一个WEB应用程序，也可能包含其他项目生成的`.jar`包或`.aar`包。

Task是指**不可分割的最小工作单元**，用于执行构建工作（比如编译项目或执行测试）。Task可以是编译一些Java类，或者创建一个`.jar`包，或者是生成JavaDoc，或者是发布文档到仓库。总之是作为<u><font color=red>原子操作</font></u>一类的存在。

由此可以得到如下关系：

· 每个Gradle build由若干个Project组成；

· 每个Project由若干个Task组成。

具体到Android项目来说，每一个待构建的Module是一个Project，构建一个Project需要执行一系列Task，比如编译、打包这些构建过程的子过程都对应着一个Task。一个apk文件的构建包含以下Task：源码编译、资源文件编译、Lint检查、打包以生成最终的`.apk`文件等等。在通常情况下，IDE在项目创建时提供的Gradle构建脚本文件就能满足开发者基本的构建管理需要，但是如果想让Gradle在构建过程中执行一些开发者自定义的构建逻辑，就需要了解如何编写Task。

## Gradle工作流程

如下图所示，Gradle的工作流程可划分为初始化、配置和执行三大阶段。

![](pics/gradle1.png)

### 初始化阶段

初始化阶段的主要任务是创建项目的层次结构，为每一个module创建一个Project对象，对应操作就是执行`setting.gradle`里面的配置。一个`setting.gradle`文件对应一个setting对象，在`setting.gradle`中可以直接调用其中的方法。`setting.gradle`文件可调用的API详情可参考[Gradle官方文档](https://docs.gradle.org/current/javadoc/org/gradle/api/initialization/Settings.html)。

### 配置阶段

配置阶段的主要任务是配置每个Project中的`build.gradle`文件，每个`build.gradle`文件对应一个Project对象。Project对象能够调用的API可以参考[官方文档内容](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html)。配置阶段执行的代码包括`build.gralde`文件中的各种语句、闭包以及Task中的配置段语句。需要注意的是，无论执行何种Gradle命令，初始化阶段和配置阶段的代码都会被执行，在排查构建速度问题的时候可以留意，是否可以将部分代码写成Task，从而减少配置阶段消耗的时间。

### 执行阶段

最后一个阶段就是执行阶段，这一阶段的主要是执行`build.gradle`文件里面所定义的Task。在配置阶段结束后，Gradle会根据任务Task的依赖关系创建一个有向无环图，可以通过Gradle对象的`getTaskGraph`方法访问，对应的类为`TaskExecutionGraph`（API详情可参考[Gradle官方文档](https://docs.gradle.org/current/javadoc/org/gradle/api/execution/TaskExecutionGraph.html)），然后通过调用`gradle <任务名>`执行对应任务。如前面阶段图所示，这里也可以加Hook，当任务执行完之后可以通过Hook来做一些自定义的事情。

## Gradle使用方式

如果使用Android Studio进行开发，那么在创建Android项目的时候，Android Studio就会自动创建好Gradle相关的工具目录和构建脚本；或者在Intellij IDEA中创建项目时，选用Gradle作为构建工具，Intellij IDEA也同样会准备好这些东西。无论是Android项目还是一般的JVM项目，只要使用Gradle这个构建系统，那么很多东西基本上是通用的。

### Gradle文件

下表是一般项目中所涉及到的比较重要的Gradle构建系统文件：

|文件名|用途说明|
|:-----:|:-----:|
|`gradle/wrapper/gradle-wrapper.jar`|用于执行下载Gradle发行版任务的程序|
|`gradle/wrapper/gradle-wrapper.properties`|一个属性文件，负责配置Gradle的下载源和本地存储位置|
|`gradlew`|一个shell脚本，用于通过Wrapper程序执行构建任务|
|`gradlew.bat`|一个Windows批处理脚本，作用同上|
|`settings.gradle`|Gradle 7.0以前仅用于指示项目中包含多少个module，Gradle 7.0开始增加了配置仓库地址的功能|
|`gradle.properties`|为Gradle构建系统配置一些全局属性，默认使用String类型|
|`build.gradle`|Gradle 7.0以前主要负责配置编译时要用到的仓库地址、全局性插件以及一些只在`module/build.gradle`中使用到的属性，Gradle 7.0开始将配置仓库地址迁移到`settings.gradle`|
|`module/build.gradle`|负责配置模块使用到的插件和依赖库，还有项目编译时要设置的属性选项以及要执行的任务等等|
|`local.properties`|为Gradle构建系统配置本地环境属性|

### Gradle构建流程监听

Gradle提供了非常多的Hook点供开发人员修改构建过程中的行为，为了方便说明，先看下面这张图：

![](pics/gradle2.png)

Gradle在构建的各个阶段都提供了很多回调，开发者在添加对应监听时务必要注意，**监听器一定要在回调的生命周期之前添加**，否则过了某个生命周期就不会执行。一种最简单的监听方式，就是在`settings.gradle`文件中新增一个构建流程监听器，参考下面的示例代码：

```
// 在settings.gradle文件中执行监听任务
gradle.addBuildListener(new BuildListener() {
    @Override
    void settingsEvaluated(Settings settings) {
        ···
    }

    @Override
    void projectsLoaded(Gradle gradle) {
        ···
    }

    @Override
    void projectsEvaluated(Gradle gradle) {
        ···
    }

    @Override
    void buildFinished(BuildResult buildResult) {
        ···
    }
})
```

当然，从上面的示例代码中可以看到，在`settings.gradle`文件添加一个监听回调，似乎还是太简单了，只覆盖到几个生命周期。如果还想监听别的生命周期，那就需要考虑在项目级`build.gradle`直接调用Project对象提供的生命周期监听API，来执行相应的Hook操作。例如，想要为所有module应用Java插件，那么就要在项目级`build.gradle`文件中加入以下代码：

```
// 通过Gradle对象直接设置监听动作
gradle.beforeProject { project ->
    apply plugin: 'java'
}

// 等效写法
allprojects {
    beforeEvaluate { project ->
        apply plugin: 'java'
    }
}
```

> 注意，在上面的示例代码中，`gradle.beforeProject`和`gradle.beforeEvaluate`的执行时机是一样的，但前者可以应用于所有项目，而后者只应用于当前被调用的Project对象。

### Gradle仓库配置

Gradle的核心功能中就包含了依赖管理，而通常情况下，大多数依赖库是从远程仓库拉取到本地的，因此有必要了解如何在Gradle中配置仓库地址。

在Gradle 7.0以前，仓库配置是放在项目级`build.gradle`文件当中的，如下列代码所示：

```
buildscript {
    repositories {
        google()
        mavenCentral()
        ···
    }
    dependencies {
        ···
    }
}
allprojects {
    repositories {
        google()
        mavenCentral()
        ···
    }
}
```

但是从Gradle 7.0开始，仓库配置就转移到`settings.gradle`:

```
pluginManagement {
    // 此处配置的仓库地址，主要用于拉取Gradle插件
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
        ···
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    // 此处配置的仓库地址，主要用于项目所有module拉取依赖库
    repositories {
        google()
        mavenCentral()
        ···
    }
}
```

由于GFW的干扰，Google、Maven以及Gradle的官方仓库并不总是能被中国大陆的开发者们正常访问，因此有时候就需要用到下面列出的一些镜像地址：

|镜像名称|镜像地址|
|:-----:|:-----:|
|阿里云Maven中央仓库镜像|https://maven.aliyun.com/repository/central|
|阿里云JCenter仓库镜像|https://maven.aliyun.com/repository/public|
|阿里云Maven公共仓库镜像|https://maven.aliyun.com/repository/public|
|阿里云Google Maven仓库镜像|https://maven.aliyun.com/repository/google|
|阿里云Gradle插件仓库镜像|https://maven.aliyun.com/repository/gradle-plugin|

对于一些只提供`.jar`、`.aar`或`.so`格式二进制文件的依赖，由于没有远程仓库能解析，因此通常采用下面这种方式导入依赖：

```
implementation fileTree(dir: 'libs', include: ['*.jar','*.aar'])
```

如果是`.so`文件，那么只要把文件放到默认路径`src/main/jniLibs`下面就可以被自动识别出来，不用进行额外配置。有部分开发者可能不喜欢用默认路径，他们通常会采用类似下面的方式去修改`.so`文件的路径：

```
android{
   sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```

#### 依赖冲突

依赖冲突是指在一个项目中，由于不同module或依赖库都引用了同一个依赖项，但属于不同版本，导致Gradle编译时不一定能按照开发者的意愿，确定到底用哪个版本的依赖项的情况。

当发生依赖冲突的时候，Gradle构建系统会自动选择发生冲突的依赖的最新版本。如果开发者也倾向于使用最新版本，且module或依赖库中未用到新旧版本互不兼容的变更内容，那么就随意了。如果不想用最新版本，那么就要参考以下几种方式来解决依赖冲突。

+ **强制指定版本**

强制指定依赖版本的操作，通常有以下两种：

```
// strictly配置
dependencies {
    implementation 'org.apache.httpcomponents:httpclient:4.5.4'
    implementation('commons-codec:commons-codec') {
        version {
            strictly '1.9'
        }
    }
}

// resolutionStrategy.force配置
android {
    ···
    configurations {
        compileClasspath {
            resolutionStrategy.force 'commons-codec:commons-codec:1.9'
        }
    }
}

dependencies {
    implementation 'org.apache.httpcomponents:httpclient:4.5.4'
}
```

如果还想了解更多限制依赖版本的方法，可以参考https://docs.gradle.org/current/userguide/rich_versions.html#sec:strict-version

+ **排除特定依赖**

排除特定依赖的语法参考下面的示例代码：

```
dependencies {
    implementation('commons-beanutils:commons-beanutils:1.9.4') {
        // 要排除多少个依赖，就执行多少个类似的语句
        exclude group: 'commons-collections', module: 'commons-collections'
    }
}
```

这个方式适用于远程依赖，因为远程依赖当中用到的一些库版本通常是无法被使用者所干涉的。并且在一般情况下，这些“被依赖的依赖”还是默认可被传递到项目当中的。**如果Gradle构建系统由于存在这些依赖而不能正常完成编译，且开发者确定排除它们也能让项目正常运行，那么就采用这种方式来解决依赖冲突问题**。

更多关于排除依赖的资料，可以参考https://docs.gradle.org/current/userguide/dependency_downgrade_and_exclude.html#sec:excluding-transitive-deps

+ **禁止依赖传递**

禁止依赖传递可以参考下面的示例代码：

```
dependencies {
    ···
    implementation("io.coil-kt:coil-compose:2.2.2") {
        transitive = false
    }
}
```

由于依赖都是默认可传递的，也就是说如果A使用了依赖项B，而B使用了依赖项C，那么A就可以调用到C。假设C取消了可传递性，那么它只能在B中使用。这种禁止依赖传递的方式，适用于开发者对项目所使用的依赖库掌控程度较高的情形。

> 注意，在某个module中使用`implementation`导入的依赖，实际上已经不能传递使用了，想要能够传递，得调用`api`来取代`implementation`完成依赖导入。

+ **调用阶段隔离**


调用阶段隔离适用于这样一种场景：项目引用的`.aar`依赖库中包含有特定的`.jar`依赖，而项目本身又直接引用了这个`.jar`文件，在编译时就会不可避免地发生冲突。调用阶段隔离就是指让这两个发生冲突的`.jar`依赖，分别使用不同的导入语句，比如一个用`compileOnly`（编译时有效，不参与打包），另一个用`implementation`，这样它们的调用阶段就直接分离了，不会再发生冲突。类似的语句还有`runtimeOnly`（打包时有效，不参与编译）。

### Gradle插件配置

Gradle本身只是提供了基本的核心功能，其他的特性比如编译Java源码的能力，或是编译Android工程的能力等等就需要通过插件来实现了。在Gradle中一般有两种类型的插件，分别叫做**脚本插件**和**二进制插件**。脚本插件是一种额外的构建脚本，它会进一步配置构建，可以把它理解为一个普通的`build.gradle`；而二进制插件就是指已经由官方或第三方实现了`org,gradle.api.plugins`接口，并发布成二进制文件的插件。

#### Gradle插件的应用

项目要应用Gradle插件，通常只在项目级`build.gradle`和模块级`build.gradle`当中执行相应的代码。在Gradle 7.0以前，配置插件的代码如下面所示：

```
// 项目级build.gradle文件
buildscript {
    repositories {
        ···
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:X.Y.Z'
        ···
    }
}

// 模块级build.gradle文件
apply plugin: 'com.android.application'
```

Gradle 7.0开始，配置插件的代码就成了下面这种的：

```
// 项目级build.gradle文件
plugins {
    id 'com.android.application' version '7.4.2' apply false
    id 'com.android.library' version '7.4.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.7.0' apply false
    ···
}

// 模块级build.gradle文件
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}
```

可以发现，在应用插件的代码中，有时候会采用`id «plugin id»`的格式，有时候又会采用`id «plugin id» version «plugin version» [apply «false»]`格式，那么这两种格式的差别是什么？按照[Gradle官方文档](https://docs.gradle.org/current/userguide/plugins.html#sec:constrained_syntax)的说法，前一种格式用在核心Gradle插件（core Gradle plugins），或是构建脚本中已经存在的插件（比如上面示例代码中的`com.android.application`插件）。后一种格式，用在那些需要从远程仓库解析拉取下来的二进制插件。此外，`apply`语句后面的布尔值，作用是阻止项目**立即应用**该插件的默认行为（比如开发者只想在特定module构建时才调用该插件）。

更多有关Gradle插件的详细介绍，可以参考Gradle官方文档：https://docs.gradle.org/current/userguide/plugins.html

### 创建Gradle Task

#### 为指定Gradle Task动态添加Action

#### 动态改变Gradle Task依赖关系