函数（function）是执行特定功能的代码模块，可以被其他程序或代码所引用。每个C++程序都至少包含一个函数——`main()`函数，且可以定义额外的函数。与函数相关联的一个重要概念是“方法”（method），它指定义在类当中，必须通过类或类的实例对象才能进行引用的函数，这个在学习到类相关的内容时会做更深入的讨论。C++自带的库函数通常只需要了解怎么调用，因此这里把重心放在如何自定义函数上面。

## 基本结构和使用

C++函数的基本结构由返回类型、函数名、入参以及函数体四大部分构成：

```
type function_name(type_argument1 arg1,type_argument2 arg2,···) {
    statements
}
```

返回类型可以是C++的基本类型（数组除外），也可以是用户自定义类型，还可以设为`void`，即无返回值。

函数名的命名规则和普通的变量名类似。

入参需要同时声明参数类型和参数名，其中入参的参数名用于声明一个起到占位符作用的**形式参数**。

函数体就是容纳具体实现代码的地方，函数所要执行的特定功能均在此处实现。只要返回类型不是`void`，都必须仿照`main()`在函数体的最后使用`return`语句返回一个值。

C++函数的标准定义流程如下：

```
#include <iostream>

//在main()函数前声明函数原型
type function_name(type_argument1,type_argument2,···);

int main() {
    //TODO：在main()函数中调用该函数
    return 0;
}

//在main()函数之后实现该函数
type function_name(type_argument1 arg1,type_argument2 arg2,···) {
    //TODO：编写具体的实现代码
}
```

有一种简化的做法就是直接用函数实现取代函数原型声明，这样做在实际开发当中也是可以运行的：

```
#include <iostream>

//在main()函数调用之前实现该函数
type function_name(type_argument1 arg1,type_argument2 arg2,···) {
    //TODO：编写具体的实现代码
}

int main() {
    //TODO：在main()函数中调用该函数
    return 0;
}
```

需要注意的是，把函数的实现放在声明原型之后和`main()`之前也是可以的，但是不声明原型又在`main()`之后才实现函数是不被允许的。

## 进阶部分
