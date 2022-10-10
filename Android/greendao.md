[greenDAO](https://greenrobot.org/greendao/)是Square公司（没错，还是那个开发了Retrofit等众多开源项目的公司）开发的一个基于SQLite的开源数据库框架。在Google推出[Jetpack Room](Android/room.md)以及Square推出[ObjectBox](https://greenrobot.org/greendao/documentation/objectbox-compat/)取代greenDAO之前，greenDAO基本上可以认为是Android开发当中数据库框架的不二选择。

随着Google Jetpack Room的推广普及和更新迭代，以及Square开发团队将开发维护重心逐渐转移到更新的ObjectBox框架上，greenDAO在未来会慢慢被淘汰掉，加之其不支持Kotlin的实体类（data class），因此这里只介绍greenDAO的基本用法。

## 前置工作

greenDAO的导入集成方式跟其他依赖库相比有些复杂。 首先要在项目级的build.gradle文件中添加如下配置：

```
buildscript {
    dependencies {
        classpath 'org.greenrobot:greendao-gradle-plugin:3.3.0'
    }
}
```

接着在模块级的build.gradle文件中添加如下配置：

```
// 导入greenDAO插件
plugins {
    id 'org.greenrobot.greendao'
}

// 导入依赖库
dependencies {
    // 截至2022年10月，最新版本为2020年5月4日发布的3.3.0
    implementation 'org.greenrobot:greendao:3.3.0'
}

// greenDAO独立配置
greendao {
    // 数据库版本号，每次调整数据库实体类（表）都要更新该值
    schemaVersion 1
    // 自动生成的数据库相关类的package名
    daoPackage 'com.example.app.greendao'
    // 设置DaoMaster、DaoSession、Dao目录
    targetGenDir 'src/main/java'
}
```

至此，greenDAO集成的前置工作已经完成。

## 基本使用

### 创建实体类（Java Bean）

以下示例代码创建了一个名为StudentBean的实体类：

```
import org.greenrobot.greendao.annotation.Entity;
import org.greenrobot.greendao.annotation.Id;

@Entity
public class StudentBean {
    @Id(autoincrement = true)
    Long studentId;
    String name;
    int age;
    String gender;
}
```

可以注意到，StudentBean被添加了一个`@Entity`的注解，该注解跟Jetpack Room中的`@Entity`注解作用类似，都是为了让数据库框架组件能够识别出需要转化成数据库表的实体类。StudentBean当中的studentId字段，被添加了`@Id`的注解，详细作用会在下文进行介绍。

### 工程构建

创建完实体类之后，在Android Studio的菜单栏中找到`Build`-`Make Project`并点击执行，等待一段时间之后，就会发现项目当中多出了一些内容。

首先，在`src/main/java`目录下会生成一个名为greendao的文件夹（包），这个是“前置工作”一节当中，开发者在模块级build.gradle文件中配置的`greendao`编译任务生成的。打开这个文件夹，会发现里面会生成三个文件：`DaoMaster`、`DaoSession`以及`StudentBeanDao`（如果有多个实体类，会一一生成对应的XXXDao类文件）。这些文件是由greenDAO插件自动生成的，因此开发者不需要也不应该去随意编辑修改它们。

`DaoMaster`保存数据库对象（SQLiteDatabase）并管理特定模式的Dao类。它具有静态方法来创建表或将它们删除。其内部类`OpenHelper`和`DevOpenHelper`是在SQLite数据库中创建模式的`SQLiteOpenHelper`实现。

`DaoSession`管理特定模式的所有可用Dao对象，可以使用其中一个getter方法获取。DaoSession还为实体提供了一些通用的持久性方法，如插入，加载，更新，刷新和删除。最后，DaoSession对象也跟踪一个身份范围。

`StudentBeanDao`负责提供对应数据库表的CRUD功能。

简而言之，`DaoMaster`主要负责数据库的创建和表的管理，`DaoSession`主要负责提供实体类对应的Dao对象，具体的表操作，即CRUD，则由实体类对应的Dao对象负责。

经过构建之后，原来的实体类也发生了一些变化，比如新增了无参和有参这两种构造方法以及属性的getter和setter方法，类似于下列代码所示：

```
@Entity
public class StudentBean {
    @Id(autoincrement = true)
    Long studentId;
    String name;
    int age;
    String gender;

    @Generated(hash = 703692370)
    public StudentBean(Long studentId, String name, int age, String gender) {
        this.studentId = studentId;
        this.name = name;
        this.age = age;
        this.gender = gender;
    }

    @Generated(hash = 2097171990)
    public StudentBean() {
    }

    // 自动生成的getter和setter省略
    ···
}
```

### CRUD功能调用

通常情况下，greenDAO数据库框架的功能调用，就是围绕`DaoMaster`、`DaoSession`以及Dao对象来执行一系列操作。

首先是初始化数据库：

```
// 创建数据库和表
val devOpenHelper = DaoMaster.DevOpenHelper(context, "xxx.db")
// 完成表注册
val daoMaster = DaoMaster(devOpenHelper.writableDatabase)
// 获取DaoSession对象
val daoSession = daoMaster.newSession()
```

> 注意，上述操作可以进行统一封装，最后对外提供一个全局唯一的DaoSession单例。

通过DaoSession对象获取到**指定**的Dao对象之后，就可以对该Dao对象所关联的数据表执行基本的CRUD操作了：

```
// 向表中插入或替换一条数据
daoSession.studentBeanDao.insertOrReplace(···)

// 向表中插入或替换多条数据
daoSession.studentBeanDao.insertOrReplaceInTx(···)

// 更新表中一条数据
daoSession.studentBeanDao.update(···)

// 更新表中多条数据
daoSession.studentBeanDao.updateInTx(···)

// 查询表中所有数据
daoSession.studentBeanDao.loadAll()

// 查询表中具有特定键的数据
daoSession.studentBeanDao.load(···)

// 删除表中所有数据
daoSession.studentBeanDao.deleteAll()

// 删除表中具有特定键的数据
daoSession.studentBeanDao.deleteByKey(···)

// 删除一条表中特定数据
daoSession.studentBeanDao.delete(···)

// 删除多条表中特定数据
daoSession.studentBeanDao.deleteInTx(···)
```

一些复杂的查询操作，比如根据某个字段筛选出部分数据，或是关联查询等，需要通过Dao对象的`queryBuilder()`方法来获得一个`QueryBuilder`对象，进而执行复杂查询。`QueryBuilder`所提供的查询方法与数据库复杂查询的指令紧密相关，因此需要对数据库复杂查询的相关概念有足够了解才能较好地利用它们。但是限于主题和篇幅，这里不做展开。

### 数据库升级

数据库升级是所有数据库框架都避不开的一道坎。greenDAO提供的`DaoMaster.DevOpenHelper`实现类，默认的升级方式就是先销毁所有的旧数据表，然后重新创建新表再导入新数据，类似于下列代码所示：

```
/** WARNING: Drops all table on Upgrade! Use only during development. */
public static class DevOpenHelper extends OpenHelper {
    public DevOpenHelper(Context context, String name) {
        super(context, name);
    }

    public DevOpenHelper(Context context, String name, CursorFactory factory) {
        super(context, name, factory);
    }

    @Override
    public void onUpgrade(Database db, int oldVersion, int newVersion) {
        dropAllTables(db, true);
        onCreate(db);
    }
}
```

先销毁再重建的方式适用于数据可通过其他方式恢复更新，且用户设备对流量、电量以及数据加载耗时等方面的性能不敏感的情形。

如果旧数据必须保留，那么只能由开发者自己继承`OpenHelper`抽象类，然后重写里面的`onUpgrade`方法。大致思路为先复制出旧数据，然后执行移除旧表、创建新表的操作，最后再将复制出来的旧数据迁移进新表。也可以采取先将旧数据复制到新表，移除旧表后再更改新表名称的方式。

## greenDAO常用注解说明

greenDAO提供了许多注解，这里对常用注解的使用方式进行说明。

|注解|用途说明|
|:--:|:----:|
|`@Entity`|标注实体类|
|`@NotNull`|将属性设置为不可空|
|`@Convert`|将数据列转换为指定的自定义类型|
|`@Generated`|greenDAO生成的构造器或方法，被此标注的代码可以变更或者下次运行时清除|
|`@Id`|主键Long型，可以通过@Id(autoincrement = true)设置自增长。该注解标记的字段必须是Long，数据库中表示它就是主键，并且默认自增|
|`@Index`|将字段作为一个属性来创建一个索引|
|`@JoinEntity`|定义表连接关系|
|`@JoinProperty`|定义名称和引用名称属性关系|
|`@Keep`|被注解的实体类或代码段将禁止greenDAO修改|
|`@OrderBy`|指定排序|
|`@Property`|设置一个非默认关系映射所对应的列名，默认是的使用字段名|
|`@ToMany`|定义与多个实体对象的关系|
|`@ToOne`|定义与另一个实体对象的关系|
|`@Transient`|添加此标记的字段不会生成数据库表的列|
|`@Unique`|向数据库列添加一个唯一约束|

