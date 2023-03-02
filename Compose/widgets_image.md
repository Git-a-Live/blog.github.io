Image组件的作用类似于ImageView，可以用来加载展示图片。它有三种声明方式，但大同小异，区别基本只集中在传入什么类型的图片资源上：

```
// 传入Painter类型参数的声明式API
@Composable
fun Image(
    painter: Painter,
    contentDescription: String?,
    modifier: Modifier = Modifier,
    alignment: Alignment = Alignment.Center,
    contentScale: ContentScale = ContentScale.Fit,
    alpha: Float = DefaultAlpha,
    colorFilter: ColorFilter? = null
) { ··· }

// 传入ImageBitmap类型参数的声明式API，但其底层实现是基于第一种方案
@Composable
@NonRestartableComposable
fun Image(
    bitmap: ImageBitmap,
    contentDescription: String?,
    modifier: Modifier = Modifier,
    alignment: Alignment = Alignment.Center,
    contentScale: ContentScale = ContentScale.Fit,
    alpha: Float = DefaultAlpha,
    colorFilter: ColorFilter? = null,
    filterQuality: FilterQuality = DefaultFilterQuality
) { 
    val bitmapPainter = remember(bitmap) { BitmapPainter(bitmap, filterQuality = filterQuality) }
    Image(
        painter = bitmapPainter,
        ···
    ） 
}

// 传入ImageVector类型参数的声明式API，但其底层实现还是基于第一种方案
@Composable
@NonRestartableComposable
fun Image(
    imageVector: ImageVector,
    contentDescription: String?,
    modifier: Modifier = Modifier,
    alignment: Alignment = Alignment.Center,
    contentScale: ContentScale = ContentScale.Fit,
    alpha: Float = DefaultAlpha,
    colorFilter: ColorFilter? = null
) = Image(
    painter = rememberVectorPainter(imageVector),
    ···
）
```

可以看到，除去`Modifier`之外，Image组件的声明其实算是比较简单的。下面继续仿照[Text](Compose/widgets_text)，围绕除`Modifier`之外的配置参数来展开内容。

## painter配置

painter配置接收`androidx.compose.ui.graphics.painter.Painter`类型参数，它的来源基本上就是从资源目录读取到的图片：

```
Image(painter = painterResource(id = R.drawable.xxx), contentDescription = null)
```

在加入[Coil](https://coil-kt.github.io/coil/compose/)等图片加载库之后，Image组件就可以动态加载本地或网络图片了，但那得另外展开介绍。

## bitmap配置

bitmap配置接收`androidx.compose.ui.graphics.ImageBitmap`类型参数，主要作用是直接绘制一些图像，比如从一个数组中读取信息绘制出二维码来。

## imageVector配置

imageVector配置接收`androidx.compose.ui.graphics.vector.ImageVector`类型参数，它在使用上并没有`ImageBitmap`那么麻烦，反而还很类似`painterResource`：

```
// 从资源目录下读取矢量图资源
Image(imageVector = ImageVector.vectorResource(id = R.drawable.xxx), contentDescription = null)

// 开发者自己构建矢量图传入使用
Image(imageVector = ImageVector.Builder(···) ··· .build(), contentDescription = null)
```

## contentDescription配置

contentDescription是声明API中唯二需要额外传入的参数，尽管大多数情况下开发者都是传`null`。contentDescription的主要作用是描述这个组件的内容或用途，从而适配Android系统的无障碍服务。如果有需要的话，就填写这个参数，为残障人士用户提供便利。

## alignment配置

alignment配置接收`androidx.compose.ui.Alignment`参数，主要作用就是在给定的组件范围内，为加载的图片设置展示位置。目前Compose内置的`Alignment`参数总计有`Alignment.Center`等15种，除此之外，开发者还可以通过`BiasAlignment`构建自定义的`Alignment`参数（但是通常情况下很少有这种需求）。

```
Image(alignment: Alignment = Alignment.Center)
```

## contentScale配置

contentScale配置接收`androidx.compose.ui.layout.ContentScale`参数，主要作用跟ImageView的scaleType类似，用于设置图片显示宽高比例或裁剪内容范围。Compose内置的`ContentScale`参数总计有`ContentScale.Fit`等7种，开发者还可以通过实现`ContentScale`接口来构建自定义的`ContentScale`参数，但通常情况下没什么必要。

```
Image(contentScale: ContentScale = ContentScale.Fit)
```

## alpha配置

alpha配置用于设置图片（注意不是Image组件本身）的透明度，参数类型为`Float`，范围在0.0 ~ 1.0之间。

```
Image(alpha: Float = 1.0f)
```

## colorFilter配置

colorFilter配置接收`androidx.compose.ui.graphics.ColorFilter`类型参数，其主要作用，按照官方文档注释的说法，是“修改图片上每个像素的颜色”。`ColorFilter`目前对外提供的函数有`tint`、`colorMatrix`以及`lighting`这三个，从它们各自的注释来看，`ColorFilter`总体上似乎是用来给图片加一些简单特效的。不过通常情况下，开发者很少会用到这个配置，因此其默认值是`null`。