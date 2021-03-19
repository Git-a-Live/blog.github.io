WebView是Android开发当中一个十分重要的组件，主要作为内嵌浏览器使用。 当然，也可以基于WebView开发一个简易的自定义浏览器，至于为什么是简易而不是复杂完整的浏览器， 主要原因在于WebView还是有不能满足开发者需求的地方，一些第三方的WebView开源库更具优势， 因此，如果现在想深入开发一个功能完善的自定义浏览器，那么最好是选用第三方开源库。

由于使用WebView通常要联网，因此需要先在AndroidManifest.xml文件中声明网络权限：

```
<uses-permission android:name="android.permission.INTERNET"/>
```

WebView的创建很简单，只要从组件库里把它拖到UI界面上，设置好位置和大小即可。 如果仅仅是让WebView显示一个网页，那就更加简单了，只要调用WebView控件的loadUrl()方法， 传入一个http或https协议的网址字符串，编译运行之后应用界面上就会加载出指定的网页来。

但是当用户试图点击网页内的其他链接查看内容时，却发现应用自动跳转到了系统默认浏览器，刚刚点击的链接则是在外部浏览器中被打开。 出现这种情况的原因在于，上文创建流程中，WebView并没有被赋予处理其他链接的权限和功能，除了加载指定网址的页面之外，其他什么都不能做。

想让应用内嵌的WebView处理其他的链接，首先要给WebView设置一个自定义的WebViewClient，并覆写其中的shouldOverrideUrlLoading()方法：

```
webView.loadUrl("URL") //加载指定链接
webView.webViewClient = object : WebViewClient() {
     override fun shouldOverrideUrlLoading (
         view: WebView?,
         request: WebResourceRequest?
     ): Boolean {
         view?.loadUrl(request?.url.toString()) //当前链接由WebView进行处理
         return true
     }
}
```

通过这部分代码，WebView获得了处理其他链接的权限和功能，而不是只能跳转到系统浏览器去打开页面。 更为便利的是，WebView自带滚动显示功能，当加载出来的网页内容超出应用界面范围时，只要用户尝试滑动界面查看更多内容， WebView的右侧就会自动显示滑动条，以提示用户当前页面可以滚动查看。

目前绝大多数网页为了美观和提供交互功能，在开发中都使用了javaScript，因此需要为WebView设置允许javaScript交互。 具体设置方法为webView.settings.javaScriptEnabled = true，在设置完以后，Android Studio会提示这样做存在风险， 会导致XXS漏洞，容易被黑客利用。如果确定不会在页面中输入敏感信息，那么可以按照提示添加一个@SuppressLint注解，这样就不会再有相关警告了。 webView.settings还有很多属性可以进行设置，限于篇幅在这里略过。

事实上，Android原生的WebView只能处理http或https协议的网址链接，其他非http或非https的链接会导致"Webpage not available"错误， 因此还需要在创建WebViewClient实例时加入判断：若链接为http或https协议，则由WebView处理，否则交由外部应用处理。 如何让外部应用处理？答案就是使用Intent，具体的使用方式如下：

```
webView.loadUrl("URL")
webView.webViewClient = object : WebViewClient(){
     override fun shouldOverrideUrlLoading(
         view: WebView?,
         request: WebResourceRequest?
     ): Boolean {
         val url = request?.url.toString()
         if (!url.startsWith("http") || !url.startsWith("https")){
              val intent = Intent(Intent.ACTION_VIEW,Uri.parse(url)) //设置intent执行的动作和uri
              if (intent.resolveActivity(packageManager) == null){ //若设备上没有安装相关应用，则执行其他动作
                   //TODO
              } else {
                   ···
                   startActivity(intent) /*若设备上安装有相关应用，则跳转到该应用*/
              }
         }
         view?.loadUrl(request?.url.toString())
         return true
     }
}
```

接下来再介绍几个常用的方法。

首先是onPageStarted()和onPageFinished()。这两个方法主要用于在加载页面和页面加载完成的时候执行某些动作， 最常用的一个用途就是显示进度条，当然显示进度条必须放到子线程里面去异步执行，否则会触发ANR。

其次是setDownloadListener，这个方法可以通过Intent调用系统浏览器去进行下载。

对于浏览器来说，仅仅显示页面和调用其他应用可能还不够，还得有一些基本功能，比如页面前进、后退， 这时就可以先利用canGoForward()和canGoBack()判断页面是否可以前进或后退，如果可以，接着就分别执行goForward()和goBack()， 来实现网页页面的前进和后退。