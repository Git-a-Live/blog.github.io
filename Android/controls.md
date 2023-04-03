## RecyclerView

在应用中查看由大量数据组成的列表时，由于屏幕尺寸的限制，不可能将这些数据一次性全部展示。因此，开发者往往采用翻页或滚动的方式来呈现数据。这里只讨论滚动方式。

Android Studio提供了一些可以实现页面滚动的组件，最主要的是ListView、ScrollView和Recyclerview。ListView是一个相对较老的控件，它使用到Adapter这个类，可以在早期的Android开发中实现MVC架构；ScrollView通常用于实现简单的滚动效果，一般做法是直接在ScrollView中嵌套一个子元素。 还有两个组件与它相似，分别是HorizontalScrollView（水平滑动）以及NestedScrollView（支持嵌套滑动）；RecyclerView出现于Android 5.0，目前可以在很大程度上替代ListView，接下来会对它进行详细介绍。

### RecyclerView

和ListView相比，RecyclerView具有如下优势：

1. 封装了ViewHolder的回收复用，写起来更加简单；
2. 高度解耦，使用灵活，扩展性强，可简便控制Item的显示方式和样式。

RecyclerView主要由LayoutManager（管理Item的布局）、Adapter（为Item提供数据）、 Item Decoration（提供Item之间的分割线）、 Item Animator（添加、删除Item动画）四个部分组成。其中，LayoutManager和Adapter是必须使用的组件，其他两个可以视情况选用。

LayoutManager提供了三种布局管理，分别是LinerLayoutManager（以垂直或者水平列表方式展示Item）、GridLayoutManager（以网格方式展示Item）， 以及StaggeredGridLayoutManager（以瀑布流方式展示Item）。LayoutManager提供了如下常见API：

|方法名|用途|
|:-----:|:-----:|
|`canScrollHorizontally()`|设置能否横向滚动|
|`canScrollVertically()`|设置能否纵向滚动|
|`scrollToPosition()`|设置滚动到指定位置|
|`setOrientation()`|设置滚动的方向|
|`getOrientation()`|获取滚动方向|
|`findViewByPosition()`|获取指定位置的Item View|
|`findFirstCompletelyVisibleItemPosition()`|获取第一个完全可见的Item位置|
|`findFirstVisibleItemPosition()`|获取第一个可见Item的位置|
|`findLastCompletelyVisibleItemPosition()`|获取最后一个完全可见的Item位置|
|`findLastVisibleItemPosition()`|获取最后一个可见Item的位置|

LinerLayoutManager在LayoutManager的基础上还提供了`canScrollVertically()`以及`canScrollHorizontally()`等常用方法；GridLayoutManager继承于LinerLayoutManager，在使用上差别不大；StaggeredGridLayoutManager的使用在网上也有很多资料可查，在此略过。

### RecyclerView Adapter

Adapter一般通过如下方式进行创建：

```
class DemoAdapter(var dataList: List<T>): RecyclerView.Adapter<DemoAdapter.ViewHolder>() {
    
    class ViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // 使用视图绑定简化控件初始化步骤
        val binding = ViewBinding.bind(itemView)
    }

    // 创建item视图。一旦有了够用的ViewHolder，RecyclerView就会停止调用该方法，
    // 随后回收利用旧的ViewHolder以节约时间和内存
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder = ViewHolder(
        // 注意inflate方法的第三个参数必须是false，否则重复创建视图时会引发应用崩溃
        LayoutInflater.from(parent.context).inflate(R.layout.xxx, parent, false)
    )

    // 查询有多少个待展示的视图
    override fun getItemCount(): Int = dataList.size

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        // 将数据集的数据分别显示到目标位置ViewHolder的特定控件上
        holder.binding.apply {
            // 使用getOrNull和空安全语法，以保证不会因为将空对象传递给控件引发NPE
            dataList.getOrNull(position)?.let {
                ···
            }
        }
    }
}
```
在MainActivity或Fragment当中调用RecyclerView和Adapter的方式如下：

```
val demoAdapter = DemoAdapter()
// 其他LayoutManager的实现子类类似
recyclerView.layoutManager = LinearLayoutManager(context)
recyclerView.adapter = demoAdapter

val demoViewModel = DemoViewModel(application)
demoViewModel.getData().observe(this, Observer {
    // 注意是将ViewModel的数据赋值给Adapter中的数据集
    demoAdapter.data = it
    // 刷新视图上的所有数据内容
    demoAdapter.notifyDataSetChanged()
})
```

## WebView

WebView是Android开发当中一个十分重要的组件，主要作为内嵌浏览器使用。当然，也可以基于WebView开发一个简易的自定义浏览器，至于为什么是简易而不是复杂完整的浏览器，主要原因在于WebView还是有不能满足开发者需求的地方，一些第三方的WebView开源库更具优势。因此，如果现在想深入开发一个功能完善的自定义浏览器，那么最好是选用第三方开源库。

由于使用WebView通常要联网，因此需要先在AndroidManifest.xml文件中声明网络权限：

```
<uses-permission android:name="android.permission.INTERNET"/>
```

WebView的创建很简单，只要从组件库里把它拖到UI界面上，设置好位置和大小即可。 如果仅仅是让WebView显示一个网页，那就更加简单了，只要调用WebView控件的loadUrl()方法，传入一个http或https协议的网址字符串，编译运行之后应用界面上就会加载出指定的网页来。

但是当用户试图点击网页内的其他链接查看内容时，却发现应用自动跳转到了系统默认浏览器，刚刚点击的链接则是在外部浏览器中被打开。出现这种情况的原因在于，上文创建流程中，WebView并没有被赋予处理其他链接的权限和功能，除了加载指定网址的页面之外，其他什么都不能做。

想让应用内嵌的WebView处理其他的链接，首先要给WebView设置一个自定义的WebViewClient，并覆写其中的shouldOverrideUrlLoading()方法：

```
// 加载指定链接
webView.loadUrl("URL")
webView.webViewClient = object : WebViewClient() {
    override fun shouldOverrideUrlLoading (
        view: WebView?,
        request: WebResourceRequest?
    ): Boolean {
        // 当前链接由WebView进行处理
        view?.loadUrl(request?.url.toString())
        return true
    }
}
```

通过这部分代码，WebView获得了处理其他链接的权限和功能，而不是只能跳转到系统浏览器去打开页面。 更为便利的是，WebView自带滚动显示功能，当加载出来的网页内容超出应用界面范围时，只要用户尝试滑动界面查看更多内容， WebView的右侧就会自动显示滑动条，以提示用户当前页面可以滚动查看。

目前绝大多数网页为了美观和提供交互功能，在开发中都使用了javaScript，因此需要为WebView设置允许javaScript交互。 具体设置方法为webView.settings.javaScriptEnabled = true，在设置完以后，Android Studio会提示这样做存在风险， 会导致XXS漏洞，容易被黑客利用。如果确定不会在页面中输入敏感信息，那么可以按照提示添加一个@SuppressLint注解，这样就不会再有相关警告了。 webView.settings还有很多属性可以进行设置，限于篇幅在这里略过。

事实上，Android原生的WebView只能处理http或https协议的网址链接，其他非http或非https的链接会导致"Webpage not available"错误，因此还需要在创建WebViewClient实例时加入判断：若链接为http或https协议，则由WebView处理，否则通过Intent交由外部应用处理：

```
webView.loadUrl("URL")
webView.webViewClient = object : WebViewClient(){
    override fun shouldOverrideUrlLoading(
        view: WebView?,
        request: WebResourceRequest?
    ): Boolean {
         val url = request?.url.toString()
         if (!url.startsWith("http") || !url.startsWith("https")){
            // 设置intent执行的动作和uri
            val intent = Intent(Intent.ACTION_VIEW,Uri.parse(url))
            // 若设备上没有安装相关应用，则执行其他动作
            if (intent.resolveActivity(packageManager) == null){ 
                //TODO
            } else {
                ···
            }
        }
        view?.loadUrl(request?.url.toString())
        return true
    }
}
```

接下来再介绍几个常用的方法。

首先是`onPageStarted()`和`onPageFinished()`。这两个方法主要用于在加载页面和页面加载完成的时候执行某些动作，最常用的一个用途就是显示进度条，当然显示进度条必须放到子线程里面去异步执行，否则会触发ANR。

其次是`setDownloadListener()`，这个方法可以通过Intent调用系统浏览器去进行下载。

对于浏览器来说，仅仅显示页面和调用其他应用可能还不够，还得有一些基本功能，比如页面前进、后退，这时就可以先利用`canGoForward()`和`canGoBack()`判断页面是否可以前进或后退，如果可以，接着就分别执行`goForward()`和`goBack()`，来实现网页页面的前进和后退。

## SurfaceView

## TextureView