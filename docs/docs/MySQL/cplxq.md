## 视图

视图可以通过SELECT语句从指定的表中查询并返回数据。视图本身不存储数据，只是存储SELECT语句以便用户能快速执行。 创建视图使用CREATE语句：

```
CREATE VIEW <name>(col 1, col 2, ···)
AS
<SELECT-block>;
```

注意：
+ 不要在其他的视图的基础上再创建视图，否则会引起性能问题；
+ 视图中的SELECT语句内使用ORDER BY是无效的；
+ 只有未使用聚合功能的视图才可以更新数据；
+ 频繁使用的SELECT语句可以封装成视图来使用。

删除视图使用DROP语句：

```
DROP VIEW <view>;
```

## 子查询

子查询又可称为内查询，因为它在本质上是一个一次性视图，即通过SELECT语句查询一个SELECT语句块，这个SELECT语句块的内容就是所谓的子查询。

```
SELECT <contents_queried>;
FROM (<SELECT-block>)
AS <alias>;
```

标量子查询是一种只返回单一值的子查询，可以跟比较运算符搭配使用，也可以书写在几乎所有能够使用常数和列名的位置。

## 关联子查询

关联子查询常用于对表中某一部分记录集合进行比较，通过在子查询中添加一个结合条件（一般是WHERE子句）， 可以实现“限定”的作用。下面以《SQL基础教程（第二版）》当中的一个例子来进行说明。

```
SELECT product_type, product_name, sale_price
FROM Product AS P1
WHERE sale_price > (SELECT AVG(sale_price)
        FROM Product AS P2
        WHERE P1.product_type = P2.product_type
        GROUP BY product_type);
```

注意：关联条件的WHERE语句必须位于子查询的语句块中。

这个例子要执行的操作是按照商品的种类，查询并返回售价高于该种类平均售价的商品的类型、名称以及价格。 WHERE P1.product_type = P2.product_type的作用是从Product中筛选出指定的商品种类以计算平均售价。 这个指定的商品种类，就是“既存在于P1中，又存在于P2中”的种类（尽管P1和P2只是同一张表的两个别名，但把它们当成两张内容一模一样的表可能更方便理解）。 在经过这个结合条件筛选之后，就可以确保从P2中只返回某一种类商品的平均售价，即一个单一的值，而P1中用于比较的商品自然也是属于这一种类。