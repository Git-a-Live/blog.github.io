## 一、枚举

枚举是一种特殊的类，它为一组相关的值定义了一个共同的类型，并且可以在开发者使用这些值的时候确保**类型安全**。此外，它还是另一种形式的“全局变量”。

### 定义
在Swift和Kotlin中，枚举的定义语法分别为：

```
Swift:
enum SomeEnumeration {
    case value1
    case value2,value3,···
}

Kotlin:
enum class SomeEnumeration {
    value1,
    value2,
    value3,
    ···
}
```
### 使用

在Swift中，当一个变量被声明为枚举类型时，可以**直接使用**`.`**运算符获取枚举值**进行赋值：

```
var variable: SomeEnueration = .value
```

当变量的类型已知时，再次为其赋值**可以省略枚举类型名**。在使用具有显式类型的枚举值时，这种写法让代码具有更好的可读性。Kotlin目前还不支持这样的语法。

### 遍历

在Swift中枚举值可以被遍历，前提是必须**声明该枚举类型遵循CaseIterable协议**，之后Swift会生成一个allCases属性，用于表示一个包含枚举所有成员的集合：

```
enum SomeEnumeration: CaseIterable {
    case value1
    case value2,value3,···
}

for value in SomeEnueration.allCases {
    //TODO
}
```

### 关联值

有时候把其他类型的值和枚举成员值一起存储起来会很有用，这额外的信息称为关联值，并且每次在代码中使用该枚举成员时，还可以修改这个关联值。通常关联值的定义方法如下：

```
enum SomeEnumeration {
    case value(ParamType1,ParamType2,···)
    ···
}

var variable = SomeEnumeration.value(param1,param2,···)
```

如果要提取这些关联值，可以像下面那样，将所需要的paramName当作已经被赋值的变量或常量拿来用即可：

```
类型1: 常量与变量混合
SomeEnumeration.value(let paramName1, var paramName2,···)

类型2: 全部常量
let SomeEnumeration.value(paramName1, paramName2,···)

类型3: 全部变量
var SomeEnumeration.value(paramName1, paramName2,···)

print(paramName1)
print(paramName2)
···
```

### 原始值

在Swift中，枚举成员可以被默认值（称为原始值）预填充，同时，Swift还会生成rawValue属性用于返回原始值。这些原始值的**类型必须相同**，且与枚举成员一一对应，**不能重复**：

```
enum SomeEnumeration: ParamType {
    case value1(rawValue1)
    case value2(rawValue2),value3(rawValue3),···
}

print(SomeEnumeration.value1.rawValue1)
print(SomeEnumeration.value2.rawValue2)
print(SomeEnumeration.value3.rawValue3)
···
```

Kotlin也有类似的用法：

```
enum class SomeEnueration(val param: ParamType) {
    value1(param1),value2(param2)
}

println(SomeEnueration.value1.param1)
println(SomeEnueration.value2.param2)
···
```

在使用原始值为**整数**或者**字符串**类型的枚举时，不需要显式地为每一个枚举成员设置原始值，Swift将会自动赋值。整数类型的原始值还会按照枚举成员的顺序自增，比如第一个枚举成员如果未设定原始值，其整数原始值为0，其他枚举成员的原始值依次自增；如果未设定枚举成员的字符串类型原始值，则默认使用枚举成员名作为原始值。

此外，Swift还为枚举提供了一个原始值构造器，可以通过传入一个原始值来获得枚举实例：

```
let value = SomeEnumeration(rawValue)
```

由于不能保证所有传入的rawValue都一定能返回确定存在的枚举实例，有可能会拿到空值nil，因此实际上value的类型为`SomeEnumeration?`。

### 递归枚举

递归枚举是一种枚举类型，它有一个或多个枚举成员使用该枚举类型的实例作为关联值。使用递归枚举时，编译器会插入一个间接层。开发者可以在枚举成员前加上`indirect`来表示该成员可递归，亦或是在枚举类型开头加上`indirect`关键字来表明它的所有成员都是可递归的：

```
部分枚举成员可递归:
enum SomeEnumeration {
    case value1(ParamType1,···)
    indirect case value2(SomeEnumeration,···)
    ···
}

所有枚举成员可递归:
indirect enum SomeEnumeration {
    case value1(ParamType1,···)
    case value2(SomeEnumeration,···)
    ···
}
```

## 二、集合

### 数组

数组（Arrays）是一种**有序**数据的集。在Swift和Kotlin中，这种数据结构都是相似的。

#### 构造

Swift中数组的构造方式主要有以下几种：

```
方式1:
var array = Array<Element>(params) //Array函数有几种重载，可以根据需要使用

方式2:
var array = [Element]()

方式3:
var array = [value1,value2,value3,···]
```

**如果想让一个数组不可变，那就用**`let`**来修饰被赋值的变量**。

Kotlin对应的数据结构是List，默认实现为ArrayList，通常使用以下方式构造：

```
方式1:
var array = ArrayList<Element>(capacity) //capacity可为空

方式2:
var array = listOf<Element>() //若需要使用可变数组则换成mutableListOf()

方式3:
var array = listOf(value1,value2,value3,···) //若需要使用可变数组则换成mutableListOf()
```

除此之外，Swift和Kotlin都支持若干**相同**类型的数组通过`+`进行组合。

#### 访问和修改

##### 1.获取数组内非空元素个数和数组容量

对于非空元素个数，Swift通过`.count`来获取，Kotlin则通过`.size`；对于数组容量，Swift提供了`.capacity`来直接获取数组的容量（空数组为0），Kotlin则需要利用`.lastIndex` + 1来间接获得数组容量（空数组返回-1，加上1之后正好是0）。

##### 2.判断是否为空数组

Swift和Kotlin分别提供了`.isEmpty`跟`.isEmpty()`来判断一个数组是否为空，而Kotlin还额外提供了`.isNotEmpty()`和`.isNullOrEmpty()`两个函数。

##### 3.访问数组元素

几乎所有的高级语言都提供了`数组名 + 下标索引`的方法来访问数组元素，Swift和Kotlin都采用`数组名[下标索引]`的方式来访问数组中存在的指定位置的元素。此外，两种语言中的数组都是**可迭代**的，因此可以使用for-in循环来进行遍历。

##### 4.修改数组元素

修改数组元素通常指将指定位置元素的值修改成另一个值。Swift可以直接使用`数组名[下标] = 值`的形式对数组指定位置元素进行修改。

而Kotlin要麻烦一些，如果使用`listOf()`构造数组，则该数组是**不可变**的，无法修改。只有使用`ArrayList<>()`或者`mutableListOf()`构造出的数组才能和Swift一样这么干，尽管对于可变数组，Kotlin又提供了`.set()`函数来修改指定位置元素，但IDE通常会建议不要用这种方式。

添加数组元素通常分为以下几种情况：

1）添加到数组首位

Swift提供了需要指定元素位置和值的`.insert()`函数，Kotlin则使用`.add()`函数。当然两者也可以选择使用`[待添加元素]+[原数组]`的方式进行拼接。

2）添加到数组中间某个位置

Swift的`.insert()`函数和Kotlin的`.add()`都可以完成这个工作。

3）添加到数组末尾

Swift可以使用`.append()`，Kotlin还是使用`.add()`。使用`[原数组]+[待添加元素]`的方式还是允许的。

删除数组元素相对添加而言就简单一些了，Swift提供了需要指定元素位置的`.remove()`函数，Kotlin则使用`.removeAt()`函数。当然，针对首位和末尾元素，Swift额外提供了`removeFirst()`和`.removeLast()`方法，因为依靠开发者自行手动输入元素索引可能存在隐患。同理，Kotlin也针对末尾元素提供了`.lastIndex`作为专门使用的索引。

无论是Swift还是Kotlin，都提供了可以一次性删除数组内所有元素的`.removeAll()`函数。

### 集合

集合（Sets）是一种**无序无重复**数据的集，Swift中的集合严格遵循了这一定义。而Kotlin分为**保留元素插入顺序**的`LinkedHashSet`和**不声明元素顺序**的`HashSet`两种实现，只有`HashSet`跟Swift所定义的集合相似，但Kotlin默认实现的是前者。后者与前者相比，可以使用**较少的内存**来存储相同数量的元素。

能够存储在集合中的类型，必须提供相应的**计算哈希值的方法**，这就是所谓的“可哈希化”。Swift中所有的基本类型，以及没有关联值的枚举成员值都是默认可哈希化的。

#### 构造

Swift中，集合有以下几种构造方式：

```
方式1:
var variable = Set<Element>()

方式2:
var variable: Set = [value1,value2,value3,···]
```
和数组一样，如果不想让集合被修改，就用`let`修饰被赋值的变量。

Kotlin中根据集合是否可变，有以下几种构造方式：

```
不可变:
var variable = setOf<Element>() //空集合不能进行转型操作
var variable = setOf(value1,value2,···) //如有需要可使用 as HashSet进行强制转型

可变:
var variable = mutableSetOf<Element>()
var variable = mutableSetOf(value1,value2,···)//如有需要可使用 as HashSet进行强制转型
```

#### 访问和修改

##### 1.获取集合内非空元素个数和集合容量

Swift和Kotlin分别依靠`.count`和`.size`来获取集合内非空元素个数以及容量。

##### 2.判断是否为空集合

和数组一样，Swift和Kotlin分别利用`.isEmpty`跟`.isEmpty()`来判断一个集合是否为空，Kotlin依然额外提供了`.isNotEmpty()`和`.isNullOrEmpty()`两个函数。

##### 3.访问集合元素

由于集合内部是无序的，因此无法像数组那样提供`集合名[下标]`的方式访问内部元素，通常只会用for-in循环对元素遍历。当然，遍历还是可以排序的，Swift提供了`.sorted()`方法，可用于对集合内部元素按照特定顺序排序后**返回一个有序数组**。

此外，判断集合内是否包含某个元素是不受排列顺序影响的，Swift和Kotlin都提供了`.contains()`函数用于执行这个功能。

##### 4.修改集合元素

因为集合内部是无序的，所以修改集合元素不需要指定位置，只有简单的添加和删除。

在Swift中，如果变量没有被`let`修饰，使用 `.insert()`可以添加一个新的元素，使用`.remove()`可以删除一个存在的元素（不存在就会返回nil），使用`.removeAll()`则可以移除集合内所有元素。

Kotlin中，只有可变集合才有写操作，对于一个可变集合，使用`.add()`可以添加一个新的元素，使用`.remove()`可以删除一个存在的元素，使用`.removeAll()`可以移除集合内所有元素。

##### 5.集合运算

Swift和Kotlin都提供了专门的方法用于集合间的运算：

1. 交集运算

Swift利用`setA.intersection(setB)`的方法返回一个集合，里面包含集合A与集合B**共有**的元素。Kotlin对应使用的是`setA.intersect(setB)`方法。

2. 并集运算

Swift和Kotlin都利用`setA.union(setB)`的方法返回一个集合，里面包含集合A与集合B**所有**的元素。

3. 差集运算

Swift利用`setA.subtracting(setB)`的方法返回一个集合，里面包含*仅存在于*集合A的元素。Kotlin对应使用的是`setA.subtract(setB)`方法。

4. 对称差运算

Swift利用`setA.symmetricDifference(setB)`方法返回一个集合，里面**排除**了集合A与集合B共有的元素，仅包含分别属于集合A和集合B的元素。

##### 6.集合关系判断

1. 相等

Swift和Kotlin都可以使用`==`来判断两个集合是否相等。两个集合相等的条件为：**一个集合内的元素总可以在另一个集合内找到类型和值相同的对应元素**。

2. 包含与被包含

Swift提供了`.isSubset()`方法来判断一个集合是为另一个集合的**子集**，对应地，`.isSuperset()`方法用于判断一个集合是否为另一个集合的**超集**。

`.isStrictSubset()`方法用于判断一个集合是否为另一个集合的**严格子集**，`.isStrictSuperset()`方法则用于判断判断一个集合是否为另一个集合的**严格超集**。所谓“严格”，意在强调两个集合**不相等**。

3. 相交

所谓“相交”，是指两个集合包含有**共同**的元素，这个概念就囊括了相等、包含的情形。Swift提供了`.isDisjoint()`方法来判断两个集合是否相交。

### 字典

字典（Dictionaries）是一种**无序键值对**的集。Kotlin中对应的数据结构是`Map`（更确切地说是`HashMap`，而非默认实现的`LinkedHashMap`）。字典中的值必须是**可哈希化**的。

#### 构造

Swift通常使用以下几种方式来构建Dictionariy：

```
方式1:
var variable = Dictionary<Key, Value>()

方式2:
var variable = [Key,Value]()

方式3:
var variable = [Key1:Value1, Key2:Value2,···]
```

Kotlin常使用以下方式构造Map：

```
LinkedHashMap:
var map = LinkedhashMap<Key,Value>()

HashMap:
var map = HashMap<Key,Value>()
```

#### 访问和修改

##### 1.获取字典内数据项数量

和数组、集合一样，Swift与Kotlin依然分别使用`.count`和`.size`来获取字典内的数据项数量。

##### 2.判断字典是否为空

和和数组、集合一样，Swift与Kotlin依然分别使用`.isEmpty`和`.isEmpty()`来判断字典是否为空。Kotlin继续提供`.isNotEmpty()`和`.isNullOrEmpty()`两个函数。

##### 3.访问字典数据项

无论是Swift还是Kotlin，都支持以`字典名[Key]`的语法来获取Key所对应的Value（若不存在该Key则返回空）。当然，Kotlin还跟Java一样，提供了`.get()`方法，通过Key来获取Value。

字典也是可迭代的，因此可以用for-in循环来遍历。

##### 4.修改字典数据项

对于Swift和Kotlin而言，修改字典数据项最简便的方式就是`字典名[Key] = newValue`。尽管Kotlin还保留有`.set()`方法，但IDE往往会建议使用下标语法。

要向字典中添加新的键值对，Swift和Kotlin依然一致地使用下标语法。

要删除字典中特定的键值对，Swift采用的是`字典名[Key] = nil`或`.removeValue()`方法，而Kotlin提供了`.remove()`函数。如果想删除字典中所有的键值对，Swift和Kotlin都提供了`.removeAll()`。

## 三、元组

元组（*tuples*）是一种**把若干个类型相同或不同的值**组合起来的复合值，其构造方式通常为：

```
完整版:
var tuples = (name1:value1, name2:value2, name3:value3, ···)

简化版:
var tuples = (value1, value2, value3, ···)
```

Kotlin可以用`Any`类型的的数组来模仿这一数据结构，例如一个同时包含字符串、数字和布尔值的`Any`数组：

```
var array = listOf("Kotlin", 100, true)
```

***元组这种数据结构只能用于简单的相关值分组，复杂的数据结构应当选择结构体和类***。

元组的内容分解（decompose）是指从元组中**提取出单独的变量或常量**，Swift通常使用以下几种方式提取：

```
方式1:
var (name1, name2, name3, ···) = tuples
print(name1) //name1被赋予value1
print(name2) //name2被赋予value2
print(name3) //name3被赋予value3
···

方式2:
print(tuples.0) //元组中第一个相关值
print(tuples.1) //元组中第二个相关值
print(tuples.2) //元组中第三个相关值
···

方式3:
print(tuples.name1) //元组中第一个相关值的名称
print(tuples.name2) //元组中第二个相关值的名称
print(tuples.name3) //元组中第三个相关值的名称
···
```