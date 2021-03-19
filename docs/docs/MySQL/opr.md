## 算术运算符

基本的算术运算符包括+（加法）、-（减法）、*（乘法）和/（除法），在这一基础之上，还有比较运算符， 包括=（等于）、>（大于）、<（小于）、<=（不大于）、>=（不小于）以及<>（不等于）。在MySQL中， 可以使用不带FROM的SELECT语句来显示运算结果；比较运算需要使用WHERE子句。

```
--算术运算：
SELECT <expression> [AS <result>]; --AS是可选的

--比较运算：
SELECT <name_cols>
FROM <name_table>
WHERE <expression>;
```

算术运算符不能用于NULL，对字符串和数字运用比较运算符时，要注意它们的区别。

## 逻辑运算符

常用的逻辑运算符包括AND（与）、OR（或）和NOT（非），在判断NULL时，需要使用IS NULL或IS NOT NULL。 逻辑运算符通常用于WHERE子句中。

```
SELECT <name_col>
FROM <name_table>
WHERE [NOT <cond>]
        [<cond 1> AND <cond 2>]
        [<cond 1> OR <cond 2>]
        [<cond> IS NULL]
        [<cond> IS NOT NULL];
```

在MySQL中，由于NULL的存在，逻辑运算真值表不再是简单的TRUE（真）和FALSE（假），而是又多了一个“不确定（UNKNOWN）”。