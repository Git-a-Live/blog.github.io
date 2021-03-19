## 一、概述

**Groovy是Java的超集**。这意味着Java代码在Groovy中是完全兼容的，而且可以实现混编。Groovy和Java乃至Kotlin一样，都是先被编译成`.class`文件，而后用Java虚拟机运行，因此和Java具有类似的跨平台特性。Groovy在兼容Java的同时也有自己的语法，而且比Java原生的还要简洁一些（<u>比如一些函数不需要写一长串的引用，直接调用即可</u>），并且和Kotlin一样都不需要使用分号。

目前大量使用Groovy的地方主要是Gradle脚本。Gradle脚本的扩展名为`.gradle`，在Android开发中常用于依赖导入的控制以及其他配置。

## 二、基本语法

### 变/常量声明

在Groovy中，变/常量的声明方式为

```
变量:
def variable = value

常量:
def final variable = value
```

声明变量或常量可以不声明类型，而是根据赋值类型自行推导。常量在声明的同时要进行赋值以实施初始化，变量则可以选择等用到的时候再赋值。除此之外，由于不声明类型，**变量可以被任意类型的value赋值，并随之改变自己的类型**。换言之，一个Groovy变量随时都可以从一个类型转换成另一种类型，<u>除非它在类型上做了显式声明</u>：

```
//允许的操作:
def variable = 100
varible = "Hello"
variable = true
variable = new JavaClass()

//无法实现的操作:
int variable = 100
variable = "Hello"
```
### 基本输入输出

从<u><font color=red>IDE终端</font></u>接收用户输入的基本方式为调用`System.in.newReader().readLine()`方法，如果是在IDE以外的控制台（如PowerShell、CMD或者Terminal等）接收，则使用`System.console().readLine`方法。打印输出内容可以直接使用`print`或`println`方法。

### 基本数据类型

由于Groovy是Java的超集，因此Java的基本数据类型（int、double、String等）乃至引用类型（StringBuilder等）都能够在Groovy当中使用，需要注意的是，Groovy的List和Map两种类型在某些方面跟Java之间存在差异。

Groovy的List声明方式为：

```
//空List:
def variable = []

//元素类型相同的List:
def variable = [0,1,2]

//元素类型不同的List:
def variable = ["A",0,true]
```

除此之外，List类型变量的用法与Java基本没有差别。

Groovy的Map声明方式为：

```
//空Map:
def variable = [:]

//被初始化的Map:
def variable = [key1:value1, key2:value2, ···]

//从Map中取值, 等效于Java中的map.get(key):
println variable.key
```

<u><font color=red>注意，虽然key可以是任何类型，但是强烈不建议使用数字类型变量或是纯数字的字符作为key</font></u>。因为在使用点运算符获取value时，数字类型的key是不会显示的，也就无法调用；而用纯数字的字符作为key，在调用时会被IDE提示错误。尽管这两个问题可以通过调用`get()`解决，但出于良好可读性的考虑，最好不要用数字或数字类型的变量作为key。

除此之外，Map类型变量的用法与Java基本没有差别。

### 基本操作符

除了具备和其他高级语言一样的最基本的赋值操作、算术操作、逻辑运算操作和关系运算操作之外，Groovy还有**位运算**操作。位运算操作符有以下几种：

1. **与运算（&）**

2. **或运算（｜）**

3. **异或运算（^）**

4. **按位取反运算（～）**

### 流程控制

#### 循环

Groovy的循环语句有以下几种：

* **upto**

```
initValue.upto(endValue) {
    //TODO
}
```

表示语句循环执行`endValue - initValue + 1`次，注意`initValue`不可大于`endValue`。

* **times**

```
totalValue.times {
    //TODO
}
```

表示语句循环执行`totalValue`次。

* **step**

```
initValue.step(endValue, stepLength) {
    //TODO
}
```

表示语句从`initValue`开始，以步长`stepLength`迭代至`endValue`。

#### 分支

Groovy的条件判断语句和Java一样，直接按照Java的编程习惯使用即可。

### 闭包与方法

闭包（Closure）是Groovy中一种特殊的对象，其本质上是若干语句所组成的用于执行某一功能的代码块，跟其他高级语言中的函数和方法类似。所不同的是，**闭包还允许将闭包作为参数传入其中参与执行**。

闭包的声明类似于函数式编程，其具体方式如下：

```
//完整声明:
def closure = {
    param1,param2,··· ->
    //TODO
    return value
}

//无传入参数和返回值:
def closure = {
    //TODO
}
```

Groovy的方法声明如下：

```
//完整声明:
def method(param1,param2 = defaultValue,···) {
    //TODO
    return value
}

//无传入参数和返回值:
def method() {
    //TODO
}
```

无论是闭包还是方法，它们的调用方式跟普通的函数没有区别：

```
def variable = closure(param1,param2,···)
variable = method(param1,param2,···)
```