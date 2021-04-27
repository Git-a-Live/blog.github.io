对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。阿里云OSS是常见的对象存储服务之一，其详细介绍可见于[官网](https://help.aliyun.com/document_detail/31817.html?spm=a2c4g.11174283.2.6.135c7da2QJGqDy)。本文主要介绍的是如何在Android项目中简单集成使用阿里云OSS，如需了解更为详细的内容，可以借助Google或查阅[官方技术文档](https://help.aliyun.com/document_detail/32042.html?spm=a2c4g.11186623.6.1304.695c1272foMz1C)。

><font color=red>注意，以下内容默认用户已经搭建好OSS服务器并开通阿里云OSS服务。</font>

## 前置工作

和[腾讯Bugly](Android/bugly)一样，阿里云OSS也可以通过Gradle导入依赖：

```
implementation "com.aliyun.dpa:oss-android-sdk:$specific_version"
```

>注意，截至2021年4月25日，阿里云OSS开发团队在[GitHub](https://github.com/aliyun/aliyun-oss-android-sdk)上提供的版本仅到2.9.3，而且导入方式也相当陈旧和不符合规范。想查看最新版本的SDK，需要前往该项目位于JCenter上的[页面](https://mvnrepository.com/artifact/com.aliyun.dpa/oss-android-sdk)。

阿里云OSS在使用过程中需要的权限如下：

```
<uses-permission android:name="android.permission.INTERNET" />

<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
```

## 初始化

在使用SDK发起对OSS的请求前，用户需要初始化一个OSSClient实例，并对它进行一些必要设置。OSSClient是OSS服务的Android客户端，它为调用者提供了一系列的方法进行操作、管理存储空间（Bucket）和文件（Object）等。OSSClient的生命周期通常和应用生命周期保持一致，因此建议在应用启动时创建一个全局的OSSClient，在应用结束时销毁即可。

在正式创建OSSClient实例之前，有三个参数需要进行初始化：EndPoint、OSSCredentialProvider以及ClientConfiguration。

+ **EndPoint**

EndPoint表示OSS对外服务的访问域名，跟阿里云服务器所在地区有关。比如，用户访问某台位于杭州的阿里云OSS服务器，那么所用的EndPoint为`http://oss-cn-hangzhou.aliyuncs.com`。其他地区服务器的EndPoint可见于[阿里云OSS官方文档](https://help.aliyun.com/document_detail/31837.htm?spm=a2c4g.11186623.2.10.569a533806eWEs#concept-zt4-cvy-5db)。

EndPoint是用于创建OSSClient实例的主要参数之一，其类型为String，例如：

```
val endPoint: String = "http://oss-cn-hangzhou.aliyuncs.com"
```

+ **OSSCredentialProvider**

阿里云官方推荐使用<font color=red>STS鉴权模式</font>，通过OSSAuthCredentialsProvider的方式来直接访问鉴权应用服务器，这样token过期后可以自动更新，同时规避AccessKeyId和AccessKeySecret直接保存在移动端的风险。示例代码如下：

```
val stsServer = "···"
val credentialProvider = OSSAuthCredentialsProvider(stsServer)
```

当然，在某些情况下，用户并不想用STS鉴权模式，而是直接读取AccessKeyId和AccessKeySecret，然后构建一个OSSCredentialProvider实例出来。这时可以参考以下的示例代码：

```
val accessKeyId: String = ···
val accessKeySecret: String = ···
val securityToken: String = ···
val credentialProvider = OSSStsTokenCredentialProvider(accessKeyId,accessKeySecret,securityToken)
```

>注意，在上述代码中，accessKeyId、accessKeySecret和securityToken既可以从自建服务器上远程获取，也可以保存在本地直接读取，但是securityToken有过期失效的风险。

+ **ClientConfiguration**

ClientConfiguration用于对OSSClient实例进行网络参数配置，官方给出的一个<font color=red>自定义</font>配置用例如下：

```
val conf = ClientConfiguration()
conf.connectionTimeout = 10 * 1000 // 连接超时，默认15秒。
conf.socketTimeout = 10 * 1000 // socket超时，默认15秒。
conf.maxConcurrentRequest = 10 // 最大并发请求数，默认5个。
conf.maxErrorRetry = 10 // 失败后最大重试次数，默认2次。
```

>注意，如果不需要使用自定义配置，就不必创建ClientConfiguration实例。

在配置完上述三个参数之后，就可以创建OSSClient实例了：

```
// 使用自定义ClientConfiguration
val oss = OSSClient(context, endpoint, credentialProvider, conf)

// 使用默认的ClientConfiguration
val oss = OSSClient(context, endpoint, credentialProvider)
```

## 基本使用

### 存储空间

存储空间是用户用于存储对象（Object）的容器，所有的对象都必须隶属于某个存储空间。存储空间具有各种配置属性，包括地域、访问权限、存储类型等。用户可以根据实际需求，创建不同类型的存储空间来存储不同的数据。

同一个存储空间的内部是扁平的，**没有文件系统的目录等概念**，所有的对象都直接隶属于其对应的存储空间。阿里云OSS的存储空间有以下特点：

+ 每个用户可以拥有多个存储空间
+ 存储空间的名称在OSS范围内必须是全局唯一的，一旦创建之后无法修改名称
+ 存储空间内部的对象数目没有限制

存储空间有自己的命名规范：

+ 只能包括小写字母、数字和短划线（-）
+ 必须以小写字母或者数字开头和结尾
+ 长度必须在3~63字节之间。 

新建一个存储空间的示例代码如下：

```
// 创建被命名好的存储空间实例
val createBucketRequest = CreateBucketRequest("···")
// 指定Bucket的ACL权限，此处设置存储空间的访问权限为公共读，默认为私有读写
createBucketRequest.bucketACL = CannedAccessControlList.PublicRead
// 异步创建存储空间
val createTask = oss.asyncCreateBucket(createBucketRequest, 
        object : OSSCompletedCallback<CreateBucketRequest?, CreateBucketResult?> {
    override fun onSuccess(request: CreateBucketRequest?, result: CreateBucketResult?) {
        Log.d("asyncCreateBucket", "Success")
    }

    override fun onFailure(
        request: CreateBucketRequest?, 
        clientException: ClientException, 
        serviceException: ServiceException
    ) {
        // 请求异常
        if (clientException != null) {
            // 本地异常，如网络异常等
            clientException.printStackTrace()
        }
        if (serviceException != null) {
            // 服务异常
            Log.e("ErrorCode", serviceException.errorCode)
            Log.e("RequestId", serviceException.requestId)
            Log.e("HostId", serviceException.hostId)
            Log.e("RawMessage", serviceException.rawMessage)
        }
    }
})
```

上述代码在创建Bucket时，指定了Bucket的ACL和所在地域。创建存储空间有以下几个注意点：

+ 同一阿里云账号在同一地域内创建的Bucket总数不能超过100个
+ 创建Bucket时可以选择Bucket ACL权限，如果不设置ACL，默认是private
+ 每个Bucket的名称全局唯一，不能出现同名Bucket，否则会创建失败
+ 创建成功后，结果返回至Bucket所在地域。

### 上传文件

阿里云OSS的Android SDK提供了三种文件上传方式：简单上传、追加上传以及断点续传上传。简单上传包括本地文件和二进制Byte数组，最大不能超过5GB；追加上传是指通过AppendObject方法，在已上传的追加类型文件（Appendable Object）末尾直接追加内容，最大也不能超过5GB；断点续传上传支持并发、断点续传以及自定义分片大小，适用于大文件，最大不能超过48.8TB。

#### 简单上传

简单上传本地文件有两种接口可以调用，一种是同步接口，只能使用于非UI线程中，另一种就是异步接口，可以用在UI线程里面。
  
使用同步接口上传本地文件的示例代码如下：

```
// 构建上传请求
val put = PutObjectRequest("<bucketName>", "<objectName>", "<uploadFilePath>")
try {
    val putResult: PutObjectResult = oss.putObject(put)
    Log.d("PutObject", "UploadSuccess")
    Log.d("ETag", putResult.eTag)
    Log.d("RequestId", putResult.requestId)
} catch (e: ClientException) {
    // 本地异常，如网络异常等
    e.printStackTrace()
} catch (e: ServiceException) {
    // 服务异常
    Log.e("RequestId", e.requestId)
    Log.e("ErrorCode", e.errorCode)
    Log.e("HostId", e.hostId)
    Log.e("RawMessage", e.rawMessage)
}
```

使用异步接口上传本地文件的示例代码如下：

```
val put = PutObjectRequest("<bucketName>", "<objectName>", "<uploadFilePath>")
// 设置上传进度回调
put.progressCallback = OSSProgressCallback { request, currentSize, totalSize ->
    Log.d("PutObject","currentSize: $currentSize totalSize: $totalSize")
}
// 创建异步上传任务
var task = oss.asyncPutObject(put, 
        object : OSSCompletedCallback<PutObjectRequest?, PutObjectResult> {
    override fun onSuccess(request: PutObjectRequest?, result: PutObjectResult) {
        Log.d("PutObject", "UploadSuccess")
        Log.d("ETag", result.eTag)
        Log.d("RequestId", result.requestId)
    }

    override fun onFailure(
        request: PutObjectRequest?,
        clientExcepion: ClientException,
        serviceException: ServiceException
    ) {
        // 请求异常
        if (clientExcepion != null) {
            // 本地异常，如网络异常等
            clientExcepion.printStackTrace()
        }
        if (serviceException != null) {
            // 服务异常
            Log.e("ErrorCode", serviceException.errorCode)
            Log.e("RequestId", serviceException.requestId)
            Log.e("HostId", serviceException.hostId)
            Log.e("RawMessage", serviceException.rawMessage)
        }
    }
})
// 可以取消任务
task.cancel()
// 等待任务完成
task.waitUntilFinished()
```

调用同步接口上传二进制Byte数组的示例代码如下：

```
// 构建随机生成的二进制Byte数组
val uploadData = ByteArray(100 * 1024)
Random().nextBytes(uploadData)

// 构造上传请求，注意第三个参数
val put = PutObjectRequest("<bucketName>", "<objectName>", uploadData)
try {
    val putResult: PutObjectResult = oss.putObject(put)
    Log.d("PutObject", "UploadSuccess")
    Log.d("ETag", putResult.eTag)
    Log.d("RequestId", putResult.requestId)
} catch (e: ClientException) {
    // 本地异常，如网络异常等
    e.printStackTrace()
} catch (e: ServiceException) {
    // 服务异常
    Log.e("RequestId", e.requestId)
    Log.e("ErrorCode", e.errorCode)
    Log.e("HostId", e.hostId)
    Log.e("RawMessage", e.rawMessage)
}
```

#### 追加上传

使用异步方式追加上传的示例代码如下：

```
val append = AppendObjectRequest("<bucketName>", "<objectName>", "<uploadFilePath>")
// 设置文件元数据
val metadata = ObjectMetadata()
metadata.contentType = "application/octet-stream"
append.metadata = metadata
// 设置追加位置
append.position = 0
// 设置回调
append.progressCallback = OSSProgressCallback { request, currentSize, totalSize -> 
    Log.d("AppendObject", "currentSize: $currentSize totalSize: $totalSize") 
}
// 异步追加上传
val task = oss.asyncAppendObject(append, 
        object : OSSCompletedCallback<AppendObjectRequest?, AppendObjectResult> {
    override fun onSuccess(request: AppendObjectRequest?, result: AppendObjectResult) {
        Log.d("AppendObject", "AppendSuccess")
        Log.d("NextPosition", "" + result.nextPosition)
    }

    override fun onFailure(
        request: AppendObjectRequest?, 
        clientExcepion: ClientException, 
        serviceException: ServiceException
    ) {
        // 异常处理
    }
})
```

在使用Append方式上传时，需要注意对追加位置（Position）这个参数进行正确的设置：

+ 如果是创建一个Appendable Object，追加位置设为0
+ 如果对Appendable Object进行内容追加时，追加位置设为Object当前长度


有两种方式可以获取该Object长度：

+ 通过上传追加后的返回内容获取
+ 通过head object获取文件长度

#### 断点续传上传

在无线网络下，上传比较大的文件持续时间长，可能会遇到因为网络条件差、用户切换网络等原因导致上传中途失败，整个文件需要重新上传。为此，Android SDK提供了断点续传上传功能。

对于移动端来说，如果不是大文件（例如小于5GB的文件），不建议使用此种方式上传。因为断点续传上传是通过[分片上传](https://help.aliyun.com/document_detail/133805.html?spm=a2c4g.11186623.6.1320.45795d88sFU9aK)实现的，上传单个文件需要进行多次网络请求，效率不高。

使用断点续传上传有以下几点需要注意：

+ 文件上传前应指定断点记录的保存文件夹，避免上传失败时，因没有断点记录消耗大量时间和流量重传整个大文件
+ 断点续传上传仅支持上传本地文件，且仅在本次上传生效
+ 断点续传上传
+ 若使用STS鉴权模式，需保证拥有访问InitMultipartUpload、UploadPart、ListParts、CompleteMultipartUpload以及AbortMultipartUpload等接口的权限
+ 默认已开启每个分片上传时的MD5校验，无需在request中设置Content-Md5头部
+ 若同一任务一直得不到续传，可能会在OSS上积累无用碎片，因此可以为Bucket设置[lifeCycle规则](https://help.aliyun.com/document_detail/31904.htm?spm=a2c4g.11186623.2.5.45795d88sFU9aK#concept-bmx-p2f-vdb)的方式来定时清理碎片

完整的断点续传上传过程的示例代码如下：

```
// 创建本地断点记录保存文件夹
val recordDirectory: String = getExternalFilesDir(null)?.absolutePath.toString() + "/oss_record/"
val recordDir = File(recordDirectory)

// 确保断点记录的保存文件夹已存在，否则就新建断点记录的保存文件夹
if (!recordDir.exists()) {
    recordDir.mkdirs()
}

// 创建断点续传上传请求，并指定断点记录文件的保存路径，保存路径为断点记录文件的绝对路径
val request = ResumableUploadRequest("<bucketName>", "<objectKey>", "<uploadFilePath>", recordDirectory)
// 调用OSSAsyncTask cancel()方法时，若DeleteUploadOnCancelling设置为false，则不删除断点记录文件
// 如果不设置此参数，则默认值为true，表示删除断点记录文件，下次再上传同一个文件时就要重新上传
request.setDeleteUploadOnCancelling(false)
// 设置上传回调
request.progressCallback = OSSProgressCallback<ResumableUploadRequest?> { request, currentSize, totalSize -> 
    Log.d("resumableUpload", "currentSize: $currentSize totalSize: $totalSize") 
}
        
val resumableTask = oss.asyncResumableUpload(request, 
        object : OSSCompletedCallback<ResumableUploadRequest?, ResumableUploadResult?> {
    override fun onSuccess(request: ResumableUploadRequest?, result: ResumableUploadResult?) {
        Log.d("resumableUpload", "success!")
    }

    override fun onFailure(
            request: ResumableUploadRequest?, 
            clientExcepion: ClientException, 
            serviceException: ServiceException
    ) {
        // 异常处理
    }
})

// 等待完成断点上传任务
resumableTask.waitUntilFinished()
```

### 下载文件

OSS Android SDK提供了四种文件下载方式：下载到本地文件、流式下载、范围下载以及限定条件下载。这四种方式的使用场景和使用方式会在下面一一展开介绍。

#### 下载到本地文件

将OSS服务器上的文件“原汁原味”地下载到本地是最为简单和常见的应用方式，阿里云OSS Android SDK下载到本地文件的示例代码如下：

```
//objectKey等同于objectname，表示从OSS下载文件时需要指定包含文件后缀在内的完整路径，例如abc/efg/123.jpg
val get = GetObjectRequest("BucketName", "objectKey")

oss.asyncGetObject(get, 
        object : OSSCompletedCallback<GetObjectRequest?, GetObjectResult> {
    override fun onSuccess(request: GetObjectRequest?, result: GetObjectResult) {
        // 开始读取数据
        val length = result.contentLength
        val buffer = ByteArray(length.toInt())
        var readCount = 0
        while (readCount < length) {
            try {
                readCount += result.objectContent.read(buffer, readCount, length.toInt() - readCount)
            } catch (e: Exception) {
                OSSLog.logInfo(e.toString())
            }
        }
        // 将下载后的文件存放在指定的本地路径
        try {
            val fout = FileOutputStream("download_filePath")
            fout.write(buffer)
            fout.close()
        } catch (e: Exception) {
            // 打印日志
            OSSLog.logInfo(e.toString())
        }
    }

    override fun onFailure(
        request: GetObjectRequest?, 
        clientException: ClientException, 
        serviceException: ServiceException
    ) {
        // 读取失败处理异常
    }
})
```

#### 流式下载

如果要下载的文件太大，或者一次性下载耗时太长，可以选择采用流式下载，一次处理部分内容，直到完成文件的下载。下载指定文件后将获得文件的输入流，此操作要求用户对该Object有读权限。

同步调用流式下载的示例代码如下：

```
// 构造下载文件请求
// objectKey等同于objectName，表示从OSS下载文件时需要指定包含文件后缀在内的完整路径，例如abc/efg/123.jpg
val get = GetObjectRequest("<bucketName>", "<objectKey>")
// 设置下载进度回调
get.setProgressListener { request, currentSize, totalSize -> 
    OSSLog.logDebug("getobj_progress: $currentSize  total_size: $totalSize", false) 
}

try {
    // 同步执行下载请求，返回结果
    val getResult: GetObjectResult = oss.getObject(get)
    Log.d("Content-Length", "" + getResult.contentLength)

    // 获取文件输入流
    val inputStream = getResult.objectContent
    val buffer = ByteArray(2048)
    var len: Int
    while (inputStream.read(buffer).also { len = it } != -1) {
        // 处理下载的数据，比如图片展示或者写入文件等
    }

    // 下载后可以查看文件元信息
    val metadata = getResult.metadata
    Log.d("ContentType", metadata.contentType)
} catch (e: ClientException) {
    // 本地异常如网络异常等
    e.printStackTrace()
} catch (e: ServiceException) {
    // 服务异常
    Log.e("RequestId", e.requestId)
    Log.e("ErrorCode", e.errorCode)
    Log.e("HostId", e.hostId)
    Log.e("RawMessage", e.rawMessage)
} catch (e: IOException) {
    e.printStackTrace()
}
```

异步调用流式下载的示例代码如下：

```
val get = GetObjectRequest("<bucketName>", "<objectKey>")
// 设置下载进度回调
get.setProgressListener { request, currentSize, totalSize ->
    OSSLog.logDebug("getobj_progress: $currentSize  total_size: $totalSize", false)
}
val task = oss.asyncGetObject(get, 
        object : OSSCompletedCallback<GetObjectRequest?, GetObjectResult> {
    override fun onSuccess(request: GetObjectRequest?, result: GetObjectResult) {
        // 请求成功
        val inputStream = result.objectContent
        val buffer = ByteArray(2048)
        var len: Int
        try {
            while (inputStream.read(buffer).also { len = it } != -1) {
                // 处理下载的数据
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }

    override fun onFailure(
            request: GetObjectRequest?, 
            clientExcepion: ClientException, 
            serviceException: ServiceException
    ) {
        // 请求异常
        if (clientExcepion != null) {
            // 本地异常，如网络异常等
            clientExcepion.printStackTrace()
        }
        if (serviceException != null) {
            // 服务异常
            Log.e("ErrorCode", serviceException.errorCode)
            Log.e("RequestId", serviceException.requestId)
            Log.e("HostId", serviceException.hostId)
            Log.e("RawMessage", serviceException.rawMessage)
        }
    }
})
```

#### 范围下载

如果仅需要文件中的部分数据，可以使用范围下载，下载指定范围内的数据。所谓的指定范围，是指从某个字节所在位置到另一个字节所在位置的区间。范围下载的示例代码如下：

```
// objectKey等同于objectName，表示下载文件到OSS时需要指定包含文件后缀在内的完整路径，例如abc/efg/123.jpg
val get = GetObjectRequest("<bucketName>", "<objectKey>")

// 设置范围
get.range = Range(0, 99) // 下载0到99字节共100个字节，文件范围从0开始计算
// get.setRange(new Range(100, Range.INFINITE)); // 下载从100个字节到结尾
        
val task = oss.asyncGetObject(get, 
        object : OSSCompletedCallback<GetObjectRequest?, GetObjectResult> {
    override fun onSuccess(request: GetObjectRequest?, result: GetObjectResult) {
        // 请求成功
        val inputStream = result.objectContent
        val buffer = ByteArray(2048)
        var len: Int
        try {
            while (inputStream.read(buffer).also { len = it } != -1) {
                // 处理下载的数据
            }
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }

    override fun onFailure(
            request: GetObjectRequest?, 
            clientExcepion: ClientException, 
            serviceException: ServiceException
    ) {
        // 请求异常
        if (clientExcepion != null) {
            // 本地异常，如网络异常等
            clientExcepion.printStackTrace()
        }
        if (serviceException != null) {
            // 服务异常
            Log.e("ErrorCode", serviceException.errorCode)
            Log.e("RequestId", serviceException.requestId)
            Log.e("HostId", serviceException.hostId)
            Log.e("RawMessage", serviceException.rawMessage)
        }
    }
})
```

#### 限定条件下载

下载文件时，可以指定一个或多个限定条件。如果满足限定条件，则下载文件；否则就返回错误，且不下载文件。阿里云OSS Android SDK提供的限定条件有：

|参数|描述|
|:--|:--|
|If-Modified-Since|如果指定的时间早于实际修改时间，则正常传输文件，否则返回错误（304 Not modified）|
|If-Unmodified-Since|如果指定的时间等于或者晚于文件实际修改时间，则正常传输文件，否则返回错误（412 Precondition failed）|
|If-Match|如果指定的ETag和OSS文件的ETag匹配，则正常传输文件，否则返回错误（412 Precondition failed）|
|If-None-Match|如果指定的ETag和OSS文件的ETag不匹配，则正常传输文件，否则返回错误（304 Not modified）|

>注意，If-Modified-Since和If-Unmodified-Since可以同时存在；If-Match和If-None-Match可以同时存在。ETag可以通过ossClient.getObjectMeta方法获取。

限定条件下载的示例代码如下：

```
val request = GetObjectRequest("bucketName", "objectName")
// 设置请求头信息
request.requestHeaders = mapOf(···)
val task = oss.asyncGetObject(request, 
        object : OSSCompletedCallback<GetObjectRequest?, GetObjectResult> {
    override fun onSuccess(request: GetObjectRequest?, result: GetObjectResult?) {
        // TODO
    }

    override fun onFailure(
            request: GetObjectRequest?,
            clientException: ClientException?,
            serviceException: ServiceException?
    ) {
        // TODO
    }

})
// 等待完成限定条件下载任务
task.waitUntilFinished()
```