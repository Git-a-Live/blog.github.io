[Glide](https://github.com/bumptech/glide)是一个快速高效的开源媒体管理和图片加载框架，以简便的接口方式封装有多媒体解码、内存与磁盘缓存以及资源池，适用于Android设备。它可以从多个源去加载和显示图片，同时也兼顾缓存和在做图片处理的时候维持一个低内存消耗。而说到图片加载框架，可能还得提到Google官方自己的[Volley](/Android/volley)，但是这两者的侧重点其实不太一样，所以Google在开发者大会上能够很大方地推荐使用Glide。然而，Glide开发团队提醒开发者如果把Glide跟Volley一起使用，会发生大量[内存抖动](/Android/perf?id=内存溢出)（🤣）。

## 准备工作

在Android开发中使用Glide，首先要添加依赖：

```
implementation 'com.github.bumptech.glide:glide:$specific_version'
annotationProcessor 'com.github.bumptech.glide:compiler:$specific_version'
```

>注意，截至2021年8月24日，Glide最新版本为4.12.0。

## 基本使用

Glide最基本的调用方式如下：

```
Glide.with(context)
    .load(uri/url/resId/drawable/file)
    .into(imageView)
```

可以看到，Glide所加载图片的来源确实很多，无论是否位于本地设备上，只要能访问得到就基本上可以加载进来，而且还可以指定相应的ImageView控件负责展示图片，使用起来还是比较方便的。

## 进阶使用

Glide的方法基本上都能采用链式调用，除去最基础的`with()`、`load()`以及`into()`之外，还有以下常用方法可以按照开发需要进行调用：

|方法|用途|示例说明|
|:-----|:---------|:---------|
|`placeholder()`|加载未完成状态图片|`placehoder(drawable/resId)`，仅支持本地资源|
|`error()`|加载错误状态图片|`error(drawable/resId)`，建议只用本地资源|
|`crossFade()`|淡入效果动画|`crossFade(time)`，默认动画维持300ms，可以传入时间参数来缩短或延长动画效果|
|`dontAnimate()`|不显示动画|直接显示图片，没有动画效果|
|`override()`|设置图片大小|`override(horizontalSize, verticalSize)`，若图片没有自动适配到Glide就使用该方法|
|`centerCrop()`|图像裁剪|将图像裁剪至完全填充ImageView的程度，但可能导致显示不完整|
|`fitCenter()`|图像缩放|将图像缩放至能在ImageView完全显示的程度，但可能不会完全填充|
|`asGif`|展示Gif图片|建议在判断所加载图片确实为Gif的情况下调用，且必须紧跟`with()`调用|
|`asBitmap()`|展示静态图片|在展示Gif图片可能引起OOM的情况下调用，且必须紧跟`with()`调用|
|`asDrawable()`|展示Drawable资源|必须紧跟`with()`调用|
|`diskCacheStrategy()`|设置磁盘缓存策略|`diskCacheStrategy(DiskCacheStrategy.ALL)`，具体策略可查看DiskCacheStrategy里面的枚举值|
|`skipMemoryCache()`|设置是否禁用内存缓存|`skipMemoryCache(true)`，在图片来源固定、但需要频繁更新的场景中使用|
|`apply()`|设置可复用的配置|`apply(option)`，传入的是一个RequestOptions对象，该对象也能调用上述部分方法|

更多使用方法可以参考Glide官方的[开发文档](http://bumptech.github.io/glide/)。

## Glide缓存机制

Glide在请求和加载图片时使用到了缓存机制。缓存机制在节约流量以及内存方面具有重要的意义，尤其是在重复加载相同资源的情况下，通过缓存可以减少不必要的网络请求，进而提高加载速度，提升用户体验。Glide使用的缓存机制分为内存缓存和磁盘缓存两方面，前者的主要作用是**防止应用重复将图片数据读取到内存中**，后者的主要作用是**防止应用从网络等地方重复下载和读取图片数据**。无论是内存缓存还是磁盘缓存，都用到了LruCache算法。

> LruCache算法的基本原理为：维护一个缓存对象列表，并依照访问顺序来决定这些缓存对象的排列顺序。最近访问的对象将被排到队列头部，一直未被访问的对象被排到队尾。排在队尾的缓存对象通常就是等待被清理的。Android当中的LruCache算法实现，利用的是LinkedHashMap这一数据结构，它可以直接输出最近访问的结果列表，这样就极大降低了实现难度。

在请求新的图片资源之前，Glide会进行逐级检查，过程大致如下：

1. 检查该图片最近是否已经加载过，并仍存在于内存当中；
2. 检查该图片现在是否仍被控件展示（弱引用）；
3. 检查该图片之前是否已被编码、转换并写入磁盘缓存；
4. 检查构建该图片的资源之前是否被写入过文件缓存；
5. 上述步骤未能找到图片时，再从指定来源请求加载相应的图片资源。

从上面的步骤中可以大致了解到Glide读取图片的顺序：1）内存；2）弱引用；3）磁盘。这里也顺便介绍Glide保存图片的顺序：1）弱引用；2）内存；3）磁盘。

为了尽可能节约内存，Glide还有一个`Bitmap`复用机制。该机制建立在`BitmapFactory.Options.inMutable`的使用上，当设置这一属性为true时，相应的`Bitmap`对象就会被设为可复用的。

### 内存缓存

内存缓存分为两级：Lru（Least Recently Used）Cache缓存和弱引用缓存。前者利用LruCache算法清理旧缓存，后者则是通过弱引用来保护正在使用的资源不被LruCache算法清理掉。

### 磁盘缓存

Glide在实施磁盘缓存的时候，会默认在[应用内部存储空间](Android/io2?id=内部存储空间)的`cache`目录下创建一个缓存路径，通常命名为`image_manager_disk_cache`，大小为250 MB。