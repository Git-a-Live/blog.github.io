AlertDialog组件是一个全新的实现。在传统的Android UI开发中，编写自定义弹窗的繁琐程度跟RecyclerView的Adapter有得一拼，而且复用性也不怎么好。但是在Compose当中，开发团队提供了尽可能简单的声明API，让开发者能够更轻松地上手使用弹窗组件。

```
// 一种声明方式，可以高度自定义按钮布局
@Composable
fun AlertDialog(
    onDismissRequest: () -> Unit,
    buttons: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    title: (@Composable () -> Unit)? = null,
    text: @Composable (() -> Unit)? = null,
    shape: Shape = MaterialTheme.shapes.medium,
    backgroundColor: Color = MaterialTheme.colors.surface,
    contentColor: Color = contentColorFor(backgroundColor),
    properties: DialogProperties = DialogProperties()
) { ··· }

// 另一种声明方式，只能分别自定义确认和取消按钮布局
@Composable
fun AlertDialog(
    onDismissRequest: () -> Unit,
    confirmButton: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    dismissButton: @Composable (() -> Unit)? = null,
    title: @Composable (() -> Unit)? = null,
    text: @Composable (() -> Unit)? = null,
    shape: Shape = MaterialTheme.shapes.medium,
    backgroundColor: Color = MaterialTheme.colors.surface,
    contentColor: Color = contentColorFor(backgroundColor),
    properties: DialogProperties = DialogProperties()
) { ··· }

// Dialog的底层实现
@Composable
fun Dialog(
    onDismissRequest: () -> Unit,
    properties: DialogProperties = DialogProperties(),
    content: @Composable () -> Unit
) { ··· }
```

## onDismissRequest配置

onDismissRequest对应于传统Dialog当中的`setOnCancelListener`，主要作用就是配置用户在点击外围界面关闭弹窗时，所要执行的动作。

## buttons配置

buttons是一个Composable作用域，主要用来给开发者自定义弹窗按钮的数量样式等，自由度较高。

## confirmButton / dismissButton配置

confirmButton和dismissButton同样是Composable作用域，但是只能自定义预置好的确认按钮和取消按钮的样式。默认情况下，AlertDialog会尝试把确认和取消按钮水平地放在彼此的旁边，如果没有足够的水平空间，则采用竖直上下放置。

## title / text配置

title的Composable作用域主要用来配置弹窗的标题栏，而text则用来配置弹窗的内容部分。

## properties配置

properties配置接收`androidx.compose.ui.window.DialogProperties`类型参数，主要作用是供开发者自定义弹窗的行为，比如进行返回操作时是否关闭弹窗（默认为true）、点击弹窗外围界面时是否关闭弹窗（默认为true）、弹窗的安全策略（如是否允许在截图时展示弹窗内容等）以及是否限制弹窗内容宽度小于屏幕宽度（默认为true）。

## content配置

content作用域里面就是完全的自定义弹窗界面了，这就是为什么Dialog声明API会属于最底层的缘故。