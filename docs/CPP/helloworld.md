按照IT界的惯例，第一个程序是输出“Hello World”，这里自然也是继续遵循传统。但是要知道，“Hello World”的意义不仅仅是检测IDE、编译环境是否正常，它还提供了一些与程序相关的基本信息。

## 基本结构

下面是CLion创建的“Hello World”样例代码：

```
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

这个样例展示了C++程序最基本的结构：
+ 预处理器编译指令`#include`
+ 函数头`int main()`
+ 函数体
+ 结束main函数的return语句

乍一看C++程序的基本结构跟C语言好像没什么太大的区别，确实如此，毕竟都是C嘛！但是仔细观察就会发现两者的不同：首先，C++的预处理器编译指令导入的是`iostream`，而C语言是`stdio.h`；其次，C语言用`printf()`函数来打印，C++用的是`cout`，而且还采用了函数式引用。

事实上，更为常见的一种程序结构是这样的：

```
#include <iostream>

int main() {
    using namespace std;
    cout << "Hello, World!" << endl;
    return 0;
}
```

`using namespace std`是一个编译指令，用于声明**名称空间**。所谓名称空间是一项C++特性，类似于Java中的import。比如，一个程序里面存在两个隶属于不同库、但函数头完全一样的函数，这时候就需要在函数前面加上名称空间的前缀以示区分。就像上面的`cout`和`endl`一样，如果不加前缀或不声明为std，IDE就会提示代码有错误，因为此时IDE根本不知道`cout`和`endl`调用的是哪个库、哪个版本的函数。

假设有一个库名为example，它也包含`cout`和`endl`，为了在程序当中把它们与C++标准库的那两个区分开来，就需要这样做：

```
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    example::cout << "Hi, World!" << example::endl;
    return 0;
}
```

当然，如果是简单的程序，没有用到别的库，在通常情况下可以使用`using namespace std`来偷懒，这样就不必在每个函数前面都加上`std::`的前缀。

## 基本语句

### 声明变量

和C语言一样，C++的任何变量在使用前都必须声明，否则IDE会捕捉到这一错误并予以提示：

```
int number;
string word;
```

### 变量赋值

绝大多数语言都使用`=`来对变量进行赋值，C++也不例外：

```
//方式一：
int number = 0;

//方式二：
int num1 = 1, num2 = 2, num3 = 3;

//方式三：
int num1, num2, num3;
num1 = num2 = num3 = 6;
```

注意到方式三里面，三个变量被连续赋值，此时的赋值顺序是从右往左。

### 输出语句

C++使用的是`std::cout`来输出打印，它比C语言的`printf()`函数更为智能，尤其是不需要指明要打印的变量是何种类型（这一特性来源于面向对象）。Java当中的`System.out.println()`也同样具备这样的特性。C++的`std::cout`通常有以下用法：

```
//方式一：
std::cout << "Hi" << std::endl;

//方式二：
int age = 20;
std::cout << "Alice is " << age << " years old" << std::endl;

//方式三：
int num1 = 0;
int num2 = 1;
int num3 = 2;
std::cout << "Three numbers: "
<< num1 << " , "
<< num2 << " , "
<< num3 << std::endl;
```

C++可以使用`std::cout`进行字符串拼接，这一特性与Java使用`+`拼接字符串一样。

这里注意到`std::cout`跟`std::endl`一起使用，后者用来换行，跟`\n`的效果是一样的。在通常情况下，如果想为一行字符串换行，使用`\n`可以减少输入量，而纯粹打印一行空行，用`std::endl`更为方便一点。

### 输入语句

C++使用`std::cin`来实现输入，与C语言的`scanf()`不同。它的常用方式为：

```
int age;
std::cout << "How old are you?" << std::endl;
std::cin >> age;
std::cout << "You're " << age << " years old now." << std::endl;
```

在某些情况下，C++程序被编译成`.exe`文件后一运行，控制台窗口仅仅在屏幕上一闪而过，这时候可以在程序的`return 0;`语句前加上一句`std::cin.get()`，之后再编译成`.exe`文件运行，控制台窗口便不会再出现这种情况。原因在于，`std::cin.get()`会等待用户的输入操作，只要控制台窗口执行到这一句就会处于等待状态，只有当用户按下任意按键进行输入时，后面的语句才会继续执行下去。除了`std::cin.get()`语句，`system("pause")`也有类似的作用。

### 循环语句

#### for循环

C++for循环语句的常用方式很简单：

```
for (int i = 0; i < some_value; i++) {
    //TODO: 循环执行若干次
}
```

和C语言一样，C++的for循环也是由这几个部分组成的：

+ 初始值`int i = 0`（初始值的赋值变量可以在外面声明）
+ 执行条件`i < some_value`
+ 步长`i++`（可以改得更大或者更小）
+ 循环体

#### while/do-while循环

C++的while循环常见使用方式如下：

```
while(some_condition) {
    //TODO: 当some_condition为真时循环执行
}
```

### 分支语句

#### if/if-else语句

#### switch语句

### 常用运算符

|运算符|作用|
|:--|:--|
|+||
|-||
|*||
|/||
|%||
|\\||
|==||
|!=||
|>||
|<||
|>=||
|<=|
|&&|||
|\|\|||
|!||
