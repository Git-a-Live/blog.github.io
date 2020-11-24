

## SQL语句种类

SQL语句可以分为三大类：DDL（数据定义语言）、DML（数据操纵语言）以及DCL（数据控制语言）：

+ DDL：包含CREATE、DROP以及ALTER
  
  CREATE: 创建数据库和表等对象；

  DROP: 删除数据库和表等对象；

  ALTER: 修改数据库和表等对象的结构。

+ DML：包含SELECT、INSERT、UPDATE以及DELETE
  
  SELECT: 查询表中数据；

  INSERT: 向表中插入数据；

  UPDATE: 更新表中数据；

  DELETE: 删除表中数据。

+ DCL：包含COMMIT、ROLLBACK、GRANT以及REVOKE
  
  COMMIT: 确认对数据库中的数据进行的变更；

  ROLLBACK: 取消对数据库中的数据进行的变更；

  GRANT: 赋予用户操作权限；

  REVOKE: 取消用户的操作权限。

## 基本书写规则

1. SQL语句以分号结尾；
2. 关键字不区分大小写；
3. 字符串和日期需要用单引号括起来标识，数字不用；
4. 单词之间用空格或换行分隔