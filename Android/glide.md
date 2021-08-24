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