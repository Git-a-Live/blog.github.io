Room是Google提供的一个ORM库，它在SQLite的基础上提供了一个抽象层，让用户能够在充分利用SQLite的强大功能的同时，获享更强健的数据库访问机制。**Google官方强烈建议使用Room而非直接使用SQLite来访问数据库**，主要原因在于：

1. SQLite没有针对原始SQL查询的编译时验证；

2. 开发者需要使用大量样板代码，在SQL查询和数据对象之间进行转换。

## 前置工作

要使用Room，需要在相应模块（注意不是项目本身）的build.gradle文件中添加如下依赖：

```
plugin {
    id 'kotlin-kapt' //kapt是必要依赖
}

dependencies {
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:$room_version"

    // optional - Test helpers
    testImplementation "androidx.room:room-testing:$room_version"
}
```

## Dao、Entity以及Database

Room提供了三大核心组件：

Database：@Database用来注解类，并且注解的类必须是继承自RoomDatabase的抽象类。 该类主要作用是创建数据库和创建Daos（data access objects，数据访问对象）。

Entity：@Entity用来注解实体类，@Database通过entities属性引用被@Entity注解的类， 并利用该类的所有字段作为表的列名来创建表。

Dao：@Dao用来注解一个接口或者抽象方法，该类的作用是提供访问数据库的方法。 在使用@Database注解的类中必须定一个不带参数的方法，这个方法返回使用@Dao注解的类。

它们的关系如下图所示：

![](pics/room_architecture.png)

### Dao

Dao的全称为Data access object（数据访问对象），专门提供访问数据库的方法，而数据库的数据最终是要通过Dao来进行CRUD操作。Dao的创建方法如下：

```
@Dao //该注解使得对象成为一个Dao接口
interface DemoDao {
    @Insert　//这类注解表示这是由官方定义好的SQL命令，开发者只需要传入参数即可
    suspend fun insertXxx(···) //使用协程的写法

    @Update
    fun updateXxx(···)

    @Delete
    fun deleteXxx(···)

    @Query("···")  //在@Query()中写入指定的SQL语句，例如"SELECT * FROM DemoEntity"
    fun foo(···)
    ···
}
```

在Dao中创建的函数都不需要写出函数体，函数只是负责传入参数或返回查询到的记录，具体实现交由Room的底层。因为是依靠注解去执行具体的SQL命令，这也就意味着开发者不光要熟悉Android的开发工具和开发语言，还需要掌握一定的SQL指令。

Android Studio目前支持在编写SQL指令时高亮提示，从而确保指令编写使用的是正确语法。

>注意，接口可以替换成抽象类，同理接口方法就被替换成抽象方法。如果多个Dao具有高度相似的函数，那么就可以将其抽象出来作为一个基础接口，其他Dao定义为抽象类继承该接口。

### Entity

Entity对应的是数据库中的表。Entity类的创建方式如下：

```
@Entity //该注解使得对象成为一个Entity类
data class DemoEntity(
    @PrimaryKey(autoGenerate = true) var primaryKey: Type1, //创建自增主键，直接赋值为0即可
    @ColumnInfo(name = "col1") var col1: Type2,  //创建列并标注列名
    var col2: Type3, //如果不使用@ColumnInfo则以变/常量名称作为列名
    ···
) {
    //TODO
}
```

从上面的代码中可以看到，Entity类的创建过程和数据库创建表的过程很相似。 Entity类中必须对每个列（包括主键）都创建get/set方法，否则无法对表中的记录进行操作（Kotlin使用data class会自动实现这些get/set方法）。

### Database

Database的创建方法如下：

```
@Database(entities = [DemoEntity_1::class, DemoEntity_2::class, ···], version = xxx, exportSchema = ···) 
abstract class DemoDatabase: RoomDatabase() {
    companion object {
        @Volatile
        private var instance: DemoDatabase? = null

        fun getImpl(context: Context): ScoreQueryDatabase = instance ?: synchronized(this) {
            Room.databaseBuilder(context, DemoDatabase::class.java,"database_name")
                    .build()
                    .also { instance = it }
                    //如果想强制在主线程中访问数据库，必须在.build()前加上.allowMainThreadQueries()
        }
    }

    abstract suspend fun getDemoDao1(): DemoDao1 //使用协程的写法
    abstract fun getDemoDao2(): DemoDao2
    ···
    //有多少个Entity，就要定义多少个Dao以及返回Dao的抽象方法，同时@Database的entities里也要注明有对应的Entity
}
```

在实际开发过程中，Entity和Dao可以集中到一起组成一个Module，供上层其他模块调用，而每个模块分别定义自己的Database，根据需要接入指定的Dao。最后可以在本模块中，或者更上层的模块中定义下面要讲到的Repository，以集中编写数据库操作方法，对外（特别是ViewModel）提供简洁的接口。

## Repository

Repository并不是架构组件库的一部分，而是代码解耦和架构比较推荐的方法。Repository类用于访问多个数据源，处理数据操作，并为应用程序提供一个整洁的API。Repository一般负责管理数据的查询线程，可使用多个后端。通常情况下，Repository主要实现从服务端或本地的数据库中拉取数据的逻辑。Repository通常还会跟ViewModel配合使用，即Repository提供封装好了的业务逻辑API，ViewModel编写调用这些API的方法，然后再由其他组件调用ViewModel提供的方法。

## 数据库版本迁移

当存储数据的表发生结构变化时，就需要进行数据库版本迁移。如果试图在修改表的结构之后不做其他任何改动，就将项目打包发布， 那么这个应用在设备上运行时可能就会出现崩溃的情况。数据库版本迁移通常是指Entity类中的表（这里称为Entity表）结构发生变更之后， 在保留数据的情况下，将Entity表的新结构应用到数据库中的表，使二者结构一致。 当然，如果确实不打算保留数据，通常采取以下两种方法进行所谓的“数据库版本迁移”：

方法一：卸载原来的应用，安装新的应用。采用这种方法的话，新应用中只修改Entity表的结构，其他文件设置不做任何改动；

方法二：在修改了Entity表结构之后，需要进入Database类文件，修改当前版本号，然后在调用`Room.databaseBuilder().build()`的时候， 在`.build()`前加上`.fallbackToDestructiveMigration()`，执行之后数据库就只会将Entity表的结构应用到数据库原表，先前所有数据都被清空。

在实际开发中，上述两种方法基本不可能考虑，因为用户更新应用的时候通常不会采用卸载旧版本的方式，而是直接覆盖安装新版本，同时， 一个每次更新都会清空所有数据的应用是不会被用户容忍的。所以必须采用下面这种看起来很繁琐的方法，才能在保留数据的前提下完成数据库版本迁移。

首先在Database类中创建一个迁移策略：

```
private val migration_M2N = object : Migration(m,n) { //从版本m迁移至版本n
    override fun migrate(database: SupportSQLiteDatabase) {
        //此处用于调用SQLite语句
        database.execSQL( "SQLite语句1" )
        database.execSQL( "SQLite语句2" )
        database.execSQL( "SQLite语句3" )
        ···
    }
}
```
然后在Database类的getDatabase()方法中按如下方式调用Room.databaseBuilder()：

```
Room.databaseBuilder(···).addMigrations(migration_M2N).build()
```

之后编译运行，通过Android Studio右侧的Device File Explorer - data - data - 应用包名 - databases，导出所有数据库文件， 通过相关工具查看这些文件的内容，检查设备上数据库表的结构是否和当前的Entity表一致。

注意，如果Entity表的结构发生改变会影响到数据的输入输出，那么项目的其他源文件就要做相应的修改，以免发生遗漏导致编译运行过程中报错。

SQLite添加列的方式很简单，但是删除列很麻烦，要按照以下步骤进行：

1. 通过CREATE TABLE创建一个新表，该表结构就是原表删除指定列以后的结构；

2. 利用INSERT INTO将原表中保留列的数据插入到新表；

3. 使用DROP TABLE删除原表；

4. 依靠ALTER TABLE将新表改名，其名字和原表一致。

如果将设备上原来安装的应用卸载掉，再直接安装包含迁移策略的新版本应用，是否会因为设备上没有相应旧版本的数据库而发生错误呢？答案是在实际应用中并不会发生这种情况。

## 数据库的分页显示

Paging（分页）是一种按需逐步加载数据的方法。当本地或服务端的数据库中保存大量数据时，如果不采用分页的方式， 那么用户在访问的时候，应用就会把所有数据（包括用不到的）都加载进来。这种做法会增加流量、内存、电量等资源的消耗， 增大数据呈现的延迟程度，降低用户体验。

Paging的核心之一在于构建使用Adapter。Paging所使用的Adapter跟之前相比更为复杂， 最为重要的一点就是不保存视图data的地址，而是利用DiffUtil.ItemCallback对比视图原有的data（旧数据集）传入视图的data（新数据集）。 如果新数据集相对于旧数据集没有发生改变，那么activity就不执行视图更新的操作；反之就只更新发生改变的部分。 

在之前的做法中，无论何种情况都调用notifyDataSetChanged()，强制更新视图上的所有内容，这是一种很不经济的做法，在大型项目中这种做法的缺陷会尤为明显。 

使用Paging需要引入下面的依赖：

```
implementation "androidx.paging:paging-runtime:${latest_version}"
```

DiffUtil.ItemCallback的使用方法具体如下列代码所示：

```
class DemoPagedAdapter :
    PagedListAdapter<User,DemoPagedAdapter.ViewHolder>(object : DiffUtil.ItemCallback<User>() {
        //此处用于对比新旧数据集，以便后续其他组件只对视图进行局部更新
        override fun areItemsTheSame(oldItem: User, newItem: Profile): Boolean {
            //TODO：覆写该方法，判断两个Item是否一致，通常对比Item中具有唯一标识作用的字段。
        }

        override fun areContentsTheSame(oldItem: User, newItem: Profile): Boolean {
            //TODO：覆写该方法，判断两个Item的内容是否一致
        }
    }) {

    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        //TODO
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        //TODO
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        //TODO
    }
}
```

Paging的另一个核心在于使用PagedList\<User>类型的数据。这就要求在activity里面调用LivePagedListBuilder(···).build()之前， 必须调用返回DataSource.Factory\<Int, User>类型数据的方法，从数据源获取数据。这些方法一般分布在ViewModel、Repository以及Dao等文件中， 下面是几个例子：

```
//Dao文件
@Query("···")
fun  Foo(): DataSource.Factory<Int, User>

//ViewModel文件
fun getData(): DataSource.Factory<Int, User>{
    return demoRepository.getData()
}

//Repository文件
fun getData(): DataSource.Factory<Int, User> {
    return data
}
```

activity中调用Paging的Adapter的具体示例如下：

```
val demoPagedAdapter = DemoPagedAdapter()
recyclerView.layoutManager = LinearLayoutManager(···)
recyclerView.adapter = demoPagedAdapter
val demoViewModel = DemoViewModel(application)
val demoPaged = LivePagedListBuilder(demoViewModel.getData(),pageSize: Int).build()
// 获取数据并设置视图每次加载的内容数量
demoPaged.observe(this, Observer {
  demoPagedAdapter.submitList(it)
  // 只进行局部更新，不会再像notifyDataSetChanged()那样将视图的内容全部刷新
})
```