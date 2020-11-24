虽然WebView和其他第三方开源库封装性很高，但也给带来另一个问题：如果只是单纯发送请求和接收响应， 但不必对返回的数据进行解析处理将其显示成一个网页，显然用不到WebView。 

为了解决这个功能，Android提供了HttpURLConnection和HttpsURLConnection两个类，用于发送请求和接受处理响应。 它们在使用上基本没有差别，只不过代码量比较多，显得很繁琐。当然，如果官方提供的东西不好用，自然还有第三方的开源库可以替代。 OkHttp就是目前开发者首选的第三方网络通信库。

在使用OkHttp之前，需要在Gradle文件中添加如下依赖：

```
implementation 'com.squareup.okhttp3:okhttp:${lateset_version}'
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
fun demoOkHttp(url:String,call:okhttp3.Callback){
     val okHttpClient = OkHttpClient()
     val request = Request.Builder()
                .url(url)
                .build()
     okHttpClient.newCall(request).enqueue(call)
     //enqueue()方法内部开启子线程
     //注意：子线程内部不能直接使用return返回数据，必须使用回调机制
}

//调用：
DemoHttpClass.demoOkHttp(url,object:okhttp3.Callback { //利用回调机制将响应数据返回给调用方
     override fun onFailure(call:Call, e:IOException) {
        //TODO
     }
     override fun onResponse(call:Call, response:Response) {
        //TODO
     }
})
```

这里只是简单介绍了一下OkHttp的基本用法。总的来说，OkHttp的封装性很高，有许多人性化的设计， 使得开发者可以通过很少的代码来实现比较复杂的网络操作，这也是它能够成为同类型开源库的首选的重要原因。