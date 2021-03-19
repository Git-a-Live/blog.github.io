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

### 主键与外键

主键用于唯一地标识表中的每条信息，可以是单一字段，也可以是多个字段的组合。外键用于关联两张表，设置外键的表称为子表，它所关联的表即为父表。一张表的外键要么为空，要么必须是另一张表的主键，且数据类型一致。

主键用法如下：

```
--单一字段：
CREATE TABLE <name> (
    ···
    <column's name> <data type> PRIMARY KEY,
    ···
);

--多个字段：
CREATE TABLE <name> (
    ···
    PRIMARY KEY(column 1, column 2, ···)
    ···
);
```

外键用法如下：

```
CREATE TABLE <name> (
    ···
    CONSTRAINT <foreign key's alias> FOREIGN KEY (column 1, column 2, ···)
                        REFERENCES <table 2> (column 1, column 2, ···)
    ···
);
```

### 约束

非空约束可以确保被约束的字段都有非空值，其语法如下：

```
CREATE TABLE <name> (
    ···
    <column's name> <data type> NOT NULL,
    ···
);
```

唯一性约束可以确保该字段在当前表中不会重复，其语法如下：

```
CREATE TABLE <name> (
    ···
    <column's name> <data type> UNIQUE,
    ···
);
```

### 属性

自增属性通常用于为表中插入的新记录生成唯一ID，且只能约束一个主键字段。其语法如下：

```
CREATE TABLE <name> (
    ···
    <column's name> <data type> PRIMARY KEY AUTO_INCREMENT,
    ···
);
```

默认值属性用于为字段设置默认值，如果不设置该属性，而插入新记录时也没有为该字段赋值，系统就会尝试将其设置为空值。其语法如下：

```
CREATE TABLE <name> (
    ···
    <column's name> <data type> DEFAULT <default value>,
    ···
);
```

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