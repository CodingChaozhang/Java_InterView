## 四、常见SQL语句书写

## 4.1 SQL语句主要分为哪几类？

**数据定义语言DDL(Data Defination Language)：**主要为**create** **drop** **alter**等操作，即对逻辑结构等有操作的，其中包括表结构，视图和索引。

**数据查询语言DQL(Data Query Language)：**主要为select操作，即以select关键字的各种简单查询，连接查询都属于DQL。

**数据操纵语言DML(Data Manipulation Language)：**主要为**insert**，**update**，**delete**等操作。其中DQL和DML共同组建了常用的增删改查操作。

## 4.2 初级SQL语言

### 4.2.1 SQL-select

**1.SQL-select语法**

```
SELECT 列名称 FROM 表名称
```

以及：

```
SELECT * FROM 表名称
```

**注释：**SQL 语句对大小写不敏感。SELECT 等效于 select。

**2.SQL-select实例**

实例1：如需获取名为 "LastName" 和 "FirstName" 的列的内容（从名为 "Persons" 的数据库表），请使用类似这样的 SELECT 语句：

```
SELECT LastName,FirstName FROM Persons
```

**"Persons" 表:**

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

**结果：**

| LastName | FirstName |
| :------- | :-------- |
| Adams    | John      |
| Bush     | George    |
| Carter   | Thomas    |

**实例2：**现在我们希望从 "Persons" 表中选取所有的列。

请使用符号 * 取代列的名称，就像这样：

```
SELECT * FROM Persons
```

**提示：**星号（*）是选取所有列的快捷方式。

**结果：**

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

### 4.2.2 SQL-select-distinct

在表中，可能会包含重复值，有时候我们希望仅仅列出不同(distinct)的值。关键词DISTINCT用于返回唯一不同的值。

**1.SQL-select-distinct语法**

```
SELECT DISTINCT 列名称 FROM 表名称
```

**2.SQL-select-distinct实例**

如需从 Company" 列中仅选取唯一不同的值，我们需要使用 SELECT DISTINCT 语句：

```
SELECT DISTINCT Company FROM Orders 
```

**"Orders"表：**

| Company  | OrderNumber |
| :------- | :---------- |
| IBM      | 3532        |
| W3School | 2356        |
| Apple    | 4698        |
| W3School | 6953        |

**结果：**

| Company  |
| :------- |
| IBM      |
| W3School |
| Apple    |

现在，在结果集中，"W3School" 仅被列出了一次。

### 4.2.3 SQL-select-where

如需要有条件地从表中选取数据，可将WHERE子句添加到SELECT语句中。

**1.SQL-select-where语法**

```
SELECT 列名称 FROM 表名称 WHERE 列 运算符 值
```

下面的运算符可在 WHERE 子句中使用：

| 操作符  | 描述         |
| :------ | :----------- |
| =       | 等于         |
| <>      | 不等于       |
| >       | 大于         |
| <       | 小于         |
| >=      | 大于等于     |
| <=      | 小于等于     |
| BETWEEN | 在某个范围内 |
| LIKE    | 搜索某种模式 |

**注释：**在某些版本的 SQL 中，操作符 <> 可以写为 !=。

**2.SQL-select-where实例**

**实例1：**

如果只希望选取居住在城市 "Beijing" 中的人，我们需要向 SELECT 语句添加 WHERE 子句：

```
SELECT * FROM Persons WHERE City='Beijing'
```

**"Persons" 表**

| LastName | FirstName | Address        | City     | Year |
| :------- | :-------- | :------------- | :------- | :--- |
| Adams    | John      | Oxford Street  | London   | 1970 |
| Bush     | George    | Fifth Avenue   | New York | 1975 |
| Carter   | Thomas    | Changan Street | Beijing  | 1980 |
| Gates    | Bill      | Xuanwumen 10   | Beijing  | 1985 |

**结果：**

| LastName | FirstName | Address        | City    | Year |
| :------- | :-------- | :------------- | :------ | :--- |
| Carter   | Thomas    | Changan Street | Beijing | 1980 |
| Gates    | Bill      | Xuanwumen 10   | Beijing | 1985 |

**3.引号的使用**

请注意，我们在例子中的条件值周围使用的是单引号。

SQL 使用单引号来环绕*文本值*（大部分数据库系统也接受双引号）。如果是*数值*，请不要使用引号。

**文本值：**

```
这是正确的：
SELECT * FROM Persons WHERE FirstName='Bush'

这是错误的：
SELECT * FROM Persons WHERE FirstName=Bush
```

**数值：**

```
这是正确的：
SELECT * FROM Persons WHERE Year>1965

这是错误的：
SELECT * FROM Persons WHERE Year>'1965'
```

### 4.2.4 SQL-AND & OR

AND和OR运算符用于基于一个以上的条件对记录进行过滤。

**1.AND 和 OR 运算符**

AND 和 OR 可在 WHERE 子语句中把两个或多个条件结合起来。

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

**2.AND运算符实例**

| LastName | FirstName | Address        | City     |
| :------- | :-------- | :------------- | :------- |
| Adams    | John      | Oxford Street  | London   |
| Bush     | George    | Fifth Avenue   | New York |
| Carter   | Thomas    | Changan Street | Beijing  |
| Carter   | William   | Xuanwumen 10   | Beijing  |

使用 AND 来显示所有姓为 "Carter" 并且名为 "Thomas" 的人：

```
SELECT * FROM Persons WHERE FirstName='Thomas' AND LastName='Carter'
```

**结果：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |

**3.OR运算符实例**

使用 OR 来显示所有姓为 "Carter" 或者名为 "Thomas" 的人：

```
SELECT * FROM Persons WHERE firstname='Thomas' OR lastname='Carter'
```

**结果：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |
| Carter   | William   | Xuanwumen 10   | Beijing |

### 4.2.5 结合AND和OR运算符的实例

我们也可以把 AND 和 OR 结合起来（使用圆括号来组成复杂的表达式）:

```
SELECT * FROM Persons WHERE (FirstName='Thomas' OR FirstName='William')
AND LastName='Carter'
```

**结果：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |
| Carter   | William   | Xuanwumen 10   | Beijing |

### 4.2.6 SQL ORDER BY

ORDER BY语句用于根据指定的列对结果集进行排序。

ORDER BY语句默认按照升序对记录进行排序。

如果希望按照降序对记录进行排序，可以使用DESC关键字。

**1.实例**

Orders 表:

| Company  | OrderNumber |
| :------- | :---------- |
| IBM      | 3532        |
| W3School | 2356        |
| Apple    | 4698        |
| W3School | 6953        |

实例1：以字母顺序显示公司名称：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company
```

**结果：**

| Company  | OrderNumber |
| :------- | :---------- |
| Apple    | 4698        |
| IBM      | 3532        |
| W3School | 6953        |
| W3School | 2356        |

实例2：以字母顺序显示公司名称（Company），并以数字顺序显示顺序号（OrderNumber）：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company, OrderNumber
```

结果：

| Company  | OrderNumber |
| :------- | :---------- |
| Apple    | 4698        |
| IBM      | 3532        |
| W3School | 2356        |
| W3School | 6953        |

实例3：以逆字母顺序显示公司名称

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC
```

**结果：**

| Company  | OrderNumber |
| :------- | :---------- |
| W3School | 6953        |
| W3School | 2356        |
| IBM      | 3532        |
| Apple    | 4698        |

实例4：以逆字母顺序显示公司名称，并以数字顺序显示顺序号：

```
SELECT Company, OrderNumber FROM Orders ORDER BY Company DESC, OrderNumber ASC
```

**结果：**

| Company  | OrderNumber |
| :------- | :---------- |
| W3School | 2356        |
| W3School | 6953        |
| IBM      | 3532        |
| Apple    | 4698        |

### 4.2.7 SQL-insert-into

insert into语句用于向表格中插入新的行。

**1. insert into语法**

```
INSERT INTO 表名称 VALUES (值1, 值2,....)
```

我们也可以指定所要插入数据的列：

```
INSERT INTO table_name (列1, 列2,...) VALUES (值1, 值2,....)
```

**2. insert into实例**

"Persons" 表：

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |

SQL 语句：

```
INSERT INTO Persons VALUES ('Gates', 'Bill', 'Xuanwumen 10', 'Beijing')
```

结果：

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |
| Gates    | Bill      | Xuanwumen 10   | Beijing |

**3. insert into实例2**

在指定的列中插入数据。

**"Persons" 表：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |
| Gates    | Bill      | Xuanwumen 10   | Beijing |

**SQL 语句：**

```
INSERT INTO Persons (LastName, Address) VALUES ('Wilson', 'Champs-Elysees')
```

**结果：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Carter   | Thomas    | Changan Street | Beijing |
| Gates    | Bill      | Xuanwumen 10   | Beijing |
| Wilson   |           | Champs-Elysees |         |

### 4.2.8 SQL-update

update语句用于修改表中的数据。

**1.update语法**

```
UPDATE 表名称 SET 列名称 = 新值 WHERE 列名称 = 某值
```

**2.实例**

**Person:**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Gates    | Bill      | Xuanwumen 10   | Beijing |
| Wilson   |           | Champs-Elysees |         |

**(1)实例1-更新某一行中的一个列**

我们为 lastname 是 "Wilson" 的人添加 firstname：

```
UPDATE Person SET FirstName = 'Fred' WHERE LastName = 'Wilson' 
```

**结果：**

| LastName | FirstName | Address        | City    |
| :------- | :-------- | :------------- | :------ |
| Gates    | Bill      | Xuanwumen 10   | Beijing |
| Wilson   | Fred      | Champs-Elysees |         |

**(2)实例2-更新某一行中的若干列**

修改地址（address），并添加城市名称（city）：

```
UPDATE Person SET Address = 'Zhongshan 23', City = 'Nanjing'
WHERE LastName = 'Wilson'
```

**结果：**

| LastName | FirstName | Address      | City    |
| :------- | :-------- | :----------- | :------ |
| Gates    | Bill      | Xuanwumen 10 | Beijing |
| Wilson   | Fred      | Zhongshan 23 | Nanjing |

### 4.2.9 SQL-delete

delete语句删除表中的行。

**1.delete语法**

```
DELETE FROM 表名称 WHERE 列名称 = 值
```

**2.delete实例**

**Person:**

| LastName | FirstName | Address      | City    |
| :------- | :-------- | :----------- | :------ |
| Gates    | Bill      | Xuanwumen 10 | Beijing |
| Wilson   | Fred      | Zhongshan 23 | Nanjing |

**(1) 删除某行**

"Fred Wilson" 会被删除：

```
DELETE FROM Person WHERE LastName = 'Wilson' 
```

**结果:**

| LastName | FirstName | Address      | City    |
| :------- | :-------- | :----------- | :------ |
| Gates    | Bill      | Xuanwumen 10 | Beijing |

**(2) 删除所有行**

可以在不删除表的情况下删除所有的行。这意味着表的结构、属性和索引都是完整的：

```
DELETE FROM table_name
```

或者：

```
DELETE * FROM table_name
```

## 4.3 高级SQL语言

### 4.3.1 SQL TOP

从表中选取头两条记录。

```
select top 2 * from persons
```

从表中选取50%的记录

```
SELECT TOP 50 PERCENT * FROM Persons
```

### 4.3.2 SQL LIKE

LIKE操作符用于在where子句中搜索列中的指定模式。

Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

**例子1**

从“Persons”表中选取居住在以“N”开始的城市里的人

```
SELECT * FROM Persons
WHERE City LIKE 'N%'
```

**提示：**"%" 可用于定义通配符（模式中缺少的字母）。

**例子2**

从 "Persons" 表中选取居住在以 "g" 结尾的城市里的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE '%g'
```

**例子3**

从 "Persons" 表中选取居住在包含 "lon" 的城市里的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE '%lon%'
```

**例子4**

使用 NOT 关键字，我们可以从 "Persons" 表中选取居住在*不包含* "lon" 的城市里的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City NOT LIKE '%lon%'
```

### 4.3.3 SQL 通配符

SQL 通配符必须与 LIKE 运算符一起使用。

在 SQL 中，可使用以下通配符：

| 通配符                     | 描述                       |
| :------------------------- | :------------------------- |
| %                          | **替代一个或多个字符**     |
| _                          | **仅替代一个字符**         |
| [charlist]                 | 字符列中的任何单一字符     |
| [^charlist]或者[!charlist] | 不在字符列中的任何单一字符 |



Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

**使用%通配符的实例**

从上面的 "Persons" 表中选取居住在以 "Ne" 开始的城市里的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE 'Ne%'
```

从 "Persons" 表中选取居住在包含 "lond" 的城市里的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE '%lond%'
```

**使用_通配符**

从上面的 "Persons" 表中选取名字的第一个字符之后是 "eorge" 的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE FirstName LIKE '_eorge'
```

从 "Persons" 表中选取的这条记录的姓氏以 "C" 开头，然后是一个任意字符，然后是 "r"，然后是任意字符，然后是 "er"：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE LastName LIKE 'C_r_er'
```

**使用[charlist]通配符**

从上面的 "Persons" 表中选取居住的城市以 "A" 或 "L" 或 "N" 开头的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE '[ALN]%'
```

从上面的 "Persons" 表中选取居住的城市*不以* "A" 或 "L" 或 "N" 开头的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE City LIKE '[!ALN]%'
```

### 4.3.4 SQL IN

IN操作符允许我们在WHERE子句中规定多个值。

Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

**实例**

从上表中选取姓氏为 Adams 和 Carter 的人：

我们可以使用下面的 SELECT 语句：

```
SELECT * FROM Persons
WHERE LastName IN ('Adams','Carter')
```

**结果集：**

| Id   | LastName | FirstName | Address        | City    |
| :--- | :------- | :-------- | :------------- | :------ |
| 1    | Adams    | John      | Oxford Street  | London  |
| 3    | Carter   | Thomas    | Changan Street | Beijing |

### 4.3.5 SQL BETWEEN

BETWEEN操作符在WHERE子句中使用，作用是选取介于两个值之间的数据范围。

Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |
| 4    | Gates    | Bill      | Xuanwumen 10   | Beijing  |

**实例**

如需以字母顺序显示介于 "Adams"（包括）和 "Carter"（不包括）之间的人，请使用下面的 SQL：

```
SELECT * FROM Persons
WHERE LastName
BETWEEN 'Adams' AND 'Carter'
```

如需使用上面的例子显示范围之外的人，请使用 NOT 操作符：

```
SELECT * FROM Persons
WHERE LastName
NOT BETWEEN 'Adams' AND 'Carter'
```

### 4.3.6 SQL Alias(别名)

通过使用 SQL，可以为列名称和表名称指定别名（Alias）。

**表的 SQL Alias 语法**

```
SELECT column_name(s)
FROM table_name
AS alias_name
```

**列的 SQL Alias 语法**

```
SELECT column_name AS alias_name
FROM table_name
```

**Alias实例：使用表名称别名**

假设我们有两个表分别是："Persons" 和 "Product_Orders"。我们分别为它们指定别名 "p" 和 "po"。

现在，我们希望列出 "John Adams" 的所有定单。

我们可以使用下面的 SELECT 语句：

```
SELECT po.OrderID, p.LastName, p.FirstName
FROM Persons AS p, Product_Orders AS po
WHERE p.LastName='Adams' AND p.FirstName='John'
```

**Alias实例：使用列名别名**

```
SELECT LastName AS Family, FirstName AS Name
FROM Persons
```

### 4.3.7 SQL join

SQL join用于根据两个或多个表中的列之间的关系，从这些表中查询数据。

**引用两个表**

可以通过引用两个表的方式，从两个表中获取数据：

谁订购了产品，并且他们订购了什么产品？

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons, Orders
WHERE Persons.Id_P = Orders.Id_P 
```

**使用join**

可以使用关键词 JOIN 来从两个表中获取数据。

如果我们希望列出所有人的定购，可以使用下面的 SELECT 语句：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P = Orders.Id_P
ORDER BY Persons.LastName
```

下面列出了您可以使用的 JOIN 类型，以及它们之间的差异。

- **JOIN: 如果表中有至少一个匹配，则返回行**
- **LEFT JOIN: 即使右表中没有匹配，也从左表返回所有的行**
- **RIGHT JOIN: 即使左表中没有匹配，也从右表返回所有的行**
- **FULL JOIN: 只要其中一个表中存在匹配，就返回行**

#### 1.Inner Join

"Persons" 表：

| Id_P | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

"Orders" 表：

| Id_O | OrderNo | Id_P |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 1    |
| 4    | 24562   | 1    |
| 5    | 34764   | 65   |

**内连接INNER JOIN实例**

现在，我们希望列出所有人的定购。

您可以使用下面的 SELECT 语句：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

结果集：

| LastName | FirstName | OrderNo |
| :------- | :-------- | :------ |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |

INNER JOIN 关键字在表中存在至少一个匹配时返回行。如果 "Persons" 中的行在 "Orders" 中没有匹配，就不会列出这些行。

#### 2. LEFT JOIN

LEFT JOIN 关键字会从左表 (table_name1) 那里返回所有的行，即使在右表 (table_name2) 中没有匹配的行。

"Persons" 表：

| Id_P | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

"Orders" 表：

| Id_O | OrderNo | Id_P |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 1    |
| 4    | 24562   | 1    |
| 5    | 34764   | 65   |

**LEFT JOIN实例**

现在，我们希望列出所有的人，以及他们的定购 - 如果有的话。

您可以使用下面的 SELECT 语句：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
LEFT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

结果集：

| LastName | FirstName | OrderNo |
| :------- | :-------- | :------ |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
| Bush     | George    |         |

LEFT JOIN 关键字会从左表 (Persons) 那里返回所有的行，即使在右表 (Orders) 中没有匹配的行。

#### 3. SQL RIGHT JOIN

RIGHT JOIN 关键字会右表 (table_name2) 那里返回所有的行，即使在左表 (table_name1) 中没有匹配的行。

"Persons" 表：

| Id_P | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

"Orders" 表：

| Id_O | OrderNo | Id_P |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 1    |
| 4    | 24562   | 1    |
| 5    | 34764   | 65   |

**右连接(RIGHT JOIN)实例**

现在，我们希望列出所有的定单，以及定购它们的人 - 如果有的话。

您可以使用下面的 SELECT 语句：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
RIGHT JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

结果集：

| LastName | FirstName | OrderNo |
| :------- | :-------- | :------ |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
|          |           | 34764   |

RIGHT JOIN 关键字会从右表 (Orders) 那里返回所有的行，即使在左表 (Persons) 中没有匹配的行。

#### 4.SQL FULL JOIN

只要其中某个表存在匹配，FULL JOIN 关键字就会返回行。

"Persons" 表：

| Id_P | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

"Orders" 表：

| Id_O | OrderNo | Id_P |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 1    |
| 4    | 24562   | 1    |
| 5    | 34764   | 65   |

现在，我们希望列出所有的人，以及他们的定单，以及所有的定单，以及定购它们的人。

您可以使用下面的 SELECT 语句：

```
SELECT Persons.LastName, Persons.FirstName, Orders.OrderNo
FROM Persons
FULL JOIN Orders
ON Persons.Id_P=Orders.Id_P
ORDER BY Persons.LastName
```

结果集：

| LastName | FirstName | OrderNo |
| :------- | :-------- | :------ |
| Adams    | John      | 22456   |
| Adams    | John      | 24562   |
| Carter   | Thomas    | 77895   |
| Carter   | Thomas    | 44678   |
| Bush     | George    |         |
|          |           | 34764   |

### 4.3.8 SQL UNION

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。

**Employees_China:**

| E_ID | E_Name         |
| :--- | :------------- |
| 01   | Zhang, Hua     |
| 02   | Wang, Wei      |
| 03   | Carter, Thomas |
| 04   | Yang, Ming     |

**Employees_USA:**

| E_ID | E_Name         |
| :--- | :------------- |
| 01   | Adams, John    |
| 02   | Bush, George   |
| 03   | Carter, Thomas |
| 04   | Gates, Bill    |

**使用UNION命令的实例**

实例

列出所有在中国和美国的不同的雇员名：

```
SELECT E_Name FROM Employees_China
UNION
SELECT E_Name FROM Employees_USA
```

结果

| E_Name         |
| :------------- |
| Zhang, Hua     |
| Wang, Wei      |
| Carter, Thomas |
| Yang, Ming     |
| Adams, John    |
| Bush, George   |
| Gates, Bill    |

**注释：**这个命令无法列出在中国和美国的所有雇员。在上面的例子中，我们有两个名字相同的雇员，他们当中只有一个人被列出来了。UNION 命令只会选取不同的值

**使用UNION ALL实例**

实例：

列出在中国和美国的所有的雇员：

```
SELECT E_Name FROM Employees_China
UNION ALL
SELECT E_Name FROM Employees_USA
```

结果

| E_Name         |
| :------------- |
| Zhang, Hua     |
| Wang, Wei      |
| Carter, Thomas |
| Yang, Ming     |
| Adams, John    |
| Bush, George   |
| Carter, Thomas |
| Gates, Bill    |

### 4.3.9 SQL SELECT INTO

SQL SELECT INTO语句可用于创建表的备份复件。

SELECT INTO 语句从一个表中选取数据，然后把数据插入另一个表中。

SELECT INTO 语句常用于创建表的备份复件或者用于对记录进行存档。



**SQL SELECT INTO实例-制作备份复件**

下面的例子会制作 "Persons" 表的备份复件：

```
SELECT *
INTO Persons_backup
FROM Persons
```

**SQL SELECT INTO 实例 - 带有 WHERE 子句**

下面的例子通过从 "Persons" 表中提取居住在 "Beijing" 的人的信息，创建了一个带有两个列的名为 "Persons_backup" 的表：

```
SELECT LastName,Firstname
INTO Persons_backup
FROM Persons
WHERE City='Beijing'
```

**SQL SELECT INTO 实例 - 被连接的表**

下面的例子会创建一个名为 "Persons_Order_Backup" 的新表，其中包含了从 Persons 和 Orders 两个表中取得的信息：

```
SELECT Persons.LastName,Orders.OrderNo
INTO Persons_Order_Backup
FROM Persons
INNER JOIN Orders
ON Persons.Id_P=Orders.Id_P
```

### 4.3.10 SQL CREATE DB

CREATE DATABASE 用于创建数据库。

现在我们希望创建一个名为 "my_db" 的数据库。

我们使用下面的 CREATE DATABASE 语句：

```
CREATE DATABASE my_db
```

可以通过 CREATE TABLE 来添加数据库表。

### 4.3.11 SQL CREATE TABLE

CREATE TABLE 语句用于创建数据库中的表。

本例演示如何创建名为 "Person" 的表。

该表包含 5 个列，列名分别是："Id_P"、"LastName"、"FirstName"、"Address" 以及 "City"：

```
CREATE TABLE Persons
(
Id_P int,
LastName varchar(255),
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

### 4.3.12 SQL 约束(Constraints)

约束用于限制加入表的数据的类型。

可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

我们将主要探讨以下几种约束：

- NOT NULL
- UNIQUE
- PRIMARY KEY
- FOREIGN KEY
- CHECK
- DEFAULT

#### 1.SQL NOT NULL约束

NOT NULL 约束强制列不接受 NULL 值。

NOT NULL 约束强制字段始终包含值。这意味着，如果不向字段添加值，就无法插入新记录或者更新记录。

下面的 SQL 语句强制 "Id_P" 列和 "LastName" 列不接受 NULL 值：

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

#### 2.SQL UNIQUE约束

UNIQUE 约束唯一标识数据库表中的每条记录。

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束。

下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 UNIQUE 约束：

**MySQL:**

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

**SQL Server / Oracle / MS Access:**

```
CREATE TABLE Persons
(
Id_P int NOT NULL UNIQUE,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 UNIQUE 约束，以及为多个列定义 UNIQUE 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

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



当表已被创建时，如需在 "Id_P" 列创建 UNIQUE 约束，请使用下列 SQL：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD UNIQUE (Id_P)
```

如需命名 UNIQUE 约束，并定义多个列的 UNIQUE 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD CONSTRAINT uc_PersonID UNIQUE (Id_P,LastName)
```

如需撤销 UNIQUE 约束，请使用下面的 SQL：

**MySQL:**

```
ALTER TABLE Persons
DROP INDEX uc_PersonID
```

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
DROP CONSTRAINT uc_PersonID
```

#### 3.SQL PRIMARY KEY约束

PRIMARY KEY 约束唯一标识数据库表中的每条记录。

**主键必须包含唯一的值。**

**主键列不能包含 NULL 值。**

每个表都应该有一个主键，并且每个表只能有一个主键。

**实例**

下面的 SQL 在 "Persons" 表创建时在 "Id_P" 列创建 PRIMARY KEY 约束：

**MySQL:**

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

**SQL Server / Oracle / MS Access:**

```
CREATE TABLE Persons
(
Id_P int NOT NULL PRIMARY KEY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

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

如果在表已存在的情况下为 "Id_P" 列创建 PRIMARY KEY 约束，请使用下面的 SQL：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD PRIMARY KEY (Id_P)
```

如果需要命名 PRIMARY KEY 约束，以及为多个列定义 PRIMARY KEY 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD CONSTRAINT pk_PersonID PRIMARY KEY (Id_P,LastName)
```

如需撤销 PRIMARY KEY 约束，请使用下面的 SQL：

**MySQL:**

```
ALTER TABLE Persons
DROP PRIMARY KEY
```

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
DROP CONSTRAINT pk_PersonID
```

#### 4.SQL FOREIGN KEY约束

一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。

让我们通过一个例子来解释外键。请看下面两个表：

"Persons" 表：

| Id_P | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

"Orders" 表：

| Id_O | OrderNo | Id_P |
| :--- | :------ | :--- |
| 1    | 77895   | 3    |
| 2    | 44678   | 3    |
| 3    | 22456   | 1    |
| 4    | 24562   | 1    |

请注意，"Orders" 中的 "Id_P" 列指向 "Persons" 表中的 "Id_P" 列。

"Persons" 表中的 "Id_P" 列是 "Persons" 表中的 PRIMARY KEY。

"Orders" 表中的 "Id_P" 列是 "Orders" 表中的 FOREIGN KEY。

FOREIGN KEY 约束用于预防破坏表之间连接的动作。

FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一。

**实例**

下面的 SQL 在 "Orders" 表创建时为 "Id_P" 列创建 FOREIGN KEY：

**MySQL:**

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

**SQL Server / Oracle / MS Access:**

```
CREATE TABLE Orders
(
Id_O int NOT NULL PRIMARY KEY,
OrderNo int NOT NULL,
Id_P int FOREIGN KEY REFERENCES Persons(Id_P)
)
```

如果需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
CREATE TABLE Orders
(
Id_O int NOT NULL,
OrderNo int NOT NULL,
Id_P int,
PRIMARY KEY (Id_O),
CONSTRAINT fk_PerOrders FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
)
```

如果在 "Orders" 表已存在的情况下为 "Id_P" 列创建 FOREIGN KEY 约束，请使用下面的 SQL：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Orders
ADD FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
```

如果需要命名 FOREIGN KEY 约束，以及为多个列定义 FOREIGN KEY 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Orders
ADD CONSTRAINT fk_PerOrders
FOREIGN KEY (Id_P)
REFERENCES Persons(Id_P)
```

如需撤销 FOREIGN KEY 约束，请使用下面的 SQL：

**MySQL:**

```
ALTER TABLE Orders
DROP FOREIGN KEY fk_PerOrders
```

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Orders
DROP CONSTRAINT fk_PerOrders
```

#### 5.SQL CHECK 约束

CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。

下面的 SQL 在 "Persons" 表创建时为 "Id_P" 列创建 CHECK 约束。CHECK 约束规定 "Id_P" 列必须只包含大于 0 的整数。

**My SQL:**

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

**SQL Server / Oracle / MS Access:**

```
CREATE TABLE Persons
(
Id_P int NOT NULL CHECK (Id_P>0),
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255),
CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
)
```

如果在表已存在的情况下为 "Id_P" 列创建 CHECK 约束，请使用下面的 SQL：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD CHECK (Id_P>0)
```

如果需要命名 CHECK 约束，以及为多个列定义 CHECK 约束，请使用下面的 SQL 语法：

**MySQL / SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ADD CONSTRAINT chk_Person CHECK (Id_P>0 AND City='Sandnes')
```

如需撤销 CHECK 约束，请使用下面的 SQL：

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
DROP CONSTRAINT chk_Person
```

**MySQL:**

```
ALTER TABLE Persons
DROP CHECK chk_Person
```

#### 6.SQL DEFAULT

DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

下面的 SQL 在 "Persons" 表创建时为 "City" 列创建 DEFAULT 约束：

**My SQL / SQL Server / Oracle / MS Access:**

```
CREATE TABLE Persons
(
Id_P int NOT NULL,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255) DEFAULT 'Sandnes'
)
```

如果在表已存在的情况下为 "City" 列创建 DEFAULT 约束，请使用下面的 SQL：

**MySQL:**

```
ALTER TABLE Persons
ALTER City SET DEFAULT 'SANDNES'
```

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ALTER COLUMN City SET DEFAULT 'SANDNES'
```

如需撤销 DEFAULT 约束，请使用下面的 SQL：

**MySQL:**

```
ALTER TABLE Persons
ALTER City DROP DEFAULT
```

**SQL Server / Oracle / MS Access:**

```
ALTER TABLE Persons
ALTER COLUMN City DROP DEFAULT
```

###  4.3.14 SQL CREATE INDEX

CREATE INDEX语句用于在表中创建索引。

在不读取整个表的情况下，索引使数据库应用程序可以更快地查找数据。

**CREATE INDEX 实例**

本例会创建一个简单的索引，名为 "PersonIndex"，在 Person 表的 LastName 列：

```
CREATE INDEX PersonIndex
ON Person (LastName) 
```

如果您希望以*降序*索引某个列中的值，您可以在列名称之后添加保留字 *DESC*：

```
CREATE INDEX PersonIndex
ON Person (LastName DESC) 
```

假如您希望索引不止一个列，您可以在括号中列出这些列的名称，用逗号隔开：

```
CREATE INDEX PersonIndex
ON Person (LastName, FirstName)
```

### 4.3.15 SQL Drop

可以使用 DROP INDEX 命令删除表格中的索引。

**用于 Microsoft SQLJet (以及 Microsoft Access) 的语法:**

```
DROP INDEX index_name ON table_name
```

**用于 MS SQL Server 的语法:**

```
DROP INDEX table_name.index_name
```

**用于 IBM DB2 和 Oracle 语法:**

```
DROP INDEX index_name
```

**用于 MySQL 的语法:**

```
ALTER TABLE table_name DROP INDEX index_name
```

**SQL DROP TABLE 语句**

DROP TABLE 语句用于删除表（表的结构、属性以及索引也会被删除）：

```
DROP TABLE 表名称
```

**SQL DROP DATABASE 语句**

DROP DATABASE 语句用于删除数据库：

```
DROP DATABASE 数据库名称
```

**SQL TRUNCATE TABLE 语句**

如果我们仅仅需要除去表内的数据，但并不删除表本身，那么我们该如何做呢？

请使用 TRUNCATE TABLE 命令（仅仅删除表格中的数据）：

```
TRUNCATE TABLE 表名称
```

### 4.3.16 SQL ALTER TABLE

ALTER TABLE 语句用于在已有的表中添加、修改或删除列。

Persons 表:

| Id   | LastName | FirstName | Address        | City     |
| :--- | :------- | :-------- | :------------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |
| 2    | Bush     | George    | Fifth Avenue   | New York |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |

**SQL ALTER TABLE 实例**

现在，我们希望在表 "Persons" 中添加一个名为 "Birthday" 的新列。

我们使用下列 SQL 语句：

```
ALTER TABLE Persons
ADD Birthday date
```

请注意，新列 "Birthday" 的类型是 date，可以存放日期。数据类型规定列中可以存放的数据的类型。

新的 "Persons" 表类似这样：

| Id   | LastName | FirstName | Address        | City     | Birthday |
| :--- | :------- | :-------- | :------------- | :------- | :------- |
| 1    | Adams    | John      | Oxford Street  | London   |          |
| 2    | Bush     | George    | Fifth Avenue   | New York |          |
| 3    | Carter   | Thomas    | Changan Street | Beijing  |          |

现在我们希望改变 "Persons" 表中 "Birthday" 列的数据类型。

我们使用下列 SQL 语句：

```
ALTER TABLE Persons
ALTER COLUMN Birthday year
```

请注意，"Birthday" 列的数据类型是 year，可以存放 2 位或 4 位格式的年份。

接下来，我们删除 "Person" 表中的 "Birthday" 列：

```
ALTER TABLE Person
DROP COLUMN Birthday
```

### 4.3.17 SQL AUTO INCREMENT

Auto-Increment会在新纪录插入表时生成一个唯一的数字。

**用于 MySQL 的语法**

下列 SQL 语句把 "Persons" 表中的 "P_Id" 列定义为 auto-increment 主键：

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

MySQL 使用 AUTO_INCREMENT 关键字来执行 auto-increment 任务。

默认地，AUTO_INCREMENT 的开始值是 1，每条新记录递增 1。

**用于 SQL Server 的语法**

下列 SQL 语句把 "Persons" 表中的 "P_Id" 列定义为 auto-increment 主键：

```
CREATE TABLE Persons
(
P_Id int PRIMARY KEY IDENTITY,
LastName varchar(255) NOT NULL,
FirstName varchar(255),
Address varchar(255),
City varchar(255)
)
```

MS SQL 使用 IDENTITY 关键字来执行 auto-increment 任务。

默认地，IDENTITY 的开始值是 1，每条新记录递增 1。

### 4.3.18 SQL CREATE VIEW

视图是可视化的表。在SQL中，视图是基于SQL语句的结果集的可视化的表。

视图包含行和列，就像一个真实的表。视图中的字段就是来自一个或多个数据库中的真实表中的字段。我们可以向视图添加SQL函数，WHERE以及JOIN语句，也可以提交数据。

**SQL CREATE VIEW实例**

样本数据库 Northwind 拥有一些被默认安装的视图。视图 "Current Product List" 会从 Products 表列出所有正在使用的产品。这个视图使用下列 SQL 创建：

```
CREATE VIEW [Current Product List] AS
SELECT ProductID,ProductName
FROM Products
WHERE Discontinued=No
```

我们可以查询上面这个视图：

```
SELECT * FROM [Current Product List]
```

Northwind 样本数据库的另一个视图会选取 Products 表中所有单位价格高于平均单位价格的产品：

```
CREATE VIEW [Products Above Average Price] AS
SELECT ProductName,UnitPrice
FROM Products
WHERE UnitPrice>(SELECT AVG(UnitPrice) FROM Products) 
```

我们可以像这样查询上面这个视图：

```
SELECT * FROM [Products Above Average Price]
```

另一个来自 Northwind 数据库的视图实例会计算在 1997 年每个种类的销售总数。请注意，这个视图会从另一个名为 "Product Sales for 1997" 的视图那里选取数据：

```
CREATE VIEW [Category Sales For 1997] AS
SELECT DISTINCT CategoryName,Sum(ProductSales) AS CategorySales
FROM [Product Sales for 1997]
GROUP BY CategoryName 
```

我们可以像这样查询上面这个视图：

```
SELECT * FROM [Category Sales For 1997]
```

我们也可以向查询添加条件。现在，我们仅仅需要查看 "Beverages" 类的全部销量：

```
SELECT * FROM [Category Sales For 1997]
WHERE CategoryName='Beverages'
```

现在，我们希望向 "Current Product List" 视图添加 "Category" 列。我们将通过下列 SQL 更新视图：

```
CREATE VIEW [Current Product List] AS
SELECT ProductID,ProductName,Category
FROM Products
WHERE Discontinued=No
```

您可以通过 DROP VIEW 命令来删除视图。

```
DROP VIEW view_name
```

### 4.3.19 SQL日期

MySQL 使用下列数据类型在数据库中存储日期或日期/时间值：

- **DATE - 格式 YYYY-MM-DD**
- **DATETIME - 格式: YYYY-MM-DD HH:MM:SS**
- **TIMESTAMP - 格式: YYYY-MM-DD HH:MM:SS**
- **YEAR - 格式 YYYY 或 YY**

SQL Server 使用下列数据类型在数据库中存储日期或日期/时间值：

- DATE - 格式 YYYY-MM-DD
- DATETIME - 格式: YYYY-MM-DD HH:MM:SS
- SMALLDATETIME - 格式: YYYY-MM-DD HH:MM:SS
- TIMESTAMP - 格式: 唯一的数字

### 4.3.20 SQL Nulls

我们如何仅仅选取在 "Address" 列中带有 NULL 值的记录呢？

我们必须使用 IS NULL 操作符：

```
SELECT LastName,FirstName,Address FROM Persons
WHERE Address IS NULL
```

结果集：

| LastName | FirstName | Address |
| :------- | :-------- | :------ |
| Adams    | John      |         |
| Carter   | Thomas    |         |

**提示：**请始终使用 IS NULL 来查找 NULL 值。

### 4.3.21 SQL limit

```
select * from 表名 limit start=0,count
```

说明：

从start开始，获取count条数据；start默认值为0，即当用户需要获取数据的前n条的时候可以直接写上xxx limit n;



## 4.4 SQL常用函数

### 4.4.1 GROUP BY

GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

**实例**

我们拥有下面这个 "Orders" 表：

| O_Id | OrderDate  | OrderPrice | Customer |
| :--- | :--------- | :--------- | :------- |
| 1    | 2008/12/29 | 1000       | Bush     |
| 2    | 2008/11/23 | 1600       | Carter   |
| 3    | 2008/10/05 | 700        | Bush     |
| 4    | 2008/09/28 | 300        | Bush     |
| 5    | 2008/08/06 | 2000       | Adams    |
| 6    | 2008/07/21 | 100        | Carter   |

现在，我们希望查找每个客户的总金额（总订单）。

我们想要使用 GROUP BY 语句对客户进行组合。

我们使用下列 SQL 语句：

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
```

结果集类似这样：

| Customer | SUM(OrderPrice) |
| :------- | :-------------- |
| Bush     | 2000            |
| Carter   | 1700            |
| Adams    | 2000            |

### 4.4.2 HAVING

在SQL中增加HAVING子句的原因是，WHERE关键字无法与合计函数一起使用。

**实例**

我们拥有下面这个 "Orders" 表：

| O_Id | OrderDate  | OrderPrice | Customer |
| :--- | :--------- | :--------- | :------- |
| 1    | 2008/12/29 | 1000       | Bush     |
| 2    | 2008/11/23 | 1600       | Carter   |
| 3    | 2008/10/05 | 700        | Bush     |
| 4    | 2008/09/28 | 300        | Bush     |
| 5    | 2008/08/06 | 2000       | Adams    |
| 6    | 2008/07/21 | 100        | Carter   |

现在，我们希望查找订单总金额少于 2000 的客户。

我们使用如下 SQL 语句：

```
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

结果集类似：

| Customer | SUM(OrderPrice) |
| :------- | :-------------- |
| Carter   | 1700            |



现在我们希望查找客户 "Bush" 或 "Adams" 拥有超过 1500 的订单总金额。

我们在 SQL 语句中增加了一个普通的 WHERE 子句：

```
SELECT Customer,SUM(OrderPrice) FROM Orders
WHERE Customer='Bush' OR Customer='Adams'
GROUP BY Customer
HAVING SUM(OrderPrice)>1500
```

结果集：

| Customer | SUM(OrderPrice) |
| :------- | :-------------- |
| Bush     | 2000            |
| Adams    | 2000            |

## 4.3 超键、候选键、主键、外键分别是什么？

- **超键：**在关系中能唯一标识元组的属性集成为关系模式的超键。一个属性可以作为一个超键，多个属性组合在一起也可以作为一个超键。
- **候选键：**是最小的超键，即没有冗余元素的超键。
- **主键：**数据库表中对存储数据对象予以唯一和完整标识的数据列或属性的组合。一个数据列只能有一个主键，且主键的取值不能缺失，即不能为空值（NULL）。
- **外键：**在一个表中存在的另一个表的主键称为此表的外键。

**结合实例具体解释：**

假设有如下两个表：

学生（学号，姓名，性别，身份证号，教师编号）

教师（教师编号，姓名，工资）

**超键：**

由超键的定义可知，学生表中含有学号或者身份证号的任意组合都为此表的超键。如：（学号）、（学号，姓名）、（身份证号，性别）等。

**候选键：**

候选键属于超键，它是最小的超键，就是说如果再去掉候选键中的任何一个属性它就不再是超键了。学生表中的候选键为：（学号）、（身份证号）。

**主键：**

主键就是候选键里面的一个，是人为规定的，例如学生表中，我们通常会让“学号”做主键，教师表中让“教师编号”做主键。

**外键：**

外键比较简单，学生表中的外键就是“教师编号”。外键主要是用来描述两个表的关系。

## 4.4 SQL约束有几种？

- **NOT NULL：**用于控制字段的内容一定不能为空(NULL);
- **UNIQUE：**控制字段内容不能重复，一个表允许有多个Unique约束；
- **PRIMARY KEY：**用于控制字段内容不能重复，但它在一个表中只允许出现一个；
- **FOREIGN KEY：**用于预防破坏表之间连接的动作，也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一；
- **CHECK：**用于控制字段的值范围。

## 4.5 SQL的关联查询

**实例：**

T_A A表 T_B B标，id为表与表相关联的字段` 创建相关表结构

```
CREATE TABLE Table_B( id INT(2), serNum VARCHAR(10) ); 
CREATE TABLE Table_A( id INT(2), serNum VARCHAR(10) ); 
INSERT INTO table_a (id, serNum)
 VALUES (1,'A000101'),(2,'A000102'),(3,'A000103'),(5,'A000104'),(8,'A000105'),(4,'A000106');
INSERT INTO table_b (id, serNum) 
 VALUES (1,'B000201'),(2,'B000202'),(3,'B000203'),(6,'B000204'),(7,'B000205'),(9,'B000206');
```

Table_A id serNum

```
 1  A000100  
 2  A000102  
 3  A000103  
 5  A000104  
 8  A000105  
 4  A000106  
```

Table_B id serNum

```
 1  B000201  
 2  B000202  
 3  B000203  
 6  B000204  
 7  B000205  
 9  B000206  
```

### 4.5.1 inner join 内连接查询

![img](file://E:/%E7%AC%94%E8%AE%B0/JAVA/Java%E5%A4%8D%E4%B9%A0%E6%A1%86%E6%9E%B6-%E6%95%B0%E6%8D%AE%E5%BA%93/Mysql/imgs_mysql/4.png?lastModify=1616899593)

```
SELECT a.*,b.*
FROM table_a a
INNER JOIN table_b b
ON a.id=b.id
```

查询结果：    id    serNum               id  serNum

```
 1  A000100       1  B000201  
 2  A000102       2  B000202  
 3  A000103       3  B000203  
```

### 4.5.2 left join 左关联查询

以左表作为基础表去关联右表，查询的结果为左表的子集。

![img](file://E:/%E7%AC%94%E8%AE%B0/JAVA/Java%E5%A4%8D%E4%B9%A0%E6%A1%86%E6%9E%B6-%E6%95%B0%E6%8D%AE%E5%BA%93/Mysql/imgs_mysql/5.png?lastModify=1616899593)

```
SELECT a.*,b.*
FROM table_a a
LEFT JOIN table_b b
ON a.id=b.id
```

查询结果： id        serNum               id   serNum

```
 1  A000100       1  B000201  
 2  A000102       2  B000202  
 3  A000103       3  B000203  
 5  A000104      (NULL)  (NULL)   
 8  A000105      (NULL)  (NULL)   
 4  A000106      (NULL)  (NULL)  
```

### 4.5.3 right join 右关联查询

以右表作为基础表去关联左表，查询的结果作为右表的子集。

![img](file://E:/%E7%AC%94%E8%AE%B0/JAVA/Java%E5%A4%8D%E4%B9%A0%E6%A1%86%E6%9E%B6-%E6%95%B0%E6%8D%AE%E5%BA%93/Mysql/imgs_mysql/6.png?lastModify=1616899593)

```
SELECT a.*,b.*
FROM table_a a
RIGHT JOIN table_b b
ON a.id=b.id
```

查询结果：     id    serNum              id  serNum

```
 1  A000100       1  B000201  
 2  A000102       2  B000202  
 3  A000103       3  B000203  
(NULL) (NULL)     6 B000204
(NULL) (NULL)     7 B000205
(NULL) (NULL)     9 B000206
```

### 4.5.4 左连接-内连接

取左表的部分集合，但又不存在右表中。

![img](file://E:/%E7%AC%94%E8%AE%B0/JAVA/Java%E5%A4%8D%E4%B9%A0%E6%A1%86%E6%9E%B6-%E6%95%B0%E6%8D%AE%E5%BA%93/Mysql/imgs_mysql/7.png?lastModify=1616899593)

```
SELECT a.*,b.*
FROM table_a a
LEFT JOIN table_b b
ON a.id=b.id
WHERE b.id IS NULL
```

查询结果：     id    serNum       id           serNum

```
 5  A000104  (NULL)  (NULL)  
 8  A000105  (NULL)  (NULL)  
 4  A000106  (NULL)  (NULL)
```

### 4.5.5 右连接

取有表的部分数据，但又不存在左表中 ![img](file://E:/%E7%AC%94%E8%AE%B0/JAVA/Java%E5%A4%8D%E4%B9%A0%E6%A1%86%E6%9E%B6-%E6%95%B0%E6%8D%AE%E5%BA%93/Mysql/imgs_mysql/8.png?lastModify=1616899593)



```
SELECT a.*,b.*
FROM table_a a
RIGHT JOIN table_b b
ON a.id=b.id
WHERE a.id IS NULL
```

查询结果：     id    serNum       id       serNum

```
(NULL) (NULL) 6    B000204
(NULL) (NULL) 7    B000205
(NULL) (NULL) 9    B000206
```

## 4.6 子查询语句

- **条件：**一条SQL语句的查询结果作为另一条查询语句的条件或查询结果；
- **嵌套：**多条SQL语句嵌套使用，内部的SQL查询语句称为子查询；

**子查询的三种情况**

1. 子查询是单行单列的情况：结果集是一个值，父查询使用：=、 <、 > 等运算符

```
-- 查询工资最高的员工是谁？ 
select  * from employee where salary=(select max(salary) from employee);   

```

1. 子查询是多行多列的情况：结果集类似于一张虚拟表，不能用于where条件，用于select子句中做为子表

```
-- 1) 查询出2011年以后入职的员工信息
-- 2) 查询所有的部门信息，与上面的虚拟表中的信息比对，找出所有部门ID相等的员工。
select * from dept d,  (select * from employee where join_date > '2011-1-1') e where e.dept_id =  d.id;    

-- 使用表连接：
select d.*, e.* from  dept d inner join employee e on d.id = e.dept_id where e.join_date >  '2011-1-1'  

```

## 4.7 mysql中in和exists区别

MySQL中的in语言是把外表和内表做hash连接，而exists语句是对外表做loop循环，每次loop循环再对内表进行查询。一直大家都认为exists比in语句的效率要高，这种说法其实是不准确的，这个要区分环境的。

- **如果要查询的两个表大小相等，那么用in和exists差别不大；**
- **如果两个表中一个较小，一个是大表，那么子查询表大的用exists，子查询表小的用in；**
- not in 和not exists：如果查询语句使用了not in，那么内外表都进行全表扫描，没有用到索引；而not extsts的子查询依然能用到表上的索引。所以无论那个表大，用**not exists都比not in要快。**

## 4.8 varchar和char的区别(重点)

**char的特点：**

- **char表示定长字符串，长度是固定的；**
- 如果插入数据的长度小于char的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比varchar快很多，甚至能快50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于char来说，最多能存放的字符个数为255，和编码无关；

**varchar的特点：**

- **varchar表示可变长的字符串，长度是可变的；**
- 插入的数据是多长，就按照多长存储；
- varchar在存取方面与char相反，它存取速度慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- **对于varchar来说，最多能存放的字符个数为65532**

总之，结合性能角度(char更快)和节省磁盘空间角度(varchar更小)，具体情况还需具体来设计数据库才是妥当的做法。

## 4.9 mysql中int(10)和char(10)以及varchar(10)之间的区别

- **int(10)中的10表示显示的数据长度**，不是存储数据的大小；**char(10)和varchar(10)的10表示存储数据的大小**，即表示存储多少个字符、
- **char(10)表示存储定长的10个字符**，不足10个就用空格补齐，占用更多的存储空间
- **varchar(10)表示存储10个变长的字符**，存储多少个就是多少个，空格也按一个字符存储，这一点和char(10)的空格是不同的，char(10)的空格表示占位不算一个字符
- **int(10) 10位的数据长度 9999999999**，占32个字节，int型4位 **char(10) 10位固定字符串**，不足补空格 最多10个字符 **varchar(10) 10位可变字符串**，不足补空格 最多10个字符

## 4.10 FLOAT和DOUBLE的区别是什么？

- FLOAT类型数据可以存储至多8位十进制数，并在内存中占4字节；
- DOUBLE类型数据可以存储至多18位十进制数，并在内存中占8字节；

## 4.11 drop、delete与truncate的区别

三者都表示删除，但是三者有一些差别：

|          | Delete                                   | Truncate                       | Drop                                                 |
| -------- | ---------------------------------------- | ------------------------------ | ---------------------------------------------------- |
| 类型     | 属于DML                                  | 属于DDL                        | 属于DDL                                              |
| 回滚     | 可回滚                                   | 不可回滚                       | 不可回滚                                             |
| 删除内容 | 表结构还在，删除表的全部或者一部分数据行 | 表结构还在，删除表中的所有数据 | 从数据库中删除表，所有的数据行，索引和权限也会被删除 |
| 删除速度 | 删除速度慢，需要逐行删除                 | 删除速度快                     | 删除速度最快                                         |

因此，在不再需要一张表的时候，用drop；在想删除部分数据行时候，用delete；在保留表而删除所有数据的时候用truncate。

## 4.12 UNION与UNION ALL的区别？

- ****如果使用UNION ALL，不会合并重复的记录行****
- **效率 UNION 高于 UNION ALL**