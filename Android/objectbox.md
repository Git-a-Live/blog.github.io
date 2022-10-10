[ObjectBox](https://objectbox.io/)跟greenDAO师出同门，也是一个开源的数据库框架。ObjectBox从开发伊始就是为了取代greenDAO。根据Square开发团队的说法，greenDAO是基于SQLite的ORM（Object Relational Mapping，对象关系映射）数据库框架，而ObjectBox是一种NoSQL（非关系型）数据库框架。传统关系型数据库（如MySQL、Oracle以及PostgreSQL等）和新兴的非关系型数据库之间的区别，还有各自的优缺点，限于篇幅这里不做展开，可以在网络上自行查阅相关资料。

ObjectBox针对不同平台（Android、iOS,、Windows,乃至Linux）提供了采用不同语言开发的、但提供统一形式API的依赖库，对于Android开发而言，主要是使用[ObjectBox-Java](https://github.com/objectbox/objectbox-java)库。和greenDAO不同，ObjectBox-Java不仅支持Java Bean，还支持Kotlin的data class，如果项目主要采用Kotlin开发，使用ObjectBox无疑是更为合适的。

> ObjectBox的数据库文件格式不是以往的`.db`，而是`.mdb`；并且默认存放在应用专属存储的`files/objectbox/objectbox`目录下。

## 前置工作

ObjectBox的导入和集成跟greenDAO类似。首先要在项目级build.gradle文件中添加以下配置：

```
buildscript {
    dependencies {
        classpath 'io.objectbox:objectbox-gradle-plugin:$specified_version'
    }
}
```

接着在模块级build.gradle文件中添加以下配置：

```
// 导入ObjectBox插件
plugins {
    id 'io.objectbox'
}
```

注意，ObjectBox不需要添加额外的依赖库和编译任务，执行到上一步应用插件就已经完成全部的集成工作。

## 基本使用

### 创建实体类

以下示例代码创建了一个名为ParentEntity的实体类：

```
@Entity
data class ParentEntity(
    @Id
    var parentId: Long = 0,
    var name: String = "",
    var age: Int = 0
)
```

ObjectBox对于实体类的要求如下：

1. 实体类中**必须**包含有Long类型的，起到ID作用的字段；
2. 实体类**必须**提供无参构造器，对data class而言，需要为所有字段加上默认值。
3. 实体类中的字段<font color=red>不能为private</font>，否则会因为字段不可见导致编译失败。

至此，ObjectBox所需的实体类已经创建完成，相关注解会在下文进行详细介绍。

### 工程构建

和greenDAO一样，ObjectBox也需要通过相同步骤完成工程构建，才能生成数据库功能所需的文件，尤其是`MyObjectBox`这个类。`MyObjectBox`跟`BuildConfig`很相似，都是在工程编译之后才会生成。它的作用类似于greenDAO的`DaoMaster`和`DaoSeesion`的结合体，后续的功能调用也是基于`MyObjectBox`来展开。

### CRUD功能调用

首先进行初始化，获取到一个`BoxStore`对象：

```
val boxStore = MyObjectBox.builder()
            .androidContext(context) // 传入上下文
            .name(···)               // 设置数据库文件的存放文件夹
            .baseDirectory(···)      // 设置数据库文件夹所在路径，不能和directory()一起使用
            .directory(···)          // 需传入数据库文件的完整路径名，不能跟name()或baseDirectory()一起使用
            .fileMode(···)           // 设置数据库文件的权限属性
            .maxSizeInKByte(···)     // 设置数据库文件的最大体积，以KB为单位
            .initialDbFile(···)      // 如果有现成的数据库文件用于初始化就调用
            ...
            .build()
```

## ObjectBox常用注解说明

## 从greenDAO迁移至ObjectBox