Slider组件对应于传统的SeekBBar控件，它的作用也是通过拖动滑块，让用户在一定数值范围内选择一个数值进行配置。

```
@Composable
fun Slider(
    value: Float,
    onValueChange: (Float) -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    valueRange: ClosedFloatingPointRange<Float> = 0f..1f,
    steps: Int = 0,
    onValueChangeFinished: (() -> Unit)? = null,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    colors: SliderColors = SliderDefaults.colors()
) { ··· }
```

## value配置

value表示的是Slider组件的起始值，通常跟下面要介绍到的valueRange搭配使用。如果value的值超出了valueRange的范围，那么它就会失效，Compose会自动将Slider的滑块设置到最值的位置上：若value超过valueRange的最大值，滑块的初始位置就在进度条最大值处，反之在进度条最小值处。

## onValueChange配置

onValueChange用于配置进度条值的监听动作，类似于调用SeekBBar的`setOnSeekBarChangeListener()`。

## valueRange配置

valueRange需要传入一个闭区间的Float数值范围。

## steps配置

用于设置滑块从valueRange最小值到最大值滑动过程中的行为表现。steps最小只能设为0，此时表示滑块的滑动是连续的，没有停顿和突变；若设置的值大于0，则表示将进度条划分为steps + 1等分，滑块每次滑动steps + 1分之一进度条的距离，此时滑块的滑动就是所谓“离散”的。

## onValueChangeFinished配置

onValueChangeFinished用于监听用户停止滑动、选定进度值的操作，如果有业务逻辑需要在这种时机执行，那么传入相应的lambda表达式即可。

## colors配置

colors配置接收`androidx.compose.material.SliderColors`参数，通常使用`SliderDefaults.colors()`进行配置，主要用于设置Slider的滑块、进度条以及进度条背景等颜色。`SliderDefaults.colors()`需要传入的参数如下列代码所示：

```
@Composable
fun colors(
    thumbColor: Color = MaterialTheme.colors.primary,
    disabledThumbColor: Color = MaterialTheme.colors.onSurface
            .copy(alpha = ContentAlpha.disabled)
            .compositeOver(MaterialTheme.colors.surface),
    activeTrackColor: Color = MaterialTheme.colors.primary,
    inactiveTrackColor: Color = activeTrackColor.copy(alpha = InactiveTrackAlpha),
    disabledActiveTrackColor: Color = MaterialTheme.colors.onSurface.copy(alpha = DisabledActiveTrackAlpha),
    disabledInactiveTrackColor: Color = disabledActiveTrackColor.copy(alpha = DisabledInactiveTrackAlpha),
    activeTickColor: Color = contentColorFor(activeTrackColor).copy(alpha = TickAlpha),
    inactiveTickColor: Color = activeTrackColor.copy(alpha = TickAlpha),
    disabledActiveTickColor: Color = activeTickColor.copy(alpha = DisabledTickAlpha),
    disabledInactiveTickColor: Color = disabledInactiveTrackColor.copy(alpha = DisabledTickAlpha)
)
```

thumbColor大致对应于SeekBar控件的`android:thumb`属性；activeTrackColor（滑块已经过区域的颜色）大致对应于`android:progressDrawable`里面的`android:id/progress`属性；而inactiveTrackColor（滑块尚未经过区域的颜色）则大致对应于`android:progressDrawable`里面的`android:id/secondaryProgress`属性。此外，activeTickColor和inactiveTickColor分别表示滑块已经过和尚未经过的刻度颜色。从上面的源码中还可以看出，部分inactiveColor的默认值，实际上只是在activeColor的基础上加了一个透明度的配置，因此有时候想偷懒的话，可以只传activeColor。
