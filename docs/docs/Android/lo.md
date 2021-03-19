布局（layout）是UI界面设计中最基本的组件，它的主要作用就是作为各个控件的“容器”，并提供一系列设计、定位以及排列各个控件的设置属性。在传统的Android开发中，UI界面的设计依赖于XML文件的编写，但是随着Android Studio的更新发展 ，UI界面设计开发逐步仿照VB和Xcode，采用可视化开发方式。<font color=red>掌握XML文件编写可以了解布局和控件的运作原理，掌握可视化操作则能够极大提高开发效率</font>，二者相辅相成，不宜偏废。

>注意，在实际开发中，要避免布局过度嵌套，以减轻界面绘制的资源消耗和性能负担。

## ConstraintLayout

ConstraintLayout（约束布局）是Google官方目前所推荐使用的基于可视化编辑的一种界面布局。它是目前Android Studio开发App时的默认布局，基本上只需要通过在可视化编辑器上拖放组件即可完成界面的开发，很大程度摆脱了传统的XML文件编写开发方式。

ConstraintLayout的出现是为了在Android应用布局中保持扁平的层次结构，减少布局的嵌套，为应用创建响应快速而灵敏的界面。就目前而言，它可以直接替代后面提到的RelativeLayout，而按照Google官方的想法，它将取代其他所有的基本类型布局。

从操作的角度来说，ConstraintLayout的使用跟Xcode的StoryBoard类似，都是贯彻“所见即所得”的设计思想，大大降低界面开发的上手难度。但是控件的约束数量随着控件增加而快速增长，也容易带来一些复杂的问题。

使用ConstraintLayout所要掌握的重点在于**设置“约束”和属性面板**，这需要大量的开发经历来积累足够的经验，才能灵活运用。

## LinearLayout

LinearLayout（线性布局）顾名思义，就是会将所包含的控件**在线性方向上依次排列**的一种布局。

控件的排列方向由属性`android:orientation`来决定，系统提供的值有vertical和horizontal。

控件在**未设置**`android:orientation`属性的排列方向的对齐方式则由`android:layout_gravity`来决定，系统提供的值有top、bottom以及center_vertical等。

控件的在某一方向的尺寸大小由`android:layout_weight`来决定，其计算方式为将布局内所有组件的`android:layout_weight`属性值相加，然后用各个控件的该属性值除以刚才的总值，算出该控件在某一方向占据多少比例的屏宽或屏高。

## RelativeLayout

RelativeLayout（相对布局）是一种通过**相对定位**方式让控件出现在任意位置的布局。这里的“相对定位”不仅仅是针对屏幕边缘，还可以针对控件。

控件通常使用`android:layout_alignXXX`和`android:layout_xxx`两类属性来决定自身相对于屏幕边缘和某个控件的位置。

## FrameLayout

FrameLayout（帧布局）是一种应用场景较少的布局，它的特点是没有丰富的定位方式，所有控件都会默认摆放在布局的**左上角**，但是可以通过设置控件的`android:layout_gravity`属性来决定其对齐方式。
