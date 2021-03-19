## 插入数据

插入数据使用INSERT语句：

```
--向指定列插入单行数据：
INSERT INTO <table>(col 1, col 2, ···)
VALUES (val 1, val 2, ···);

--向指定列插入多行数据：
START TRANSACTION;
INSERT INTO <table>(col 1, col 2, ···)
VALUES (val 1, val 2, ···);
INSERT INTO <table>(col 1, col 2, ···)
VALUES (val 1, val 2, ···);
···
COMMIT;

--向所有列插入数据：
INSERT INTO <table>VALUES (val 1, val 2, ···);

--从其他表复制数据（可以设置筛选条件）：
INSERT INTO <table 1>(col 1, col 2, ···)
SELECT col 1, col 2, ···
FROM <table 2>;
```

## 删除数据
删除表中数据使用DELETE语句：

```
--删除所有数据：
DELETE FROM <table>; --在MySQL中不推荐使用

TRUNCATE <table>; --在MySQL中推荐使用

--删除特定数据：
DELETE FROM <table>
WHERE <cond>;
```

## 更新数据

更新表中数据使用UPDATE语句：

```
--更新所有数据：
UPDATE <table> 
    SET <col> = <expression>; --不安全的操作

UPDATE <table> 
    SET <col> = <expression>
WHERE <cond>; --推荐的安全操作

--更新多列数据：
UPDATE <table>
    SET <col 1> = <expr 1>,
        <col 2> = <expr 2>,
        <col 3> = <expr 3>,
···
WHERE <cond>;
```

## 事务

事务包含了一系列处理。标准SQL语句中并没有规定事务开始的语句，因此MySQL规定以START TRANSACTION作为事务开始语句。 事务的结束则统一使用COMMIT或ROLLBACK语句。

```
START TRANSACTION;
statement 1;
statement 2;
statement 3; ···
COMMIT; --或者ROLLBACK
```

事实上，几乎所有的数据库都无需开始指令，因为它们基本都默认自动提交模式，即一个语句就是一个事务。 换句话说，事务从连接数据库就已经开始了，不会特意提醒用户在某个时间点开始执行。