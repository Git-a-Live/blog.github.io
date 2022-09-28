[ZXing](https://github.com/zxing/zxing)是Google开源的一个1D/2D条码图像处理库，使用Java实现，可支持多种格式的1D/2D条码。它包含了联系到其他语言的端口。ZXing可以实现使用手机的内置摄像头完成条码的扫描及解码。ZXing的名字来源于“zebra crossing”，直译过来就是斑马条纹，所以项目的Logo也是一匹斑马：

![](https://camo.githubusercontent.com/1c0ff84f124fc90e102e85e7c61f2d21e480570632e72b06cdef2109e6c13fe8/68747470733a2f2f7261772e6769746875622e636f6d2f77696b692f7a78696e672f7a78696e672f7a78696e672d6c6f676f2e706e67)

ZXing只提供了基本的条码生成识别功能，如果想满足开发需求，可能需要自行封装。按照开发团队的说法：

> The project is in maintenance mode, meaning, changes are driven by contributed patches. Only bug fixes and minor enhancements will be considered. ...There is otherwise no active development or roadmap for this project. It is "DIY".
> 
> 本项目处于维护模式，即只有提交的补丁会引起项目变化——除bug修复和主要的功能增强外，其他提交都不考虑。……此外，本项目也没有什么活跃的开发计划或路线图，开发者可DIY自行封装。

限于篇幅，这里只介绍二维码（QR Code）的生成和识别。

## 依赖库导入

ZXing支持`.jar`文件直接导入，不过Google在Maven Center上已经提供了[远程依赖](https://mvnrepository.com/artifact/com.google.zxing/core)方式，所以Android开发中使用远程依赖更为方便：

```
dependencies {
    implementation 'com.google.zxing:core:${specified_version}'
}
```

由于ZXing有生成、识别条码的功能，因此集成使用时必须配置相机和存储等权限：

```
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
```

在Android 6.0或更高版本的系统上，还需要配置动态权限获取的业务逻辑。

## 基本使用

### 生成二维码图片

首先要设置生成二维码所需的重要参数：
```
// 用Hashtable类型设置二维码参数
val qrParam = Hashtable<EncodeHintType, Any>().apply {
    // 设置二维码纠错级别，这里选择最高级别H
    put(EncodeHintType.ERROR_CORRECTION, ErrorCorrectionLevel.H)
    // 设置字符编码方式
    put(EncodeHintType.CHARACTER_SET, Charsets.UTF_8.name())
}
```

接着构建BitMatrix类型对象：
```
// 将内容转换成一个具有特定尺寸的比特矩阵，条码格式设置为BarcodeFormat.QR_CODE，即二维码
val bitMatrix = MultiFormatWriter().encode(content, BarcodeFormat.QR_CODE, width, height, qrParam)
```

然后将BitMatrix转换成一维数组：
```
// 利用Kotlin扩展函数为BitMatrix对象提供转换一维数组的功能
fun BitMatrix.generateQRData(): IntArray {
    val data = IntArray(width * height)
    for (y in 0 until height) {
        for (x in 0 until width) {
            // 在这里设置像素点颜色
            data[y * width + x] = if (get(x, y)) Color.BLACK else Color.WHITE
        }
    }
    return data
}
```

最后利用一维数组的数据生成Bitmap图片：
```
// BitMatrix对象转换成一维数组
val data = bitMatrix.generateQRData()
// 创建一张bitmap图片，采用最高图片效果ARGB_8888
val bitmap = Bitmap.createBitmap(bitMatrix.width, bitMatrix.height, Bitmap.Config.ARGB_8888).apply {
    // 传入二维码颜色数组，生成图片颜色
    setPixels(data, 0, width, 0, 0, width, height) 
}
```

至此，利用特定内容生成二维码图片的流程已经基本完成（类似下图所示），只要将生成的Bitmap图片传递给ImageView控件就可以展示了。

![](pics/zxing.png)

有些二维码的中心位置还会有图标，下面介绍如何为二维码设置中心图标。首先要将图标转换成Bitmap对象（转换过程略），然后进行以下初始化操作：
```
// 利用Kotlin扩展函数为Bitmap类型的中心图标提供初始化功能
fun Bitmap.initPortrait(portraitSize: Int): Bitmap? {
    var bitmap: Bitmap? = null
    try {
        val matrix = Matrix().apply {
            setScale(portraitSize * 1F  / width, portraitSize * 1F / height)
        }
        bitmap = Bitmap.createBitmap(this, 0, 0, width, height, matrix, true)
    }
    return bitmap
}
```

接着开始为之前生成的二维码图片添加中心图标：
```
// 利用Kotlin扩展函数为二维码的Bitmap图片提供绘制中心图标的功能
fun Bitmap.createQRCodeBitmapWithPortrait(qrCodeSize: Int, portraitSize: Int, portrait: Bitmap) {
    portrait.initPortrait(portraitSize)?.let {
        val left = (qrCodeSize - it.width) / 2
        val right = left + it.width
        val top = (qrCodeSize - it.height) / 2
        val bottom = top + it.height
        // 设置图标要显示的位置，即居中显示
        val rect = Rect(left, top, right, bottom)
        // 取得二维码图片上的画笔，即要在二维码图片上绘制图标
        val canvas = Canvas(this)
        // 设置要绘制的图标范围大小
        val rect2 = Rect(0, 0, it.width, it.height)
        // 开始绘制
        canvas.drawBitmap(it, rect2, rect, null)
    }
}
```

在上述流程完成之后，二维码就添加上了中心图标，类似下图所示：

![](pics/zxing2.png)


### 识别二维码内容

识别二维码需要先配置以下参数：
```
// 设置二维码读取配置参数
val readerParam = EnumMap<DecodeHintType, Any>(DecodeHintType::class.java).apply {
    // 配置优化精度
    put(DecodeHintType.TRY_HARDER, true)
    // 配置解码所使用的编码格式
    put(DecodeHintType.CHARACTER_SET, Charsets.UTF_8)
}
```

接着创建二维码解析功能所需的核心对象——解析器：
```
val qrReader = QRCodeReader()
```

然后对转换成Bitmap的二维码图片进行处理：
```
// 定义一个整型数组，长度为转换成Bitmap图片的长度 ✖️ 宽度
val array = IntArray(bitmap.width * bitmap.height)

// 将Bitmap的像素点全部导入到刚才定义好的数组
bitmap.getPixels(array, 0, width, 0, 0, width, height)

// 将Bitmap转换成BinaryBitmap
val binaryBitmap = BinaryBitmap(HybridBinarizer(RGBLuminanceSource(bitmap.width, bitmap.height, array)))
```

最后传给二维码解析器进行解析，将内容解析出来：
```
val cintent = qrReader.decode(binaryBitmap, readerParam).text
```

至此，利用ZXing已经可以开发出最基本的二维码生成与识别功能。