Android开发中，图片、音频、视频以及文档是最基础的多媒体文件，之前已经介绍了图片，接下来主要介绍音视频的播放。 由于篇幅所限，仅涉及如何从SD卡中读取音视频文件并实现最基础的播放停止功能，音视频解码等更深入的技术并不会涉及。

## MediaPlayer

音频和视频在读取和播放等操作上有着非常相似的地方，所以只要掌握其中一种，另外一种也很容易就能理解并运用。 先来看看如何从SD卡中读取并播放音频。

在Android中播放音频，一般是通过MediaPlayer类来实现的。MediaPlayer的工作流程如下：

1. 创建MediaPlayer对象，调用setDataSource()设置音频文件路径；

2. 调用prepare()方法使MediaPlayer进入准备状态；

3. 调用start()方法播放音频，调用pause()方法暂停音频，调用stop()方法停止播放等等；

4. 退出应用程序时，调用release()方法释放当前关联的音频资源。

在了解上述的MediaPlayer工作流程之后，就可以写出一个简易音乐播放器。

和图片一样，首先要在AndroidManifest.xml文件中声明读写外部存储文件的权限，然后在代码中进行授权情况判断：

```
val permission = Manifest.permission.WRITE_EXTERNAL_STORAGE
val granted = PackageManager.PERMISSION_GRANTED
val access = ContextCompat.checkSelfPermission(applicationContext, permission)
if (access != granted){
    ActivityCompat.requestPermissions(this, arrayOf(permission),REQUEST_CODE)
} else {
    //TODO
}
···
override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array< out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == REQUEST_CODE
                    && grantResults.isNotEmpty()
                    && grantResults[0] == PackageManager.PERMISSION_GRANTED
                ){
            //TODO
        } else{
            //TODO
        }
    }
```

接着创建MediaPlayer对象，然后设置音频文件路径，并使MediaPlayer对象进入准备状态：

```
val mediaPlayer = MediaPlayer()
try {
    val file = File(PATH,"music.mp3")
    mediaPlayer.setDataSource(file.path)
    mediaPlayer.prepare()
} catch (e: IOException){
    //TODO
}
```

最后调用相关方法，即可实现基本的音频播放功能。

MediaPlayer还有以下常用方法：

>reset() //将MediaPlayer对象重置到初始创建状态

>seekTo() //从指定的地方开始播放音频

>isPlaying() //判断当前MediaPlayer对象是否在播放音频

>getDuration() //获取载入的音频文件时长

视频播放和音频播放是相似的，因为视频播放会使用到一个封装了MediaPlayer的VideoView控件， 两者在基本操作上的区别不是很大。但是VideoView和WebView一样，只是一个能实现简单功能的原生控件， 不能指望仅仅依靠它就可以写出一个复杂且功能完善的强大的视频播放器，如果有更多更高的需求，那么应该去选用一些第三方开源库。

接下来简单介绍一下VideoView的使用。由于读取SD卡上的视频文件同样需要声明权限，具体代码和音频部分基本一样，所以此处不再赘述。 VideoView的几个使用方法如下：

>setVideoPath() //设置视频文件的路径

>start() //播放视频

>pause() //暂停播放视频

>resume() //从头播放视频

>seekTo() //从指定的地方开始播放视频

>suspend() //释放关联的视频资源

>isPlaying() //判断当前MediaPlayer对象是否在播放视频

>getDuration() //获取载入的视频文件时长

## AudioManager

## MediaRecorder

## AudioRecorder

## AudioTrack

## ExoPlayer