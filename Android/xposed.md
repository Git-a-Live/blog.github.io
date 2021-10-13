## 安装Xposed框架

>注意，设备必须处于Root状态，无论是真机还是模拟器均可。

首先前往[官网](https://xposed-installer.com/)下载Xposed Installer，然后安装到设备上。接着打开安装好的Xposed Installer，如下图所示。

![](https://gitee.com/dellg3/images/raw/master/xposed1.png)

点击“安装/更新”，下载安装Xposed框架：

![](https://gitee.com/dellg3/images/raw/master/MuMu20200617122321.png)

安装并重启之后：

![](https://gitee.com/dellg3/images/raw/master/MuMu20200617122813.png)

## 编写测试Xposed模块

首先使用Android Studio创建一个新项目，然后在gradle文件中添加如下依赖并同步：

```
···
dependencies {
    ···
    compileOnly 'de.robv.android.xposed:api:$latest_version'
    compileOnly 'de.robv.android.xposed:api:$latest_version:sources'

}
```
之后在AndroidManifest.xml文件的application标签下添加如下信息：

```
        <!-- 1、标识自己是否为一个Xposed模块 -->
        <meta-data
            android:name="xposedmodule"
            android:value="true"/>
            
        <!-- 2、Xposed模块的描述信息 -->
        <meta-data
            android:name="xposeddescription"
            android:value="a sample for xposed"/>
            
        <!-- 3、支持Xposed框架的最低版本 -->
        <meta-data
            android:name="xposedminversion"
            android:value="53"/>
```

编写劫持文件：

```
···
import de.robv.android.xposed.IXposedHookLoadPackage
import de.robv.android.xposed.XC_MethodHook
import de.robv.android.xposed.XposedHelpers.findAndHookMethod
import de.robv.android.xposed.callbacks.XC_LoadPackage

class DemoHooks: IXposedHookLoadPackage {
    override fun handleLoadPackage(lpparam: XC_LoadPackage.LoadPackageParam?) {
        if (lpparam?.packageName.equals("当前包名")){
            val clazz = lpparam?.classLoader?.loadClass("待Hook方法所在类的路径")
            val hook = object : XC_MethodHook(){
                override fun beforeHookedMethod(param: MethodHookParam?) {
                    super.beforeHookedMethod(param)
                    //TODO
                }

                override fun afterHookedMethod(param: MethodHookParam?) {
                    super.afterHookedMethod(param)
                    //TODO
                }
            }
            findAndHookMethod(clazz,"方法名",待Hook方法参数1类型,待Hook方法参数2类型,···,hook)
        }
    }
}
```

创建asset文件夹以及xposed_init文件：

通过File - New - Folder - Assets Folder创建assets文件夹，然后对assets文件夹右键 - New - File，将该文件命名为xposed_init，在其中写入上一步创建的劫持文件的完整路径然后保存即可：

> com.example.app.DemoHooks

完成其他部分的代码编写之后，编译运行，将项目打包成应用并安装到设备上。模块被安装后就会马上被Xposed所识别，并弹出提示要求激活模块并重启（或软重启）设备。在“模块”页面中勾选刚刚安装好的应用，然后回到“框架”页面打开左上角菜单，点击“重启设备”或者“软重启”：

![](https://gitee.com/dellg3/images/raw/master/MuMu20200617132635.png)

![](https://gitee.com/dellg3/images/raw/master/MuMu20200617132703.png)

重启之后再启动刚刚安装的Xposed模块，就可以通过日志、Toast等方式观察到被Hook的效果了。