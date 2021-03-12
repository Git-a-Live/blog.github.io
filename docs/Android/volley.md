## 何为Volley

[Volley](https://developer.android.com/training/volley/)是Google官方在2013年推出的一款网络封装库（同时也是一个图片加载框架），主要用于**数据量小且通信频繁的网络操作**场景当中。

按照官方的说法，Volley具有以下特点：

```
· Automatic scheduling of network requests.
· 自动调度网络请求。

· Multiple concurrent network connections.
· 多个并发网络连接。

· Transparent disk and memory response caching with standard HTTP cache coherence.
· 透明磁盘和具有标准HTTP缓存一致性的内存响应缓存。

· Support for request prioritization.
· 支持请求设置优先级。

· Cancellation request API. You can cancel a single request, 
  or you can set blocks or scopes of requests to cancel.
· 取消请求API，既可以取消单个请求，也可以设置取消某个时间段或范围内的请求。

· Ease of customization, for example, for retry and backoff.
· 便于自定义，如自定义重试或退避时间。

· Strong ordering that makes it easy to correctly populate your UI 
  with data fetched asynchronously from the network.
· 强大的排序功能，确保从网络上异步获取到的数据能正确填充到界面上。

· Debugging and tracing tools.
· 调试和追踪工具。
```

同时，官方也强调：

```
Volley is not suitable for large download or streaming operations, 
since Volley holds all responses in memory during parsing. 
For large download operations, consider using an alternative like DownloadManager.

Volley不适用于大量下载或流式传输操作，因为Volley在解析过程中会把所有响应缓存到内存中（响应数据量太大会耗光内存）。
如果需要进行大量下载的操作，最好选择DownloadManager之类的方式。
```

目前Volley并没有集成在Android SDK当中，因此要想使用Volley，需要先在项目中导入依赖如下：

```
dependencies {
    ...
    implementation 'com.android.volley:volley:$some_version'
}
```

最新版Volley可以前往Volley所在的[GitHub](https://github.com/google/volley)页面查看。

><font color=red>注意，任何使用网络功能的应用都必须先在AndroidManifest文件中添加访问网络的权限。</font>

## 基本使用

### 简单请求响应

使用Volley完成一次简单的请求响应，主要依靠“三板斧”：RequestQueue、目标访问地址和Request实例对象。

首先需要创建一个RequestQueue对象：

```
//使用默认配置的RequestQueue
val queue = Volley.newRequestQueue(context)
```

接着设置目标访问地址：

```
//目标访问地址字符串中必须包含协议名称，否则Volley会报异常
val url = "https://..."
```

然后创建一个Request实例对象：

```
//Request是一个接口，StringRequest只是实现该接口的子类之一，Volley还自带JsonRequest和ImageRequest
val request = StringRequest(Request.Method.XXX,url,
    Request.Listener<String> {
        //TODO: 处理响应返回的字符串
    },
    Response.ErrorListener {
        //TODO: 处理请求失败
    }
)
```

最后把Request实例对象添加到RequestQueue里面，剩下的交由Volley自动完成：

```
queue.add(request)
```

在调用`add()`方法时，Volley会运行一个**缓存处理线程**和一个**网络调度线程池**。当请求被添加到队列后，缓存线程会拾取该请求并对其进行分类。如果该请求所需的响应内容已经缓存到本地，系统就会在缓存线程上解析本地缓存，并在主线程上传送解析后的响应内容，否则系统会将其放进网络队列中，进行在线请求。此时线程池中第一个可用的网络线程会从队列中获取该请求，执行HTTP事务，在**工作器线程**上解析响应并写入缓存，最后将解析后的响应发送回主线程以供使用。

需要注意的是，诸如I/O、解析和解码等开销较大的操作都是在工作器线程上执行的。此外，不管请求是从哪个线程发起，响应最终都会被传送到主线程。

### 取消请求

取消某个待处理请求最简单的方式，就是调用这个请求的Request实例对象的`cancel()`方法，例如：

```
request.cancel()
```

一旦调用该方法，后续的响应处理也不会进行，这意味着取消请求是“不计后果”的。但是要注意，如果有些环节依赖于该请求的响应处理结果，那么就要谨慎考虑是否调用这个方法。

在某些情形下，开发者可能需要取消一部分请求，而这些请求数量众多又不好一一手动调用`cancel()`方法，这时候可以为它们设置一个标记，例如：

```
request1.tag = "someTag"
request2.tag = "someTag"
request3.tag = "someTag"
...
```

然后由RequestQueue对象统一取消带有特定标记的请求：

```
queue.cancelAll("someTag")
```

## 进阶使用

### 设置RequestQueue

在基本使用中，RequestQueue对象是由`newRequestQueue()`方法创建的，其配置也是Volley的默认配置。如果想满足特定的需求，就需要对RequestQueue进行自定义设置。

RequestQueue有三个构造方法：

```
RequestQueue(Cache cache, Network network, int threadPoolSize, ResponseDelivery delivery)

RequestQueue(Cache cache, Network network, int threadPoolSize)

RequestQueue(Cache cache, Network network)
```

这些构造方法的使用在Voilley中有相关的注释说明，此处不做赘述。调用其中的一个，传入指定的参数即可完成对RequestQueue对象的自定义配置。以最简单的一个构造方法为例：

```
//设置缓存
val cache = DiskBasedCache(cacheDir, 1024 * 1024)
//设置请求处理程序
val network = BasicNetwork(HurlStack())
//创建自定义RequestQueue对象
val requestQueue = RequestQueue(cache, network).apply {
    start()
}
···
```

注意到自定义RequestQueue对象在创建之后需要马上调用`start()`方法，该方法的作用是创建并启动缓存分类器。在这之后，就跟前面基本使用一样了。

**Google官方建议自定义RequestQueue对象采用单例模式进行调用**。

### 使用Request实例对象

前面提到，Request是一个接口，StringRequest只是实现该接口的子类之一，Volley还自带JsonRequest和ImageRequest。Volley把响应分成了三种类型进行处理，开发者可以根据需要进行选用。

#### StringRequest

StringRequest提供了一种简单粗暴的响应处理方式——全部转成字符串，这在某些情况下是很好用的，比如访问的地址直接返回了一串字符串。

StringRequest的构造方法有两个：

```
StringRequest(int method, String url, Listener<String> listener, @Nullable ErrorListener errorListener)

StringRequest(String url, Listener<String> listener, @Nullable ErrorListener errorListener)
```

第一个构造方法在基本使用中已经展示过了，第二个构造方法默认发起GET请求。两个构造方法都需要输入目标访问地址、响应成功监听器以及响应错误监听器。对响应的处理主要在这两个监听器里进行。

#### JsonRequest

JsonRequest顾名思义就是用来处理Json类型的响应，但是通常情况下并不直接使用它，而是它底下的两个子类：JsonObjectRequest和JsonArrayRequest。

JsonObjectRequest的构造方法也有两个：

```
JsonObjectRequest(String url, @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener, @Nullable ErrorListener errorListener)

JsonObjectRequest(int method, String url, @Nullable JSONObject jsonRequest,
            Listener<JSONObject> listener, @Nullable ErrorListener errorListener)
```

第一个构造方法不带请求方式，由传入的JSONObject对象决定，传入空对象时进行GET请求，反之则进行POST请求。两个构造方法都需要传入响应监听器，和StringRequest一样。

和JsonObjectRequest一样，JsonArrayRequest还是两个：

```
JsonArrayRequest(String url, Listener<JSONArray> listener, @Nullable ErrorListener errorListener)

JsonArrayRequest(int method, String url, @Nullable JSONArray jsonRequest, 
            Listener<JSONArray> listener, @Nullable ErrorListener errorListener)
```

第一个构造方法默认发起GET请求。两个构造方法同样都要传入响应监听器。

#### ImageRequest

ImageRequest用于处理图片，这也是Volley能作为一个简单图片加载框架的原因。当然，需求比较复杂的情况下还是应该使用专门的图片加载框架。

ImageRequest依然有两个构造方法，但是其中一个已经被标注废弃，因此只需要了解还在使用的另一个即可：

```
ImageRequest(String url, Response.Listener<Bitmap> listener, int maxWidth,
            int maxHeight, ScaleType scaleType, Config decodeConfig,
            @Nullable Response.ErrorListener errorListener)
```

这个构造方法需要传入的参数比较多，除了目标访问地址和响应监听器以外，还需要传入最大展示宽度（maxWidth）、最大展示高度（maxHeight）、图片填充方式（scaleType）以及图片解码方式（decodeConfig）。其中，图片填充方式类似于电脑桌面的契合度，有居中、拉伸等等；图片解码方式有六种，有一种由于失真度太高已经被弃用，剩余五种（ALPHA_8、RGB_565 、ARGB_8888、RGBA_F16以及HARDWARE）可以通过Google和Volley的注释说明去了解，此处不做深入展开。

#### 自定义Request子类

根据二八定律，上述几个由Google官方提供的Request子类已经可以满足大部分的简单需求，但是有些需求光用它们还是满足不了，于是就需要实现Request接口，设计出满足开发者需求的子类。但是具体问题具体分析，不同的需求肯定会衍生出不同的Request子类，究竟要如何自定义并不能一概而论。所以最好的方式就是参考现成的几个子类进行设计，也可以参考Google官方的[简单介绍](https://developer.android.com/training/volley/request-custom)，这里不再深入。

