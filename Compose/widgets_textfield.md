TextField组件对应于传统的EditText控件，用于接收用户输入的数据信息。

```
@Composable
fun TextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    textStyle: TextStyle = LocalTextStyle.current,
    label: @Composable (() -> Unit)? = null,
    placeholder: @Composable (() -> Unit)? = null,
    leadingIcon: @Composable (() -> Unit)? = null,
    trailingIcon: @Composable (() -> Unit)? = null,
    isError: Boolean = false,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
    keyboardActions: KeyboardActions = KeyboardActions(),
    singleLine: Boolean = false,
    maxLines: Int = Int.MAX_VALUE,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    shape: Shape =
        MaterialTheme.shapes.small.copy(bottomEnd = ZeroCornerSize, bottomStart = ZeroCornerSize),
    colors: TextFieldColors = TextFieldDefaults.textFieldColors()
) { ··· }
```

## value配置

value配置通常不会写死一个固定值，否则就会强制覆盖掉用户输入的任何内容。一般的做法是订阅一个状态，随用户输入发生变化。

## readOnly配置

按照官方注释的说法，readOnly配置主要用于控制TextField组件是否可以编辑。当设为true时，TextField里面的内容不可修改，但是可以被复制，通常适用于存在不可编辑的预填充内容的场景。

## label配置

label配置是一个Composable作用域，其主要作用是为TextField组件增加一个“标签”装饰，且不仅限于文本。官方在label的注释中提到，当TextField组件获得或失去焦点的时候，label的文本部分会随之出现缩放的动画效果。

## placeholder配置

placeholder配置也是一个Composable作用域，官方注释称它在TextField组件获得焦点且尚未输入内容的时候会显示出来。

## leadingIcon配置

leadingIcon配置同样是一个Composable作用域，它的主要作用就是在TextField组件起始位置添加一些起到装饰作用的内容（包括但不限于图标），比如一个表示搜索含义的放大镜图标等等。

## trailingIcon配置

trailingIcon配置跟leadingIcon的作用一样，只不过是放在TextField组件末尾位置。一些常见的装饰有表示发送内容含义的发送图标，或是一个邮箱后缀文本等等。

## isError配置

isError通常不是直接设置一个固定值来使用的，而是订阅一个状态。当这个状态返回true的时候（比如检测到文本格式不正确），TextField组件里面的label、已经输入的文本内容、trailingIcon甚至包括组件底部的横线等都会改变成相应的颜色，以提示用户所输入的内容存在问题，需要改正。

## visualTransformation配置

visualTransformation配置接收`androidx.compose.ui.text.input.VisualTransformation`类型参数，主要作用是调整TextField组件的内容显示模式。默认值是`VisualTransformation.None`，表示输入内容以明文显示；如果要用于输入密码的场景，可以改用`PasswordVisualTransformation()`，同时还可以传入自定义的字符用于隐藏实际内容。比如Compose默认使用的是`·`，那么文本“1234”就会被转换隐藏为“····”，如果改用`*`，就会显示成“****”。

## keyboardOptions配置

keyboardOptions配置接收`androidx.compose.foundation.text.KeyboardOptions`类型参数，主要用于配置软键盘，默认值为`KeyboardOptions.Default`。`KeyboardOptions`包含四个参数，分别用于设置大小写、自动订正、允许输入的文本类型（比如无限制文本、纯数字、邮箱或电话号码等）以及输入法点击回车键时执行的动作（比如搜索或发送等）。

## keyboardActions配置

keyboardActions配置接收`androidx.compose.foundation.text.KeyboardActions`类型参数，主要作用是让开发者对`KeyboardOptions`当中点击回车键所执行的动作进行自定义设置。`KeyboardActions`传入的参数全都是lambda表达式，比如`onDone`、`onGo`、`onSearch`还有`onSend`等等。

## singleLine配置

singleLine配置主要是让开发者根据实际使用场景，来设置是否允许TextField组件换行显示内容。按照官方注释的说法，如果设置为单行显示，那么软键盘就不会有回车键出现，并且下面要提到的maxLines会被强制设置成1。

## maxLines配置

maxLines配置主要是用于限制用户输入内容的**可视行数**（同时也可能是TextField组件的高度）。例如，当用户输入的文本内容超过TextField单行所能显示的长度时，TextField会自动进行换行处理；若换行数达到maxLines设置的上限值仍不能将内容展示完全，则TextField会自动执行滚动操作，将先前输入的内容滚动移出可视范围，确保后面输入的内容能始终处在可视范围内。此外，用户还可以通过手动触摸滑动方式，查看被移出可视范围的文本内容。

## colors配置

colors配置接收`androidx.compose.material.TextFieldColors`类型参数，主要作用是集中配置Textfield组件当中文本、背景以及光标等元素的颜色。默认值是`TextFieldDefaults.textFieldColors()`，但是开发者可以向这个函数传入自定义参数来进行调整，具体参数和含义可以参考下面的代码：

```
@Composable
fun textFieldColors(
    // 输入的文字颜色
    textColor: Color = LocalContentColor.current.copy(LocalContentAlpha.current),

    // 禁用 TextField 时，已有的文字颜色
    disabledTextColor: Color = textColor.copy(ContentAlpha.disabled),

    // 输入框的背景颜色，当设置为 Color.Transparent 时，将透明
    backgroundColor: Color = MaterialTheme.colors.onSurface.copy(alpha = BackgroundOpacity),

    // 输入框的光标颜色
    cursorColor: Color = MaterialTheme.colors.primary,

    // 当 TextField 的 isError 参数为 true 时，光标的颜色
    errorCursorColor: Color = MaterialTheme.colors.error,

    // 当输入框处于焦点时，底部指示器的颜色
    focusedIndicatorColor: Color = MaterialTheme.colors.primary.copy(alpha = ContentAlpha.high),

    // 当输入框不处于焦点时，底部指示器的颜色
    unfocusedIndicatorColor: Color = MaterialTheme.colors.onSurface.copy(alpha = UnfocusedIndicatorLineOpacity),

    // 禁用 TextField 时，底部指示器的颜色
    disabledIndicatorColor: Color = unfocusedIndicatorColor.copy(alpha = ContentAlpha.disabled),

    // 当 TextField 的 isError 参数为 true 时，底部指示器的颜色
    errorIndicatorColor: Color = MaterialTheme.colors.error,

    // TextField 输入框前头的颜色
    leadingIconColor: Color = MaterialTheme.colors.onSurface.copy(alpha = IconOpacity),

    // 禁用 TextField 时 TextField 输入框前头的颜色
    disabledLeadingIconColor: Color = leadingIconColor.copy(alpha = ContentAlpha.disabled),

    // 当 TextField 的 isError 参数为 true 时 TextField 输入框前头的颜色
    errorLeadingIconColor: Color = leadingIconColor,

    // TextField 输入框尾部的颜色
    trailingIconColor: Color = MaterialTheme.colors.onSurface.copy(alpha = IconOpacity),

    // 禁用 TextField 时 TextField 输入框尾部的颜色
    disabledTrailingIconColor: Color = trailingIconColor.copy(alpha = ContentAlpha.disabled),

    // 当 TextField 的 isError 参数为 true 时 TextField 输入框尾部的颜色
    errorTrailingIconColor: Color = MaterialTheme.colors.error,

    // 当输入框处于焦点时，Label 的颜色
    focusedLabelColor: Color = MaterialTheme.colors.primary.copy(alpha = ContentAlpha.high),

    // 当输入框不处于焦点时，Label 的颜色
    unfocusedLabelColor: Color = MaterialTheme.colors.onSurface.copy(ContentAlpha.medium),

    // 禁用 TextField 时，Label 的颜色
    disabledLabelColor: Color = unfocusedLabelColor.copy(ContentAlpha.disabled),

    // 当 TextField 的 isError 参数为 true 时，Label 的颜色
    errorLabelColor: Color = MaterialTheme.colors.error,

    // Placeholder 的颜色
    placeholderColor: Color = MaterialTheme.colors.onSurface.copy(ContentAlpha.medium),

    // 禁用 TextField 时，placeholder 的颜色
    disabledPlaceholderColor: Color = placeholderColor.copy(ContentAlpha.disabled)
)
```

## BasicTextField组件

如果想要更高程度的自定义效果，开发者还可以选用BasicTextField组件：

```
@Composable
fun BasicTextField(
    value: String,
    onValueChange: (String) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    readOnly: Boolean = false,
    textStyle: TextStyle = TextStyle.Default,
    keyboardOptions: KeyboardOptions = KeyboardOptions.Default,
    keyboardActions: KeyboardActions = KeyboardActions.Default,
    singleLine: Boolean = false,
    maxLines: Int = Int.MAX_VALUE,
    visualTransformation: VisualTransformation = VisualTransformation.None,
    // 当输入框内文本触发更新时候的回调，包括了当前文本的各种信息
    onTextLayout: (TextLayoutResult) -> Unit = {},
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    // 输入框光标的颜色
    cursorBrush: Brush = SolidColor(Color.Black),
    // 是一个允许在 TextField 周围添加修饰的 @Composable lambda表达式，需要在布局中调用 innerTextField() 才能完成 TextField 的构建
    decorationBox: @Composable (innerTextField: @Composable () -> Unit) -> Unit =
        @Composable { innerTextField -> innerTextField() }
) { ··· }
```