Android多媒体框架支持播放各种常见媒体类型，以便开发者能轻松地将音频、视频和图片集成到应用中。本节内容主要介绍Android多媒体框架中基础的API使用，至于更深入的音视频开发，则在其他章节展开。

## MediaPlayer

`MediaPlayer`是多媒体框架最重要的组成部分之一。`MediaPlayer`对象能够获取、解码以及播放音频和视频，而且只需进行极少设置。它支持多种不同的媒体源，例如：

+ `res/raw`资源；
+ 在线资源（流式传输）；
+ 内部URI，例如从Content Resolver获取到的URI。

下面就从不同媒体源对`MediaPlayer`的使用展开介绍。不过在此之前，需要注意申请运行时可能会用到的一些权限，比如播放在线媒体内容需要申请网络权限，访问设备磁盘上的多媒体文件需要申请存储空间访问权限，以及播放内容时防止屏幕变暗或处理器进入休眠状态需要申请唤醒锁定权限：

```
<uses-permission android:name="android.permission.WAKE_LOCK" />
```

最后再提供一张Google官方绘制的`MediaPlayer`运行时的状态机示意图：

![](pics/mediaplayer.gif)

根据这张状态机示意图，开发者可以知道调用什么方法会进入什么状态，从某一个状态切换到下一个状态需要调用什么方法，在开发时就能更好地运用这些API。

更多有关`MediaPlayer`的详细资料，可以参阅[Google官方文档](https://developer.android.google.cn/guide/topics/media/mediaplayer)。

### `res/raw`资源

`res/raw`资源是指系统不会尝试以任何特定方式解析的文件。一般来说，现成的音频和视频这些多媒体文件都可以放到这个目录下供应用读取调用。

```
var mediaPlayer: MediaPlayer? = null

if (mediaPlayer == null) {
    // 从res/raw目录下读取多媒体文件进行播放，需要调用MediaPlayer.create()方法，
    // 这个方法内部自动调用了prepare()，因此开发者不需要自行再去调用
    mediaPlayer = MediaPlayer.create(this, R.raw.local_resource)
}

// 开始播放或继续播放多媒体文件
mediaPlayer?.start()

// 暂停播放多媒体文件
mediaPlayer?.pause()

// 停止播放多媒体文件，实际操作是注销了MediaPlayer的所有监听回调
mediaPlayer?.stop()

// 在使用完MediaPlayer之后，务必及时释放MediaPlayer资源
mediaPlayer?.release()
mediaPlayer = null
```

### 在线资源

MediaPlayer支持HTTP流式传输，可以播放在线资源。需要特别注意的是，如果应用禁止明文通信，仅允许使用HTTPS协议，而传入的在线资源用的是HTTP协议，那么MediaPlayer就有可能出现无法正常解析播放该资源的异常情况。针对这种问题，一个可以参考的解决方法是将在线资源url的协议头换成HTTPS。另外，加载在线资源通常是耗时的，因此一般会采用下面示例代码的方式来进行异步加载：

```
// 传入在线资源的url
val url = "http://···"

var mediaPlayer: MediaPlayer? = MediaPlayer().apply {
    // 设置MediaPlayer异步准备完毕之后的回调
    setOnPreparedListener { it: MediaPlayer ->
        it.start()
        ···
    }

    // 设置MediaPlayer播放完毕之后的回调
    setOnCompletionListener { it: MediaPlayer ->
        ···
        it.release()
        mediaPlayer = null
    }

    // 设置MediaPlayer发生错误时的回调
    setOnErrorListener { mp, what, extra ->
        mp.release()
        mediaPlayer = null
         Log.e("MyTag","what = $what, extra = $extra")
        true
    }

    try {
        // 将在线资源地址传入MediaPlayer，注意该方法随时可能抛出异常
        setDataSource(url)

        // 设置在线资源的流媒体形式，比如音频就按照下面这种方式来设置
        setAudioAttributes(AudioAttributes.Builder()
            .setContentType(AudioAttributes.CONTENT_TYPE_MUSIC)
            .setLegacyStreamType(AudioManager.STREAM_MUSIC)
            .build())

        // 由于加载在线资源相对耗时，最好调用prepareAsync()，而非prepare()
        prepareAsync()
    } catch (e: Exception) {
        e.printStackTrace()
        ···
    }
}

// 释放MediaPlayer资源步骤省略
···
```

> 注意，`MediaPlayer.setDataSource()`实际上不光能传入在线资源url，还可以传入磁盘上存在的多媒体文件的路径，从而进行加载。也就是说，`MediaPlayer.setDataSource()`既能加载在线资源，又能加载本地资源。

### 内部URI

通过[Content Resolver](Android/contpro?id=使用content-resolver访问其他应用的数据)，开发者可以从本地解析出多媒体文件的URI，然后调用`MediaPlayer.setDataSource()`完成加载，如下面代码所示：

```
// 获取到多媒体资源的URI
val myUri: Uri = ···

var mediaPlayer: MediaPlayer? = MediaPlayer().apply {
    ···

    // 由于其他过程跟加载在线资源基本一样，这里只展示如何传入URI
    setDataSource(applicationContext, myUri)

    ···
}
```

### 使用唤醒锁定

当应用在后台播放媒体内容时，设备可能随时会进入休眠状态。由于Android系统尝试在设备处于休眠状态时节省电量，因此它会尝试关闭手机上任何不必要的功能，包括CPU和WLAN硬件。因此开发者需要考虑如何防止系统休眠干扰内容播放。

为了确保媒体播放在这些情况下能继续运行，开发者必须使用“唤醒锁定”。唤醒锁定可以告诉系统：当前应用正在使用一些即使在手机处于闲置状态时也应该可用的功能。需要注意的是，开发者应始终谨慎使用唤醒锁定功能，因为它们会显著缩短设备的电池续航时间。

为确保CPU在`MediaPlayer`播放时继续运行，在初始化时应调用`setWakeMode()`方法。完成该操作后，`MediaPlayer`会在播放时保持指定的锁定状态，并在暂停或停止播放时自动释放锁定，如下面代码所示：

```
var mediaPlayer: MediaPlayer? = MediaPlayer().apply {
    ···

    // 其他初始化过程省略，只展示如何启用唤醒锁定功能
    setWakeMode(applicationContext, PowerManager.PARTIAL_WAKE_LOCK)

    ···
}
```

## MediaRecorder

Android多媒体框架支持捕获和编码各种常见的音频和视频格式。如果设备硬件支持，开发者可以使用`MediaRecorder`的相关API来录制音频。本节内容主要介绍如何使用 `MediaRecorder`实现基本的录音功能。更多关于`MediaRecorder`的详细资料，尤其是`MediaMuxer`的使用，可以参阅[Google官方文档](https://developer.android.google.cn/guide/topics/media/mediarecorder)。

### `MediaRecorder`的基本使用

首先来看如何初始化以及启动录音：

```
// 在Android 12+的设备上，MediaRecorder的构造方法需要传入Context，低于Android 12的设备则继续使用无参构造方法
var mediaRecorder: MeiaRecorder? = MediaRecorder().apply {
    try {
        // 设置音频源，一般为设备的麦克风
        setAudioSource(AudioSource.MIC)

        // 设置录音输出格式，在Android 8.0+设备上建议使用MPEG_2_TS，因为这有利于进行流式传输
        setOutputFormat(if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            MediaRecorder.OutputFormat.MPEG_2_TS
        } else {
            MediaRecorder.OutputFormat.AAC_ADTS
        })

        // 设置输出的音频文件名和格式，通常格式选用MP3
        setOutputFile("XXXXXX.mp3")

        // 设置音频编码器
        setAudioEncoder(AudioEncoder.AAC)

        // 设置音频采样频率，单位Hz，可根据实际选用的编码器来决定
        setAudioSamplingRate(16000)

        // 设置录音的声道数，1表示单声道，2表示双声道
        setAudioChannels(2)

        // 设置抛出错误通知的回调
        setOnErrorListener { mr, what, extra ->
            ···
        }

        // 设置信息事件通知的回调
        setOnInfoListener { mr, what, extra ->  
            ···
        }

        // 调用prepare()就表示MediaRecorder对象已经配置完毕，因此所有的set操作都必须在prepare()之前调用
        prepare()
    } catch (e: Exception) {
        ···
    }
}

// 开始录音
mediaRecorder?.start()

// 暂停录音
mediaRecorder?.pause()

// 恢复录音
mediaRecorder?.resume()
```

> 注意，在Android 10+的设备上，`MediaRecorder`提供了`registerAudioRecordingCallback()`这一回调注册方法，用于接收音频录制过程中一些事件的通知。另外，使用`registerAudioRecordingCallback()`要配套使用`unregisterAudioRecordingCallback()`以注销回调监听，防止引发内存泄漏。

录制结束后务必及时释放`MediaRecorder`资源：

```
mediaRecorder?.apply {
    try {
        stop()
        // 调用reset()方法时，MediaRecorder对象尚可重用
        reset()
        // 调用release()方法后，MediaRecorder对象就无法重用了
        release()
    } catch (e: Exception) {
        ···
    }
}
mediaRecorder = null
```

## ExoPlayer

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



## AudioRecorder

## AudioManager

## AudioTrack