在Android的UI界面开发中，所有控件都是直接或间接继承自**View**（视图），而所有布局都是直接或间接继承自**ViewGroup**（事实上ViewGroup也是继承自View）。View作为Android中最基本的UI组件，可以在屏幕上绘制一块区域，并响应这块区域的各种事件。当系统控件远远不能满足开发需求的时候，可以构建自定义控件来解决这一问题。

## XML文件编写

为了解决重复编写布局代码的问题，可以将某一类布局（如标题栏和设置页等）各个元素提取出来，编写成一份通用的XML文件，在使用时直接引用该文件即可。引用布局有些特殊，需要在对应XML中通过下面这种方式来引用：

```
<include layout="···">
```

XML文件编写主要分为三大内容，一是布局选用，二是控件摆放，三是形式定义。控件摆放和普通的Android项目开发没有什么区别，这里不作赘述。

### 布局选用

在[基本布局](Android/lo)中已经简略介绍过常见的四大布局，它们各有特点，按照开发需要选用即可。不过Google官方推荐使用ConstraintLayout，那么以ConstraintLayout作为基本容器即可。

布局不需要开发者自己手动编写，右键项目`res`目录下的指定文件夹（如`layout`），通过`New`➡`Android Resource File`打开如下图所示的配置窗口：

![](pics/Screenshot%202020-12-09%20132554.png)

依次填写布局文件名称、选择Resource type为“Layout”，然后点击“OK”就创建好布局文件了。

### 样式定制

样式定制包括布局和控件，涉及形状、颜色、尺寸以及主题等属性，都是通过编写XML属性来进行设置，下面做一些简要介绍。

+ **形状**

布局和控件的形状都可以通过`android:background`引用相应的XML文件来设置属性，所引用的文件通常放在`res`目录的`drawable`文件夹下。创建引用文件和布局类似，都可以通过Android Studio的快捷方式来创建。

刚创建好的形状文件只有\<selector>标签，里面只能新增\<item>标签，通常情况下Android Studio还会提示`item`后面可以设置一些属性，但是形状文件一般用不到，因此可以不设置。在此基础上，只有再添加\<shape>标签，才算真正开始设置形状：

```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape>

        </shape>
    </item>
</selector>
```

`shape`后面同样可以设置一些属性，比如`android:shape`。`android:shape`有四个类型（oval、ring、line、rectangle）可以选择，用来设定基本形状。如果不设置，那么形状就默认是矩形的。

\<shape>标签内部有六大子标签，分别是：

>size：
```
<size android:height="设置高度"
    android:width="设置宽度"/>
```
>corners：
```
<corners android:radius="统一设置所有圆角大小，不能和其他选项共存"
                android:bottomLeftRadius="设置左下圆角大小"
                android:bottomRightRadius="设置右下圆角大小"
                android:topLeftRadius="设置左上圆角大小"
                android:topRightRadius="设置右上圆角大小"/>
```

>solid：
```
<solid android:color="设置填充颜色"/>
```

>padding：
```
<padding    
    android:left="内部到左边缘边距"   
    android:top="内部到顶部边距"   
    android:right="内部到右边缘边距"   
    android:bottom="内部到底部边距"/>
```

>gradient：
```
<gradient  
    android:type="linear线性渐变（默认）| radial放射渐变 | sweep扫描式渐变"
    android:angle="渐变角度，必须为45的倍数，0为从左到右，90为从上到下"
    android:centerX="渐变中心X的相对位置，范围为0～1"   
    android:centerY="渐变中心Y的相对位置，范围为0～1"   
    android:startColor="渐变开始点的颜色"
    android:centerColor="渐变中间点的颜色，在开始与结束点之间"
    android:endColor="渐变结束点的颜色"
    android:gradientRadius="渐变的半径，只有当渐变类型为radial时才能使用"
    android:useLevel="使用LevelListDrawable时就要设置为true；设为false时才有渐变效果"/>
```

>stroke：
```
<stroke        
    android:width="描边的宽度"
    android:color="描边的颜色"
    android:dashWidth="虚线的宽度，值为0时是实线"
    android:dashGap="虚线的间隔"/>
```

+ **颜色**

颜色可以通过`android:color`属性来设置。

+ **尺寸**

尺寸可以通过`android:width`和`android:height`等属性来设置。

+ **主题**

主题可以通过`android:theme`属性来设置，对应的引用文件编写方式如下：

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="样式名称" parent="继承样式">
        <item name="选择属性名称">设置属性值</item>
        ···
    </style>
</resources>
```

## 控件组合封装

一些具有固定用途和操作的控件可以注册封装成一个组合<font color=red>（实际上就是一个包含布局的自定义控件）</font>，在要调用的时候只需要将整个组合导入就能使用，不需要再重新注册和重写方法，减少重复代码。通常做法如下：

1. **继承父类创建控件**

新建一个继承于某个Layout的子类（比如ConstraintLayout）：

```
class MyLayout(context: Context,attrs: AttributeSet): ConstraintLayout(context, attrs) {
    init {
        LayoutInflater.from(context).inflate(R.layout.xxx,this) //引用布局文件，添加父布局this
        //布局内控件注册，编写业务逻辑
    }
}
```

2. **布局文件添加控件**

在一个空的布局文件中，以控件的形式添加上一步自定义的Layout子类：

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android" android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.example.app.MyLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
    
</androidx.constraintlayout.widget.ConstraintLayout>
```

>注意，导入自定义控件时，必须指明控件的完整类名。

3. **注册控件响应事件**

在自定义控件中注册响应事件，就跟在Activity和Fragment等组件中注册控件类似，没有太大的差异，因此不做赘述。

## 自定义View

自定义View包括直接继承View和直接继承ViewGroup进行功能定制，跟之前自定义控件相比，自定义View要复杂一些，因为这涉及到View的构造函数和属性等。通用的方法如下：

1. **创建子类**

继承View（或ViewGroup）并重写至少一个构造函数：

```
class MyView(context: Context): View(context) {
    init {
        //TODO
    }
    constructor(context: Context, @Nullable attrs: AttributeSet): this(context){
        //TODO
    }
    ···
}
```

2. **编写属性文件**

Android系统的控件中，以`android:xxx`形式出现的都是系统自带的属性，如果要编写自定义属性，就需要在`values`目录下创建属性文件，并仿照下面的格式编写属性名称和属性值：

```
<?xml version="1.0" encoding="utf-8"?>
    <resources>
        <declare-styleable name="属性文件名">
            <attr name="属性名称" format="属性类型，包括string、integer和fraction（百分数）等" />
            ···
        </declare-styleable>
    </resources>
```

在布局文件中，自定义View通过`app: xxx`的格式来获取并设置自定义属性，在其构造方法中还可通过TypedArray对象调用`getXXX()`方法来获取自定义属性的值。

3. **导入和使用**

自定义View导入和使用方式同[自定义控件](#控件组合封装)相似，这里不再赘述。
