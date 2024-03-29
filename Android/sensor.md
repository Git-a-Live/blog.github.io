## 传感器概览

大多数Android设备都有内置传感器，用来测量运动、屏幕方向和各种环境条件。这些传感器能够提供高度精确的原始数据，非常适合用来监测设备的三维移动或定位，或监测设备周围环境的变化。

### 基本概念

Android 平台支持三大类传感器：

+ **运动传感器**

    这个类别中包含加速度计、重力传感器、陀螺仪和旋转矢量传感器，可用于测量三个轴向上的加速力和旋转力。

+ **环境传感器**
  
    这个类别中包含气压计、光度计和温度计，可用于测量各种环境参数，如环境气温、气压、照度和湿度。

+ **位置传感器**

    这个类别中包含屏幕方向传感器和磁力计，可用于测量设备的物理位置。

Android设备传感器所定义的坐标系如下图所示，注意这跟进行UI开发时所用到的坐标系完全不一样：

![](pics/sensor1.png)

### 传感器框架

利用Android传感器框架，开发者可以访问设备上提供的传感器并从中获取原始数据。传感器框架提供有多个类和接口，可用于执行各种与传感器相关的任务，例如：

+ 确定设备上有哪些传感器；
+ 确定单个传感器的特性，如最大量程、制造商、功率要求和分辨率等；
+ 获取原始传感器数据，并设置获取传感器数据的最低频率；
+ 注册和注销用于监控传感器变化的传感器事件监听器。

传感器框架位于`android.hardware`，包含了以下类和接口：

|类/接口|用途说明|
|:-----:|:-----:|
|`SensorManager`|可用于创建传感器服务的实例，从而执行访问传感器、注册和注销监听、获取屏幕方向信息、报告传感器精度、设置数据采集频率以及校准传感器等操作|
|`Sensor`|可用于创建特定类型传感器的实例，从而获取传感器的特性|
|`SensorEvent`|可用于创建传感器事件对象，该对象能提供原始传感器数据、传感器类型、数据精度以及事件时间戳等信息|
|`SensorEventListener`|可用于接收传感器数值或精度发生变化时发出的事件通知，并通过回调方法将事件传递给其他业务逻辑使用|

### 最佳实践原则

+ **仅在前台采集传感器数据**

    在Android 9+的设备上，应用运行于后台的会导致部分类型的传感器无法接收到事件。因此应当在应用位于前台或使用前台服务时检测传感器事件。

+ **不用传感器时应注销监听器**

    在不使用传感器或传感器活动暂停时，应及时注销传感器的监听器。注册传感器监听器后，只要不注销，那么即使监听器的活动已暂停，传感器仍会继续采集数据并消耗电池资源。

+ **不要阻塞` onSensorChanged()`方法**

    传感器数据能以很高的频率变化，这意味着系统可能会频繁调用 `onSensorChanged()`方法。因此在`onSensorChanged()`方法中应尽可能执行较少的任务，以免造成阻塞。如果应用的某些业务逻辑需要过滤或删减传感器数据，则应在`onSensorChanged()`方法之外执行该任务。

+ **避免使用已废弃的方法或传感器类型**

    有一些方法和常量已被弃用，在开发时应当避免用到它们。比如`TYPE_ORIENTATION`传感器类型已被弃用，应改用`getOrientation()`方法来获取屏幕方向数据；再比如，`TYPE_TEMPERATURE`传感器类型也已被弃用，在搭载`Android 4.0`的设备上应当改用`TYPE_AMBIENT_TEMPERATURE`类型的传感器。

+ **使用传感器之前应先进行验证**

    在尝试从传感器采集数据之前，应始终先验证设备上是否存在该传感器。不要因为某种传感器很常用就假设它存在，因为没有人要求设备制造商必须在其设备中提供任何特定的传感器。

+ **谨慎选择传感器采集频率**

    使用`registerListener()`方法注册传感器时，务必选择合适的数据采集频率。因为传感器能以非常高的频率提供数据，所以如果选用过高的频率，有可能会采集到大量无用的传感器数据，导致白白浪费系统资源和电池电量。

## 运动传感器

### 重力传感器

### 线性加速度计

### 旋转矢量传感器

### 有效运动传感器

### 计步器传感器

### 步测器传感器

## 环境传感器

### 光照、压力及温度传感器

### 湿度传感器

## 位置传感器

### 游戏旋转矢量传感器

### 地磁旋转矢量传感器

### 地磁场传感器

### 近程传感器