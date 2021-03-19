WorkManager是Google推出的用于在**后台处理定时执行任务**的一种组件，它可以根据操作系统的版本自动选择底层是使用AlarmManager实现还是JobScheduler实现。

WorkManager与[Service](Android/service)有很大不同，它只是一个处理定时任务的工具，可确保应用退出甚至手机重启的情况下，之前注册的任务仍可得到执行。此外，WorkManager注册的周期性任务<font color=red>不一定会准时执行</font>，因为系统可能会将几个触发时间临近的几个任务放在一起执行，从而减少CPU唤醒次数和资源消耗。

## 基本用法

在使用WorkManager之前，需要在build.gradle文件中添加如下依赖：

```
dependencies {
    //Java syntax
    implementation "androidx.work:work-runtime:$work_version"

    //Kotlin + coroutines
    implementation "androidx.work:work-runtime-ktx:$work_version"
}
```

WorkManager的基本用法通常有三步：

1. **定义一个后台任务，并实现具体的业务逻辑**；
   ```
    class DemoWorker(appContext: Context, workerParams: WorkerParameters):
       Worker(appContext, workerParams) {
        override fun doWork(): Result {
            //在此处编写具体的后台任务逻辑
            return Result.success() //失败则返回Result.failure()或Result.retry()
        }
    }
   ```
2. **配置该后台任务的运行条件和约束信息，并构建后台任务请求**；
   ```
    //单次任务请求：
    val request = OneTimeWorkRequest.Builder(DemoWorker::class.java)
    ··· //链式配置
    .build()

    //周期任务请求：
    val request = PeriodicWorkRequest.Builder(DemoWorker::class.java, time, TimeUnit.XXX)
    ··· //链式配置
    .build()
    //周期性任务的时间间隔不能小于15分钟，以降低设备性能消耗
   ```
3. **将该后台任务请求请求传入WorkManager的`enqueue()`方法中，系统会在适合的时间运行**。
   ```
    WorkManager.getInstance(context).enqueue(request)
   ```

## 配置WorkRequest

WorkRequest的构建实际上可以非常复杂，主要就是通过[Builder](DesignPattern/创建型设计模式?id=三、builder)链式调用若干设置方法对WorkRequest的运行条件和约束信息进行配置。常用的设置方法有：

+ setInitialDelay()：设置后台任务在指定时间后开始运行
+ addTag()：设置任务标签，同一标签可以管理多个后台任务
+ setBackoffCriteria()：如果后台任务返回Result.retry()，则在一段时间后重新执行任务
+ setConstraints()：设置后台任务的运行条件

运行条件的编写方法和WorkRequest类似：

```
val constraint = Constraints.Builder()
    ··· //链式配置
    .build()
```

常见设置方法：

+ setRequiredNetworkType()：设置后台任务在某种网络类型下运行
+ setRequiresCharging()：设置后台任务在某个充电状态下运行
+ setTriggerContentMaxDelay()：设置触发后台任务的最大延迟时间
+ setTriggerContentUpdateDelay()：设置触发内容更新的延迟时间

## 链式任务

WorkManager的链式任务机制很简单，就是调用一个`beginWith()`和若干`then()`来有序执行一系列后台任务：

```
WorkManager.getInstance(context)
.beginWith(task1)
.then(task2)
.then(···)
···
.enqueue()
```

>注意，使用WorkManager的链式任务时，只有前一个后台任务执行成功之后，下一个后台任务才会执行。

<font color=red>在一些定制过的Android系统上，应用随时可能会被杀掉，因此在这种情况下，不能依赖WorkManager去实现核心功能。</font>