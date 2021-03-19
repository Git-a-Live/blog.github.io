## 常用聚合函数

+ COUNT()：可用于计算表中数据的行数，当其参数为*(所有列)时，输出的行数包含NULL；当参数为指定列时，输出的行数不包含NULL。

```
--所有列聚合：
SELECT COUNT(*)
FROM <name_table>;

--指定列聚合：
SELECT COUNT(<name_col>)
FROM <name_table>;
```

+ SUM()：可用于计算指定列中所有数据的合计值，会自动排除包含NULL的项目。

```
SELECT SUM(<name_col>)
FROM <name_table>;
```

+ AVG()：可用于计算指定列中所有数据的平均值，会自动排除包含NULL的项目。

```
SELECT AVG(<name_col>)
FROM <name_table>;
```

+ MAX()：可用于获取指定列中数据最大值。

```
SELECT MAX(<name_col>)
FROM <name_table>;
```

+ MIN()：可用于获取指定列中数据最小值。

```
SELECT MIN(<name_col>)
FROM <name_table>;
```

在聚合函数的参数前加上DISTINCT，可以排除重复数据。

```
SELECT Aggregate Function(DISTINCT <param>)
FROM <name_table>;
```

更多聚合函数，详见[MySQL官网](https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions-and-modifiers.html)。

## 分组

GROUP BY用于按指定列分组汇总数据，这个子句**总是**位于FROM或WHERE的后面。GROUP BY中的指定列被称为聚合键或分组列。 当数据中包含NULL时，GROUP BY也会把NULL划分为一组进行汇总。

```
SELECT <name_col1>,
         <name_col2>,
         ···
         Aggregate Function(<param>)
FROM <name_table>
WHERE <cond>
GROUP BY <name_col1>,
         <name_col2>,
         ···;
```

注意：
+ 使用GROUP BY的SELECT中只能包含常数、聚合函数以及GROUP BY中的聚合键；
+ GROUP BY中不能使用列的别名；
+ GROUP BY的显示结果是无序的。

## 为聚合结果指定条件

WHERE子句只能对数据行指定条件，而使用HAVING子句可以对分组指定条件。HAVING子句位于GROUP BY后面，而且包含的要素和使用了GROUP BY的SELECT语句一样。

```
SELECT <name_col1>,
         <name_col2>,
         ···
         Aggregate Function(<param>)
FROM <name_table>
WHERE <cond>
GROUP BY <name_col1>,
         <name_col2>,
         ···
HAVING <cond>;
```

## 对查询结果进行排序

ORDER BY用于对查询结果按照指定列进行排序，通常写在SELECT的末尾。 ORDER BY中的指定列被称为排序键，排序列后面还可以指定排序方式，默认是升序排列。

```
SELECT <name_col1>,
         <name_col2>,
         ···
         Aggregate Function(<param>)
FROM <name_table>
WHERE <cond>
GROUP BY <name_col1>,
         <name_col2>,
         ···
HAVING <cond>
ORDER BY <name_col1> [DESC],
         <name_col2> [DESC],
         ···;
```

注意：
+ ORDER BY可以指定多个排序键；
+ NULL总是排在查询结果的开头或末尾；
+ 排序键可以使用别名；
+ ORDER BY可以使用存在于表中、但不在SELECT语句中的列；
+ ORDER BY可以使用聚合函数；
+ ORDER BY中不要使用列编号；