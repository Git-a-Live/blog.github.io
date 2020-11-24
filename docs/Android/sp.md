SharedPreferences可实现简单的数据持久化保存（.xml格式文件），前提是用户没有特意删除数据，以及硬件设备没有被破坏。 

目前，它仅支持boolean、float、int、long和string等基本类型的存储，其他自定义的复合类型无法存储。 SharedPreferences主要用于存储一些配置信息，类似于Windows下常用的.ini文件。 

它将数据保存在自己创建的文件当中， 通过保存上一次用户所做的修改或者自定义参数设定，使得程序再次启动后能够依然保持原有设置。 通过使用键值对的方式进行存储，SharedPreferences可以方便地管理写入和读取。

SharedPreferences对象的获取方式有两种：

```
//Activity直接获取：
val shp = getSharedPreferences(keyName, MODE_PRIVATE)

//非Activity需要传入Context对象：
val shp = context.getSharedPreferences(keyName, MODE_PRIVATE)
```
如果只是读取数据，SharedPreferences对象只需要调用get类方法就够了：

```
val string = shp.getString(keyName, defValue)
val int = shp.getInt(keyName, defValue)
···
```

但是对数据进行写入以及持久化保存，还必须再创建一个SharedPreferences的内部类对象——Editor：

```
val editor = shp.edit()
editor.putInt(keyName, value)
editor.putString(keyName, value)
editor.apply()
//editor.commit
```

注意，在上面的过程中，实施写入操作之后，必须调用apply()或commit方法，否则内容不会写入。

apply()方法和commit()方法的区别在于：前者没有返回值，而且将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘；后者会返回布尔值以供判断写入是否成功，并且同步地提交到硬件磁盘（在多线程中并发使用commit()方法会发生阻塞降低执行效率）。