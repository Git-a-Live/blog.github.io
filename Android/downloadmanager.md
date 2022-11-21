[DownloadManager](https://developer.android.google.cn/reference/android/app/DownloadManager)是Google官方在Android SDK当中提供的一个系统服务，用于负责那些基于HTTP/HTTPS协议的耗时下载任务。使用DownloadManager，开发者可以简化文件下载功能的开发。

## 前置工作

使用DownloadManager不需要额外导入任何依赖库，只需要在AndroidManifest.xml文件中加入网络以及存储访问权限：

```
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

## 基本使用

由于DownloadManager是系统服务，因此通常跟其他系统服务一样，要通过以下方式获取`DownloadManager`对象：

```
val downloadManager = context.getSystemService(Context.DOWNLOAD_SERVICE) as DownloadManager
```

获得这一对象之后，才能执行后面的一系列业务逻辑。

### 创建和执行下载请求

创建下载请求的方式较为简单，通常如下列代码所示：

```
import android.app.DownloadManager.Request

val downloadUrl: String = "···"
val file: File = File(···)
val downloadRequest: Request = Request(Uri.parse(downloadUrl))
        // 设置可以执行下载任务的网络环境
        .setAllowedNetworkTypes(Request.NETWORK_WIFI or Request.NETWORK_MOBILE)
        // 设置通知栏可见性、标题以及任务描述
        .setNotificationVisibility(Request.VISIBILITY_VISIBLE)
        .setTitle("···")
        .setDescription("···")
        // 禁止或允许数据漫游时下载
        .setAllowedOverRoaming(true)
        // 设置文件存放目录
        .setDestinationUri(Uri.fromFile(file))
        // 设置文件类型
        .setMimeType("···")
        // 禁止或允许使用计费流量下载
        .setAllowedOverMetered(true)
        // 设置下载任务的请求头
        .addRequestHeader(···)

// 提交下载任务并获取任务ID以便进行追踪
val downloadId: Long = downloadManager.enqueue(downloadRequest)
```

如果下载任务启动时遭遇网络环境不满足、SD卡未挂载完成以及超过最大并发数等异常情况，DownloadManager会自行暂停下载任务，直到恢复正常下载条件。

### 获取下载信息

获取下载信息需要通过`DownloadManager.Query`对象和ContentProvider组件：

```
val downloadQuery: DownloadManager.Query = DownloadManager.Query().setFilterById(downloadId)
val cursor: Cursor? = downloadManager.query(downloadQuery)

cursor?.apply {
    if (moveToFirst()) {
        // 已经下载文件大小
        getLong(getColumnIndexOrThrow(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR))
        // 下载文件的总大小
        getLong(getColumnIndexOrThrow(DownloadManager.COLUMN_TOTAL_SIZE_BYTES))
        // 下载状态代码
        val currentStatus = getInt(getColumnIndex(DownloadManager.COLUMN_STATUS))
        // 下载文件的URI
        getString(getColumnIndexOrThrow(DownloadManager.COLUMN_LOCAL_URI))
        when (currentStatus) {
            DownloadManager.STATUS_SUCCESSFUL -> {
                // TODO: 下载成功
            }
            DownloadManager.STATUS_FAILED -> {
                // TODO: 下载失败
            }
            else -> Unit
        }
    }
}
```

如果要监听下载状态，实际上有两种方式，一种比较简单，即通过子线程定时更新`Cursor`对象，然后按照上面的示例代码把状态信息读取出来，再传递给回调；另一种则是继承`ContentObserver`类，重写其`onChange()`方法，然后对其注册和注销监听。后者调用`onChange()`的频率通常不受开发者控制，因此在界面待更新元素过多且网速较快的情况下，快速执行`onChange()`将会对界面性能产生不利影响。

