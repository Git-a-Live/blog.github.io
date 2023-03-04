Box布局可以让包裹其中的子项依照顺序堆叠起来，大致对应于传统Android UI开发中，各控件在ConstrainLayout里面的堆叠效果——代码中排序靠前的控件位于堆叠的最底层，排序靠后的控件位于堆叠的最上层——这就是Box名称含义在实际效果（先放的物品位于箱底，后放的物品位于上层）上的一种阐释。

```
@Composable
inline fun Box(
    modifier: Modifier = Modifier,
    // 用于设置子项的排布方式，比如Alignment.TopStart表示全都堆叠到左上角（LTR情形）
    contentAlignment: Alignment = Alignment.TopStart,
    // 用于设置Box的尺寸约束是否对它所包裹的子项起效
    propagateMinConstraints: Boolean = false,
    content: @Composable BoxScope.() -> Unit
) { ··· }
```