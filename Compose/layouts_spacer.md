Spacer能够提供一个空白的布局，正如它的名称一样。如果要问这种空白布局有什么用，大概就类似于在传统布局文件中用一个View来充当占位符。Spacer是所有布局当中配置最少的一个，只有一个`Modifier`参数可以配置。通常情况下，Spacer的`Modifier`只会用到与宽高尺寸相关的配置。按照官方注释的说法就是，它能配置的只有`Modifier.width`、`Modifier.height`以及`Modifier.size`这类参数。

```
@Composable
@NonRestartableComposable
fun Spacer(modifier: Modifier) { ··· }
```