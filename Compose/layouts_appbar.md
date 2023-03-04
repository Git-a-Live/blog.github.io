正如它的名字一样，TopAppBar组件的主要用途就是给应用提供一个符合Material Design风格的自定义顶部栏。传统的Android UI开发中，具有类似功能的玩意儿被称为“ActionBar”。但是要想自定义，一般得先在代码中移除掉系统默认的ActionBar，然后编写一个自定义View放在界面顶端充当ActionBar。Compose团队考虑到这一点，于是为开发者提供了一个现成的布局来使用。

```
// 可高度自定义的声明API
@Composable
fun TopAppBar(
    modifier: Modifier = Modifier,
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: Dp = AppBarDefaults.TopAppBarElevation,
    contentPadding: PaddingValues = AppBarDefaults.ContentPadding,
    content: @Composable RowScope.() -> Unit
) { ··· }

// 固定样式的声明API
@Composable
fun TopAppBar(
    // TopAppBar的标题
    title: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    // 通常设置一个用于返回上一级页面的IconButton
    navigationIcon: @Composable (() -> Unit)? = null,
    // 通常用在TopAppBar末尾处设置若干提供其他功能的IconButton
    actions: @Composable RowScope.() -> Unit = {},
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    elevation: Dp = AppBarDefaults.TopAppBarElevation
) { ··· }
```

既然应用顶部有TopAppBar，那么应用底部是否有对应的BottomAppBar？答案是“有的”。Compose团队在`androidx.compose.material.AppBar.kt`文件中确实提供了这个布局：

```
@Composable
fun BottomAppBar(
    modifier: Modifier = Modifier,
    backgroundColor: Color = MaterialTheme.colors.primarySurface,
    contentColor: Color = contentColorFor(backgroundColor),
    cutoutShape: Shape? = null,
    elevation: Dp = AppBarDefaults.BottomAppBarElevation,
    contentPadding: PaddingValues = AppBarDefaults.ContentPadding,
    content: @Composable RowScope.() -> Unit
) { ··· }
```

可以看到，BottomAppBar的声明API，跟TopAppBar可高度自定义的声明API极为相似，无非是多了一个cutoutShape参数。这个参数的主要作用是为BottomAppBar设置镂空形状——通常情况下这种形状跟FloatingActionButton里面的形状是一样的。官方注释中还明确表示，BottomAppBar通常会跟FloatingActionButton一起在Scaffold布局中使用。后面的内容还会介绍到一种跟BottomAppBar相似的布局——BottomNavigation，为什么Compose官方在已经开发BottomAppBar的情况下，还要再开发一个BottomNavigation？主要原因可能是BottomAppBar的声明API相对于BottomNavigation而言更偏底层（虽然BottomNavigation底层不是用BottomAppBar来实现的），不如直接提供一个简单一点的，也能减少开发者的工作量。

最后需要注意的是，无论是TopAppBar还是BottomAppBar，在实践当中通常都是用在后面会介绍到的Scaffold布局当中。
