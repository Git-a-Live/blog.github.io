Coloum和Row都是Compose当中最基础的布局组件，前者的作用是让包裹在其中的其他Compose组件以竖直方式自上而下排列，而后者则是让包裹其中的其他Compose组件以水平方式自左向右排列。它们的声明API实际上极为相似，可以通过下面的源码看出来：

```
// Column布局的声明API
@Composable
inline fun Column(
    modifier: Modifier = Modifier,
    // 用于设置布局内子项在竖直方向上的排布位置
    verticalArrangement: Arrangement.Vertical = Arrangement.Top,
    // 用于设置布局内子项在水平方向上的排布位置
    horizontalAlignment: Alignment.Horizontal = Alignment.Start,
    content: @Composable ColumnScope.() -> Unit
) { ··· }

// Row布局的声明API
@Composable
inline fun Row(
    modifier: Modifier = Modifier,
    // 用于设置布局内子项在水平方向上的排布位置
    horizontalArrangement: Arrangement.Horizontal = Arrangement.Start,
    // 用于设置布局内子项在竖直方向上的排布位置
    verticalAlignment: Alignment.Vertical = Alignment.Top,
    content: @Composable RowScope.() -> Unit
) { ··· }
```

如果要在传统的Android UI开发中，给Column和Row找一个对应的东西，大概就是LinearLayout了。实际上，基于Column和Row编写Compose界面的过程，确实跟早期Android开发在`.xml`文件中疯狂嵌套LinerLayout排布控件差不多。不过，传统的View系统在嵌套大量LinearLayout之后很快就会遭遇到性能问题，而Compose则几乎不受影响。此外，Column和Row布局的真正想要高度自定义，最终必须依靠`Modifier`，并且需要提醒的是，某些`Modifier`参数可能还会覆盖掉先前在布局或组件中已经设好的配置。