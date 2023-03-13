## 蓝牙技术简介

蓝牙（Bluetooth）是一种无线技术标准，可实现固定设备、移动设备和楼宇个人域网之间的短距离数据交换。蓝牙技术具有十分广泛的应用，它具有耗电量低、成本低、安全性、稳定性、易用性等优点，在物联网设备上的占有率非常高。目前蓝牙技术可以分为传统蓝牙（Bluetooth Classic）和低功耗蓝牙（Bluetooth Low Energy，BLE）两种，它们的主要区别如下：

![](pics/bluetooth.svg)

+ 传统蓝牙也被称为Bluetooth基本速率/增强数据速率（BR/EDR），是一种低功率无线电，可在2.4GHz非授权工业、科学和医疗（ISM）频段的79个频道上进行数据传输。它支持点对点设备通信，并且主要用于实现无线音频流，目前已成为支撑无线扬声器、耳机和车载娱乐系统等技术的标准无线电协议。传统蓝牙还能应用于包括移动打印在内的其他数据传输场景。

+ 低功耗蓝牙是为非常低的功率操作而设计的。BLE在2.4GHz非授权ISM频段的40个信道上传输数据，支持多种通信拓扑结构，目前已从点对点通信扩展到广播通信。 此外，基于[Bluetooth mesh](https://www.bluetooth.com/learn-about-bluetooth/recent-enhancements/mesh/)，蓝牙还支持创建可靠的、大规模的设备网络。尽管蓝牙最初以其设备通信功能而闻名，但现在BLE也被广泛用于设备定位技术当中，以满足人们对**室内高精度定位服务**日益增长的需求。

Android平台支持蓝牙网络堆栈，能让设备以无线方式与其他蓝牙设备交换数据。应用框架提供了通过Android Bluetooth API访问蓝牙功能的权限。这些API允许应用以无线方式连接到其他蓝牙设备，从而实现点对点或多点广播的无线功能。按照Google官方的说法，通过这些API，开发者能让应用实现以下功能：

+ 扫描其他蓝牙设备；
+ 在本地蓝牙适配器中查询已配对过的设备；
+ 建立[RFCOMM](https://www.bluetooth.com/specifications/specs/rfcomm-1-1/)通信通道；
+ 通过“服务发现”来连接其他设备；
+ 在设备间进行数据传输；
+ 管理多个连接。

设备通过蓝牙进行交互的流程大致为：为了让支持蓝牙的设备能够在彼此之间传输数据，它们必须先通过**配对过程**形成通信通道。其中一台设备（可检测到的设备）需将自身设置为“允许接收传入的连接请求”的状态，另一台设备会使用**服务发现过程**找到此可检测到的设备。在可检测到的设备接受配对请求后，这两台设备会完成**绑定过程**，并在此期间**交换安全密钥**。二者会缓存这些密钥，以供日后使用。

在完成配对和绑定过程后，两台设备会建立会话交换信息。当会话完成时，发起配对请求的设备会**释放**已将其链接到可检测设备的通道，但是请注意，这两台设备仍能保持绑定状态——因为双方都已经完成配对，并持有对方的安全密钥。只要二者处于彼此的检测范围内且均未移除绑定，便可自动重新连接。

## 蓝牙基本使用

本节内容主要介绍如何通过Android Bluetooth API来完成使用蓝牙进行通信的四大必需任务：设置蓝牙、查找局部区域内的配对设备或可用设备、连接设备，以及在设备之间传输数据。

### 权限配置

按照Google官方文档的介绍，使用蓝牙功能需要声明如下权限：

```
<manifest>
    <!-- 在Android 12以下的旧设备上声明传统蓝牙的权限 -->
    <uses-permission android:name="android.permission.BLUETOOTH"
        android:maxSdkVersion="30" />
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"
        android:maxSdkVersion="30" />

    <!-- 在Android 10 - 11的设备上声明以下权限用于通过蓝牙给设备进行定位 -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />

    <!-- 在Android 9或更低版本的设备上声明该权限用于通过蓝牙给设备进行定位 -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" 
        android:maxSdkVersion="28"/>

    <!-- 在Android 12+的设备上声明该权限以寻找蓝牙设备，若不需要从扫描结果中获取设备的定位信息，则声明“neverForLocation” -->
    <uses-permission android:name="android.permission.BLUETOOTH_SCAN"
        android:usesPermissionFlags="neverForLocation" />

    <!-- 在Android 12+的设备上声明该权限设置本机可见性 -->
    <uses-permission android:name="android.permission.BLUETOOTH_ADVERTISE" />

    <!-- 在Android 12+设备上声明该权限以同其他已配对设备通信 -->
    <uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
    ...
</manifest>
```

如果开发者的应用项目将蓝牙作为关键功能，那么还需要参考下面的方式声明应用需要用到蓝牙功能，并指定使用的蓝牙类型：

```
<!-- 声明使用传统蓝牙 -->
<uses-feature android:name="android.hardware.bluetooth" android:required="true"/>

<!-- 声明使用低功耗蓝牙 -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

声明这些的作用是什么呢？主要是给开发者上架应用到Google Play用的。如果Google Play检测到用户手机没有指定类型的蓝牙功能，且应用又必须使用到蓝牙，那么就会在商店搜索结果中自动隐藏那些需要用到蓝牙功能的应用；反过来，如果开发者希望自己的应用也能在那些缺少蓝牙功能的设备上使用，那就把`android:required`设为false，并且还要在代码中增加检查蓝牙功能是否存在的业务逻辑，防止运行时直接引发应用崩溃。

### 设置蓝牙

在使用蓝牙功能之前，必须先判断设备是否支持蓝牙——开发者不能对使用本应用的设备做出过于随意的假设。下面的示例代码提供了两种检测方式：

```
// 调用系统服务，判断是否存在BluetoothAdapter
fun checkBluetoothAvailability(context: Context): Boolean {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        val bluetoothManager: BluetoothManager? = context.getSystemService(BluetoothManager::class.java)
        bluetoothManager != null && bluetoothManager.adapter != null
    } else {
        // 注意，BluetoothAdapter.getDefaultAdapter()目前已经被标记为废弃
        BluetoothAdapter.getDefaultAdapter() != null
    }
}

// 通过PackageManager直接判断是否有蓝牙服务
fun Context.hasBluetooth(): Boolean = packageManager.hasSystemFeature(PackageManager.FEATURE_BLUETOOTH)
```

接着就是启动蓝牙了。注意，启动蓝牙的操作在Android 6.0以下、Android 6.0 ~ Android 11以及Android 12+的业务逻辑是存在差异的，具体可以参考下面的示例代码：

```
fun Activity.enableBluetooth() {
    // 在Android 12+，由于在AndroidManifest.xml里面声明了BLUETOOTH_CONNECT权限，因此必须先检查授权情况
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        if (ActivityCompat.checkSelfPermission(this, 
                Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(arrayOf(Manifest.permission.BLUETOOTH_CONNECT), REQUEST_CODE)
        }
    } else if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
        // Android 6.0以下只能用BluetoothAdapter来操作
        BluetoothAdapter.getDefaultAdapter()?.let {
            if (!it.isEnabled) {
                it.enable()
            }
        }
        return
    }

    // 对于Android 6.0 ~ Android 11的设备，可以直接请求开启蓝牙，不需要额外授权
    bluetoothManager?.adapter?.let {
        if (!it.isEnabled) {
            // 如果不需要返回结果，就用startActivity()，否则用startActivityForResult()
            startActivity(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE))
        }
    }
}
```

如果要让应用监听蓝牙启动状态并执行相应的业务逻辑，可以创建一个BroadcastReceiver专门监听`BluetoothAdapter.ACTION_STATE_CHANGED`这个广播。该广播包含有`EXTRA_STATE`和 `EXTRA_PREVIOUS_STATE`这两个字段，分别表示新旧两种状态。它们返回的状态值有`STATE_TURNING_ON`、`STATE_ON`、`STATE_TURNING_OFF`以及`STATE_OFF`这四种，开发者可以根据这四种状态来编写对应的业务逻辑。

如果要**在应用中**关闭蓝牙（注意不是让用户在系统任务栏中关闭），那么业务逻辑上就要有一些调整了：

```
fun Activity.disableBluetooth() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        if (ActivityCompat.checkSelfPermission(this,
                Manifest.permission.BLUETOOTH_CONNECT) != PackageManager.PERMISSION_GRANTED) {
            requestPermissions(arrayOf(Manifest.permission.BLUETOOTH_CONNECT), 114514)
        }
    }

    // 尽管Android 6.0以下和Android 6.0+的设备获取BluetoothAdapter的方式不同，但它们都要调用被废弃的disable()方法才能主动关闭蓝牙
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
        BluetoothAdapter.getDefaultAdapter()
    } else {
        bluetoothManager?.adapter
    }?.let {
        if (it.isEnabled) {
            it.disable()
        }
    }
}
```

### 查找设备

### 连接设备

## 低功耗蓝牙

## BLE音频