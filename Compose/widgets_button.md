Button组件对应的是传统UI开发中的Button控件，其声明API如下：

```
@OptIn(ExperimentalMaterialApi::class)
@Composable
fun Button(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    elevation: ButtonElevation? = ButtonDefaults.elevation(),
    shape: Shape = MaterialTheme.shapes.small,
    border: BorderStroke? = null,
    colors: ButtonColors = ButtonDefaults.buttonColors(),
    contentPadding: PaddingValues = ButtonDefaults.ContentPadding,
    content: @Composable RowScope.() -> Unit
) { ··· }
```

Compose中基于Button原始声明API衍生出来的Button组件

## onClick配置

onnClick接收一个lambda表达式，其主要作用跟传统Button控件通过`setOnClickListener`去设置点击动作监听器基本一样。

## enabled配置

enabled配置可以传入一个布尔值，从而控制Button组件的使能状态。

## interactionSource配置

interactionSource配置接收`androidx.compose.foundation.interaction.MutableInteractionSource`类型参数。按照官方注释的说法，这个参数表示的是Button组件发射的一系列事件所组成的“交互流”，开发者可以根据这些交互事件的类型，来改变Button组件的外观样式。按照这些信息，大致可以认为interactionSource配置的作用，类似于检测传统Button控件的`isPressed`以及`isSelected`之类的状态，然后根据这些状态去改变控件的外观。

## elevation配置

elevation配置接收`androidx.compose.material.ButtonElevation`参数，默认值是`null`，主要作用是设置Button组件的悬浮状态（以便渲染出阴影）。

## shape配置

shape配置接收`androidx.compose.ui.graphics.Shape`类型参数，默认值是`MaterialTheme.shapes.small`，主要作用是设置Button组件的形状样式。Compose内置Material主题包，就包含有`MaterialTheme.shapes.small`、`MaterialTheme.shapes.medium`以及`MaterialTheme.shapes.large`三种形状样式，供开发者直接使用。

## border配置

border配置接收`androidx.compose.foundation.BorderStroke`类型参数，其主要作用是为Button组件绘制一定样式的边缘。

## colors配置

colors配置接收`androidx.compose.material.ButtonColors`类型参数，其主要作用为一次性配置组件背景颜色和content作用域颜色。Compose内置的`ButtonDefaults.buttonColors()`函数可以用来进行设置。

## contentPadding配置

contentPadding配置接收`androidx.compose.foundation.layout.PaddingValues`类型参数，其主要作用为设置content作用域边界到Button组件边缘的距离。

## content配置

content是一个布局组件作用域，Button组件默认使用的是Row布局，用于水平排布一些图标和文本内容样式。

## IconButton组件

IconButton组件从名字上就能看出它的用途，类似于传统的ImageButton控件。

```
@Composable
fun IconButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    content: @Composable () -> Unit
) { ··· }
```

IconButton需要注意的地方就是content配置，IconButton的icon部分就是在这里添加的，当然开发者也可以添加其他组件，配合onClick操作组合出能执行复杂业务逻辑的IconButton。

## FloatingActionButton组件

FloatingActionButton组件对应的是传统的FloatingActionButton控件，两者作用相似。

```
@OptIn(ExperimentalMaterialApi::class)
@Composable
fun FloatingActionButton(
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    shape: Shape = MaterialTheme.shapes.small.copy(CornerSize(percent = 50)),
    backgroundColor: Color = MaterialTheme.colors.secondary,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: FloatingActionButtonElevation = FloatingActionButtonDefaults.elevation(),
    content: @Composable () -> Unit
) { ··· }
```

FloatingActionButton的声明API跟IconButton相比，多了四个配置参数。在源码当中，这四个参数实际上是给Surface布局使用的，通过配置Surface布局来调整FloatingActionButton的样式。FloatingActionButton组件的elevation参数用于设置悬浮效果，默认值是`FloatingActionButtonDefaults.elevation()`。事实上，`FloatingActionButtonDefaults.elevation()`也包含有默认值，开发者可以通过修改这个函数的入参来调整FloatingActionButton组件的悬浮效果。