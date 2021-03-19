## 一、声明常量与变量

在Swift语言中，使用`let`来声明**常量**，使用`var`  来声明**变量**，这与Kotlin十分相似：

```
Swift:
let a = 0
var b = 1

Kotlin:
val a = 0
var b = 1
```

`注意`：Swift可以在**一行代码**中以逗号分隔的形式声明**多个**常量或变量，而Kotlin目前还无法做到：

```
Swift:
let a = 0, b = 1, c = 2
var a = 0, b = 1, c = 2

Kotlin:
val a = 0
val b = 1
val c = 2

var a = 0
var b = 1
var c = 2
```

## 二、声明类型

与Kotlin相似，Swift也是采用两种方式来使得常量或变量被声明为指定类型，一种是`直接声明`，另一种是`类型推导`，二者语法基本一样：

```
Swift:
let s: String  = "Hello"
var num = 0

Kotlin:
val s: String = "Hello"
var num = 0
```

## 三、常量与变量的命名

在Kotlin中，命名通常需要遵守与Java相似的规则；而Swift语言的常量与变量名不能包含数学符号，箭头，保留的（或者非法的）Unicode 码位，连线与制表符，也不能以数字开头，并尽量避免与关键字同名。换句话说，只要开发者愿意，英文字母以外的合法字符都是可以用于命名常量和变量的。但是出于可读性的考虑，最好还是使用有意义的英文字符串来命名。

## 四、简单输入输出

Kotlin使用`Scanner`和`next`函数来读取控制台的输入，Swift则使用`readLine`函数：

```
Swift:
let s = readLine()

Kotlin:
val scanner = Scanner(System.`in`)
var s = scanner.nextLine()
```

但是二者的输出都使用具有相同名称的print函数

```
Swift:
print("Hello World")

Kotlin:
print("Hello World")
```
同样地，无论是Kotlin还是Swift，它们的print函数都可以具有多种形式（通过重载来接收不同的输入）。

## 五、注释与分号

Kotlin和Swift都使用`//`作为单行注释符号，`/**/`则用于多行注释。二者通常都**不使用**分号来分隔每行代码，但由于Swift允许在同一行内写多个独立语句，因此在这种情形下需要使用分号。

## 六、基本类型

### 1.整数

Swift的整数类型包括：

· `Int`：有符号整数

· `UInt`：无符号整数

Int和UInt都可以在后面添加8、16、32以及64来指定整数长度，如果不添加，则默认与当前所在平台的**原生字长**一致。为了提高Swift代码的可维护性和可复用性，通常能不使用UInt就不使用UInt（避免类型转换），只有*在必须存储与平台原生字长相同的无符号整数*的情况下才使用它。

当然，如果是和C进行混编，可能还需要用到诸如`CLong`、`CLongLong`、`CUnsignedLongLong`等类型。

相比较而言，Kotlin的整数类型只有四种，包括：

· `Byte`：8位整数

· `Short`：16位整数

· `Int`：32位整数

· `Long`：64位整数

### 2.浮点数

无论是Kotlin还是Swift，都包含Double和Float两种类型的浮点数。Float是32位的浮点数，Double是64位的浮点数。

同样地，在和C混编的情况下，Swift可能还需要调用到`CDouble`、`CLongDouble`、`CFloat`等类型。

### 3.字符和字符串

Swift的字符类型为Character（Kotlin为Char），字符串类型和Kotlin一样均为String。

对于多行字符串，二者都使用`"""`包裹起这些字符串内容，并且一般不需要在里面进行字符的转义，直接使用即可（除非再使用"""，就需要加上`\`进行转义）。

对于转义字符，特殊字符的转义二者基本一致，unicode字符的转义有所差别：Kotlin中使用`\uXXXX`，而Swift使用`\u{XXXX}`来表示。

字符拼接在Swift和Kotlin中都是**不被允许**的，而字符串拼接都可以使用`+`来执行，此外，Swift还可以调用append函数，与Kotlin中的StringBuilder类似。

在字符串的比较中，Swift和Kotlin都可以对*字符相等、前缀相等*以及*后缀相等*这三种情况来进行相应的比较操作。

· `字符相等`

Kotlin和Swift都可以使用`==`和`!=`来比较两个字符串是否完全相等，同时二者也都具有类似的equal函数可以调用。值得注意的是，在Swift语言中，即使两个字符串是由不同的Unicode标量构成，只要可扩展的字形群集有同样的语言意义和外观就认为它们标准相等。

· `前缀相等`：

在Swift中，判断字符串是否具有特定前缀，可以使用hasPrefix函数，Kotlin则使用startWith函数。

· `后缀相等`：

与前缀相等判断类似，Swift使用hasSuffix函数执行判断，Kotlin使则用endWith函数。

### 4.布尔值

Swift的布尔值类型表示为Bool，Kotlin表示为Boolean。它们都采用true和false来表示逻辑运算中的真和假。

### 5.数值字面量

在Swift和Kotlin语言中，整数的字面量可以有如下写法：

· `十进制`：不加前缀

· `二进制`：加**0b**前缀

· `八进制`：加**0o**前缀（Kotlin目前不支持）

· `十六进制`：加**0x**前缀

Kotlin可以在整数后面加上`L`来标识该整数为Long类型，或是在浮点数后面加上`f`/`F`后缀标识其为Float类型，Swift没有这个机制。但是二者都可以使用科学记数法，即在浮点数后面添加e或E的后缀。

对于十六进制浮点数，Swift要求必须添加p或P后缀。

## 七、类型转换

### 1.数字转换

无论是Swift还是Kotlin，无论是在整数间、整数和浮点数间还是浮点数间的转换，首先都应当遵循“**不引发溢出**”的原则。对于Swift，整数和浮点数相互转换需要**显式地指定类型**，即使用`SomeType(params)`（比如Int( )、Double( )、Float( )等）类型的函数将数字转换成需要的类型。

### 2.字符串转换

Swift可以使用`Int( )`等函数将String类型的数字转换成对应的类型，也可以反过来用`String( )`函数将数字转换成String类型。

在Kotlin中，String类型的数字可以通过调用toByte、toShort等函数直接转换，但是从数字转换成String，则需要**先将该数字的值赋给一个变量或常量，之后让被赋值的变量或常量调用toString函数**。





