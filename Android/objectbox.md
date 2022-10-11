[ObjectBox](https://objectbox.io/)跟greenDAO师出同门，也是一个开源的数据库框架。ObjectBox从开发伊始就是为了取代greenDAO。根据Square开发团队的说法，greenDAO是基于SQLite的ORM（Object Relational Mapping，对象关系映射）数据库框架，而ObjectBox是一种NoSQL（非关系型）数据库框架。传统关系型数据库（如MySQL、Oracle以及PostgreSQL等）和新兴的非关系型数据库之间的区别，还有各自的优缺点，限于篇幅这里不做展开。

根据开发团队的说法，从greenDAO迁移至ObjectBox有以下优势：

+ **更快的执行速度**：ObjectBox采用的NoSQL方案，速度是SQLite的10倍以上；
+ **更强大的关系支持**：ObjectBox提供了更改追踪、级联放置以及灵活的加载策略；
+ **不需要掌握SQL**：ObjectBox对SQL语句进行了高度封装，简化使用方式；
+ **现代API**：简化API，提供响应式查询以及对RxJava的支持；
+ **更清晰的实体类代码**：ObjectBox不像greenDAO那样需要在工程目录下中生成代码，也不会修改开发者源文件；
+ **支持Kotlin**：支持Kotlin data class以取代原来greenDAO的Java Bean。 

从greenDAO迁移至ObjectBox的流程，可查看[ObjectBox DaoCompat](https://greenrobot.org/greendao/documentation/objectbox-compat/)相关的文档。

ObjectBox针对不同平台（Android、iOS,、Windows,乃至Linux）提供了采用不同语言开发的、但提供统一形式API的依赖库，对于Android开发而言，主要是使用[ObjectBox-Java](https://github.com/objectbox/objectbox-java)库。

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

1. 实体类中**必须**包含有Long类型的，起到ID作用的字段，且不能使用private修饰；
2. 实体类**必须**提供无参构造器，对data class而言，需要为所有字段加上默认值。

至此，ObjectBox所需的实体类已经创建完成，相关注解的含义和使用可以在[ObjectBox官方文档](https://docs.objectbox.io/entity-annotations)中做进一步了解。

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

获取指定类型的`Box`对象，以便对特定实体类所关联的数据表进行操作，比如下列代码所示的ParentEntity：

```
val parentBox: Box<ParentEntity> = boxStore.boxFor(ParentEntity::class.java)
```

`Box`对象提供的基本操作如下：

```
// 插入若干条数据，若数据已存在就进行更新
parentBox.put(ParentEntity(···), ParentEntity(···), ···)
parentBox.put(collection<ParentEntity>(···)）

// 通过指定ID获取一条数据
val parent: ParentEntity = parentBox[parentId]

// 获取所有数据
val parents: List<ParentEntity> = parentBox.all

// 根据ID移除若干条数据
parentBox.remove(Id1, Id2, ···)
parentBox.removeByIds(collection<Long>(···))

// 根据实体或实体集合移除若干条数据
parentBox.remove(parent1, parent2, ···)
parentBox.remove(collection<ParentEntity>(···))

// 移除所有数据
parentBox.removeAll()

// 获取表中数据的数量
val amount: Long = parentBox.count()
```

查询操作比较复杂，这里需要进行单独介绍。首先是获取一个`Query`对象（如下列代码所示），这个对象相当于构建了一条SQL查询语句，所以里面会有大量的跟SQL查询指令相关的方法调用，需要开发者对SQL查询语句有一定了解才能较好地运用。

```
// 以下只展示了部分常用的查询方法
val query: Query<ParentEntity> = parentBox.query(···) // 传入自定义查询条件
    .and()                                                   // 查询条件AND连接符
    .between(···)                                            // 查询指定属性位于两个值之间的数据
    .contains(···)                                           // 查询指定属性包含某种内容的数据
    .or()                                                    // 查询条件OR连接符
    .equal(···)                                              // 查询指定属性等于某个值的数据
    .notEqual(···)                                           // 查询指定属性不符合条件的数据
    .in(···)                                                 // 查询指定属性存在于指定条件中的数据
    .notIn(···)                                              // 查询指定属性不存在于指定条件中的数据
    .isNull(···)                                             // 查询指定属性为空的数据
    .notNull(···)                                            // 查询指定属性不为空的数据
    .greater(···)                                            // 查询指定属性大于某个值的数据
    .less(···)                                               // 查询指定属性小于某个值的数据
    .greaterOrEqual(···)                                     // 查询指定属性不小于某个值的数据
    .lessOrEqual(···)                                        // 查询指定属性不大于某个值的数据
    .startsWith(···)                                         // 查询指定属性以某种内容开头的数据
    .endsWith(···)                                           // 查询指定属性以某种内容结尾的数据
    .order(···)                                              // 按指定属性升序查询
    .orderDesc(···)                                          // 按指定属性降序查询
    .build()
```

上面提到的“属性”，实际上就是指数据列。实体类的属性列表在构建工程的时候由ObjectBox自行生成，跟`MyObjectBox`位于相同目录下，是一个Java文件，名称为实体类名字后面加上一个下划线。比如ParentEntity实体类对应的属性列表文件名称为“ParentEntity_”。

在构建完`Query`对象之后，就可以调用下列代码所示的常用方法获取要查询的数据：

```
// 获取所有符合查询条件的数据列表
val parents: MutableList<ParentEntity> = query.find()

// 获取符合查询条件的唯一数据，可能为空
val parent: ParentEntity? = query.findUnique()

// 获取符合查询条件的数据数量
val amount: Long = query.count()
```

**<font color=red>查询结束之后，还需要`Query`对象调用`close()`方法以结束查询，释放资源。</font>**

至此，ObjectBox基本的CRUD功能操作介绍已经全部结束。

### 异步任务

ObjectBox提供了一些可以异步执行事务的API，包括基于内建线程池的方案和基于Kotlin协程的方案。

基于内建线程池的方案有两个API，分别是`callInTxAsync()`和`runInTxAsync()`。这两个API的区别只在于**第一个**参数：`callInTxAsync()`需要传入一个**有返回值**的`Callable`对象，而`runInTxAsync()`需要传入一个**没有返回值**的`Runnable`对象。两者的使用方式大同小异：

```
boxStore.callInTxAsync({
    // 需要提供返回值
}) { result, error ->
    // 处理结果和异常
}

store.runInTxAsync({
    // 不需要提供返回值
}) { result, error ->
    // 处理结果和异常
}
```

基于Kotlin协程方案的API是`awaitCallInTx()`，这个API是一个**挂起函数**（所以要在协程作用域里面调用），需要传入一个`Callable`对象：

```
val job = GlobalScope.launch {
    try {
        val result = boxStore.awaitCallInTx {
            // 执行事务并返回结果
        }
        ···
    } catch(e: Exception) {
        ···
    }
}
```