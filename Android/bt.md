蓝牙是一种无线技术标准，可实现固定设备、移动设备和楼宇个人域网之间的短距离数据交换。 蓝牙技术具有十分广泛的应用，它具有耗电量低、成本低、安全性、稳定性、易用性等优点，在物联网设备上的占有率非常高。

使用蓝牙需要声明权限，通常使用的权限为下列两个：

```
<uses-permission android:name="android.permission.BLUETOOTH"/>

<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```

第一个权限用于在应用程序中请求或者建立蓝牙连接并传递数据，第二个权限则用于初始化设备发现功能或者对蓝牙设置进行更改。 通常情况下，仅仅是打开和关闭蓝牙，并不需要进行动态授权。而一旦涉及到更多危险权限时，如依靠蓝牙进行定位或收发数据等， 就必须进行动态授权，否则无法使用。

打开和关闭蓝牙有两种方式，第一种是通过Intent，另一种则是通过调用相关的方法。

通过Intent打开蓝牙的代码如下：

```
startActivity(Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE))
//如果有需要，可以换成startActivityForResult()
```

通过Intent启动蓝牙时，系统会自动弹出一个对话框，询问用户是否授权该应用启动蓝牙。注意，授权仅限于本次使用， 如果用户关闭了蓝牙，下一次再想通过该应用打开蓝牙，就要再进行一次授权，实质上跟动态授权没什么区别， 而且Intent也不能很方便地执行关闭蓝牙的动作。当应用程序的使用频率较低而且不需要通过该应用关闭蓝牙时，可以考虑使用这个方法。

第二种方法就是调用BluetoothAdapter对象的enable()和disable()方法。 利用这种途径不仅能让用户乃至应用自行决定何时启用或关闭蓝牙，还不用像Intent那样进行类似于动态授权的工作。 具体的代码如下：

```
val bluetoothAdapter = BluetoothAdapter.getDefaultAdapter()
bluetoothAdapter.enable() //启用蓝牙
bluetoothAdapter.disable() //关闭蓝牙
```

BluetoothAdapter是蓝牙开发过程中接触到的第一个类，通常还需要跟BluetoothDevice、BluetoothServerSocket以及BluetoothSocket等类配合使用。除了enable()和disable()之外，它还有以下常见方法：

>getDefaultAdapter() //获取本地蓝牙适配器

>setName(String name) //设置蓝牙名称

>isEnabled() //判断蓝牙是否打开

>name //获取本地蓝牙的名称

>address //获取本地蓝牙适配器的地址

>bondedDevices //获取已经绑定的蓝牙设备

>getRemoteDevice(byte[] address) //获取远程蓝牙设备

>getRemoteDevice(String address) //获取远程蓝牙设备

>startDiscovery() //开始搜索附近蓝牙

>cancelDiscovery() //停止搜索蓝牙

>listenUsingInsecureRfcommWithServiceRecord(String name, UUID uuid) //创建BluetoothServerSocket

蓝牙设备的搜索分为被动搜索和主动搜索。被动搜索顾名思义，就是本设备启动蓝牙之后，在一定时间内（通常是120秒）可以被其他设备发现； 主动搜索则是主动扫描探测附近是否存在蓝牙设备，如果有，就利用一个广播接收器来接收搜索到的设备。

被动搜索的代码很简单，具体如下：

```
if (bluetoothAdapter.scanMode != BluetoothAdapter.SCAN_MODE_CONNECTABLE_DISCOVERABLE){
    val intent = Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE)
    intent.putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION,120)
    startActivity(intent)
}
```

首先是判断本设备是否处于可被发现的状态，如果是不可被发现，那么就会通过Intent来设置当前设备可被发现。 当然，由于用到了Intent，因此会弹出系统对话框询问用户授权。如果用户没有打开蓝牙功能，那么它还可以在经过授权之后自行启动蓝牙。 然而被动搜索能做的也仅仅是这些了，如果希望应用程序能做得更多，就需要使用主动搜索。

在介绍主动搜索之前，先简单了解一下如何获取已经配对的设备。蓝牙（其一）中已经提到过，BluetoothAdapter对象的常用方法当中， 有一个.bondedDevices的方法可以获取已经配对的设备，当然，这个方法返回的是一个Set对象，其内部元素的类型为BluetoothDevice。 如果要将里面绑定的设备通过一个列表展示出来，那么可以采用如下方式：

```
if (bluetoothAdapter.bondedDevices.isEmpty()){
    //TODO
}else{
    for (bluetoothDevice in bluetoothAdapter.bondedDevices){
        //TODO
    }
}
```

bluetoothDevice是一个BluetoothDevice对象，可以通过`.name`、`.address`以及`.uuid`等来获取已配对设备的信息。