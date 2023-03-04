Scaffold的主要作用是，为开发者提供一个Material Design风格的**基本视图界面结构**。Scaffold布局的声明API总计有18个参数，几乎涵盖了一个按照Material Design设计准则开发的视图界面应该包含的界面元素及其样式效果。

```
@Composable
fun Scaffold(
    modifier: Modifier = Modifier,
    scaffoldState: ScaffoldState = rememberScaffoldState(),
    topBar: @Composable () -> Unit = {},
    bottomBar: @Composable () -> Unit = {},
    snackbarHost: @Composable (SnackbarHostState) -> Unit = { SnackbarHost(it) },
    floatingActionButton: @Composable () -> Unit = {},
    floatingActionButtonPosition: FabPosition = FabPosition.End,
    isFloatingActionButtonDocked: Boolean = false,
    drawerContent: @Composable (ColumnScope.() -> Unit)? = null,
    drawerGesturesEnabled: Boolean = true,
    drawerShape: Shape = MaterialTheme.shapes.large,
    drawerElevation: Dp = DrawerDefaults.Elevation,
    drawerBackgroundColor: Color = MaterialTheme.colors.surface,
    drawerContentColor: Color = contentColorFor(drawerBackgroundColor),
    drawerScrimColor: Color = DrawerDefaults.scrimColor,
    backgroundColor: Color = MaterialTheme.colors.background,
    contentColor: Color = contentColorFor(backgroundColor),
    content: @Composable (PaddingValues) -> Unit
) { ··· }
```

## scaffoldState配置

scaffoldState配置接收`androidx.compose.material.ScaffoldState`类型参数，主要作用

## topBar配置

## bottomBar配置

## snackbarHost配置

## floatingActionButton配置

## floatingActionButtonPosition配置

## isFloatingActionButtonDocked配置

## drawerContent配置

## drawerGesturesEnabled配置

## drawerShape配置

## drawerScrimColor配置

## content配置