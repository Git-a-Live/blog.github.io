## 数组

数组是高级语言中最为常见的复合类型之一。它是一种用于存储**相同类型**元素的数据结构，通常采用基于连续存储单元地址的顺序存储方式。

### 基本结构和使用

C++中定义一个数组的典型方式如下：

```
int numbers[10] = {0,1,2,3,4,5,6,7,8,9,0}; 
char alphabeta[5];
double d[3] {1.0, 2.0, 3.0};
string s[] = {"A","B","C"}; //IDE自动为该数组计算元素个数
float f[] {0.0, 1.0, 2.0, 3.0}; //IDE自动为该数组计算元素个数
```

可以注意到，一个数组由三个部分构成：

+ 数组类型
+ 数组名称
+ 元素数目或数组长度（通常是常量，但是可以利用`new`运算符避开这种限制）

即创建一个数组的基本格式为：`type array_name[array_size]`

值得注意的是，在定义数组的同时，用户可以选择初始化，也可以选择不初始化，而后者带来的后果就是，**被定义好的数组无法再次被赋值，但是可以对数组里面的元素进行操作**。另外，要表示一个数组，其格式为`{元素1,元素2,元素3,···}`。将一个现成的数组以初始化的方式赋值给变量或常量是允许的，哪怕变量用的是`auto`进行类型自动推导。

还有一点值得注意，那就是数组的初始化同样遵循[自动转换的基本规则](/CPP/helloworld?id=基本类型转换)，禁止缩窄转换。

如果要访问数组中的某个元素，那么就采用下标的方式进行访问：

```
int numbers[10] = {0,1,2,3,4,5,6,7,8,9,0}; 
int num = numbers[3];
```

和其他绝大多数的高级语言一样，C++的数组下标也是从0开始的，这意味着数组中最后一个元素的下标为数组元素数目减一。如果使用的下标不在数组元素数目范围内（比如上面的例子当中访问的是numbers[-1]或者numbers[11]），程序在编译时可能不会报错，但是运行时就会出现问题，通常IDE会提示数组越界。为了能够更安全方便地使用数组以满足特定需求，C++又提供了数组的两个替代品，它们分别是vector和array。

### vector

模板类vector是一种使用`new`和`delete`自动完成内存管理的**动态数组**，可以在运行阶段设置数组长度，而不必像数组那样必须对长度进行初始化。此外，vector不仅可以在末尾附加数据，还可以在中间插入数据。

vector基本上就是C++中动态数组的默认实现，要使用vector，首先需要包含它的头文件：

```
#include <vector>
```

接着在程序中以如下格式创建vector对象：

```
vector<typeName> vectorName(vectorLength) //vectorLength可以是整型常量，也可以是整型变量
```

vector元素的访问方式有两种，一种跟数组一样，采用下标方式，而另外一种则是通过调用成员函数`at()`，如下面所示：

```
someVector[0]; //访问索引为0的元素
someVector.at(0); //同上
```

这两种方式的区别在于，下标方式不会检查索引是否越界，而成员函数会在运行期间捕获到非法索引并中断程序运行，从而禁止这种不安全的行为——这种检查带来的结果是，程序运行时间要更长一些。除此之外，vector对象还可以通过调用`begin()`和`end()`这两个成员函数来确定数组边界，从而更方便地避免数组越界情况的发生。

### array

模板类array是C++ 11引入的一个新型数组，它和数组一样具有**固定长度**，但是使用起来比数组更为方便和安全。

要使用array，同样需要导入头文件：

```
#include <array>
```

接着以如下格式创建array对象：

```
array<typeName, arrayLength> arrayName{};
```

同vector一样，array元素也有两种访问方式，一种是下标，另一种是成员函数`at()`，它们的差异和vector相同，因此不再赘述。

>注意，数组、vector和array有以下异同：
>1. array和数组都采用静态内存分配的方式（即栈）来存储元素，而vector则是将元素保存在自由存储区或者堆当中；
>2. 一个数组不能被赋值给另一个数组，但是一个vector（array）对象可以赋值给另一个vector（array）对象。

## 字符串

### string类

string类是C++库提供的一个专门用于存储字符串变量的数据类型，它跟用char数组存储字符串相比更为方便和安全。

要使用string类，必须先在程序中添加对应的头文件，如下列代码所示：

```
#include <string>
```

string类位于`std`名称空间当中，因此需要进行声明或采用`std::string`的方式来进行引用。string类型变量——确切地说是对象——跟char数组有很大的不同，前者就是一个表示字符串的实体，而后者将一个字符串拆分成若干个字符保存到一组存储单元当中。

string类型对象的声明和初始化方式都很简单，示例如下：

```
string str1;
str1 = "Hello World";

string str2 = "This is a string.";
```

string类型的对象可以在声明时不进行初始化，而且其内容长度也是可以调整的——是不是感觉非常眼熟？string类和vector一样，也是一种动态数组，这就是为什么string类比单纯的char数组更为方便和安全的重要原因之一。

string类型对象的简单输入输出和其他基本类型一样，都可以用`cin >>`和`cout <<`来进行操作，而不用考虑其内部的工作原理。

### C风格字符串

## 结构

## 共用体