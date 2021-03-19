在数学领域，集合表示一个由0个或若干个元素组成的整体。与此相似，在数据库领域， 集合代表的是由各种记录（表、视图以及查询结果等）组合而成的整体。数学意义上的集合运算，包括交、并、补、差等运算； 而数据库中的集合运算也具有相似的概念，如加减等。

在数据库中，集合运算要注意以下事项：
+ 两张表中记录列数必须一致；
+ 数据类型相同的列方可进行运算；
+ ORDER BY子句只能使用一次，并且位于末尾。

## 表的并集运算

表的并集运算，指的是对两张表的记录进行合并，同时在查询结果中剔除重复的记录，也被称作加法：

```
SELECT <columns> FROM <table 1>
UNION [ALL] --使用ALL可以保存重复的记录
SELECT <columns> FROM <table 2>;
```

## 表的交集运算

表的交集运算，指的是提取出两张表的公共部分（即相同的记录），在标准SQL语句中，可以直接使用下面的语法：

```
SELECT <columns> FROM <table 1>
INTERSECT
SELECT <columns> FROM <table 2>;
```

但是MySQL目前尚未支持这种语法，因此使用以下语句来替代：

```
--方式一：
SELECT DISTINCT <columns>
FROM <table 1>
INNER JOIN <table 2> USING(<columns>); --表的联结会在下面介绍

--方式二：
SELECT DISTINCT <columns>
FROM <table 1>
WHERE <columns> IN
(SELECT <columns> FROM <table 2>);
```

## 表的差集运算

表的差集运算，指的是从一张表中剔除掉同时存在于另一张表的记录并返回剩余记录。这个语法和交集运算一样，MySQL目前也是不支持的。标准SQL的语法为：

```
SELECT <columns> FROM <table 1>
EXCEPT
SELECT <columns> FROM <table 2>;
```

MySQL的替代语法为：

```
ELECT DISTINCT <columns>
FROM <table 1>
WHERE <columns> NOT IN
(SELECT <columns> FROM <table 2>);
```

## 表的联结

表的联结，指的是从多张表中获取指定的数据列并返回查询结果。联结又分为内联结和外联结，前者只能选取出同时存在于多张表中的数据 （这就是MySQL可以利用内联结模拟交集运算的原因），而后者可以获取主表中所有数据，以及若干存在于其他表中的数据。

还有一种被称为交叉联结的运算方式， 从本质上来说，它是所有联结的基础。内联结和外联结从本质上来说就是筛选并返回指定的交叉联结结果。交叉联结的作用是将参与联结的若干张表中的所有数据进行交叉组合 （做笛卡尔积运算），然后输出一个记录数为所有表行数乘积的查询结果。 然而这种联结在实际场景中并不具有太大的实用性，而且还会大量消耗硬件资源，因此并不常用。

+ 内联结

使用内联结有两个地方要注意：一是使用INNER JOIN联结两张表，二是使用ON子句来指定联结键，即两张表联结所使用的列，联结键可以有多个。 如果需要经常使用内联结查询结果，最好是保存成视图。内联结的一般用法如下：

```
SELECT <alias of table 1>.<column 1>,
<alias of table 1>.<column 2>,
<alias of table 1>.<column 3>,
···
FROM <table 1> AS <alias> INNER JOIN <table 2> AS <alias>
ON <join key>  --ON必须位于FROM和WHERE之间
WHERE <cond>;
```

+ 外联结

外联结又分左联结和右联结，前者表示以左边的表作为主表，后者表示以右边的表作为主表。

```
SELECT <alias of table 1>.<column 1>,
<alias of table 1>.<column 2>,
<alias of table 1>.<column 3>,
···
FROM <table 1> AS <alias> LEFT/RIGHT OUTER JOIN <table 2> AS <alias>
ON <join key>
WHERE <cond>;
```