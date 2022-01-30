进行Android开发，除了需要掌握如何使用Android Studio之外，还要知道使用命令行工具，尤其是ADB（Android Debug Bridge）工具。由于Android系统的内核是Linux，因此一部分Linux指令是可以直接使用的，但需要先通过ADB调出相应的shell工具。如果安装了Android Studio，那么ADB工具会自动安装配置好，否则需要手动安装并和Java一样，需要在系统环境变量中进行额外配置。ADB工具可以在Android Studio的“Terminal”窗口使用，也可以在相应的系统命令行程序里调用。

## 常用ADB指令

|用途|指令|示例说明|
|:--|:--|:--|
|启动ADB服务|adb start-server||
|终止ADB服务|adb kill-server||
|重启ADB并以Root身份运行|adb root||
|启动Linux Shell|adb shell||
|查看当前连接的设备|adb devices||
|通过网络连接设备|adb connect ip:port|adb connect 192.168.0.100:5555（常见的端口号是5555）|
|取消设备连接|adb disconnect ip:port|adb disconnect 192.168.0.100:5555|
|从设备复制文件/目录到计算机|adb pull device_dir pc_dir|adb pull /sdcard/example.png（如果不指明pc_dir，则默认为开发者执行该指令时所处的目录）|
|从计算机复制文件/目录到设备|adb push pc_dir device_dir||
|从设备外部安装应用|adb install pc_dir/apk_name.apk|adb install C:\User\User\Desktop\example.apk|
|利用设备内部安装包安装应用|adb shell pm install -r absolute_dir/package_name.apk|adb shell pm install -r /data/data/files/example.apk（注意必须使用绝对路径，参数-r表示强制安装）|
|卸载应用|adb uninstall package_name|adb uninstall com.baidu.searchbox|
|清除应用所有数据|adb shell pm clear package_name|adb shell pm clear com.android.browser|
|以管理员权限卸载应用|adb shell pm uninstall --user 0 package_name|adb shell pm uninstall --user 0 com.baidu.searchbox|
|以管理员权限强制停用应用|adb shell pm disable-user package_name|adb shell pm disable-user com.baidu.searchbox|
|强制停止应用运行|adb shell am force-stop package_name|adb shell am force-stop com.baidu.searchbox|
|查看设备安装的应用列表|adb shell pm list packages|不使用参数表示列出所有包，-f表示列出包和包相关联的文件，-d表述列出被禁用的包，-e表示列出已启用的包，-s过滤出系统包，-3过滤出第三方包，-i输出包和安装信息|
|查看当前所处窗口|dumpsys window \| grep mCurrentFocus|需要先执行adb shell进入shell环境再使用|
|查看系统任务栈情况|adb shell dumpsys activity||
|启动应用程序|adb shell am start package_name|adb shell am start com.android.browser|
|针对特定设备执行命令|adb -s devices_name shell ······|adb -s 192.168.0.100:5555 shell am force-stop com.baidu.searchbox|
|输入文本内容|adb shell input text "some_content"|adb shell input text "fuck Baidu"|
|输入按键事件|adb shell input keyevent key_code|adb shell input keyevent 4（常用key_code：电源键26，菜单键82，返回键4）|
|启动Activity|adb shell am start -n package_name/package_name.SomeActivity|adb shell am start -n com.android.browser/com.android.browser.BrowserActivity（注意Activity要在AndroidManifest文件中设置为export = true）|
|通过指定组件启动服务|adb shell am startservice -n package_name/.service_class_name|adb shell am startservice -n com.example.app/.DemoService|
|通过指定action启动服务|adb shell am startservice -a some_action|adb shell am startservice -a "android.intent.action.CALL"|
|发送广播|adb shell am broadcast -a action_or_category|adb shell am broadcast -a "android.intent.action.AdupsFota.WriteCommandReceiver"|
|当前窗口截屏|adb shell screencap -p /sdcard/pic_name.png||
|监控当前运行的应用程序有哪些|adb shell am monitor||

## 常用Linux命令

>注意，**所有命令都可以通过"man [command]"来查看详情，包括man命令本身。 man可以提供被查询命令的所有选项和参数类型等信息。**

### 基础命令

+ exit：退出shell
+ echo：在终端输出字符串或者从变量提取出来的值
+ date：以指定格式显示和设置日期
+ reboot：重启系统
+ poweroff：关机
+ wget：从指定地址下载文件
+ ps：查看系统进程状态
+ top：动态监视进程活动与系统负载等信息
+ pidof：查询某个指定服务进程的PID值
+ kill：终止某个指定PID的服务进程
+ killall：终止某个指定名称的服务所对应的全部进程
+ clear：将之前的终端信息隐藏至上一页
+ reset：彻底清空终端屏幕（运行速度缓慢）
+ alias：为系统命令设置别名

### 系统状态检测命令

+ ipconfig：获取网卡配置与网络状态等信息
+ uname：查看系统内核与系统版本等信息
+ uptime：查看系统的负载信息
+ free：显示当前系统中内存的使用量信息
+ who：查看当前登入主机的用户终端信息
+ last：查看所有系统的登录记录
+ history：显示历史执行过的命令
+ sosreport：收集系统配置及架构信息并输出诊断文档

### 文本文件编辑命令

+ cat：查看内容较少的纯文本文件
+ more：查看内容较多的纯文本文件
+ head：查看纯文本文档的前N行
+ tail：查看纯文本文档的后N行或持续刷新内容
+ tr：替换文本文件中的字符
+ wc：统计指定文本的行数、字数、字节数
+ stat：查看文件的具体存储信息和时间等信息
+ cut：按“列”提取文本字符
+ diff：比较多个文本文件的差异

### 文件目录管理命令

+ pwd：显示用户当前所处的工作目录
+ cd：切换工作路径
+ ls：显示目录中的文件信息
+ touch：创建空白文件或设置文件的时间
+ mkdir：创建空白的目录
+ cp：复制文件或目录
+ mv：剪切文件或将文件重命名
+ rm：删除文件或目录
+ wc：统计指定文本的行数、字数、字节数
+ dd：按照指定大小和个数的数据块来复制文件或转换文件
+ file：查看文件的类型

## 打包与搜索命令

+ zip / unzip：打包/解压文件
+ tar：打包或解压文件
+ grep：在文本中执行关键词搜索
+ find：按照指定条件来查找文件