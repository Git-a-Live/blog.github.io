## Modifier

在使用Compose进行UI开发的过程中，开发者会经常看到一个熟悉的身影：`Modifier`。可以这么说，它几乎遍布所有组件和布局的入参，并发挥着不可小视的样式配置作用。本节内容将对`Modifier`这个工具展开介绍。

### 何为Modifier

顾名思义，`Modifier`就是一个样式效果修改器，其源码文件位于`androidx.compose.ui`包下。它既可以给组件和布局添加上额外的样式效果，也能够加上行为监听，几乎所有的Compose界面元素都可以通过它来进行自定义。在Compose源码当中，`Modifier`其实是一个接口，但它还对外提供了一个同名的伴生对象，作为默认值来使用。`Modifier`有一个`CombinedModifier`子类，从名字上就能看出它的主要作用是什么。尽管`Modifier`是一个接口，内部总共也就提供了5个API，但这丝毫不影响它发挥作用。因为Compose开发团队为`Modifier`编写了大量扩展函数，而它们就是开发者所经常调用的东西。

### Modifier的扩展函数

下表是从Compose源码当中所整理出来的，一些常用的`Modifier`扩展函数。

|函数名|用途备注|
|:-----:|:-----:|
|`width()`|单独设置Compose界面元素的宽度，通常传入`Dp`类型参数|
|`height()`|单独设置Compose界面元素的高度，通常传入`Dp`类型参数|
|`size()`|一次性设置Compose界面元素的宽高，通常传入`Dp`类型参数|
|`widthIn()`|单独设置Compose界面元素的宽度范围，通常传入两个分别代表最大值和最小值的`Dp`类型参数|
|`heightIn()`|单独设置Compose界面元素的高度范围，通常传入两个分别代表最大值和最小值的`Dp`类型参数|
|`sizeIn()`|一次性设置Compose界面元素的宽高范围，通常传入四个分别代表宽高最值的`Dp`类型参数|
|`requiredWidth()`|强制要求Compose界面元素保持指定宽度，不受任何测量方式的影响。传入`Dp`类型参数|
|`requiredHeight()`|强制要求Compose界面元素保持指定高度，不受任何测量方式的影响。传入`Dp`类型参数|
|`requiredSize()`|强制要求Compose界面元素保持指定宽高，不受任何测量方式的影响。传入`Dp`类型参数|
|`fillMaxWidth()`|设置Compose界面元素的宽度，填满包裹该元素作用域的宽度|
|`fillMaxHeight()`|设置Compose界面元素的高度，填满包裹该元素作用域的高度|
|`fillMaxSize()`|设置Compose界面元素填满包裹该元素作用域|
|`wrapContentWidth()`|设置包裹Compose界面元素的作用域宽度，同内部包裹元素的最大宽度一致|
|`wrapContentHeight`|设置包裹Compose界面元素的作用域高度，同内部包裹元素的最大高度一致|
|`wrapContentSize()`|设置包裹Compose界面元素的作用域宽高，仅刚好包裹内部元素|
|`offset()`|设置Compose界面元素在水平和竖直方向上的位置偏移量，受LTR/RTL方向设置的影响，同一个偏移量可能使得元素向右移（LTR）或向左移（RTL）。偏移量单位可能是dp，也可能是px|
|`absoluteOffset()`|设置Compose界面元素在水平和竖直方向上的绝对位置偏移量，全部按照LTR方向进行平移。偏移量单位可能是dp，也可能是px|
|`padding()`|设置Compose界面元素上下左右的边距，起始位置和末尾位置受LTR/RTL设置影响。根据不同的实现，可能需要分别传上下左右四个边距参数，也可能只需要传一个统一的边距值|
|`absolutePadding()`|设置Compose界面元素上下左右的边距，起始位置和末尾位置全部按照LTR方向来设置，目前仅支持传入上下左右四个边距参数|
|`alpha()`|设置Compose界面元素的透明度，范围[0f, 1f]|
|`aspectRatio()`|设置Compose界面元素的宽高比|
|`background()`|设置Compose界面元素的背景样式|
|`blur()`|设置Compose界面元素的模糊效果，从Android 12起才支持|
|`border()`|设置Compose界面元素的边缘样式|
|`clickable()`|设置Compose界面元素被点击时的动作|
|`clip()`|将Compose界面元素裁剪成制定形状|
|`clipToBounds()`|将Compose界面元素按包裹该元素的作用域形状进行裁剪|
|`animateContentSize()`|为包裹Compose界面元素的作用域，在其随内部元素尺寸发生变化时添加平滑的动画效果|
|`captionBarPadding()`|在应用界面上为caption bar这类系统UI添加边距，以便能够将其容纳|
|`navigationBarsPadding()`|在应用界面上为导航栏这类系统UI添加边距，以便能够将其容纳|
|`imePadding()`|在应用界面上为输入法界面添加边距，以便能够将其容纳，而不是遮挡一些重要的界面元素|
|`statusBarsPadding()`|在应用界面上为状态栏这类系统UI添加边距，以便能够将其容纳|
|`paddingFromBaseline()`|为Compose界面元素设置上下边距，参照基准为包裹该元素的作用域顶部和底部|
|`weight()`|为Compose界面元素在Column和Row布局中设置权重，从而自动分配宽高尺寸|
|`rotate()`|设置Compose界面元素围绕自己中心旋转一定角度，单位为°。非负数表示顺时针旋转，反之为逆时针旋转|
|`scale(）`|设置Compose界面元素的缩放比例|
|`shadow()`|设置Compose界面元素的阴影效果|
|`horizontalScroll()`|当Compose界面元素的内容超过其宽度所能展示的范围时，允许内容水平滑动展示|
|`verticalScroll()`|当Compose界面元素的内容超过其高度所能展示的范围时，允许内容上下滑动展示|
|`matchParentSize()`|设置被Box布局包裹的Compose界面元素，其宽高尺寸与Box布局一致|

## MaterialTheme

## CompositionLocal