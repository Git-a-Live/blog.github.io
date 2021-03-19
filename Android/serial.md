序列化是指将数据结构或对象转换成二进制或十六进制等字节序列的过程，反序列化顾名思义，就是序列化的逆操作。 在某种程度上，序列化和反序列化类似于编码和解码过程。序列化和反序列化通常用于数据的存储和传输。 在Android开发中，常用的序列化方式有：Serializable、Parcelable、JSON以及XML。

Serializable是Java原生的序列化方法，其使用方式非常简单，就是先让对象直接实现Serializable接口：

```
//Java代码：
public class Demo implements Serializable{
    private static final long serialVersionUID = xxxxxxxxxxxxxxxxxxL;
    ···
}

//Kotlin代码：
class Demo : Serializable{
    companion object {
        private const val serialVersionUID = xxxxxxxxxxxxxxxxxxL
    }
    ···
}
```

Serializable接口没有定义任何方法，它是一个空接口。这样的接口被称为"标记接口"（Marker Interface）， 实现了标记接口的类仅仅是给自身贴了个"标记"，并没有增加任何方法。

注意，使用Serializable时，应当在对象中设置一个Long类型的serialVersionUID，而不是让系统自己生成， 否则当应用中的对象结构发生改变，再次编译运行时会抛出如下异常：

```
java.io.InvalidClassException: com.demo.demo.Demo;
Incompatible class (SUID): com.demo.demo.Demo: static final long serialVersionUID =xxxxxxxxxxxxxxxxxxL;
but expected com.demo.demo.Demo: static final long serialVersionUID =yyyyyyyyyyyyyyyyyL;
```

在让对象实现Serializable接口之后，就要利用ObjectOutputStream把对象写入字节流中：

```
//Java代码：
try {
    ···
    Demo demo = new Demo();
    ObjectOutputStream objectOutputStream = new ObjectOutputStream(new openFileOutput(FILE_NAME,MODE));
    //设置写入的文件名和写入模式
    objectOutputStream.writeObject(demo);
    objectOutputStream.flush(); //将缓冲区内的字节流全部输出到文件，不用等缓冲区满
    objectOutputStream.close(); //关闭字节流输出，释放资源
     ···
}catch (IOException e){
    //TODO
}

//Kotlin代码：
try {
    ···
    val demo = Demo()
    val objectOutputStream = ObjectOutputStream(openFileOutput(FILE_NAME,MODE))
    objectOutputStream.writeObject(demo)
    objectOutputStream.flush()
    objectOutputStream.close()
    ···
}catch (e: IOException){
     //TODO
}
```

如果要从文件中读取数据，即反序列化，使用ObjectInputStream把字节流写入内存：

```
//Java代码：
try {
    ···
    ObjectInputStream objectInputStream = new ObjectInputStream(new openFileInput(FILE_NAME));
    Demo demo = (Demo) objectInputStream.readObject(); //强制转换类型，便于后续输出
    ···
}catch (IOException e){
    //TODO
}

//Kotlin代码：
try {
     ···
    val objectInputStream = ObjectInputStream(openFileInput(FILE_NAME))
    val demo = objectInputStream.readObject() as Demo
    ···
}catch (e: IOException){
    //TODO
}
```

反序列化时，Java对象是由JVM直接构造出来的，不调用构造方法，因此构造方法内部的代码，在反序列化时根本不可能执行。

此外还要注意，Java的序列化机制可以导致一个实例能直接从byte[]数组创建，而不必经过构造方法，因此存在一定的安全隐患。 一个精心构造的byte[]数组被反序列化后可以执行特定的Java代码，从而导致严重的安全漏洞。 

实际上，Java本身提供的基于对象的序列化和反序列化机制既存在安全性问题，也存在兼容性问题（仅适用于Java）。 更好的序列化方法是通过JSON这样的通用数据结构来实现，只输出基本类型（包括String）的内容，而不存储任何与代码相关的信息。