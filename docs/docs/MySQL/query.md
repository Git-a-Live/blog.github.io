## 查看所有列

```
SELECT * FROM <name_table>
```
## 查看特定列

```
SELECT <name_col1>, <name_col2>, ··· FROM <name_table>
```

## 在查询结果中设置列的别名

```
SELECT <name_col1> AS alias_1,
<name_col2> AS alias_2, ···
FROM <name_table>
```
注意：设置中文别名时需要用双引号括起来（MySQL可以不用，但标准SQL中应该这么做）

## 在查询结果中剔除重复行

```
SELECT DISTINCT <name_cols> FROM <name_table>
```

注意：DISTINCT只能位于SELECT和第一个列名之间

## 指定查询条件

```
SELECT <name_cols>, ···
FROM <name_table>
WHERE <cond1>
AND <cond2>
AND ···;
```

注意：WHERE必须位于FROM后面