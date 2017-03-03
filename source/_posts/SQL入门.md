---
title: SQL入门
date: 2017-03-03 20:40:04
categories: coding
tags:
  - SQL
---

本文总结了一些 SQL 操作的基本介绍

## SQL 简介

SQL 是用于访问和处理数据库的标准的计算机语言。

**SQL 对大小写不敏感！**

可以把 SQL 分为两个部分：数据操作语言 (DML) 和 数据定义语言 (DDL)。

查询和更新指令构成了 SQL 的 DML 部分：

* `SELECT` - 从数据库表中获取数据
* `UPDATE` - 更新数据库表中的数据
* `DELETE` - 从数据库表中删除数据
* `INSERT INTO` - 向数据库表中插入数据

SQL 的数据定义语言 (DDL) 部分使我们有能力创建或删除表格。我们也可以定义索引（键），规定表之间的链接，以及施加表间的约束。

* `CREATE DATABASE` - 创建新数据库
* `ALTER DATABASE` - 修改数据库
* `CREATE TABLE` - 创建新表
* `ALTER TABLE` - 变更（改变）数据库表
* `DROP TABLE` - 删除表
* `CREATE INDEX` - 创建索引（搜索键）
* `DROP INDEX` - 删除索引

<!--more-->

## SQL 数据类型

不同的数据库中对数据类型的定义有所不同，我在这里按照 MySQL 中的数据类型介绍，如果需要使用其他数据库的数据类型，可以查看[w3school | SQL 数据类型](http://www.w3school.com.cn/sql/sql_datatypes.asp)

在 MySQL 中，有三种主要的类型：文本、数字和日期/时间类型。

**Text 类型：**

![](http://ojt6zsxg2.bkt.clouddn.com/0d1589d4e1c8050751ca47e78bc0f189.png)

**Number 类型：**

![](http://ojt6zsxg2.bkt.clouddn.com/96ad55e98b6b7eba19072bc70b61dd31.png)

**Date 类型：**

![](http://ojt6zsxg2.bkt.clouddn.com/0c5d432edee1b368e2f586bd49202102.png)

## 运算符

运算符用于指定一个SQL语句中的条件，并作为连词多个条件在一份声明中。

* 算术运算符
* 比较操作符
* 逻辑运算符

### SQL算术运算符

![](http://ojt6zsxg2.bkt.clouddn.com/4762033d8dbf4de96a713cf23aa4c373.png)

### SQL比较操作符

![](http://ojt6zsxg2.bkt.clouddn.com/5d54c8d8e292d32990aeaaa0ad6492a7.png)

### SQL逻辑运算符

![](http://ojt6zsxg2.bkt.clouddn.com/b114ccbc13ec83ed53304e7fa726e5b9.png)

## 数据库级的操作

CREATE DATABASE语句的基本语法如下：

```
CREATE DATABASE DatabaseName;
```

查看所有的数据库：

```
show databases;
```

删除一个现有的数据库

```
DROP DATABASE DatabaseName;
```

选择一个数据库

```
USE DatabaseName;
```



## SELECT

`SELECT` 语句用于从表中选取数据。

结果被存储在一个结果表中（称为结果集）。

```
SELECT 列名称 FROM 表名称 //SELECT LastName,FirstName FROM Persons
SELECT * FROM 表名称  //星号（*）是选取所有列的快捷方式。
```

### SELECT DISTINCT

在表中，可能会包含重复值。这并不成问题，不过，有时您也许希望仅仅列出不同（distinct）的值。

关键词 `DISTINCT` 用于返回唯一不同的值。

```
SELECT DISTINCT 列名称 FROM 表名称
```

## WHERE

如需有条件地从表中选取数据，可将 `WHERE` 子句添加到 `SELECT` 语句。

```
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```

```
操作符            描述
=	             等于
<>	            不等于 //在某些版本的 SQL 中，操作符 <> 可以写为 !=
>	             大于
<	             小于
>=	           大于等于
<=	           小于等于
BETWEEN	     在某个范围内
LIKE	     搜索某种模式
```

举例：

```
SELECT * FROM Persons WHERE City='Beijing'
```

**引号的使用**

请注意，我们在例子中的条件值周围使用的是单引号。

SQL 使用单引号来环绕文本值（大部分数据库系统也接受双引号）。如果是数值，请不要使用引号。

## `AND` & `OR `

`AND` 和 `OR` 运算符用于基于一个以上的条件对记录进行过滤。

`AND` 和 `OR` 可在 `WHERE` 子语句中把两个或多个条件结合起来：

* 如果第一个条件和第二个条件都成立，则 `AND` 运算符显示一条记录。
* 如果第一个条件和第二个条件中只要有一个成立，则 `OR` 运算符显示一条记录。


举例：

```
SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter'
```

我们也可以把 `AND` 和 `OR` 结合起来（使用圆括号来组成复杂的表达式）:

```
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William')
AND LastName='Carter'
```

## ORDER BY

`ORDER BY` 语句用于对结果集进行排序。

`ORDER BY` 语句默认按照升序对记录进行排序。

如果您希望按照降序对记录进行排序，可以使用 `DESC` 关键字。

举例：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company
```

以字母顺序显示公司名称（Company），接着按照数字顺序显示顺序号（OrderNumber）：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber
```

以逆字母顺序显示公司名称：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC
```

以逆字母顺序显示公司名称，并以数字顺序显示顺序号：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```

## INSERT INTO

语法

```
INSERT INTO 表名称 VALUES (值1, 值2,....)
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

## UPDATE

语法：

```
UPDATE 表名称 SET 列名称 = 新值 WHERE ...
```

举例：

```
UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing'
WHERE LastName = 'Wilson'
```

## DELETE

DELETE 语句用于删除表中的行。

```
DELETE FROM 表名称 WHERE 列名称 = 值
```

可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的

```
DELETE * FROM table_name
```











































