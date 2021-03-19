## 文件I/O

### 读取文件

读取文件的关键调用方法为`File('dir','filename')`，其典型的一种使用方式如下：

```
new File('dir','filename').eachLine { line ->
    println line
}
```

通过调用new File( )来获得一个File实例对象，然后使用**函数式编程**的方式来输出文件的内容。File对象有许多可以调用的方法（比如创建目录的mkdir()，删除文件的delete()等），此处不作赘述。

### 写入文件

写入文件的关键调用方法为`File('dir','filename')`，其典型的一种使用方式如下：

```
new File('dir','filename').withWriter('utf-8') { writer ->
    writer.writeLine 'Example Content'
}
```

如果是比较简单的输出内容，还可以使用下面这种方法：

```
new File('dir','filename') << '''This is an example.
Hello Groovy!
Hello World!'''
```

上述方法还可以用来将一个文件的内容复制到另一个文件：

```
new File('dir1','filename1') << new File('dir2','filename2')
```

### 遍历文件树

遍历文件树主要有以下五种方式：

```
//遍历全部文件和目录:
def dir = 'directory'
dir.eachFile { file ->
    println file.name
}

//遍历特定文件:
def dir = 'directory'
dir.eachFileMatch(pattern) { file ->     
    println file.name
}

//递归遍历文件和目录:
def dir = 'directory'
dir.eachFileRecurse { file ->     
    println file.name
}

//递归遍历文件:
def dir = 'directory'
dir.eachFileRecurse(FileType.FILES) { file ->      
    println file.name
}

//更复杂的遍历:
def dir = 'directory'
dir.traverse { file ->
    //TODO
}
```

## 执行外部命令

按照Groovy官方文档的解释：

> Groovy provides a simple way to execute command line processes. Simply write the command line as a string and call the execute() method. E.g., on a *nix machine (or a windows machine with appropriate *nix commands installed).
>
> *Groovy提供了一种可以执行命令行指令的简单方法，只需在类Unix系统或是类Unix命令行工具上，将命令行指令编写成字符串并调用excute()方法即可。*

事实上，这里的外部命令主要是指**各个操作系统自带的命令行指令**，只不过Linux、macOS等类Unix系统的指令有许多相同的地方，故而文档中强调"类Unix系统或类Unix命令行工具"。如果使用的是Windows系统，那么就应该把Windows上的命令行指令代入执行。

官方提供了一个例子：

```
def process = "ls -l".execute()
println "Found text ${process.text}" //打印执行后的结果
```

更多关于Groovy执行命令行指令的内容，详见[官方文档](http://docs.groovy-lang.org/latest/html/documentation/working-with-io.html)。

## 范围

和Kotlin、Swift类似，Groovy也有"区间"的概念，这个概念在Groovy中被称作"范围"。
范围有以下几种定义类型：

```
//闭区间:
def range = 1..10 //[1,10]升序排列
range = 10..1 //[10,1]降序排列

//左闭右开区间:
def range = 1..<10 //[1,10)

//字符区间:
def range = 'a'..'z' //[a,z]升序排列
range = 'z'..'a' //[z,a]降序排列
```

<font color=red>注意：字符区间内各个字符的排列是按照它们在<u>字符集</u>（比如UTF8）里的顺序排序的，这意味着区间里只能使用字符集里存在的字符</font>。

## 正则表达式

Groovy中创建模式匹配对象的方式有两种：

```
//方式1:
def regex = ~'matchPattern'

//方式2:
def regex =~ 'matchPattern'
```

其中`matchPattern`就是通用的[正则表达式](https://www.wikiwand.com/en/Regular_expression)。

要比较某个字符串是否匹配某个模式，使用以下方法：

```
str ==~ 'matchPattern'
```

## 异常处理

Groovy处理异常的方式和Java一样，都是使用`try`包裹可能会抛出异常的代码块，并在`catch`部分对异常进行处理。

## 特征

按照Groovy官方文档的解释：

> Traits are a structural construct of the language which allow −
>   · Composition of behaviors.
>   · Runtime implementation of interfaces.
>   · Compatibility with static type checking/compilation
> They can be seen as interfaces carrying both default implementations and state. 
>
> *特征是允许下列行为的语言的一种结构性构造：*
>   · *行为的组合；（对应Java的多重继承）*
>   · *接口的运行时实现；*
>   · *与静态检查或编译兼容*
> *它们可以被当成是包含了默认实现和状态的**接口**。*

Groovy中声明特征的方式为：

```
trait MyTrait {
    //TODO
}
```

特征更类似于Java中的可继承类，即使它是通过`implements`关键字被其他类所继承。因为特征内部可以定义完整的方法和属性，继承特征的子类也不需要强制覆写方法（Java中继承接口的子类即便不会调用某个抽象方法，也需要将该方法的空覆写形式保留）既可以不覆写方法而直接调用其默认实现，也可以选择将默认方法覆写成自己需要的形式。其方式类似于下面这种：

```
class Demo implements MyTrait {
    //TODO
}
```

正如上面所述的那样，特征类似于一个普通的可继承类，因此也可以使用`implements`来继承某个接口，并实现接口所定义的抽象方法。从这个角度来说，可以认为特征是实现了*Bridge*的一种对象。

## 注释与注解

Groovy的注释和注解方式跟Java一样。