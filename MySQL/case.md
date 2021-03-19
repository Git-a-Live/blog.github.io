CASE表达式相当于其他编程语言当中的条件（分支）语句，分为简单CASE表达式和搜索CASE表达式。

## 简单CASE表达式

简单CASE表达式的一般用法为：

```
CASE <expression>
    WHEN <expression 1> THEN <expression>
    WHEN <expression 2> THEN <expression>
    WHEN <expression 3> THEN <expression>
    ···
    ELSE <expression>
END
```

将上面的语句与其他语言中的switch-case进行对比，就可以发现有高度相似之处：

```
switch(<case>) {
    case <cond 1>: //TODO
    case <cond 2>: //TODO
    ···
    default: //TODO
}
```

## 搜索CASE表达式

搜索CASE表达式的一般用法为：

```
CASE WHEN <expression 1> THEN <expression>
    WHEN <expression 2> THEN <expression>
    WHEN <expression 3> THEN <expression>
    ···
    ELSE <expression>
END
```

将上面的语句与其他语言的if-else进行对比，同样能发现许多相似之处：

```
if (cond 1) {
    //TODO
} else if (cond 2) {
    //TODO
} ··· else {
    //TODO
}
```