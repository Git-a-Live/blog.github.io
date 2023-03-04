BottomNavigation组件大致对标的是传统的TabLayout，只不过它的名称强调了它通常只是用于底部导航。按照官方注释的说法，BottomNavigation组件“允许用户在一个应用程序内的主要目的地之间移动”，也就是以往应用底部导航栏所执行的功能。BottomNavigation并非单打独斗，而是要配合Scaffold以及BottomNavigationItem来使用。

```
@Composable
fun BottomNavigation(
    modifier: Modifier = Modifier,
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: Dp = BottomNavigationDefaults.Elevation,
    content: @Composable RowScope.() -> Unit
) { ··· }
```

BottomNavigation组件的声明API较为简单，这里不做展开，主要介绍BottomNavigationItem。BottomNavigationItem的声明API是一个RowScope的扩展函数，这表明它只能在Row布局作用域中使用。

```
@Composable
fun RowScope.BottomNavigationItem(
    // 订阅一个状态以判断当前子项是否被选中
    selected: Boolean,
    // 设置子项被选中（点击）时的动作，一般是执行界面跳转
    onClick: () -> Unit,
    // 子项的Icon，一般位于文本标签的上方
    icon: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
    // 为子项设置文本标签
    label: @Composable (() -> Unit)? = null,
    // 设置为false时，子项的文本标签只会在被点击选中时才显示
    alwaysShowLabel: Boolean = true,
    interactionSource: MutableInteractionSource = remember { MutableInteractionSource() },
    // 子项被选中时的Icon和文本标签颜色
    selectedContentColor: Color = LocalContentColor.current,
    // 子项未被选中时的Icon和文本标签颜色
    unselectedContentColor: Color = selectedContentColor.copy(alpha = ContentAlpha.medium)
) { ··· }
```

由于BottomNavigation组件一般不会只有一个BottomNavigationItem（否则还要什么页面导航），因此在BottomNavigation的content作用域里，开发者可以选择逐个手写BottomNavigationItem，也可以利用集合类迭代设置BottomNavigationItem，如下面代码所示：

```
val items = listOf(···)
Scaffold(
    ···
    bottomBar = {
        BottomNavigation {
            items.forEachIndexed { index, item ->
                BottomNavigationItem(···)
            }
        }
    }
) { ··· }
```

除此之外，Compose官方对于BottomNavigationItem的行为样式也有一些建议：

+ 如果有**三个**跳转目的地，那么就保持展示各个子项的Icon和文本标签；

+ 如果有**四个**跳转目的地，被选中的子项展示Icon和文本标签，未被选中的子项可以只展示Icon，但也**建议**展示文本标签；

+ 如果有**五个**跳转目的地，被选中的子项展示Icon和文本标签，未被选中的子项只展示Icon，文本标签**只在空间充足的情况下**展示。