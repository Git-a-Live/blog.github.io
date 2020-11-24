## 表的创建
在创建表之前，需要先创建用于存储表的数据库，其指令如下：

```
CREATE DATABASE <name>
```
数据库创建完成以后，可以输入如下指令以创建表：

```
CREATE TABLE <name>
(column 1, data type, column constraint,
 column 2, data type, column constraint,
 column 3, data type, column constraint,
 ···
 constraint 1, constraint 2, ···);
 ```

注意：数据库、表以及列的名称只能以英文字母、数字和下划线（_）组成，且以英文字母作为开头，不能使用重复名称。

## 表内设置

数据类型：对应上面的data type，包含INTEGER、CHAR、VARCHAR以及DATE等类型；

约束条件：对应上面的constraint，包含NULL、NOT NULL以及PRIMARY KEY等；

## 删除和更新

删除表只需要如下指令：

```
DROP TABLE <name>
```

向表中添加列：

```
ALTER TABLE <name> ADD COLUMN <definition of the column>
```

从表中删除列：

```
ALTER TABLE <name> DROP COLUMN ‹name of the column ›
```

向表中插入数据：

```
//单行数据：
INSERT INTO <name> VALUES (value 1, value 2, value 3, ···);

//多行数据：
START TRANSACTION;
INSERT INTO <name> VALUES (value 1, value 2, value 3, ···);
INSERT INTO <name> VALUES (value 1, value 2, value 3, ···);
INSERT INTO <name> VALUES (value 1, value 2, value 3, ···);
INSERT INTO <name> VALUES (value 1, value 2, value 3, ···);
···
COMMIT;
```

## 表名更改

表名更改的指令也很简单：

```
RENAME TABLE <old name> TO <new name>
```