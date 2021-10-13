[Retrofit](https://square.github.io/retrofit/)是Square公司在OkHttp的基础上进一步封装，基于“请求域名固定”、“接口功能归类”以及“底层细节屏蔽”的开发实际情况，所设计开发出的更具面向对象思维的应用层网络通信库。由于Retrofit是第三方开源库，因此在Android项目中使用前需要导入依赖：

```
//必选，添加Retrofit、OkHttp以及Okio
implementation "com.squareup.retrofit2:retrofit:$latest_version"

//可选，根据所要解析的数据格式选用对应的转换解析库，解析JSON就选用转换解析JSON的库
implementation "com.squareup.retrofit2:converter-gson:$latest_version"
```

## 接口定义

基于“接口功能归类”的开发事实，使用Retrofit应当**根据不同的服务器接口功能，定义出相应的接口，并为其创建单独的文件**。此外，通常Retrofit接口文件的命名需要遵循`功能+Service`的格式，以确保见名知意。

定义接口的完整方式如下：

```
interface FeatureSevice {
    @注解名(注解内容)
    fun foo(@注解名(注解内容) param: ParamType): Call<ConvertType>
    ···
}
```

>注意，每个接口方法都需要添加[注解](#注解详解)，并且需要将返回值声明成Call类型（使用Call Adapter可以自定义返回类型），通过泛型来指定服务器响应的数据应该转换成什么对象。

## 实例创建

实例创建主要包括Retrofit对象实例以及接口对象实例的创建，具体方式为：

```
//Retrofit对象实例：
val retrofit = Retrofit.Builder()
      .baseUrl("请求根路径，通常是域名或主机地址") //必须调用，注意不带路径要以
      .addConverterFactory(转换库) //需要解析数据时调用
      ···
      .build()

//接口对象实例：
 val service = retrofit.create(TestService::class.java)
```

接口本身是没法实例化的，但是通过Retrofit对象调用`create()`方法实现**动态代理**，进而获得一个实例，调用接口内部定义的所有方法。

事实上，Retrofit对象是全局通用的，而接口实例对象则根据需要创建，因此应当设法优化实例创建流程，确保Retrofit对象是单例，而接口对象可以根据接口变动，一种推荐的方式如下：

```
object ServiceCreator {
    private val retrofit = Retrofit.Builder()
                      .baseUrl("域名") 
                      .addConverterFactory(转换库)
                      ···
                      .build()
    fun <T> create(serviceClass: Class<T>): T = retrofit.create(serviceClass)
}
```

如果掌握了[泛型实化](Kotlin/gen?id=泛型实化)，还可以将`create()`方法改写成：

```
inline fun <reified T> create(): T = create(T::class.java)
```

## 接口调用

接口的调用方式比较简单，如下面代码所示：

```
//异步请求
featureService.foo(param).enqueue(object: Callback<SomeType>{
    override fun onResponse(call: Call<SomeType>, response: Response<SomeType>) {
        //TODO
    }

    override fun onFailure(call: Call<SomeType>, t: Throwable) {
        //TODO
    }
})

//同步请求
featureService.foo(param).execute()
```

这里的`enqueue()`用法跟OkHttp基本相似，都是在内部自动开启子线程，并将数据返回到主线程中（Handler机制），整个过程自动完成线程切换，不需要手动干预。

## 注解详解

虽然Retrofit的源码只有37个文件，但其中22个都是注解，可知注解在Retrofit的使用中占据何等重要的地位，下面将稍微做一些介绍。

### 方法注解

| 注解名称 | 含义| 用法 |
| :------- | :-------------- | :------|
| @GET     | 发送GET请求     | 一般需要添加**相对路径**或**绝对路径**或**全路径**，如果不想在@GET注解后添加请求路径，可在方法的第一个参数中用@Url注解添加请求路径
| @POST    | 发送POST请求    | 同@GET
| @PUT     | 发送PUT请求   | 同@GET
| @DELETE  | 发送DELETE请求  | 同@GET
| @HEAD    | 发送HEAD请求    | 同@GET
| @PATCH| 发送PATCH请求| 同@GET
| @OPTIONS | 发送OPTIONS请求 | 同@GET
| @HTTP    | 发送HTTP请求    | 用于发送一个自定义的HTTP请求，可替代其他的方法注解

在Retrofit中，Url的构建是一个相对复杂的工作，因为Retrofit对Url的组合构建制定了以下规则：

+ 若注解中提供了完整的url（如`https://github.com/Git-a-Live?tab=repositories`），则该url将作为请求的url；
+ 若注解中提供了**不以 `/ `开头的不完整**url（如`Git-a-Live`），则请求的url为baseUrl+注解的组合；
+ 若注解中提供了**以 `/ `开头的不完整**url（如`/Git-a-Live?tab=repositories`），则请求的url为baseUrl的主机部分+注解的组合。

### 标记注解


| 注解名称        | 含义 | 用法 |
| :-------------- | :-------------- | :--- |
| @FormUrlEncoded | 用于修饰@Field注解和@FieldMap注解      | 使用该注解表示请求正文将使用表单网址编码（即[MIME](/ComputerNetwork/Chapter_6_应用层?id=通用互联网邮件扩充mime)类型为`application / x-www-form-urlencoded`）。字段应该声明为参数，并用@Field或@FieldMap注释
| @Multipart      | 使用该注解,表示请求体包含多个部分     | 每一部分作为一个参数，且用@Part注解声明
| @Streaming      | 处理返回Response的方法的响应体     | 不将`body()`转换为byte []

### 参数注解

| 注解名称  | 含义 | 用法 |
| :-------- | :--- | :--- |
| @Query    | 添加请求参数      | 使用该注解定义的参数可为空值并被忽略；当传入一个List或array时，为每个非空item以`key = value`的形式，通过`&`拼接请求键值对，用在Url中
| @QueryMap | 以map形式添加请求参数     | map的键和值默认进行URL编码，且每一项的键和值都不能为空，否则会抛出`IllegalArgumentException`异常，用在Url构建中
| @Body     | 将一个实体类序列化为请求正文      | 使用该注解定义的参数**不可为空**，适用于不想以参数或表单的形式发送POST或PUT请求的场景
| @Field    | 发送表单请求     | 同@Query，但用于请求正文中
| @FieldMap | 以map形式发送表单请求     | 同@QueryMap，但用于请求正文中
| @Part     | 定义Multipart请求的每个part     | 使用该注解定义的参数可为空并被忽略；如果类型是okhttp3.MultipartBody.Part，内容将被直接使用； 如果类型是RequestBody，那么该值将直接与其内容类型一起使用，需要在注释中提供part名称；其他对象类型将通过使用转换器转换为适当的格式，同样要在注释中提供part名称
| @PartMap  | 以map的方式定义Multipart请求的每个part     | map中每一项的键和值都不能为空；如果类型是RequestBody，那么该值将直接与其内容类型一起使用；其他对象类型将通过使用转换器转换为适当的格式，并在注释中提供part名称

### 其他注解

| 注解名称 | 含义 | 用法 |
| :------- | :--- | :--- |
| @Path    | 在URL路径段中替换指定的参数值    |使用该注解定义的参数的值不可为空，且默认使用URL编码
| @Header  | 添加请求头     | 使用该注解定义的请求头可以为空并被自动忽略；当传入一个List或array时，将每个非空的item的值拼接到请求头中；具有相同名称的请求头不会相互覆盖，而是会照样添加到请求头中
| @Url     | 添加请求的接口地址     | **不能与@Path注解同时使用**，因为使用@Url指定请求路径时，请求路径尚不存在，@Path注解自然也就没法替换参数