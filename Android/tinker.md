[Tinker](https://github.com/Tencent/tinker)是腾讯推出的一个热修复框架。所谓热修复，是指通过服务端下发补丁包，让已安装的客户端接收后进行动态更新，从而在不用重新安装APP的情况下修复应用缺陷。热修复在缺陷修复效率上要高于应用分发平台，但是由于绕开了应用分发平台的审核机制，该技术可能会成为某些企业或开发人员入侵用户设备的跳板。

对于审核机制严格的应用分发平台，比如Google Play和App Store，最好不要使用热修复技术，否则将会面临违规和应用下架的风险。

截至2021年4月7日，Tinker有以下限制：

1. 不支持修改AndroidManifest文件，比如动态添加四大组件；
2. 不支持部分系统版本为Android 5.0的三星机型；
3. **不能在上架Google Play的应用中集成使用Tinker**。

## Tinker集成

+ **步骤一：在项目级build.gradle文件中添加依赖**

```
buildscript {
    dependencies {
        ···
        classpath "com.tencent.tinker:tinker-patch-gradle-plugin:$specific_version"
    }
}
```

+ **步骤二：在app级build.gradle文件中添加依赖**

```
dependencies {
    //optional, help to generate the final application
    annotationProcessor "com.tencent.tinker:tinker-android-anno:$specific_version"
    //tinker's main Android lib
    implementation "com.tencent.tinker:tinker-android-lib:$specific_version"
}
```

+ **步骤三：创建自定义Application和ApplicationLike以初始化SDK**

创建一个继承于TinkerApplication的Application子类：

```
public class SampleApplication extends TinkerApplication {
    public SampleApplication() {
      super(
        //tinkerFlags, which types is supported
        //dex only, library only, all support
        ShareConstants.TINKER_ENABLE_ALL,
        // This is passed as a string so the shell application does not
        // have a binary dependency on your ApplicationLifeCycle class.
        "tinker.sample.android.app.SampleApplicationLike");
    }
}
```

创建一个继承于DefaultApplicationLike的ApplicationLike子类：

```
/*方式一*/
@DefaultLifeCycle(
application = "tinker.sample.android.app.SampleApplication",             //application name to generate
flags = ShareConstants.TINKER_ENABLE_ALL)                                //tinkerFlags above
public class SampleApplicationLike extends DefaultApplicationLike {
    //TODO
}
```


