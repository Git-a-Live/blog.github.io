[ZXing](https://github.com/zxing/zxing)是Google开源的一个1D/2D条码图像处理库，使用Java实现，可支持多种格式的1D/2D条码。它包含了联系到其他语言的端口。ZXing可以实现使用手机的内置摄像头完成条码的扫描及解码。ZXing的名字来源于“zebra crossing”，直译过来就是斑马条纹，所以项目的Logo也是一匹斑马：

![](https://camo.githubusercontent.com/1c0ff84f124fc90e102e85e7c61f2d21e480570632e72b06cdef2109e6c13fe8/68747470733a2f2f7261772e6769746875622e636f6d2f77696b692f7a78696e672f7a78696e672f7a78696e672d6c6f676f2e706e67)

ZXing只提供了基本的条码生成识别功能，如果想满足开发需求，可能需要自行封装。按照开发团队的说法：

> The project is in maintenance mode, meaning, changes are driven by contributed patches. Only bug fixes and minor enhancements will be considered. ...There is otherwise no active development or roadmap for this project. It is "DIY".
> 
> 本项目处于维护模式，即只有提交的补丁会引起项目变化——除bug修复和主要的功能增强外，其他提交都不考虑。……此外，本项目也没有什么活跃的开发计划或路线图，开发者可DIY自行封装。

## 依赖库导入

ZXing支持`.jar`文件直接导入，不过Google在Maven Center上已经提供了[远程依赖](https://mvnrepository.com/artifact/com.google.zxing/core)方式，所以Android开发中使用远程依赖更为方便：

```
dependencies {
    implementation 'com.google.zxing:core:${specified_version}'
}
```

## 基本使用



