虽然WebView和其他第三方开源库封装性很高，但也给带来另一个问题：如果只是单纯发送请求和接收响应， 但不必对返回的数据进行解析处理将其显示成一个网页，显然用不到WebView。 

为了解决这个功能，Android提供了HttpURLConnection和HttpsURLConnection两个类，用于发送请求和接受处理响应。 它们在使用上基本没有差别，只不过代码量比较多，显得很繁琐。当然，如果官方提供的东西不好用，自然还有第三方的开源库可以替代。 OkHttp就是目前开发者首选的第三方网络通信库。

## OkHttp基本用法

在使用OkHttp之前，需要在Gradle文件中添加如下依赖：

```
implementation "com.squareup.okhttp3:okhttp:$lateset_version"
```

添加上述依赖后，Android Studio会导入OkHttp和Okio这两个库。
由于网络操作非常耗时，因此必须开启协程或多线程。此外，网络功能的代码大同小异，没有必要在每个用到网络功能的地方都写一遍， 所以应该把这部分代码封装起来。通常最简易的做法如下：

```
fun foo(···) {
    Thread(Runnable {
        val okHttpClient = OkHttpClient() //创建OkHttpClient对象
        val requestBody = FormBody.Builder() //创建FormBody对象，添加要提交的数据
                .add("data1_name","data")
                .add("data2_name","data")
                .···
                .build()
        val request = Request.Builder()
                .url(url)
                .post(requestBody) //在发送请求中携带要提交的数据
                .···
                .build() //构建Request对象
        val response= okHttpClient.newCall(request).execute()
        //发送请求给指定网站服务器，并接收响应返回的数据
        try {
                val rsp = response.body?.string()
                //注意在body不要调用toString()方法，否则无法将响应返回的数据正确转换成字符串形式
                //body还可以通过bytes()、byteString()、byteStream()、charStream()等方法转换成其他形式
                if (rsp == null){
                    //TODO
                } else {
                    //TODO
                }
        } catch (e: Exception){
             //TODO
        }
    }).start()
}
```

然而，OkHttp还有更为简便且周到的做法：

```
//封装：
fun demoOkHttp(url:String, call:okhttp3.Callback){
     val okHttpClient = OkHttpClient()
     val request = Request.Builder()
                .url(url)
                .build()
     okHttpClient.newCall(request).enqueue(call)
     //enqueue()方法内部开启子线程
     //注意：子线程内部不能直接使用return返回数据，必须使用回调机制
}

//调用：
DemoHttpClass.demoOkHttp(url, object:okhttp3.Callback { //利用回调机制将响应数据返回给调用方
     override fun onFailure(call:Call, e:IOException) {
        //TODO
     }
     override fun onResponse(call:Call, response:Response) {
        //TODO
     }
})
```

## OkHttp与WebSocket通信

OkHttp除了支持HTTP/HTTPS通信之外，还支持WebSocket通信。在介绍OkHttp如何进行WebSocket通信之前，需要先简单了解什么是WebSocket。

### 何为WebSocket

WebSocket诞生于2008年，是由HTML5规范引出的一项新型[应用层](ComputerNetwork/Chapter_6_应用层)协议，在2011年成为国际标准，目前市场上主流的浏览器都已支持该协议。WebSocket解决了HTTP长期以来**只能由客户端发起通信**的缺陷，实现了服务端主动向客户端推送信息，也就是在C/S体系下的[全双工](ComputerNetwork/Chapter_2_物理层?id=数据通信基础)通信（HTTP只是半双工通信）。在服务器状态连续变化的情形里面，这种主动推送消息的操作可以<font color=red>消除客户端采用轮询方式造成的效率低下和资源浪费</font>。WebSocket的主要特点包括：

+ 建立在TCP协议之上，服务器端的实现比较容易；
+ 与HTTP良好兼容，默认端口也是80和443，并且握手阶段采用HTTP协议，因此握手时不容易被屏蔽，能通过各种HTTP代理服务器；
+ 数据格式比较轻量，性能开销小，通信高效；
+ 可以发送文本，也可以发送二进制数据；
+ 没有同源限制，客户端可以与任意服务器通信。

> 同源策略是一个重要的安全策略，它用于限制一个origin的文档或者它加载的脚本如何能与另一个源的资源进行交互。该策略能帮助阻隔恶意文档，减少可能被攻击的媒介。

WebSocket通信时采用的链接形式也是[URL](ComputerNetwork/Chapter_6_应用层?id=web、url与html)，只不过协议名称换成`ws`或`wss`，`ws`和`wss`的差异可以参照`http`和`https`。WebSocket在TCP连接建立后，需要通过HTTP进行一次握手，也就是通过HTTP发送一条GET请求消息给服务器，告知服务器准备要建立WebSocket连接了，具体做法就是在头部信息中添加相关参数。然后服务器做出响应，并且将连接协议改成WebSocket，开始建立长连接。在这一过程中，WebSocket的请求和响应跟HTTP相比有以下重要差异：

+ <font color=red>请求头</font>使用`Connection:Upgrade`参数，表示客户端要连接升级，不用HTTP协议；
+ <font color=red>请求头</font>使用`Upgrade:websocket`参数，表示客户端要升级建立Websocket连接；
+ <font color=red>请求头</font>使用`Sec-Websocket-Key:<key>`参数，服务端会通过这个参数验证该请求是否有效，其中key是由客户端随机生成的；
+ <font color=red>请求头</font>使用`Sec-webSocket-Extensions:<extension>`参数，用于客户端指定一些扩展协议，比如`permessage-deflate`，这是一种WebSocket压缩协议；
+ <font color=blue>响应头</font>返回`Sec-WebSocket-Accept:<key>`参数，用以服务端告知客户端同意发起一个WebSocket连接，其中key是由服务端随机生成的，跟请求头里的不一样；
+ <font color=blue>响应头</font>返回`Sec-webSocket-Extensions:<extension>`参数，用于服务端告知客户端服务端所支持的扩展协议；
+ <font color=blue>响应头</font>返回`Sec-WebSocket-Version:<version>`参数，用于服务端告知客户端所使用的Websocket协议版本，截止2022年6月的最新版本为13；
+ <font color=blue>响应头</font>返回状态码`101`，表示服务端响应协议升级，后续的数据交互都采用WebSocket协议。

### 实现基本WebSocket通信

首先是构建发起WebSocket请求的客户端：

```
val client = OkHttpClient.Builder()
    .connectTimeout(5, TimeUnit.SECONDS) // 设置连接超时时间
    .readTimeout(5, TimeUnit.SECONDS)    // 设置读取超时时间
    .writeTimeout(5, TimeUnit.SECONDS)   // 设置写入超时时间
    .callTimeout(5, TimeUnit.SECONDS)    // 设置调用超时时间
    .pingInterval(5, TimeUnit.SECONDS)   // 设置保活心跳发送间隔
    .retryOnConnectionFailure(true)      // 允许连接失败后尝试自动重连
    .build()
```

接着构建客户端要发起的WebSocket请求：

```
val url = "wss://example.com"
val request = Request.Builder().url(url).build()
```

然后构建WebSocket监听器：

```
val listener = obj: WebSocketListener() {
    override fun onOpen(webSocket: WebSocket, response: Response) {
        super.onOpen(webSocket, response)
        // TODO: 建立WebSocket连接时相关业务逻辑
    }

    override fun onClosing(webSocket: WebSocket, code: Int, reason: String) {
        super.onClosing(webSocket, code, reason)
        // TODO: WebSocket连接正在关闭时相关业务逻辑
    }

    override fun onClosed(webSocket: WebSocket, code: Int, reason: String) {
        super.onClosed(webSocket, code, reason)
        // TODO: WebSocket关闭后相关业务逻辑
    }

    override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
        super.onFailure(webSocket, t, response)
        // TODO: WebSocket连接失败时相关业务逻辑
    }

    override fun onMessage(webSocket: WebSocket, text: String) {
        super.onMessage(webSocket, text)
        // TODO: 客户端接收到服务端推送消息时相关业务逻辑（仅限字符类型）
    }

    override fun onMessage(webSocket: WebSocket, bytes: ByteString) {
        super.onMessage(webSocket, bytes)
        // TODO: 客户端接收到服务端推送消息时相关业务逻辑（仅限二进制类型）
    }
}
```

> 注意，上述监听器回调方法均在**子线程**当中执行，自定义的业务逻辑需要注意线程切换问题。

最后通过客户端发起WebSocket请求：

```
// 一条连接只需成功建立一次，不需要反复调用newWebSocket
val webSocket = client.newWebSocket(request, listener)

// WebSocket实例对象可执行的操作
webSocket.apply {
    close(code, reason) // 主动关闭WebSocket连接，需要传入标准状态码和关闭原因
    request()           // 获取初始化当前WebSocket连接时所使用导的请求头
    cancel()            // 立即释放当前WebSocket连接持有的所有资源，并废弃所有还没发送出去的消息
    send(string)        // 向服务器发送字符串类型消息
    send(byteString)    // 向服务器发送经过编码的二进制类型消息
    queueSize()         // 获取队列中等待传输的消息的字节长度
}

// WebSocket实例对象释放WebSocket连接后，客户端还要释放线程池以回收线程资源
client.dispatcher.executorService.shutdownNow()
```

> WebSocket关闭时所使用的标准状态码可参考：https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent/code

### 搭建WebSocket模拟服务端

如果想要测试WebSocket功能，但又没有可用的服务器（尤其是不想专门租用一台云服务器）时，可以利用OkHttp提供的[MockWebSocket](https://mvnrepository.com/artifact/com.squareup.okhttp3/mockwebserver)服务，在应用当中提供一个模拟服务器的功能。MockWebSocket需要额外引入一个依赖库：

```
implementation 'com.squareup.okhttp3:mockwebserver:$specified_version'
```

新建一个`MockWebServer`对象：

```
val mockWebServer: MockWebServer = MockWebServer()
```

构建响应监听器：

```
val response: MockResponse = MockResponse().withWebSocketUpgrade(object: WebSocketListener() {
    override fun onOpen(webSocket: WebSocket, response: Response) {
        super.onOpen(webSocket, response)
        // TODO: 建立WebSocket连接时相关业务逻辑
    }

    override fun onClosing(webSocket: WebSocket, code: Int, reason: String) {
        super.onClosing(webSocket, code, reason)
        // TODO: WebSocket连接正在关闭时相关业务逻辑
    }

    override fun onClosed(webSocket: WebSocket, code: Int, reason: String) {
        super.onClosed(webSocket, code, reason)
        // TODO: WebSocket关闭后相关业务逻辑
    }

    override fun onFailure(webSocket: WebSocket, t: Throwable, response: Response?) {
        super.onFailure(webSocket, t, response)
        // TODO: WebSocket连接失败时相关业务逻辑
    }

    override fun onMessage(webSocket: WebSocket, text: String) {
        super.onMessage(webSocket, text)
        // TODO: 客户端接收到服务端推送消息时相关业务逻辑（仅限字符类型）
    }

    override fun onMessage(webSocket: WebSocket, bytes: ByteString) {
        super.onMessage(webSocket, bytes)
        // TODO: 客户端接收到服务端推送消息时相关业务逻辑（仅限二进制类型）
    }
})
```

将响应传入模拟服务端的消息队列：

```
mockWebServer.enqueue(response)

// 获取模拟服务端的地址
val url = "ws://${mockWebServer.hostName}:${mockWebServer.port}/"
```

至此，WebSocket模拟服务器已经在应用当中完成搭建，后续就是客户端与模拟服务端进行通信的步骤，这里不再赘述。

