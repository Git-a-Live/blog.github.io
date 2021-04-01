## 何为正则表达式

正则表达式（Regular Expression）是一种在程序当中用来描述某种语法规则的**字符串**，通常用于检索和替换那些匹配某种语法模式的文本。它可以实现文本的精确匹配（逐字符对应相等），但这只是一小部分功能。

正则表达式真正强大的地方在于，它可以构建出十分复杂的模式（pattern），用于匹配**任何**符合模式的字符串，而不需要考虑这些字符串代表着什么。这种抽取共性直面其本质的方式，就是正则表达式能够发挥强大作用的根本所在。

比如电子邮箱地址，Microsoft有Outlook，Google有Gmail，腾讯有QQ邮箱，而网易有163邮箱等等，对于正则表达式来说，它们都是一些`邮箱名@邮箱域名`格式的字符串。再比如手机号码，中国大陆的手机号码数不胜数，但本质上都是一串`1XX-XXXX-XXXX`格式的11位数字，而美国的手机号码就是一串`XXX-XXX-XXXX`格式的10位数字。

类似的例子还有很多，限于篇幅这里不做过多的举例说明。

## 基本使用

### 常用匹配语法

正则表达式的常用匹配语法如下：

|表达式|含义|示例说明|
|:--:|:--|:--|
|`\|`|选择，相当于编程语言里的或运算|`a\|b`可以匹配a或b|
|`+`|`+`前面的字符<font color=red>必须至少出现一次|`Goo+gle`可以匹配Google、Gooogle以及Goooogle等等|
|`?`|`?`前面的字符<font color=blue>最多只出现一次|`colou?r`可以匹配color和colour|
||放在`+`后面用作非贪婪匹配，尽可能少地匹配到字符|`(\d+?)(0*)`表示该字符串由非零值和零值两部分组成，可以匹配10、100、110以及123000等等|
|`*`|`*`前面的字符可以出现0次、1次以及若干次|`0*42`可以匹配42、042、0042以及00042等等|
|`()`|定义操作符的范围和优先度|`gr(a\|e)y`可以匹配gray和grey，等价于`gray\|grey`|
|`\`|将下一个字符标记为特殊字符|`\n`表示换行符，`\\`表示字符\|
|`^`|匹配输入字符串的<font color=green>开始</font>位置|`^A(\w+)`可以匹配Apple、Abandon以及A123等等|
|`$`|匹配输入字符串的<font color=orange>结束</font>位置|`(\w+)(uck)$`可以匹配duck、Fuck以及Suck等等|
|`.`|匹配除“\r”“\n”之外的任何单个字符|`A(.)BC`不能匹配A\rBC、A\nBC以及ABBBC之类的字符串|
|`A`|匹配指定字符|`ABC`只能匹配字符串ABC|
|`{n}`|匹配指定个数的字符|`1{5}`只能匹配字符串11111|
|`{n,m}`|匹配从n~m范围内个数的字符|`0{1,3}`能匹配字符串0、00和000|
|`{n,}`|匹配至少n个字符|`0{1,}`能匹配字符串0、00以及000等等|
|`\d`|匹配数字0~9||
|`\D`|匹配`\d`范围以外的任意字符||
|`\w`|匹配大小写字母、数字以及下划线||
|`\W`|匹配`\w`范围以外的任意字符||
|`\s`|匹配空格、Tab键||
|`\S`|匹配`\s`范围以外的任意字符||
|`\uXXX`|匹配Unicode字符|`\u548c`可以匹配Unicode字符“和”|
|`[ABC]`|匹配`[]`里面包含的任意字符||
|`[A-Z]`|匹配某个范围内的任意字符||
|`[^A-Z]`|匹配某个范围外的任意字符||

更多语法可以参考[正则表达式的Wikipedia](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)。

### 在程序中使用正则表达式

前面已经提到，正则表达式本质上就是一个描述某种语法规则的字符串，只有把这个字符串作为参数传到调用它的方法里面才能进行正则匹配。正则表达式的使用场景主要有三个：匹配、检索和替换。

#### 字符串匹配

匹配可以分为精确匹配、模糊匹配以及分组匹配，前两种主要调用的是字符串对象的`matches()`方法（在Kotlin中它是一个[infix函数](Kotlin/func?id=infix函数实现dsl)，而且接收的是Regex对象），该方法会返回一个Boolean类型的匹配结果：

```
/*精确匹配*/
"ABC" matches Regex("ABC") //true
"ABC" matches Regex("A") //false

/*模糊匹配*/
"ABC" matches Regex("A[ABC]C") //true
"ABC" matches Regex("A[^ABC]C") //false
```

事实上，精确匹配的本质就是判断两个字符串是否相等，在这种情况下并没有必要用上正则表达式，只需要调用`equals()`方法或在Kotlin中使用重载运算符`==`即可。

分组匹配不是像上面两种匹配那样只做简单判断，它还会返回匹配到对象。这种情况下就需要引入`java.util.regex`包，使用Pattern和Matcher这两个类。此外，分组匹配的“分组”二字，体现在`()`的使用上，凡是被一对`()`包住的内容都算一个组，正则表达式里有几对`()`就有几个组。下面是示例代码：

```
val phoneNumer = "0571-1234-5678"
val pattern: Pattern = Pattern.compile("(\\d{4})-(\\d{4})-(\\d{2}(\\d*))")
val matcher: Matcher = pattern.matcher(phoneNumer)
if (matcher.matches()) { //只有成功匹配到之后才能返回结果
    println(matcher.group(0)) //0571-1234-5678
    println(matcher.group(1)) //0571
    println(matcher.group(2)) //1234
    println(matcher.group(3)) //5678
    println(matcher.group(4)) //78
} else {
    println("Nothing matched")
}
```

注意到Matcher对象在成功匹配到字符串之后，返回的分组里还包含了整个字符串本身，也就是调用`matcher.group(0)`所得到的结果。其余分组按照<font color=red>从左往右、从外到内</font>的顺序依次被添加到group数组当中——因此注意不要让数组越界。

#### 字符串检索

字符串检索同样离不开Pattern和Matcher对象，在这种场景下，需要调用的是Matcher对象的`find()`方法，示例代码如下：

```
val string = "The quick brown fox jumps over the lazy dog."
val pattern = Pattern.compile("(\\w*)\\wo\\w(\\w*)")
val matcher = pattern.matcher(string)
while (matcher.find()) { //必须使用while循环才能反复检索输出
    println(string.substring(matcher.start(),matcher.end()))
} //返回的是brown、fox、dog
```

>注意，如果调用`matches()`方法，由于是直接拿整个字符串去跟正则表达式匹配，返回的结果始终为false，这样就不是检索了。

#### 字符串替换

Kotlin为字符串对象提供了许多`replace()`方法来完成字符串替换的工作，其中有两个接收字符或字符串用于精确匹配。还有两个是接收正则表达式转换而来的Regex对象进行模糊匹配和替换，以其中一个为例：

```
val string = "The quick brown fox jumps over the lazy dog."
println(string.replace(Regex("\\s"),"-")) //The-quick-brown-fox-jumps-over-the-lazy-dog.
```

正则表达式替换还有一个“反向引用”的概念。所谓反向引用，是指把分组匹配获得的分组子串，通过**模板引擎**又替换回原来的字符串当中，观察下面的示例代码：

```
val string = "The quick brown fox jumps over the lazy dog."
println(string.replace(Regex("\\s([a-z]{4})\\s")," <b>$1</b> "))
```

这段代码输出的结果是`The quick brown fox jumps <b>over</b> the <b>lazy</b> dog.`，注意到所有只包含四个字母的单词都被加上了`<b>`和`</b>`。事实上，“加上`<b>`和`<\b>`”只是表象，真正的操作是这样的：先根据正则表达式进行分组匹配，然后把获得的分组组合成` <b>XXXX</b> `形式的字符串，最后再替换掉字符串中原来的部分。

另外需要注意的是，在上面的示例代码中，`$1`不是什么随随便便填进去的东西，而是通过分组匹配拿到的分组子串，对应于Matcher对象的`group(1)`。如果示例里面不止一对`()`，可能还要用到`$2`和`$3`等等，那它们对应的就是`group(2)`和`group(3)`，以此类推。这种时候，`$1`发挥的就是模板引擎的作用了。