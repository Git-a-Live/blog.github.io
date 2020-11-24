JSON是JavaScript Object Notation的缩写，它去除了所有JavaScript执行代码，只保留JavaScript的对象格式。 它是一种比XML更为简单的数据结构，而且在序列化中使用起来更为安全。JSON作为数据传输的格式，有几个显著的优点：

+ JSON只允许使用UTF-8编码，不存在编码问题；
+ JSON只允许使用双引号作为key，特殊字符用`\`转义，格式简单；
+ 浏览器内置JSON支持，如果把数据用JSON发送给浏览器，可以用JavaScript直接处理。

JSON适合表示层次结构，因为它格式简单，仅支持以下几种数据类型：

+ 键值对：{"key": value}
+ 数组：[1, 2, 3]
+ 字符串："abc"
+ 数值（整数和浮点数）：12.34
+ 布尔值：true或false
+ 空值：null

浏览器直接支持使用JavaScript对JSON进行读写，使用JSON进行数据传输，在浏览器端非常方便， 因此，绝大多数REST API都选择JSON作为数据传输格式。

在Java中，通常可以使用Jackson、Gson以及Fastjson等第三方库来解析JSON（原生的也可以用，就是相对麻烦一些）。 利用Jackson将JavaBean变成JSON的过程是序列化，将JSON解析为JavaBean则为反序列化。可以通过自定义JsonSerializer和JsonDeserializer来定制序列化和反序列化。

以Gson为例，Gson是由Google开发的第三方库，专门用于Java的序列化和反序列化。 在项目中使用Gson的话需要在Gradle或Maven文件里面添加依赖：

```
//Gradle：
implementation 'com.google.code.gson:gson:${latest_version}'

//Maven：
<dependency>
    <groupId>com.google.code.gson< /groupId>
    <artifactId>gson< /artifactId>
    <version>${latest_version}< /version>
< /dependency>
```

通过Gson进行序列化的常用方式如下：

```
//Kotlin代码：
//对类序列化
val demo = Demo()
val string = Gson().toJson(demo) /*如果一个类中还包含另一个类，那么就会嵌套输出*/

//对数组/List序列化
val demosArray/demosList = arrayOf/listOf(
    val demo1 = Demo()
    val demo2 = Demo()
    ···
)
val string = Gson().toJson(demosArray/demosList)

//Java代码：
//对类序列化
Demo demo = new Demo()
String string = new Gson().toJson(demo)

//对数组序列化
Demo[] demoArray = {
    Demo demo1 = new Demo(),
    Demo demo2 = new Demo(),
    ···
}
String string = new Gson().toJson(demos)

//对List序列化
List< Demo> demoList = List.of(
    new Demo(),
    new Demo(),
    ···
);
String string = new Gson().toJson(demosList);
```

通过Gson进行反序列化的常用方式如下：

```
//Kotlin：
//对类反序列化
val json = {···}
val demo: Demo = Gson().fromJson(json,Demo::class.java)

//对数组反序列化
val json = [···]
val demoArray: Array< Demo> = Gson().fromJson(json,Array< Demo>::class.java)

//对List反序列化
val json = [···]
/*List*/
val demosArray: List<Demo> = Gson().fromJson(json,Array<Demo>::class.java).toList()
/*ArrayList*/
val demosArrayList: ArrayList<Demo> = Gson().fromJson(json,ArrayList< Demo>::class.java)

//Java代码：
//对类反序列化
String json = {···};
Demo demo = new Gson().fromJson(json,Demo.class);

//对数组反序列化
String json = [···];
Demo[] demoArray = new Gson().fromJson(json,Demo[].java);

//对List反序列化
String json = [···];

//方法一：将List当成数组，List特性丢失
Demo[] demoList = new Gson().fromJson(json, Demo[].java);

//方法二：利用反射，保留List特性
Type typeList = new TypeToken<List<Demo>>(){}.getType();
List<Demo> demoList = new Gson().fromJson(json, typeList);
```

如果在创建类的时候，对里面的字段加上注解@SerializedName(NAME)，那么在序列化之后， 这些加上注解的字段将不会以原来的名称出现在JSON文件中，而是显示注解赋予的名称；若不希望某个字段被序列化， 可以在该字段前加上注解@Transient（Kotlin）或者直接加上transient（Java）。