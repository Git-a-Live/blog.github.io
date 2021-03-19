Parcelable通常用于跨进程传输数据，它提供了这样一套机制：将序列化之后的数据写入到一个共享内存中， 其他进程通过Parcel可以从这块共享内存中读出字节流，并反序列化成对象。实现Parcelable的作用如下：

+ 永久性保存对象，保存对象的字节序列到本地文件中；
+ 通过序列化在网络中或进程间传递对象。

Parcelable需要通过实现Parcelable接口才能调用，其具体实现方式为：

```
//Java代码：
public class Demo implements Parcelable {
    protected Demo(Parcel parcel) {
         ···
    }

    public static final Creator<Demo> CREATOR = new Creator<Demo>() {
        @Override
        public Book createFromParcel(Parcel parcel) {
            return new Demo(parcel);
        }

        @Override
        public Demo[] newArray(int size) {
            return new Demo[size];
        }
    };

    @Override
    public int describeContents() {
        //TODO：return 0; or return 1;
    }

    @Override
    public void writeToParcel(Parcel parcel, int flags) {
        ···
    }
    ···
}

//Kotlin代码：
class Demo : Parcelable {
    //在实现Parcelable接口之后，系统会自行构造下列方法
    constructor(parcel: Parcel) : this(
        parcel.readXxx(),
        parcel.readXxx()
        ···
        //读取对象字节流，供createFromParcel()调用
    )
    ···
    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeXxx(···)
        parcel.writeXxx(···)
        ···
        //将数据写入parcel，实现序列化输出
    }

    override fun describeContents(): Int {
        //TODO：return 0 or return 1，只针对一些特殊的需要描述信息的对象，需要返回1，其他情况返回0即可
    }

    companion object CREATOR : Parcelable.Creator<Demo> {
        override fun createFromParcel(parcel: Parcel): Demo {
            return Demo(parcel)
            //通过构造函数完成读取工作
        }

        override fun newArray(size: Int): Array<Demo?> {
            return arrayOfNulls(size)
            //供该类的数组反序列化时调用
        }
    }
}
```

进程间传递对象的方式如下，先来看原进程的核心代码：

```
//Java代码：
Demo demo = new Demo();
Intent intent = new Intent(THIS_PROCESS.this, ANOTHER_PROCESS.class);
Bundle bundle = new Bundle(); //推荐使用Bundle在进程间传递数据
bundle.putParcelable(PARCEL_NAME, demo); //打包Parcel
intent.putExtra(BUNDLE_NAME,bundle); //构建传递Bundle的intent
startActivity(intent);

//Kotlin代码：
val demo = Demo()
val intent = Intent(applicationContext, ANOTHER_PROCESS::class.java)
val bundle = Bundle() 
bundle.putParcelable(KEY_NAME,demo) 
intent.putExtra(KEY_NAME,bundle)
startActivity(intent)
```

目的进程要进行反序列化，其核心代码为：

```
//Java代码：
Intent intent = getIntent()
Bundle bundle = intent.getBundleExtra(BUNDLE_NAME) 
Demo demo = bundle.getParcelable<Demo>(PARCEL_NAME) //从Parcel中提取对象

//Kotlin代码：
val bundle = intent.getBundleExtra(BUNDLE_NAME) 
val demo = bundle.getParcelable< Demo>(PARCEL_NAME)
```