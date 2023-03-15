## 蓝牙技术简介

蓝牙（Bluetooth）是一种无线技术标准，可实现固定设备、移动设备和楼宇个人域网之间的短距离数据交换。蓝牙技术具有十分广泛的应用，它具有耗电量低、成本低、安全性、稳定性、易用性等优点，在物联网设备上的占有率非常高。目前蓝牙技术可以分为传统经典蓝牙（Bluetooth Classic）和低功耗蓝牙（Bluetooth Low Energy，BLE）两种，它们的主要区别如下：

![](pics/bluetooth.svg)

+ 经典蓝牙也被称为Bluetooth基本速率/增强数据速率（BR/EDR），是一种低功率无线电，可在2.4GHz非授权工业、科学和医疗（ISM）频段的79个频道上进行数据传输。它支持点对点设备通信，并且主要用于实现无线音频流，目前已成为支撑无线扬声器、耳机和车载娱乐系统等技术的标准无线电协议。传统蓝牙还能应用于包括移动打印在内的其他数据传输场景。

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
<!-- 声明需要调用到经典蓝牙 -->
<uses-feature android:name="android.hardware.bluetooth" android:required="true"/>

<!-- 声明需要调用到低功耗蓝牙 -->
<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>
```

声明这些的作用是什么呢？主要是给开发者上架应用到Google Play用的。如果Google Play检测到用户手机没有指定类型的蓝牙功能，且应用又必须使用到蓝牙，那么就会在商店搜索结果中自动隐藏那些需要用到蓝牙功能的应用；反过来，如果开发者希望自己的应用也能在那些缺少蓝牙功能的设备上使用，那就把`android:required`设为false，并且还要在代码中增加检查蓝牙功能是否存在的业务逻辑，防止运行时直接引发应用崩溃。

### 设置蓝牙

在使用蓝牙功能之前，必须先判断设备是否支持蓝牙——开发者不能对使用本应用的设备做出过于随意的假设。下面的示例代码提供了两种检测方式：

```
// 调用系统服务，判断是否存在BluetoothAdapter
fun Context.checkBluetoothAvailability(): Boolean {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        val bluetoothManager: BluetoothManager? = getSystemService(BluetoothManager::class.java)
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
    getSystemService(BluetoothManager::class.java)?.adapter?.let {
        if (!it.isEnabled) {
            // 如果不需要返回结果，就用startActivity()，否则用startActivityForResult()
            startActivity(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE))
        }
    }
}
```

如果要让应用监听蓝牙启动状态并执行相应的业务逻辑，可以创建一个BroadcastReceiver专门监听`BluetoothAdapter.ACTION_STATE_CHANGED`这个广播。该广播包含有`EXTRA_STATE`和 `EXTRA_PREVIOUS_STATE`这两个字段，分别表示新旧两种状态。它们返回的状态值有`STATE_TURNING_ON`、`STATE_ON`、`STATE_TURNING_OFF`以及`STATE_OFF`这四种，开发者可以根据这四种状态来编写对应的业务逻辑。

除了`BluetoothAdapter.ACTION_REQUEST_ENABLE`可以开启蓝牙之外，`BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE`这个Intent也能做同样的事，唯一的区别就是后者会提示用户设备会在一段时间（比如默认120s）内处于可被发现的状态：

```
val discoverableIntent: Intent = Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE).apply {
    // 这里传入的是保持可见性的时间，单位为秒，如果设为0就表示一直可见
    putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300)
}
```

如果要**在应用中**关闭蓝牙（注意不是让用户在系统任务栏中关闭），那么业务逻辑上就要有一些调整了：

```
fun Activity.disableBluetooth() {
    // 检查授权步骤省略
    ···

    // 尽管Android 6.0以下和Android 6.0+的设备获取BluetoothAdapter的方式不同，但它们都要调用被废弃的disable()方法才能主动关闭蓝牙
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
        BluetoothAdapter.getDefaultAdapter()
    } else {
        getSystemService(BluetoothManager::class.java)?.adapter
    }?.let {
        if (it.isEnabled) {
            it.disable()
        }
    }
}
```

### 查找设备

查找蓝牙设备需要用到`BluetoothAdapter`。上一节内容已经初步介绍过如何利用`BluetoothAdapter`来开启和关闭蓝牙，这一节将介绍如何依靠它来查找其他可用的蓝牙设备。

“设备发现”是利用蓝牙查找设备过程当中一个很基础也很重要的概念。它是指在本地区域内（通常不会超过100米范围）扫描、搜索其他启用蓝牙的设备，并请求相关信息的过程。启用蓝牙的设备，必须处于可被检测的状态才能被其他设备所检测发现，从而执行后续的其他步骤。当两个未配对过的设备首次建立蓝牙连接时，用户会接收到一个配对请求；而已经配对过的设备，则会直接基于已保存的蓝牙MAC地址等信息，建立RFCOMM通道进行数据传输和通信。

#### 查找已配对过的设备

对于已经配对过的设备，Android提供了`BluetoothAdapter.getBondedDevices()`，用于返回一个包含有`BluetoothDevice`对象的集合。如下面代码所示：

```
fun Activity.getBondedBluetoothDevices(): Set<BluetoothDevice> {
    // 检查授权步骤省略
    ···

    return if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
        BluetoothAdapter.getDefaultAdapter()
    } else {
        getSystemService(BluetoothManager::class.java)?.adapter
    }?.bondedDevices ?: emptySet()
}
```

`BluetoothDevice`对象包含了蓝牙设备的名称、MAC地址以及类型等属性，这些属性在后面介绍如何建立蓝牙连接进行通信时会用到。

#### 发现其他设备

要发现周围的蓝牙设备，就要调用`BluetoothAdapter.startDiscovery()`，可以参考下面的示例代码：

```
fun Activity.startBluetoothDiscovery() {
    // 检查授权步骤省略
    ···

    // 如果要主动取消发现设备，就执行cancelDiscovery()
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
        BluetoothAdapter.getDefaultAdapter()
    } else {
        getSystemService(BluetoothManager::class.java)?.adapter
    }?.startDiscovery()
}
```

由于发现设备是一个耗时过程（一次扫描大约耗费12秒），因此从启动扫描到获取结果必然是异步的。Android提供的方案是让应用监听`BluetoothDevice.ACTION_FOUND`这个广播，从广播返回的结果中获取扫描发现的设备：

```
val receiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        intent.action?.let {
            if (BluetoothDevice.ACTION_FOUND == it) {
                intent.apply {
                    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU) {
                        getParcelableExtra<BluetoothDevice>(BluetoothDevice.EXTRA_DEVICE)
                    } else {
                        getParcelableExtra(BluetoothDevice.EXTRA_DEVICE, BluetoothDevice::class.java)
                    }?.let { device -> 
                        // TODO: 获取到BluetoothDevice对象
                    }
                }
            }
        }
    }
}

// 注册广播接收器
registerReceiver(receiver, IntentFilter(BluetoothDevice.ACTION_FOUND))
```

如果要监听蓝牙扫描状态，同样使用广播接收器，监听`ACTION_SCAN_MODE_CHANGED`广播，然后获取`EXTRA_SCAN_MODE`和`EXTRA_PREVIOUS_SCAN_MODE`这两个参数，里面包含的结果有`SCAN_MODE_CONNECTABLE_DISCOVERABLE`、`SCAN_MODE_CONNECTABLE`以及`SCAN_MODE_NONE`这三种，分别表示设备处于可检测状态、未处于可检测状态但能收到连接还有既不处于可检测状态也不能收到连接。

### 连接设备

蓝牙连接采用的是C/S模式。对于单台设备而言，它必须同时实现Server和Client两套机制，才能正常地跟其他设备进行连接。无论是作为Server还是Client，设备都得获取到`BluetoothSocket`。Server一侧负责接受连接请求并从连接中获得socket信息；Client一侧负责开启RFCOMM通道并提供socket信息。

当两台设备在同一RFCOMM通道上各自持有一个已处于连接状态的`BluetoothSocket`时，就可以认为它们已经建立连接，这时两台设备就能进行数据传输了。当然，本节内容主要讨论如何连接蓝牙设备，数据传输会在下一节内容展开。

#### 作为Server监听连接请求

Server端需要关注两样东西：`BluetoothServerSocket`和`BluetoothSocket`。前者用来监听连接请求，并在接收请求的时候向Client返回一个`BluetoothSocket`；后者用于蓝牙通信，数据传输的操作都将基于它来进行。如果Server只专注于一对一通信，那么在获得`BluetoothSocket`之后就可以把`BluetoothServerSocket`给关闭了，否则应该继续保留其存在和运行。

Server监听连接请求的操作可以参考下面的示例代码：

```
// 第一步：在已经开启蓝牙功能的前提下，创建BluetoothServerSocket对象
val bluetoothServerSocket = if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
    BluetoothAdapter.getDefaultAdapter()
} else {
     context.getSystemService(BluetoothManager::class.java)?.adapter
}?.listenUsingInsecureRfcommWithServiceRecord(STRING_NAME, UUID)

// 第二步：使用循环开启连接请求监听，如果监听到有请求连接并确认接受，就可以获得一个BluetoothSocket对象
var shouldLoop = true
while (shouldLoop) {
    val bluetoothSocket = try {
        bluetoothServerSocket?.accept()
    } catch (e: Exception) {
        // TODO: 处理异常，并停止监听
        shouldLoop = false
        null
    }

    bluetoothSocket?.let {
        // 这里对获取到的BluetoothSocket对象进行处理
        action(it)
                
        // 第三步：如果只进行一对一通信，在获得BluetoothSocket对象之后即可关闭监听
        bluetoothServerSocket?.close()
        shouldLoop = false
   }
}
```

注意，上面开启监听的步骤不能在主线程中进行，可以选择开启一个子线程或者协程来处理。

#### 作为Client发起连接请求

Client端需要关注`BluetoothDevice`和`BluetoothSocket`，前者代表的是待连接的目标设备，后者的用途跟Server端的基本一样，但是要主动调用`connect()`发起连接请求，具体可以参考下面的示例代码：

```
// 第一步：从已配对设备列表或扫描结果当中选择一个BluetoothDevice对象，用于创建BluetoothSocket
var bluetoothSocket: BluetoothSocket? = null
try {
    // 如果检测到目标设备未绑定过，则首先发起绑定请求
    if (BluetoothDevice.BOND_NONE == bluetoothDevice.bondState) {
        bluetoothDevice.createBond()
        return
    }
    // 配对绑定过的设备，其BluetoothDevice.uuids通常不会为空
    bluetoothDevice.uuids?.randomOrNull()?.uuid?.let { id ->
        // 此处必须使用目标设备的UUID来创建BluetoothSocket，否则会连接失败
        bluetoothSocket = bluetoothDevice.createRfcommSocketToServiceRecord(id)
    }
} catch (e: Exception) {
    ···
}

// 第二步：取消设备扫描搜索任务（如果有的话），否则会影响后面的连接步骤
bluetoothAdapter?.cancelDiscovery()

// 第三步：通过BluetoothSocket，对目标设备发起连接请求，注意要在主线程以外执行该逻辑
try {
     bluetoothSocket?.let {
        // 调用connect方法时，当前线程会被阻塞，直到连接成功或抛出异常
        it.connect()
        // 由于connect()方法会阻塞流程，因此这里通常可以串行拿到连接状态
        if (it.isConnected) {
            ···
        }
    }
} catch (e: Exception) {
    ···
    // 如果发生异常，应当关闭BluetoothSocket
    bluetoothSocket?.close()
}
```

对于从未配对过的两台设备而言，在Client调用`BluetoothDevice.createBond()`发起连接请求时，两边的设备会**自动**弹出类似下图所示的配对请求对话框：

![](pics/bluetooth2.png)

如果对方一直没有操作，那么连接请求会被阻塞，直到对方同意或拒绝配对，或是配对过程超时自动断开连接。

### 数据传输

两台设备建立起蓝牙连接之后，它们就可以通过`BluetoothSocket`进行数据传输了。如果看过`BluetoothSocket`的源码，就可以发现这个类是非常简单的，里面涉及到数据流的部分主要就是`getInputStream()`和`getOutpotStream`。这两个数据流将承担蓝牙通信数据传输的任务，对`InputStream`调用`read()`方法来读取数据，对`OutputStream`调用`write()`方法来写入数据。当然，这些操作不能在主线程上进行。Google官方的建议是专门用一个子线程来同时承载数据读写操作，其中用一个循环来专门负责持续地从`InputStream`中读取数据，而写入数据则可以提供另外的方法来执行。具体可以参考下面的示例代码：

```
// 在协程中发送数据
coroutineScope.launch(Dispatchers.IO) {
    try {
        bluetoothSocket?.outputStream?.write(byteArray)
    } catch (e: Exception) {
        // TODO: 注意调用write方法时必须使用try-catch捕获随时可能出现的异常
    }
}

// 在协程中监听数据传入
coroutineScope.launch(Dispatchers.IO) {
    val buffer = ByteArray(1024)
    while (true) {
        try {
            bluetoothSocket?.inputStream?.apply {
                if (available() != 0) {
                    read(buffer)
                }
            }
        } catch (e: Exception) {
            // TODO: 注意调用read方法时也要使用try-catch来捕获随时可能出现的异常
        }
    }
}
```

## 低功耗蓝牙

低功耗蓝牙（Bluetooth Low Energy）是蓝牙4.0版本起开始支持的一项蓝牙技术规范。由于经典蓝牙被Apple限制数据传输接口（需要经过MFI认证），而且功耗偏高，目前在移动互联网应用中已被逐渐边缘化。需要注意的是，蓝牙4.0版本支持单双模，单模就是通常意义上的BLE，主打低功耗、快连接和长距离的优势；而双模则是既支持BLE又能兼容经典蓝牙，后者的主要优势在于大数据和高速率。在对功耗要求不是极为严苛的情形下，使用双模蓝牙是比较理想的选择。

Android为BLE提供了平台级的支持，并且还有相应的API供开发者使用，从而完成蓝牙的四大必需功能。

### 基本概念

在BLE中有以下重要概念需要被开发者所了解：

+ **profile**

profile表示BLE设备的配置文件。Bluetooth SIG（Bluetooth Special Interest Group）为BLE设备定义了许多profile，作为通用的标准。每份profile都是一个规范，用于指导设备在特定应用场合下如何工作。需要注意的是，BLE设备可以同时实现多个profile，比如一个健康手环同时包含有心率监测和电量检测的功能，而这两个功能遵循的是各自的规范。

+ **GATT（Generic Attribute Profile）**

GATT直接翻译过来就是“通用属性配置文件”，其规范作用主要针对BLE连接当中传输的简短数据片段（被称作“属性（attributes）”）。目前现有的全部BLE应用配置文件都是基于GATT，所以值得高度关注。

+ **ATT（Attribute Protocol）**

GATT的实现建立在ATT的基础上，类似于TCP/IP，两者的关系可以描述为GATT/ATT。ATT针对BLE设备做过优化，尽可能使用较少的字节数进行数据传输，以便能在这些设备上运行。每个属性都会使用UUID作为唯一标识，它们经过ATT传输时所采用的格式，就包括了下面要介绍到的characteristic和service这两种。

+ **characteristic**

characteristic直译过来就是特征。一个characteristic可以被理解成类型，其概念跟编程语言中的class类似。characteristic包含一个value和若干个descriptor，当然某些情形下可能只有value。

+ **descriptor**

descriptor直译过来就是描述符，其作用是定义一些用于描述characteristic value的属性。例如指定一些人类可读可理解的描述信息，描述一个value的可接受范围或是其度量单位等。

+ **service**

service代表一些characteristic的集合。比如一个心率检测的服务可能就包含有心率测量之类的characteristic。

在BLE网络拓扑中，有中央和外围的概念，也有server和client的概念。位于网络拓扑中央的设备负责扫描和监听广播，外围设备则发出广播。serve和client跟前文介绍的概念有所差异，主要是指GATT server和GATT client，server负责生产数据，而client负责消费数据。

在权限配置方面，BLE跟经典蓝牙并无二致，如果忘记要给BLE配置哪些权限，可以回过头再看一遍[权限配置](Android/bluetooth?id=权限配置)这一节内容。另外，由于Android设备通常只有一种蓝牙硬件，因此设置BLE跟[设置经典蓝牙](Android/bluetooth?id=设置蓝牙)的操作步骤也基本相同，这里不再重复。

### 查找BLE设备

在Android应用中调用查找BLE设备的API比较简单，如下面代码所示：

```
// 应用授权检查过程省略
···

// 启用蓝牙功能步骤省略
····

// 创建扫描结果回调
val scanCallback = object :ScanCallback() {
    override fun onScanResult(callbackType: Int, result: ScanResult?) {
        // ScanResult里包含有BluetoothDevice对象等重要属性
        result?.let {
            ···
        }
    }

    override fun onScanFailed(errorCode: Int) {
        ···
    }

    override fun onBatchScanResults(results: MutableList<ScanResult>?) {
        ···
    }
}

// 启动BLE设备扫描任务
bluetoothAdapter.bluetoothLeScanner?.startScan(scanCallback)

// 仅扫描特定类型的BLE设备，使用回调
bluetoothAdapter.bluetoothLeScanner?.startScan(
    arrayListOf<ScanFilter>(···),
    ScanSettings.Builder(). ··· .build(),
    scanCallback
)

// 扫描特定类型的BLE设备，使用PendingIntent，Android 8.0起支持
val callbackIntent: PendingIntent = ···
bluetoothAdapter.bluetoothLeScanner?.startScan(
    arrayListOf<ScanFilter>(···),
    ScanSettings.Builder(). ··· .build(),
    callbackIntent
)

// 在获得扫描结果或扫描达到一定时限后应当主动关闭扫描任务，避免无端消耗电量
bluetoothAdapter.bluetoothLeScanner?.stopScan(scanCallback)

// 如果使用PendingIntent，则传入之前创建的PedingIntent对象来关闭扫描任务，Android 8.0起支持
bluetoothAdapter.bluetoothLeScanner?.stopScan(callbackIntent)
```

> 注意，Google官方文档提示开发者，同一个应用在同一时刻只能扫描BLE设备或经典蓝牙设备，**不能一起扫描**。

针对过滤BLE设备类型的扫描方式，这里对`ScanFilter`和`ScanSettings`做一些简单介绍。两种数据类型的对象都是通过[Builder模式](DesignPattern/创建型设计模式?id=三、builder)创建出来的，并没有什么构造方法可以直接调用，具体可以参考下面的示例代码：

```
// Scanfilter的构建方式
ScanFilter.Builder()
    .setAdvertisingDataType(···)            // 设置广播数据类型过滤条件，传入的值来源于ScanRecord.DATA_TYPE_XXX，API 33以上才能使用
    .setAdvertisingDataTypeWithData(···)    // 作用同上，但还要传入原始数据和原始数据的掩码，两者数组长度必须一致，否则无法正确匹配
    .setDeviceAddress(···)                  // 设置BLE设备MAC地址的过滤条件
    .setDeviceName(···)                     // 设置BLE设备名称的过滤条件
    .setManufacturerData(···)               // 设置BLE设备生产厂商的过滤条件
    .setServiceUuid(···)                    // 设置服务UUID的过滤条件，可以是全部匹配，也可以是部分匹配
    .setServiceSolicitationUuid(···)        // 设置服务请求UUID的过滤条件，可以是全部匹配，也可以是部分匹配
    .setServiceData(···)                    // 设置服务数据的过滤条件
    .build()

// ScanSettings的构造方式
ScanSettings.Builder()
    .setCallbackType(···)                   // 设置BLE设备扫描的回调类型，传入的值来源于ScanSettings.CALLBACK_TYPE_XXX
    .setScanMode(···)                       // 设置BLE设备扫描模式，传入的值来源于ScanSettings.SCAN_MODE_XXX
    .setLegacy(···)                         // 设置扫描结果是否仅返回传统广播，设为false才能调用setPhy(···)
    .setMatchMode(···)                      // 设置BLE设备扫描匹配模式，传入的值来源于ScanSettings.MATCH_MODE_XXX
    .setNumOfMatches(···)                   // 设置BLE设备扫描匹配数量，传入的值来源于ScanSettings.MATCH_NUM_XXX
    .setPhy(···)                            // 设置在本次扫描期间所使用的用物理层，传入的值来源于BluetoothDevice.PHY_LE_XXX，
                                               或是ScanSettings.PHY_LE_ALL_SUPPORTED
    .setReportDelay(···)                    // 设置获得扫描结果后延迟上报的时长，默认5000ms，设为0则表示立即上报
    .build()
```

### 连接到GATT server

让应用连接到GATT server的方式实际上非常简单：

```
// 应用授权检查过程省略
···

// 启用蓝牙功能步骤省略
····

// 获取BluetoothDevice对象过程省略
···

// 创建一个BluetoothGattCallback的回调实例，并在其中编写相应的GATT client操作
val bluetoothGattCallback = object :BluetoothGattCallback() {
    override fun onConnectionStateChange(gatt: BluetoothGatt?, status: Int, newState: Int) {
        ···
    }

    override fun onServicesDiscovered(gatt: BluetoothGatt?, status: Int) {
        ···
    }

    // 要重写的方法较多，此处省略
    ···
}

// 从BluetoothDevice中获取BluetoothGatt实例，并基于该实例进行GATT client的操作
var bluetoothGatt: BluetoothGatt? = bluetoothDevice?.connectGatt(context, isAutoConnect, bluetoothGattCallback, ···)

// 不再使用BLE设备时，应当主动关闭BluetoothGatt，以便系统可以释放资源
bluetoothGatt?.close()
bluetoothGatt = null
```

#### BluetoothGattCallback的使用

这里要重点介绍一下`BluetoothGattCallback`这个回调，因为它包含的回调方法实在太多了，而且许多重要的操作（比如BLE属性的读写）都要在这些回调方法中执行，所以有必要专门留出一定的篇幅给它。下表是`BluetoothGattCallback`所持有的一些重要的回调方法：

|回调方法|用途说明|
|:-----:|:-----:|
|`onCharacteristicChanged`|当characteristic被修改，且调用过`BluetoothGatt.setCharacteristicNotification()`，允许接受相关通知时会被触发|
|`onCharacteristicRead`|当调用`BluetoothGatt.readCharacteristic()`需要返回结果时会被触发|
|`onCharacteristicWrite`|当调用`BluetoothGatt.writeCharacteristic()`需要指示结果时会被触发|
|`onConnectionStateChange`|当GATT client与GATT server间的连接状态发生改变时会被触发|
|`onDescriptorRead`|当调用`BluetoothGatt.readDescriptor()`需要返回结果时会被触发|
|`onDescriptorWrite`|当调用`BluetoothGatt.writeDescriptor()`需要指示结果时会被触发|
|`onMtuChanged`|当调用`BluetoothGatt.requestMtu()`，请求对MTU（Maximum Transmission Unit）长度进行更改时会被触发|
|`onPhyRead`|当调用`BluetoothGatt.readPhy()`需要返回结果时会被触发|
|`onPhyUpdate`|当调用`BluetoothGatt.setPreferredPhy()`，或是远程设备更改了物理层信息于是返回结果时会被触发|
|`onReadRemoteRssi`|当调用`BluetoothGatt.readRssi()`需要返回结果时会被触发|
|`onReliableWriteCompleted`|当调用`BluetoothGatt.executeReliableWrite()`并完成之后会被触发|
|`onServiceChanged`|当远程设备的服务发生变化，而GATT数据库尚未进行更新时就会被触发，表明接收到了服务发生变化的事件|
|`onServicesDiscovered`|当远程设备的service、characteristic以及descriptor的列表发生变化（比如新的service被发现）时会被触发|

> 注意，上表列出的回调方法仅用于说明其触发时机和用途，因为实际上有许多同名方法已经确定会在API 33当中被废弃掉，这里为节约篇幅没有全部列出。更多详细说明，可以访问[https://developer.android.google.cn/reference/android/bluetooth/BluetoothGattCallback](https://developer.android.google.cn/reference/android/bluetooth/BluetoothGattCallback)。

### BLE数据传输

在执行读写操作之前，有必要先获得BLE设备上的服务列表：

```
val services: List<BluetoothGattService?> = bluetoothGatt?.services ?: emptyList()
```

#### 读取BLE属性

首先来看如何从GATT服务中获取想要的信息：

```
services.forEach { bluetoothGattService ->
    // 获取BluetoothGattService的类型，有SERVICE_TYPE_PRIMARY和SERVICE_TYPE_SECONDARY两种
    val type: Int = bluetoothGattService.type

    // 获取当前服务的UUID
    val serviceUUID: UUID = bluetoothGattService.uuid

    // 获取当前服务所包含的属性列表
    val characteristics: List<BluetoothGattCharacteristic?>? = bluetoothGattService.characteristics

    // 获取当前服务所包含的其他服务列表
    val includedServices: List<BluetoothGattService?>? = bluetoothGattService.includedServices

    // 根据UUID获取特定属性
    val someCharacteristic: BluetoothGattCharacteristic? = bluetoothGattService.getCharacteristic(UUID)

    // 为当前服务增加属性
    bluetoothGattService.addCharacteristic(···)

    // 为当前服务增加服务
    bluetoothGattService.addService()(···)
}
```

接着来看如何读取特定信息：

```
bluetoothGatt?.let {
    // 注意readCharacteristic()被调用之后，其结果会异步传递到BluetoothGattCallback的相关回调方法
    it.readCharacteristic(characteristic)

    // 跟readCharacteristic()一样，结果也是异步传递到相关回调方法中
    it.readDescriptor(descriptor)

    // RSSI指Received Signal Strength Indication，即接收信号强度指示，主要用在基于蓝牙4.X协议的定位服务中
    it.readRssi()

    // 读取设备物理层相关信息
    it.readPhy()
}
```

#### 写入BLE属性

```
bluetoothGatt?.let {
    // 启动可靠写入，表示信息写入过程是原子操作
    it.beginReliableWrite()

    // 将信息写入特定的BluetoothGattCharacteristic对象当中，有相应的异步回调
    it.writeCharacteristic(···)

    // 将信息写入特定的BluetoothGattDescriptor对象当中，同样有相应的异步回调
    it.writeDescriptor(···)

    // 放弃可靠写入，当开发者通过验证发现写入的信息不是预期的内容时可以调用
    it.abortReliableWrite()

    // 执行可靠写入，当开发者通过验证发现写入的信息符合预期时可以调用
    it.executeReliableWrite()
}
```

#### 接收GATT通知

BLE应用通常会要求在设备上的特定特征发生变化时收到通知，而这也可以通过下面示例代码中调用`BluetoothGatt.setCharacteristicNotification()`来实现：

```
// 调用setCharacteristicNotification()后，结果同样会异步传到相应的回调方法中
bluetoothGatt?.setCharacteristicNotification(characteristic, enabled)
```

### BLE音频

前面的内容曾提到，BLE为了降低功耗，通常只会传输规模较小的数据，像音频这样比较大型的数据流一般都会切换到经典蓝牙模式当中去传输。但是从Android 13开始，Android平台会直接支持BLE的音频功能——当然设备本身也要支持BLE音频才行。

根据Google官方的说法，BLE音频的使用场景大致可以分为四种：

1. 具有通话功能或支持VoIP的应用，且要求通话延迟低、电量消耗低以及音频质量高；
2. 游戏应用，要求低延迟语音传输和高保真音频播放；
3. 多媒体播放应用，要求能让用户在系统设置中更改偏好使用的设备；
4. 无障碍服务，主要是能让佩戴助听器的用户通过BLE音频来使用麦克风并拨打电话。

Google官方在文档中还给出了一些支持BLE音频功能开发的API，但是介绍得不多，这里也不做赘述。更多详细资料，可以访问[https://developer.android.google.cn/guide/topics/connectivity/ble-audio/overview](https://developer.android.google.cn/guide/topics/connectivity/ble-audio/overview)。