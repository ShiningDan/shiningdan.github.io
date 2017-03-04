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

## 表级的操作

### SQL 约束 (Constraints)

约束用于限制加入表的数据的类型。

可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

* NOT NULL
* UNIQUE
* PRIMARY KEY
* FOREIGN KEY
* CHECK
* DEFAULT

#### UNIQUE 约束

`UNIQUE` 约束唯一标识数据库表中的每条记录。

`UNIQUE` 和 `PRIMARY KEY` 约束均为列或列集合提供了唯一性的保证。

`PRIMARY KEY` 拥有自动定义的 `UNIQUE` 约束。

请注意，每个表可以有多个 `UNIQUE` 约束，但是每个表只能有一个 `PRIMARY KEY` 约束。

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
UNIQUE (Id_P)
)
```

如果需要命名 `UNIQUE` 约束，以及为多个列定义 `UNIQUE` 约束，请使用下面的 SQL 语法：

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
)
```

**SQL UNIQUE Constraint on ALTER TABLE**

```
ALTER TABLE Persons
ADD UNIQUE (Id_P)

ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)  // 有一个别名 uc_PersonID
```

**撤销 UNIQUE 约束**

```
ALTER TABLE Persons
DROP INDEX uc_PersonID
```

#### PRIMARY KEY 约束

`PRIMARY KEY` 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 `NULL` 值。

每个表都应该有一个主键，并且每个表只能有一个主键。

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (Id_P)
)
```

如果需要命名 `PRIMARY KEY` 约束，以及为多个列定义 `PRIMARY KEY` 约束，请使用下面的 SQL 语法：

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
)
```

**SQL PRIMARY KEY Constraint on ALTER TABLE**

```
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)

ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```

**如需撤销 PRIMARY KEY 约束，请使用下面的 SQL：**

```
ALTER TABLE Persons
DROP PRIMARY KEY

ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

#### FOREIGN KEY 约束

一个表中的 `FOREIGN KEY` 指向另一个表中的 `PRIMARY KEY`。

`FOREIGN KEY` 约束用于预防破坏表之间连接的动作。

`FOREIGN KEY` 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。


下面的 SQL 在 "Orders" 表创建时为 "Id_P" 列创建 `FOREIGN KEY`：

```
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
FOREIGN KEY (Id_P) REFERENCES Persons(Id_P)
)
```

如果在 "Orders" 表已存在的情况下为 "Id_P" 列创建 `FOREIGN KEY` 约束，请使用下面的 SQL：

```
ALTER TABLE Orders
ADD FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
```

如需撤销 `FOREIGN KEY` 约束，请使用下面的 SQL：

```
ALTER TABLE Orders
DROP FOREIGN KEY fk_PerOrders
```

### CHECK 约束

CHECK 约束用于限制列中的值的范围
。
如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CHECK (Id_P>0)
)
```

### AUTO INCREMENT

Auto-increment 会在新记录插入表中时生成一个唯一的数字。

默认地，`AUTO_INCREMENT` 的开始值是 1，每条新记录递增 1。

要让 `AUTO_INCREMENT` 序列以其他的值起始，请使用下列 SQL 语法：

```
CREATE TABLE Persons
(
P_Id int NOT NULL AUTO_INCREMENT,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
PRIMARY KEY (P_Id)
)
```

要在 "Persons" 表中插入新记录，我们不必为 "P_Id" 列规定值（会自动添加一个唯一的值）



### 基本命令

创建一个表：

```
CREATE TABLE CUSTOMERS(
   ID   INT              NOT NULL,
   NAME VARCHAR (20)     NOT NULL,
   AGE  INT              NOT NULL,
   ADDRESS  CHAR (25)  DEFAULT 'Sandnes',
   SALARY   DECIMAL (18, 2),       
   PRIMARY KEY (ID)
);
```

查看表的表头属性：

```
SQL> DESC CUSTOMERS;
+---------+---------------+------+-----+---------+-------+
| Field   | Type          | Null | Key | Default | Extra |
+---------+---------------+------+-----+---------+-------+
| ID      | int(11)       | NO   | PRI |         |       |
| NAME    | varchar(20)   | NO   |     |         |       |
| AGE     | int(11)       | NO   |     |         |       |
| ADDRESS | char(25)      | YES  |     | NULL    |       |
| SALARY  | decimal(18,2) | YES  |     | NULL    |       |
+---------+---------------+------+-----+---------+-------+
```

删除表：

```
DROP TABLE CUSTOMERS;
```

### CREATE INDEX 语句

`CREATE INDEX` 语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

您可以在表中创建索引，以便更加快速高效地查询数据。

用户无法看到索引，它们只能被用来加速搜索/查询。

**注释**：更新一个包含索引的表需要比更新一个没有索引的表更多的时间，这是由于索引本身也需要更新。因此，理想的做法是仅仅在常常被搜索的列（以及表）上面创建索引。

```
CREATE INDEX PersonIndex
ON Person (LastName, FirstName)
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

### TOP

TOP 子句用于规定要返回的记录的数目。

对于拥有数千条记录的大型表来说，TOP 子句是非常有用的。

**注释**：并非所有的数据库系统都支持 TOP 子句。

SQL Server 的语法：

```
SELECT TOP number|percent column_name(s) FROM table_name
```

MySQL 语法:

```
SELECT * FROM Persons LIMIT 5
```

### JOIN

SQL join 用于根据两个或多个表中的列之间的关系，从这些表中查询数据。

可以使用引用两个表：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons, Orders
WHERE Persons.Id_P = Orders.Id_P 
```

也可以使用 `JOIN`

```
FROM Persons
INNER JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```

`INNER JOIN` 关键字在表中存在至少一个匹配时返回行。如果 "Persons" 中的行在 "Orders" 中没有匹配，就不会列出这些行。

除了我们在上面的例子中使用的 INNER JOIN（内连接），我们还可以使用其他几种连接。
下面列出了您可以使用的 JOIN 类型，以及它们之间的差异。

* JOIN: 如果表中有至少一个匹配，则返回行(INNER JOIN 与 JOIN 是相同的)
* LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行
* RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行
* FULL JOIN: 只要其中一个表中存在匹配，就返回行

#### LEFT JOIN

`LEFT JOIN` 关键字会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行。

```
SELECT column_name(s)
FROM table_name1
LEFT JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```

#### RIGHT JOIN

`RIGHT JOIN` 关键字会右表 (table_name2) 那里返回所有的行，即使在左表 (table_name1) 中没有匹配的行。

```
SELECT column_name(s)
FROM table_name1
RIGHT JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```

#### FULL JOIN

只要其中某个表存在匹配，FULL JOIN 关键字就会返回行。

`FULL JOIN` 关键字会从左表 (Persons) 和右表 (Orders) 那里返回所有的行。如果 "Persons" 中的行在表 "Orders" 中没有匹配，或者如果 "Orders" 中的行在表 "Persons" 中没有匹配，这些行同样会列出。

```
SELECT column_name(s)
FROM table_name1
FULL JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```

### UNION 和 UNION ALL

`UNION` 操作符用于合并两个或多个 `SELECT` 语句的结果集。
请注意，`UNION` 内部的 `SELECT`语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

```
SELECT E_Name FROM Employees_China
UNION
SELECT E_Name FROM Employees_USA
```

这个命令无法列出在中国和美国的所有雇员。在上面的例子中，我们有两个名字相同的雇员，他们当中只有一个人被列出来了。`UNION` 命令只会选取不同的值。

```
SELECT column_name(s) FROM table_name1
UNION ALL
SELECT column_name(s) FROM table_name2
```

UNION ALL 命令和 UNION 命令几乎是等效的，不过 UNION ALL 命令会列出所有的值。

### SELECT INTO

**SQL SELECT INTO 语句可用于创建表的备份复件。**

SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。
SELECT INTO 语句常用于创建表的备份复件或者用于对记录进行存档。

```
SELECT column_name(s)
INTO new_table_name [IN externaldatabase] 
FROM old_tablename
```

举例：

```
SELECT *
INTO Persons IN 'Backup.mdb'
FROM Persons
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

### LIKE

LIKE 操作符用于在 WHERE 子句中搜索列中的指定模式。

举例：

我们希望从上面的 "Persons" 表中选取居住在以 "N" 开始的城市里的人：

```
SELECT * FROM Persons WHERE City LIKE 'N%'
```
"%" 可用于定义通配符（模式中缺少的字母）。


我们希望从 "Persons" 表中选取居住在以 "g" 结尾的城市里的人：

```
SELECT * FROM Persons WHERE City LIKE '%g'
```

我们希望从 "Persons" 表中选取居住在包含 "lon" 的城市里的人：

```
SELECT * FROM Persons WHERE City LIKE '%lon%'
```

#### SQL 通配符

SQL 通配符必须与 LIKE 运算符一起使用。

```
通配符	                 描述
%	              替代一个或多个字符

_	                仅替代一个字符

[charlist]	     字符列中的任何单一字符

[^charlist]
或者
[!charlist]     不在字符列中的任何单一字符
```

### IN

IN 操作符允许我们在 WHERE 子句中规定多个值。

```
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1,value2,...)
```

### BETWEEN

BETWEEN 操作符在 WHERE 子句中使用，作用是选取介于两个值之间的数据范围。

操作符 `BETWEEN ... AND` 会选取介于两个值之间的数据范围。这些值可以是数值、文本或者日期。

```
SELECT column_name(s)
FROM table_name
WHERE column_name
BETWEEN value1 AND value2
```

举例：

如需以字母顺序显示介于 "Adams"（包括）和 "Carter"（不包括）之间的人，请使用下面的 SQL：

```
SELECT * FROM Persons
WHERE LastName
BETWEEN 'Adams' AND 'Carter'
```
**重要事项**：不同的数据库对 `BETWEEN...AND` 操作符的处理方式是有差异的。某些数据库会列出介于 "Adams" 和 "Carter" 之间的人，但不包括 "Adams" 和 "Carter" ；某些数据库会列出介于 "Adams" 和 "Carter" 之间并包括 "Adams" 和 "Carter" 的人；而另一些数据库会列出介于 "Adams" 和 "Carter" 之间的人，包括 "Adams" ，但不包括 "Carter" 。

如需使用上面的例子显示范围之外的人，请使用 `NOT` 操作符：

```
SELECT * FROM Persons
WHERE LastName
NOT BETWEEN 'Adams' AND 'Carter'
```


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

填充使用另一个表中的一个表：

```
INSERT INTO first_table_name [(column1, column2, ... columnN)] 
    SELECT column1, column2, ...columnN FROM second_table_name [WHERE condition];
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

## Alias（别名）

通过使用 SQL，可以为列名称和表名称指定别名（Alias）。

### 表的 SQL Alias 语法

```
SELECT column_name(s)
FROM table_name
AS alias_name
```

### 列的 SQL Alias 语法

```
SELECT column_name AS alias_name
FROM table_name
```

举例：

```
SELECT po.OrderID, p.LastName, p.FirstName
FROM Persons AS p, Product_Orders AS po
WHERE p.LastName='Adams' AND p.FirstName='John'
```

## VIEW（视图）

视图是一个虚拟表，其内容由查询定义。同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。对其中所引用的基础表来说，视图的作用类似于筛选。

在 SQL 中，视图是基于 SQL 语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实的表中的字段。我们可以向视图添加 SQL 函数、`WHERE` 以及 `JOIN` 语句，我们也可以提交数据，就像这些来自于某个单一的表。

**SQL CREATE VIEW 语法**

```
CREATE VIEW view_name AS
SELECT column_name(s)
FROM table_name
WHERE condition
```

**注释**：视图总是显示最近的数据。每当用户查询视图时，数据库引擎通过使用 SQL 语句来重建数据。
















































