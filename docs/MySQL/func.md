## 算数函数

除了最基本的加减乘除四则运算之外，MySQL中比较常用的算术函数还包括绝对值函数ABS()、求余函数MOD()以及四舍五入函数ROUND()。

```
--ABS()
ABS(n) --相当于表达式n >= 0? n: -1*n

--MOD()
MOD(m,n) --相当于表达式m % n

--ROUND()
ROUND(m,n) --m为被四舍五入的数，n为小数点有效位数
```

## 字符串函数

常见的对字符串操作有拼接、求长度、大小写转换、替换以及截取等，这些操作在MySQL中对应的函数及用法如下：

```
--拼接函数CONCAT()
CONCAT(str 1, str 2, str 3, ···)

--求字符串长函数LENGTH()
LENGTH(str) --CJK字符串在不同的数据库中调用该函数可能会返回不同结果

--大小写转换函数LOWER()和UPPER()
LOWER(str) --对CJK字符无效
UPPER(str) --对CJK字符无效

--替换函数REPLACE()
REPLACE(target str, str replaced, str replacing)

--截取函数SUBSTRING()
SUBSTRING(target str [FROM] starting position [TO] numbers of chars in substring)
```

## 日期函数

这里只介绍可用于绝大部分DBMS的日期函数。

```
--求当前日期
SELECT CURRENT_DATE;

--求当前时间
SELECT CURRENT_TIME;

--求当前时间戳
SELECT CURRENT_TIMESTAMP;

--从时间戳中提取指定内容
EXTRACT(some part of the timestamp FROM timestamp)

--时间转换
CAST(values before casting AS target type)

--返回从左到右第一个非空字符串
COALESCE(str 1, str 2, str 3, ···)
```