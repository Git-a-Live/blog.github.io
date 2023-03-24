[ExoPlayer](https://developer.android.google.cn/codelabs/exoplayer-intro#0)是基于Android的底层媒体API所构建的一款应用级媒体播放器。与`MediaPlayer`相比，它具有多项优势：第一，`MediaPlayer`支持的媒体格式，它不仅都支持，还支持DASH和SmoothStreaming等自适应格式；第二，ExoPlayer具有高度的可定制性和可扩展性，因此能够用于许多高级用例。目前，ExoPlayer已经被包括YouTube和Google Play影视在内的Google 应用所集成使用，并且已经开源到[GitHub](https://github.com/google/ExoPlayer)上。

### ExoPlayer导入

ExoPlayer并没有被纳入Android SDK中，因此开发者要想使用ExoPlayer，就得在模块级`build.gradle`文件中以第三方依赖库的形式导入，如下面代码所示：

```
// ExoPlayer核心依赖库，必选
implementation 'com.google.android.exoplayer:exoplayer-core:X.Y.Z'

// ExoPlayer界面组件库，一般会选用
implementation 'com.google.android.exoplayer:exoplayer-ui:X.Y.Z'

// ExoPlayer支持DASH格式的依赖库，按需选用
implementation 'com.google.android.exoplayer:exoplayer-dash:X.Y.Z'
```

需要注意的是，ExoPlayer依赖库的体积非常小，最多也就数百KiB，所以除非项目对apk文件体积的要求严格到KiB级别，否则不需要太担心导入这个库会导致应用体积大幅膨胀。

### ExoPlayer的基本使用