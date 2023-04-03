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

#### `ExoPlayer`对象构建

`ExoPlayer`对象采用Builder模式创建对象，如下面代码所示：

```
val exoPlayer = ExoPlayer.Builder(context)
    .setAudioAttributes(···)                // 设置音频属性，handleAudioFocus只有在AudioAttributes.usage
                                            // 为C.USAGE_MEDIA或C.USAGE_GAME时才能设为true
    .setClock(···)                          // 设置一个时钟，仅用于测试场景
    .setBandwidthMeter(···)                 // 设置一个带宽测量器，需要开发者自己实现接口
    .setAnalyticsCollector(···)             // 播放器事件收集分析器，需要开发者自己实现接口
    .setDetachSurfaceTimeoutMs(···)         // 设置画面绘制到播放器surface的超时时长
    .setHandleAudioBecomingNoisy(···)       // 设置插入耳机时是否暂停播放
    .setLivePlaybackSpeedControl(···)       // 设置在线播放速度，需要开发者自己实现接口
    .setLoadControl(···)                    // 设置多媒体资源加载控制器，需要开发者自己实现接口
    .setLooper(···)                         // 设置循环线程，用于响应播放器的所有调用操作，并调用监听器
    .setMediaSourceFactory(···)             // 设置多媒体资源的构建工厂
    .setReleaseTimeoutMs(···)               // 设置释放资源的超时时长
    .setRenderersFactory(···)               // 设置自定义渲染器
    .setPauseAtEndOfMediaItems(···)         // 设置播放器是否需要在每播放完一个多媒体项目之后暂停播放
    .setSeekBackIncrementMs(···)            // 设置快退步长，单位毫秒
    .setSeekForwardIncrementMs(···)         // 设置快进步长，单位毫秒
    .setSkipSilenceEnabled(···)             // 设置是否在播放音频流时跳过静默的部分
    .setTrackSelector(···)                  // 设置具备特定配置属性的播放轨道（如音轨），以便适配当前播放环境
    .setUseLazyPreparation(···)             // 设置是否将播放器加载准备动作延迟到缓冲多媒体资源时
    .setVideoChangeFrameRateStrategy(···)   // 设置自定义的视频帧率更改策略，需要开发者自己实现接口
    .setVideoScalingMode(···)               // 设置视频画面缩放模式，如C.VideoScalingMode.VIDEO_SCALING_MODE_SCALE_TO_FIT
    .setWakeMode(···)                       // 设置播放器的唤醒锁定模式，如C.WakeMode.WAKE_MODE_NETWORK
    .build()
```

创建`ExoPlayer`对象之后，还需要将该对象绑定到界面的`PlayerView`控件上：

```
binding.exoPlayer.player = exoPlayer
```

#### 基础播放功能使用

初始化ExoPlayer的步骤可参考下面的示例代码：

```
// 初始化ExoPlayer
exoPlayer?.run {
    // 设置是否在准备完毕后立即播放
    playWhenReady = true
    // 构建MediaItem传入的链接必须以多媒体文件的后缀结尾，否则无法正常解析
    setMediaItem(MediaItem.fromUri(···))
    // 开始加载在线资源
    prepare()
}
```

ExoPlayer提供了一套默认的播放控制布局（如下图所示），因此开发者只是想提供基础播放功能的话，可以只用默认的播放控制布局。

![](pics/exo1.png)

当ExoPlayer使用完毕之后，就要释放资源：

```
exoPlayer?.run {
    stop()
    release()
}

exoPlayer = null
```

### ExoPlayer的进阶使用

#### 创建播放列表

#### 自适应流式传输

#### 事件监听

#### 自定义界面