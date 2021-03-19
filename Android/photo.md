在开发应用的时候，如果需要使用到拍照功能，那么通常都会选择调用系统中存在的相机应用，从而使用摄像头：

```
val intent = Intent("android.media.action.IMAGE_CAPTURE")
startActivityForResult(intent, REQUEST_CODE)
```

在调用完摄像头之后，开发者可能还需要将拍好的图片反馈到应用界面上，以方便使用者可以检查图片是否符合要求， 这时候可以通过以下代码，通过ImageView来展示缩略图：

```
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)
    if (requestCode == REQUEST_CODE && resultCode == Activity.RESULT_OK) {
        //Android相机应用会对返回Intent中的照片进行编码。下面的代码会检索此图片并将其显示在一个ImageView中
        val bitMap = data?.extras?.get("data") as Bitmap
        //相机应用的Intent会把照片作为extra中的小型Bitmap，传递给onActivityResult()，使用键名"data"
        imageView.setImageBitmap(bitMap)
    }
}
```

当然，上述过程中并不涉及到图片的存储，因此通过这种方法拍摄并显示到应用界面上的图片并不会被保存到设备中。 如果有这样的需求，那么就要指定图片保存的位置，以及图片保存时的唯一名称，然后在程序中将相机应用拍摄图片后， 从指定位置读取图片以供用户进行操作。

通常情况下，用户使用设备相机拍摄的所有照片都应保存在设备的公共外部存储设备中，以供所有应用访问。 合适的共享照片目录由具有DIRECTORY_PICTURES参数的getExternalStoragePublicDirectory()提供。 由于这种方法提供的目录会在所有应用之间共享，因此对该目录执行读写操作分别需要READ_EXTERNAL_STORAGE和WRITE_EXTERNAL_STORAGE权限。 拥有写入权限即暗示可以读取，因此，如果开发者需要将图片写入到外部存储设备，只需请求如下权限：

```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
确定文件目录后，还需要指定一个不会跟其他文件产生冲突的文件名。一种可借鉴的做法就是使用时间戳对照片进行命名。