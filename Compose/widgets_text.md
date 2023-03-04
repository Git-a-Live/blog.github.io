Text是Compose中最基础的组件之一，主要作用跟传统的TextView类似，但其底层实现却不是基于TextView。

> 注意：Compose组件不是在传统控件的基础上封装一套东西，而是从头实现了一整套全新的界面元素绘制方案。传统控件在Compose当中可能有对等的实现，但它们在本质上终究是完全不同的。

Text完整的Composable函数入参配置如下面代码所示：

```
@Composable
fun Text(
    text: String,
    modifier: Modifier = Modifier,
    color: Color = Color.Unspecified,
    fontSize: TextUnit = TextUnit.Unspecified,
    fontStyle: FontStyle? = null,
    fontWeight: FontWeight? = null,
    fontFamily: FontFamily? = null,
    letterSpacing: TextUnit = TextUnit.Unspecified,
    textDecoration: TextDecoration? = null,
    textAlign: TextAlign? = null,
    lineHeight: TextUnit = TextUnit.Unspecified,
    overflow: TextOverflow = TextOverflow.Clip,
    softWrap: Boolean = true,
    maxLines: Int = Int.MAX_VALUE,
    onTextLayout: (TextLayoutResult) -> Unit = {},
    style: TextStyle = LocalTextStyle.current
) { ··· }
```

接下来的内容就将围绕着这些参数配置，来对Text这一Compose组件进行详细介绍。由于`Modifier`遍布各个组件的声明API，且功能强大，需要单独开一个章节来介绍如何使用好这个利器，故这里不做展开。

## text配置

text配置有三个来源：1）直接传入字面量；2）从资源目录中提取使用；3）通过数据订阅进行更新。

+ **直接传入字面量**

直接传入字面量的方式最为简单，如下列代码所示：

```
Text(text = "Hello world!")
```

+ **从资源目录中提取使用**

尽管`res/layout`已经没了，但这丝毫不影响其他资源文件的使用：

```
Text(text = stringResource(id = R.string.xxx))
```

+ **通过数据订阅进行更新**

```
val content by remember { mutableStateOf("Hello World") }
Text(text = content)

// 在别的地方调用content重新赋值
content = "Hi Compose"
```

传统的TextView可以通过`Spannable`，在同一段文本中给不同部分赋予不同的效果，这在Compose中也有对应的实现，那就是`AnnotatedString`。`AnnotatedString`的简单用例如下：

```
Text(
    // 使用buildAnnotatedString来构建一段AnnotatedString文本
    text = buildAnnotatedString {
        // append用于为文本划分需要添加不同效果的部分
        append("这是")
        // withStyle函数包裹要添加效果的文本即可
        withStyle(style = SpanStyle(fontWeight = FontWeight.W900)) {
            append("最好的时代")
        }
    }
)
```

## color配置

color配置接收`androidx.compose.ui.graphics.Color`类型的参数，有以下两种构建方式：

```
// 使用预置的颜色值
Text(color = Color.Blue)

// 使用自定义的颜色值
Text(color = Color("···"))
```

## fontSize & letterSpacing & lineHeight配置

fontSize、letterSpacing（字符间距）以及lineHeight（行高间距）配置都接收`androidx.compose.ui.unit.TextUnit`类型的参数，目前有以下两种构建方式：

```
// 为Int/Float/Double类型字号调用扩展函数构建TextUnit
import androidx.compose.ui.unit.sp
Text(
    fontSize = 22.sp, 
    letterSpacing = 7.sp,
    lineHeight = 
)

// 直接构建一个TextUnit对象传入，目前仍为实验性功能，需加上注解
@OptIn(ExperimentalUnitApi::class)
@Composable
fun Foo() {
    Text(
        fontSize = TextUnit(22F, TextUnitType.Sp),
        letterSpacing = TextUnit(7F, TextUnitType.Sp)
    )
}
```

## fontStyle配置

fontStyle配置接收`androidx.compose.ui.text.font.FontStyle`类型参数，目前Compose允许使用的实际上只有两种：`FontStyle.Italic`和`FontStyle.Normal`。

```
Text(fontStyle = FontStyle.Italic)
```

## fontWeight配置

fontWeight配置接收`androidx.compose.ui.text.font.FontWeight`类型参数，用于配置字体的粗细。目前Compose内置有9种字体粗细的配置参数，如果算上它们各自对应的别名，共计有18种，如`FontWeight.W700`和它的别名`FontWeight.Bold`。

```
Text(fontWeight = FontWeight.Bold)
```

## fontFamily配置

fontFamily配置接收`androidx.compose.ui.text.font.FontFamily`类型参数，用于配置字体。目前Compose内置有`FontFamily.Monospace`等四种字体，开发者还可以将其他字体放到`res/font`目录下然后再加载使用。

```
// 使用Compose内置的字体
Text(fontFamily = FontFamily.Monospace)

// 使用资源目录下的字体
Text(fontFamily = FontFamily(Font(R.font.pingfang, FontWeight.W700)))
```

## textDecoration配置

textDecoration配置接收`androidx.compose.ui.text.style.TextDecoration`类型参数，用于给文本添加上一些“装饰”，比如下划线等。目前Compose内置的装饰效果只有三种：`TextDecoration.None`、`TextDecoration.LineThrough`以及`TextDecoration.Underline`。

```
Text(textDecoration = TextDecoration.Underline)
```

## textAlign配置

textAlign配置接收`androidx.compose.ui.text.style.TextAlign`类型参数，用于设置**文本内容在Text组件中**的位置（注意不是Text组件本身在布局中的位置）。目前Compose内置的定位有`TextAlign.Center`等6种。

```
Text(textAlign = TextAlign.Left)
```

## overflow配置

overflow配置接收`androidx.compose.ui.text.style.TextOverflow`类型参数，用于处理文本长度超出限制时的视觉效果。目前Compose内置效果有`TextOverflow.Clip`、`TextOverflow.Ellipsis`以及`TextOverflow.Visible`这三种，分别对应于“超出长度限制就截断”、“超出长度限制就以省略号隐藏后面的内容”还有“在组件边界外继续渲染出超出限制部分的文本内容”三种处理方式。

```
Text(overflow = TextOverflow.Ellipsis)
```

## softWrap配置

softWrap配置接收布尔值，用于配置超长文本是否换行，默认是执行换行操作。

```
Text(softWrap = false)
```

## maxLines配置

maxLines接收Int类型参数，用于配置Text组件最多能容纳的文本行数，默认配置的是Int类型的最大值。

```
Text(maxLines = 6)
```

## onTextLayout配置

onTextLayout接收的是带有一个`TextLayoutResult`类型入参的lambda表达式，按照官方文档的说法，这个是用来做回调的，`TextLayoutResult`类型入参中就包含有段落、文本长度等细节信息，有需要的时候可以调用。

## style配置

style配置接收`androidx.compose.ui.text.TextStyle`类型的参数，用于统一配置Text组件内文本行高、颜色以及粗细等配置，类似于传统控件在`.xml`文件中配置style。Compose的`androidx.compose.material`包中已经内置了一部分style。

```
Text(style = MaterialTheme.typography.body2)
```