## 何为谓词

与先前所介绍过的可以返回数字、字符串或者日期等函数不同，谓词就是一种只返回逻辑值（TRUE/FALSE/UNKNOWN）的函数， 因此可以想到，比较运算符是属于谓词一类的。本部分主要介绍LIKE、BETWEEN、IS NULL/IS NOT NULL、 IN/NOT IN、以及EXISTS这几个谓词的用法。

## 用法介绍

+ LIKE

LIKE用来查询包含指定内容的记录，有三种形式：

```
WHERE str LIKE 'part matched %' --查询以指定内容为开头的记录
WHERE str LIKE '% part matched' --查询以指定内容为结尾的记录
WHERE str LIKE '% part matched %' --查询包含指定内容的记录
```

注意：
1. `%`可以被`_`替代，并且后者可以被当作一个字符的占位符；
2.  使用`_`可以指定所要查询的记录的长度。

+ BETWEEN

BETWEEN用来查询存在于一个连续范围内的记录：

```
WHERE record BETWEEN critical value 1 AND critical value 2
```

+ IS NULL和IS NOT NULL

这两个谓词用于判断记录是否为空。

```
WHERE record IS NULL / IS NOT NULL
```

+ IN和NOT IN

这两个谓词用于判断记录是否符合若干离散的条件中的任意一条。

```
WHERE record IN/NOT IN (cond 1, cond 2, ···)
```

注意：有时候IN可以被OR替代，但是IN可以用在子查询中，而且更具健壮性。

+ EXISTS和NOT EXISTS

这两个谓词用于判断符合条件的记录是否存在，经常和（查询全部记录的）关联子查询配合使用。

```
WHERE EXISTS/NOT EXISTS (correlated subquery)
```