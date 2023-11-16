---
title: T-SQL 语言基础
tags:
  - T-SQL
categories:
  - 编程
date: 2020-01-13 17:57:24
---

> https://www.cnblogs.com/edisonchou/p/6106176.html  
> 一：SQL Server 的体系结构  
> 二：查询  
> 三：表表达式  
> 四：集合运算  

> https://www.cnblogs.com/edisonchou/p/6106755.html  
> 五：透视、逆透视及分组  
> 六：数据修改  
> 八：可编程对象  

> https://www.cnblogs.com/edisonchou/p/6129717.html  
> 七：事务和并发  

<!--more-->

# SQL Server 体系结构
## 数据库的物理布局
![ ](1.png)

&emsp;&emsp;数据库在物理上由数据文件和事务日志文件组成，每个数据库必须至少有一个数据文件和一个日志文件。  
&emsp;&emsp;（1）数据文件用于保存数据库对象数据。数据库必须至少有一个主文件组（Primary），而用户定义的文件组则是可选的。Primary 文件组包括 主数据文件（.mdf），以及数据库的系统目录（catalog）。可以选择性地为 Primary 增加多个辅助数据文件（.ndf）。用户定义的文件组只能包含辅助数据文件。  
&emsp;&emsp;（2）日志文件则用于保存 SQL Server 为了维护事务而需要的信息。虽然 SQL Server 可以同时写多个数据文件，但同一时刻只能以顺序方式写一个日志文件。

> `.mdf`、`.ldf`和`.ndf`  
> &emsp;&emsp;`.mdf`代表 Master Data File，`.ldf`代表 Log Data File，而`.ndf`代表 Not Master Data File（非主数据文件）。

## 架构（Schema）和对象
&emsp;&emsp;一个数据库包含多个架构，而每个架构又包括多个对象。可以将架构看作是各种对象的**容器**，这些对象可以是表（table）、视图（view）、存储过程（stored procedure）等等。
![ ](2.png)

&emsp;&emsp;此外，架构也是一个命名空间，用作对象名称的前缀。例如，架设在架构 Sales 中有一个 Orders 表，架构限定的对象名称是 Sales.Orders。如果在引用对象时省略架构名称，SQL Server 将采用一定的办法来分析出架构名称是什么。**如果不显式指定架构，那么在解析对象名称时，就会要付出一些没有意义的额外代价。**因此，建议都加上架构名称。

---

# 查询
## 单表查询
（1）关于`SELECT`子句：使用`*`号是糟糕的习惯
```sql
SELECT * FROM Sales.Shippers;
```
&emsp;&emsp;在绝大多数情况下，使用星号是一种糟糕的编程习惯，在此还是建议大家即使需要查询表的所有列，也应该显式地指定它们。

（2）关于`FROM`子句：显示指定架构名称  
&emsp;&emsp;通过显示指定架构名称，可以保证得到的对象的确是你原来想要的，而且还不必付出任何额外的代价。

（3）关于`TOP`子句：T-SQL 独有关键字  
&emsp;&emsp;① 可以使用`PERCENT`关键字按百分比计算满足条件的行数；
```sql
SELECT TOP (1) PERCENT orderid, orderdate, custid, empid
FROM Sales.Orders
ORDER BY orderdate DESC;
```
&emsp;&emsp;上面这条 SQL 就会请求最近更新过的前 1% 个订单。

&emsp;&emsp;② 可以使用`WITH TIES`选项请求返回所有具有相同结果的行。
```sql
SELECT TOP (5) WITH TIES orderid, orderdate, custid, empid
FROM Sales.Orders
ORDER BY orderdate DESC;
```
&emsp;&emsp;上面这条 SQL 请求返回与 TOP n 行中最后一行的排序值相同的其他所有行。

（4）关于`OVER`子句：为行定义一个窗口以便进行特定的运算  
&emsp;&emsp;`OVER`子句的优点在于**能够在返回基本列的同时，在同一行对它们进行聚合；也可以在表达式中混合使用基本列和聚合值列。**  
&emsp;&emsp;例如，下面的查询为 OrderValues 的每一行计算当前价格占总价格的百分比，以及当前价格占客户总价格的百分比。
```sql
SELECT orderid, custid, val,
100.0 * val / SUM(val) OVER() AS pctall,
100.0 * val / SUM(val) OVER(PARTITION BY custid) AS pctcust
FROM Sales.OrderValues;
```

（5）子句的逻辑处理顺序
![ ](3.png)

（6）运算符的优先级
![ ](4.png)

（7）`CASE`表达式  
&emsp;&emsp;① 简单表达式：将一个值与一组可能的取值进行比较，并返回满足第一个匹配的结果；
```sql
SELECT productid,productname,categoryid,categoryname=(
    CASE categoryid
        WHEN 1 THEN 'Beverages'
        WHEN 2 THEN 'Condiments'
        WHEN 3 THEN 'Confections'
        WHEN 4 THEN 'Dairy Products'
        ELSE 'Unkonw Category'
    END)
FROM Production.Products;
```

&emsp;&emsp;② 搜索表达式：将返回结果为 *TRUE* 的第一个`WHEN`逻辑表达式所关联的`THEN`子句中指定的值。如果没有任何`WHEN`表达式结果为 *TRUE* ，`CASE`表达式则返回`ELSE`子句中出现的值。（如果没有指定`ELSE`，则默认返回 *NULL* ）；
```sql
SELECT orderid, custid, val, valuecategory=(
  CASE 
    WHEN val < 1000.00    THEN 'Less than 1000'
    WHEN val BETWEEN 1000.00 AND 3000.00 THEN 'Between 1000 and 3000'
    WHEN val > 3000.00    THEN 'More than 3000'
    ELSE 'Unknown'
  END
)
FROM Sales.OrderValues
```

（8）三值谓词逻辑：`TRUE`、`FALSE`与`UNKNOWN`  
&emsp;&emsp;SQL 支持使用`NULL`表示缺少的值，它使用的是三值谓词逻辑，代表计算结果可以是`TRUE`、`FALSE`与`UNKNOWN`。在 SQL 中，对于`UNKNOWN`和`NULL`的处理不一致，这就需要我们在编写每一条查询语句时应该明确地注意到正在使用的是三值谓词逻辑。  
&emsp;&emsp;例如，我们要请求返回 region 列不等于 WA 的所有行，则需要在查询过滤条件中显式地增加一个对 *NULL* 值的测试：
```sql
SELECT custid, country, region, city
FROM Sales.Customers
WHERE region <> N'WA'
  OR region IS NULL;
```

&emsp;&emsp;另外，T-SQL 对于 *NULL* 值的处理是先输出 *NULL* 值再输出非 *NULL* 值的顺序，如果想要先输出非 *NULL* 值，则需要改变一下排序条件，例如下面的请求：
```sql
select custid, region
from sales.Customers
order by (case 
when region is null then 1 else 0
end), region;
```
&emsp;&emsp;当 region 列为 *NULL* 时返回 1，否则返回 0。非 *NULL* 值的表达式返回值为 0，因此，它们会排在 *NULL* 值（表达式返回 1）的前面。如上所示的将`CASE`表达式作为第一个拍序列，并把 region 列指定为第二个拍序列。这样，非 *NULL* 值也可以正确地参与排序，是一个完整解决方案的查询。

（9）`LIKE`谓词的花式用法  
&emsp;&emsp;① `%`（百分号）通配符
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'D%';
```

&emsp;&emsp;② `_`（下划线）通配符：下划线代表任意单个字符  
&emsp;&emsp;下面请求返回 lastname 第二个字符为 e 的所有员工：
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'_e%';
```

&emsp;&emsp;③ `[<字符列>]`通配符：必须匹配指定字符中的一个字符  
&emsp;&emsp;下面请求返回 lastname 以字符 A、B、C 开头的所有员工：
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'[ABC]%';
```

&emsp;&emsp;④ `[<字符-字符>]`通配符：必须匹配指定范围内中的一个字符  
&emsp;&emsp;下面请求返回 lastname 以字符 A 到 E 开头的所有员工：
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'[A-E]%';
```

&emsp;&emsp;⑤ `[^<字符-字符>]`通配符：不属于特定字符序列或范围内的任意单个字符  
&emsp;&emsp;下面请求返回 lastname 不以 A 到 E 开头的所有员工：
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'[^A-E]%';
```

&emsp;&emsp;⑥ `ESCAPE`转义字符  
&emsp;&emsp;如果搜索包含特殊通配符的字符串（例如 `%`，`_`，`[`、`]`等），则必须使用转移字符。下面检查 lastname 列是否包含下划线：
```sql
SELECT empid, lastname
FROM HR.Employees
WHERE lastname LIKE N'%!_%' ESCAPE '!';
```

（10）两种转换值的函数：`CAST`和`CONVERT`  
&emsp;&emsp;`CAST`和`CONVERT`都用于转换值的数据类型。
```sql
SELECT CAST(SYSDATETIME() AS DATE);
SELECT CONVERT(CHAR(8),CURRENT_TIMESTAMP,112);
```
&emsp;&emsp;需要注意的是，`CAST`是 ANSI 标准的 SQL，而`CONVERT`不是。所以，除非需要使用样式值，否则**推荐优先使用`CAST`函数，以保证代码尽可能与标准兼容。**

## 联接查询
（1）交叉联接：返回笛卡尔积，即 m*n 行的结果集
```sql
-- CROSS JOIN
select c.custid, e.empid
from sales.Customers as c
    cross join HR.Employees as e;
-- INNER CROSS JOIN    
select e1.empid,e1.firstname,e1.lastname,
    e2.empid,e2.firstname,e2.lastname
from hr.Employees as e1
    cross join hr.Employees as e2;
```

（2）内联接：先笛卡尔积，然后根据指定的谓词对结果进行过滤
```sql
select e.empid,e.firstname,e.lastname,o.orderid
from hr.Employees as e
    join sales.Orders as o
    on e.empid=o.empid;
```

> &emsp;&emsp;虽然不使用`JOIN`这种 ANSI SQL-92 标准语法也可以实现联接，但强烈推荐使用 ANSI SQL-92 标准，因为它用起来更加安全。比如，假如你要写一条内联接查询，如果不小心忘记了指定联接条件，如果这时候用的是 ANSI SQL-92 语法，那么语法分析器将会报错。

![ ](5.png)

（3）外联结：笛卡尔积 → 对结果过滤 → 添加外部行  
&emsp;&emsp;通过例子来理解外联结：根据客户的客户 ID 和订单的客户 ID 来对 Customers 表和 Orders 表进行联接，并返回客户和他们的订单信息。该查询语句使用的联接类型是左外连接，所以查询结果也包括那些没有发出任何订单的客户；
```sql
--LEFT OUTER JOIN
select c.custid,c.companyname,o.orderid
from sales.Customers as c
  left outer join sales.Orders as o
  on c.custid=o.custid;
```

&emsp;&emsp;另外，需要注意的是在对外联结中非保留值得列值进行过滤时，不要在`WHERE`子句中指定错误的查询条件。  
&emsp;&emsp;例如，下面请求返回在 2007 年 2 月 12 日下过订单的客户，以及他们的订单。同时也返回在 2007 年 2 月 12 日没有下过订单的客户。这是一个典型的左外连接的案例，但是我们经常会犯这样的错误：
```sql
select c.custid,c.companyname,o.orderid,o.orderdate
from sales.Customers as c
    left outer join sales.Orders as o
    on c.custid=o.custid 
where o.orderdate='20070212';
```

&emsp;&emsp;执行结果如下：
![ ](6.png)

&emsp;&emsp;这是因为对于所有的外部行，因为它们在 o.orderdate 列上的取值都为 *NULL* ，所以`WHERE`子句中条件 o.orderdate='20070212' 的计算结果为 *UNKNOWN* ，因此`WHERE`子句会过滤掉所有的外部行。  
&emsp;&emsp;我们应该将这个条件搬到`on`后边：
```sql
select c.custid,c.companyname,o.orderid,o.orderdate
from sales.Customers as c
    left outer join sales.Orders as o
    on c.custid=o.custid 
        and o.orderdate='20070212';
```

&emsp;&emsp;这下的执行结果如下：
![ ](7.png)

## 子查询
（1）独立子查询：不依赖于它所属的外部查询  
&emsp;&emsp;例如下面要查询 Orders 表中订单 ID 最大的订单信息，这种叫做 独立标量子查询，即返回值不能超过一个。
```sql
select orderid, orderdate, empid, custid
from sales.Orders
where empid=(select MAX(o.orderid) from sales.Orders as o);
```

&emsp;&emsp;下面请求查询返回姓氏以字符 D 开头的员工处理过的订单的 ID，这种叫做 独立多值子查询，即返回值可能有多个。
```sql
select orderid
from sales.Orders
where empid in (select e.empid 
    from hr.Employees as e
    where e.lastname like N'D%');
```

（2）相关子查询：必须依赖于它所属的外部查询，不能独立地调用它  
&emsp;&emsp;例如下面的查询会返回每个客户的订单记录中订单 ID 最大的记录：
```sql
select custid, orderid, orderdate, empid
from sales.Orders as o1
where orderid=(select MAX(o2.orderid) 
    from sales.Orders as o2
    where o2.custid=o1.custid);
```
&emsp;&emsp;简单地说，对于 o1 表中的每一行，子查询负责返回当前客户的最大订单 ID。如果 o1 表中某行的订单 ID 和子查询返回的订单 ID 匹配，那么 o1 中的这个订单 ID 就是当前客户的最大订单 ID，在这种情况下，查询便会返回 o1 表中的这个行。

（3）`EXISTS`谓词：它的输入是一个查询，如果子查询能够返回任何行，则返回 *True* ，否则返回 *False*  
&emsp;&emsp;例如下面的查询会返回下过订单的西班牙客户：
```sql
select custid, companyname
from sales.customers as c
where c.country=N'Spain' and exists (
    select * from sales.Orders as o
    where o.custid=c.custid);
```

&emsp;&emsp;同样，要查询没有下过订单的西班牙客户只需要加上`NOT`即可：
```sql
select custid, companyname
from sales.customers as c
where c.country=N'Spain' and not exists (
    select * from sales.Orders as o
    where o.custid=c.custid);
```

> &emsp;&emsp;对于`EXISTS`，它采用的是二值逻辑（ *TRUE* 和 *FALSE* ），它只关心是否存在匹配行，而不考虑`SELECT`列表中指定的列，并且无须处理所有满足条件的行。可以将这种处理方式看做是一种“短路”，它**能够提高处理效率**。  　
> &emsp;&emsp;另外，由于`EXISTS`采用的是二值逻辑，因此相较于`IN`要更加安全，可以避免对 *NULL* 值的处理。　

（4）高级子查询  
&emsp;&emsp;① 如何表示前一个或后一个记录？逻辑等式：上一个 -> 小于当前值的最大值；下一个 -> 大于当前值的最小值；
```sql
-- 上一个订单ID
select orderid, orderdate, empid, custid,
(
select MAX(o2.orderid) 
from sales.Orders as o2
where o2.orderid<o1.orderid
) as prevorderid 
from sales.Orders as o1;
```

&emsp;&emsp;② 如何实现连续聚合函数？在子查询中连续计算
```sql
-- 连续聚合
select orderyear, qty, 
(select SUM(o2.qty) 
 from sales.OrderTotalsByYear as o2
 where o2.orderyear<=o1.orderyear) as runqty 
from sales.OrderTotalsByYear as o1
order by orderyear;
```

&emsp;&emsp;执行结果如下图所示：
![ ](8.png)

&emsp;&emsp;③ 使用`NOT EXISTS`谓词取代`NOT IN`隐式排除 *NULL* 值：当对至少返回一个 *NULL* 值的子查询使用`NOT IN`谓词时，外部查询总会返回一个空集。（前面提到，`EXISTS`谓词采用的是二词逻辑而不是三词逻辑）
```sql
-- 隐式排除NULL值
select custid,companyname from sales.Customers as c
where not exists
(select * 
 from sales.Orders as o
 where o.custid=c.custid);
```

&emsp;&emsp;又如以下查询请求返回每个客户在 2007 年下过订单而在 2008 年没有下过订单的客户：
```sql
select custid, companyname
from sales.Customers as c
where exists 
(select * from sales.Orders as o1
 where c.custid=o1.custid 
 and o1.orderdate>='20070101' and o1.orderdate<'20080101')
and not exists 
(select * from sales.Orders as o2
 where c.custid=o2.custid
 and o2.orderdate>='20080101' and o2.orderdate<'20090101');
```

---

# 表表达式
&emsp;&emsp;表表达式是一种命名的查询表达式，代表一个有效的关系表。可以像其他表一样，在数据处理中使用表表达式。MSSQL 中支持 4 种类型的表表达式：

## 派生表
&emsp;&emsp;派生表（也称为表子查询）是在外部查询的 FROM 子句中定义的，只要外部查询一结束，派生表也就不存在了。  
&emsp;&emsp;例如下面代码定义了一个名为 USACusts 的派生表，它是一个返回所有美国客户的查询。外部查询则选择了派生表的所有行。
```sql
select *
from (select custid, companyname 
      from sales.Customers 
      where country='USA') as USACusts;
```

## 公用表表达式
&emsp;&emsp;公用表达式（简称 CTE，Common Table Expression）是和派生表很相似的另一种形式的表表达式，是 ANSI SQL（1999 及以后版本）标准的一部分。  
&emsp;&emsp;举个栗子，下面的代码定义了一个名为 USACusts 的 CTE，它的内部查询返回所有来自美国的客户，外部查询则选择了 CTE 中的所有行：
```sql
WITH USACusts AS
(
    select custid, companyname
    from sales.Customers 
    where country=N'USA'
)
select * from USACusts;
```
&emsp;&emsp;和派生表一样，一旦外部查询完成，CTE 的生命周期也就结束了。

## 视图
&emsp;&emsp;派生表和 CTE 都是不可重用的，而视图和内联表值函数却是可重用，它们的定义存储在一个数据库对象中，一旦创建，这些对象就是数据库的永久部分。只有用删除语句显式地删除，它们才会从数据库中移除。  
&emsp;&emsp;下面仍然继续上面的例子，创建一个视图：
```sql
IF OBJECT_ID('Sales.USACusts') IS NOT NULL
   DROP VIEW Sales.USACusts;
GO
CREATE VIEW Sales.USACusts
AS
SELECT 
    custid, companyname, contactname, contacttitle, address,
    city, region, postalcode, country, phone, fax
FROM Sales.Customers
WHERE country=N'USA';
GO
```

&emsp;&emsp;使用该视图：
```sql
SELECT * FROM Sales.USACusts;
```

&emsp;&emsp;执行结果如下：
![ ](9.png)

## 内联表值函数
&emsp;&emsp;内联表值函数能够支持输入参数，其他方面就与视图类似了。  
&emsp;&emsp;下面演示如何创建函数：
```sql
IF OBJECT_ID('dbo.fn_GetCustOrders') IS NOT NULL
   DROP FUNCTION dbo.fn_GetCustOrders;
GO
CREATE FUNCTION dbo.fn_GetCustOrders
    (@cid AS INT) RETURNS TABLE
AS
RETURN 
    SELECT
        orderid, custid, empid, orderdate, requireddate,
        shippeddate, shipperid, freight, shipname, shipaddress, shipcity,
        shipregion, shippostalcode, shipcountry
    FROM Sales.Orders
    WHERE custid=@cid;
GO
```

&emsp;&emsp;如何使用函数：
```sql
SELECT orderid, custid
FROM dbo.fn_GetCustOrders(1) AS CO;
```

&emsp;&emsp;执行结果如下：
![ ](10.png)

> **总结**：  
> &emsp;&emsp;借助表表达式可以简化代码，提高代码地可维护性，还可以封装查询逻辑。  
> &emsp;&emsp;当需要使用表表达式，而且不计划重用它们的定义时，可以使用派生表或 CTE，与派生表相比，CTE 更加模块化，更容易维护。  
> &emsp;&emsp;当需要定义可重用的表表达式时，可以使用视图或内联表值函数。如果不需要支持输入，则使用视图；反之，则使用内联表值函数。

---

# 集合运算
## UNION 并集运算
![ ](11.png)

&emsp;&emsp;在 T-SQL 中，`UNION`集合运算可以将两个输入查询的结果组合成一个结果集。需要注意的是：如果一个行在任何一个输入集合众出现，它也会在`UNION`运算的结果中出现。T-SQL 支持以下两种选项：  
（1）`UNION ALL`：不会删除重复行
```sql
-- union all
select country, region, city from hr.Employees
union all
select country, region, city from sales.Customers; 
```

&emsp;&emsp;结果得到 100 行：
![ ](12.png)

（2）`UNION`：会删除重复行
```sql
-- union
select country, region from hr.Employees
union
select country, region from sales.Customers; 
```

&emsp;&emsp;结果得到 34 行：
![ ](13.png)

## INTERSECT 交集运算
![ ](14.png)

&emsp;&emsp;在 T-SQL 中，`INTERSECT`集合运算对两个输入查询的结果取其交集，只返回在两个查询结果集中都出现的行。  
&emsp;&emsp;`INTERSECT`集合运算在逻辑上会首先删除两个输入集中的重复行，然后返回只在两个集合中中都出现的行。换句话说：如果一个行在两个输入集中都至少出现一次，那么交集返回的结果中将包含这一行。  
&emsp;&emsp;例如，下面返回既是雇员地址，又是客户地址的不同地址：
```sql
-- intersect
select country, region, city from hr.Employees
intersect
select country, region, city from sales.Customers;
```

&emsp;&emsp;执行结果如下图所示：
![ ](15.png)

> &emsp;&emsp;这里需要说的是，集合运算对行进行比较时，**认为两个 NULL 值相等**，所以就返回该行记录。

## EXCEPT 差集运算
![ ](16.png)

&emsp;&emsp;在 T-SQL 中，集合之差使用`EXCEPT`集合运算实现的。它对两个输入查询的结果集进行操作，返回出现在第一个结果集中，但不出现在第二个结果集中的所有行。  
&emsp;&emsp;`EXCEPT`结合运算在逻辑上首先删除两个输入集中的重复行，然后返回只在第一个集合中出现，在第二个结果集中不出现的所有行。换句话说：一个行能够被返回，仅当这个行在第一个输入的集合中至少出现过一次，而且在第二个集合中一次也没出现过。  
&emsp;&emsp;此外，相比`UNION`和`INTERSECT`，两个输入集合的顺序是会影响到最后返回结果的。  
&emsp;&emsp;例如，借助`EXCEPT`运算，我们可以方便地实现属于 A 但不属于 B 的场景，下面返回属于员工地址，但不属于客户地址的地址记录：
```sql
-- except 
select country, region, city from hr.Employees
except
select country, region, city from sales.Customers;
```

&emsp;&emsp;执行结果如下图所示：
![ ](17.png)

## 集合运算优先级
![ ](18.png)

&emsp;&emsp;SQL 定义了集合运算之间的优先级：`INTERSECT`最高，`UNION`和`EXCEPT`相等。  
&emsp;&emsp;换句话说：首先会计算`INTERSECT`，然后按照从左至右的出现顺序依次处理优先级相同的运算。
```sql
-- 集合运算的优先级
select country, region, city from Production.Suppliers
except
select country, region, city from hr.Employees
intersect
select country, region, city from sales.Customers;
```
&emsp;&emsp;上面这段 SQL 代码，因为`INTERSECT`优先级比`EXCEPT`高，所以首先进行`INTERSECT`交集运算。因此，这个查询的含义是：返回没有出现在员工地址和客户地址交集中的供应商地址。

## 使用表表达式避开不支持的逻辑查询处理
&emsp;&emsp;集合运算查询本身并不支持除`ORDER BY`以外的其他逻辑查询处理阶段，但可以通过表表达式来避开这一限制。  
&emsp;&emsp;解决方案就是：首先根据包含集合运算的查询定义一个表表达式，然后在外部查询中对表表达式应用任何需要的逻辑查询处理。  
（1）例如，下面的查询返回每个国家中不同的员工地址或客户地址的数量：
```sql
select country, COUNT(*) as numlocations
from (select country, region, city from hr.Employees
      union
      select country, region, city from sales.Customers) as U
group by country;
```

（2）例如，下面的查询返回由员工地址为 3 或 5 的员工最近处理过的两个订单：
```sql
select empid,orderid,orderdate 
from (select top (2) empid,orderid,orderdate 
    from sales.Orders
    where empid=3
    order by orderdate desc,orderid desc) as D1
union all
select empid,orderid,orderdate 
from (select top (2) empid,orderid,orderdate 
    from sales.Orders
    where empid=5
    order by orderdate desc,orderid desc) as D2;
```

---

# 透视、逆透视及分组
## 透视
&emsp;&emsp;所谓透视（Pivoting）就是把数据从行的状态旋转为列的状态的处理。其处理步骤为：
![ ](19.png)

&emsp;&emsp;相信很多人在笔试或面试的时候被问到如何通过 SQL 实现行转列或列转行的问题，可能很多人当时懵逼了，没关系，下面我们通过例子来理解。  
（1）准备数据
```sql
--1.0准备数据
USE tempdb;

IF OBJECT_ID('dbo.Orders', 'U') IS NOT NULL DROP TABLE dbo.Orders;
GO

CREATE TABLE dbo.Orders
(
  orderid   INT        NOT NULL,
  orderdate DATE       NOT NULL, -- prior to SQL Server 2008 use DATETIME
  empid     INT        NOT NULL,
  custid    VARCHAR(5) NOT NULL,
  qty       INT        NOT NULL,
  CONSTRAINT PK_Orders PRIMARY KEY(orderid)
);

INSERT INTO dbo.Orders(orderid, orderdate, empid, custid, qty)
VALUES
  (30001, '20070802', 3, 'A', 10),
  (10001, '20071224', 2, 'A', 12),
  (10005, '20071224', 1, 'B', 20),
  (40001, '20080109', 2, 'A', 40),
  (10006, '20080118', 1, 'C', 14),
  (20001, '20080212', 2, 'B', 12),
  (40005, '20090212', 3, 'A', 10),
  (20002, '20090216', 1, 'C', 20),
  (30003, '20090418', 2, 'B', 15),
  (30004, '20070418', 3, 'C', 22),
  (30007, '20090907', 3, 'D', 30);

SELECT * FROM dbo.Orders;
```

&emsp;&emsp;这里使用了 MS SQL2008 的`VALUES`子句格式语法，这是 2008 版本的新特性。如果你使用的是 2005 及以下版本，你需要多个`INSERT`语句。最后的执行结果如下图所示：
![ ](20.png)

（2）需求说明  
&emsp;&emsp;假设我们要生成一个报表，包含每个员工和客户组合之间的总订货量。用以下简单的分组查询可以解决这个问题：
```sql
select empid,custid,SUM(qty) as sumqty 
from dbo.Orders
group by empid,custid;
```

&emsp;&emsp;该查询的执行结果如下：
![ ](21.png)

&emsp;&emsp;不过，假设现在要求要按下表所示的的格式来生成输出结果：
![ ](22.png)
&emsp;&emsp;这时，我们就需要进行透视转换了！

（3）使用**标准 SQL** 进行透视转换  
&emsp;&emsp;Step1. 分组：`GROUP BY empid`；  
&emsp;&emsp;Step2. 扩展：`CASE WHEN custid='A' THEN qty END`；  
&emsp;&emsp;Step3. 聚合：`SUM(CASE WHEN custid='A' THEN qty END)`；
```sql
--1.1标准SQL透视转换
select empid,
    SUM(case when custid='A' then qty end) as A,
    SUM(case when custid='B' then qty end) as B,
    SUM(case when custid='C' then qty end) as C,
    SUM(case when custid='D' then qty end) as D
from dbo.Orders
group by empid;
```

&emsp;&emsp;执行结果如下图所示：
![ ](23.png)

（4）使用 T-SQL `PIVOT`运算符进行透视转换  
&emsp;&emsp;自 SQL Server 2005 开始引入了一个 T-SQL 独有的表运算符`PIVOT`，它可以对某个源表或表表达式进行操作、透视数据，再返回一个结果表。  
&emsp;&emsp;`PIVOT`运算符同样涉及前面介绍的三个逻辑处理阶段（分组、扩展和聚合）以及同样的透视转换元素，但使用的是不同的、**SQL Server 原生的语法**。  
&emsp;&emsp;下面是使用`PIVOT`运算符实现上面一样的效果：
```sql
select empid,A,B,C,D
from (select empid,custid,qty
      from dbo.Orders) as D
  pivot (sum(qty) for custid in (A,B,C,D)) as P;
```
&emsp;&emsp;其中，`PIVOT`运算符的圆括号内要指定聚合函数（本例中`SUM`）、聚合元素（本例中的 qty）、扩展元素（custid）以及目标列名称的列表（本例中的 A、B、C、D）。在`PIVOT`运算符的圆括号后面，可以为结果表制定一个别名。

> &emsp;&emsp;**Tips**：使用`PIVOT`运算符一般不直接把它应用到源表（本例中的 Orders 表），而是将其应用到一个表表达式（该表表达式只包含透视转换需要的3种元素，不包含其他属性。）此外，不需要为它显式地指定分组元素，也就不需要再查询中使用`GROUP BY`子句。

## 逆透视
&emsp;&emsp;所谓逆透视（Unpivoting）转换是一种把数据从列的状态旋转为行的状态的技术，它将来自单个记录中多个列的值扩展为单个列中具有相同值得多个记录。换句话说，将透视表中的每个源行潜在地转换成多个行，每行代表源透视表的一个指定的列值。

还是通过一个栗子来理解：  
（1）首先还是准备一下数据：
```sql
USE tempdb;

IF OBJECT_ID('dbo.EmpCustOrders', 'U') IS NOT NULL DROP TABLE dbo.EmpCustOrders;

SELECT empid, A, B, C, D
INTO dbo.EmpCustOrders
FROM (SELECT empid, custid, qty
      FROM dbo.Orders) AS D
  PIVOT(SUM(qty) FOR custid IN(A, B, C, D)) AS P;

SELECT * FROM dbo.EmpCustOrders;
```

&emsp;&emsp;下面是对这个表 EmpCustOrders 的查询结果：
![ ](24.png)

（2）需求说明  
&emsp;&emsp;要求执行你透视转换，为每个员工和客户组合返回一行记录，其中包含这一组合的订货量。期望的输出结果如下图所示：
![ ](25.png)

（3）标准 SQL 进行逆透视转换  
&emsp;&emsp;Step1. 生成副本：`CROSS JOIN`交叉联接生成多个副本  
&emsp;&emsp;Step2. 提取元素：通过`CASE`语句生成 qty 数据列  
&emsp;&emsp;Step3. 删除不相关的交叉：过滤掉 *NULL* 值
```sql
select *
from (select empid, custid,
        case custid
            when 'A' then A
            when 'B' then B
            when 'C' then C
            when 'D' then D
        end as qty
      from dbo.EmpCustOrders
        cross join (VALUES('A'),('B'),('C'),('D')) as Custs(custid)) as D
where qty is not null;
```

&emsp;&emsp;执行结果如下图所示：
![ ](26.png)

（4）T-SQL `UNPIVOT`运算符进行逆透视转换  
&emsp;&emsp;和`PIVOT`类似，在 SQL Server 2005 引入了一个`UNPIVOT`运算符，它的作用刚好和`PIVOT`运算符相反，即我们可以拿来做逆透视转换工作。`UNPIVOT`同样会经历我们上面提到的三个阶段。继续上面的栗子，我们使用`UNPIVOT`来进行逆透视转换：
```sql
select empid, custid, qty
from dbo.EmpCustOrders
  unpivot (qty for custid in (A,B,C,D)) as U;
```
&emsp;&emsp;其中，`UNPIVOT`运算符后边的括号内包括：用于保存源表列值的目标列明（这里是 qty），用于保存源表列名的目标列名（这里是 custid），以及源表列名列表（A、B、C、D）。同样，在`UNPIVOT`括号后面也可以跟一个别名。

> &emsp;&emsp;**Tips**：对经过透视转换所得的表再进行逆透视转换，并不能得到原来的表。因为你透视转换只是把经过透视转换的值再旋转岛另一种新的格式。

## 分组
&emsp;&emsp;首先了解一下分组集：分组集就是分组（`GROUP BY`子句）使用的一组属性（或列名）。在传统 SQL 中，一个聚合查询只能定义一个分组集。为了灵活而有效地处理分组集，SQL Server 2008 引入了几个重要的新功能（**它们都是`GROUP BY`的从属子句，需要依赖于`GROUP BY`子句**）：  
（1）`GROUPING SETS`从属子句  
&emsp;&emsp;使用该子句，可以方便地在同一个查询中定义多个分组集。例如下面，我们定义了4个分组集：(empid,custid)，(empid)，(custid) 和 ()：
```sql
--3.1GROUPING SETS从属子句
select empid,custid,SUM(qty) as sumqty
from dbo.Orders
group by 
  GROUPING SETS
  (
    (empid,custid),
    (empid),
    (custid),
    ()
   );
```
&emsp;&emsp;这个查询相当于执行了四个 group by 查询的并集。

（2）`CUBE`从属子句  
&emsp;&emsp;`CUBE`子句为定义多个分组集提供了一种更简略的方法，可以把`CUBE`子句看作是用于生成分组的幂集。例如：`CUBE(a,b,c)`等价于`GROUPING SETS[(a,b,c),(a,b),(a,c),(b,c),(a),(b),(c),()]`。下面我们用`CUBE`来实现上面的例子：
```sql
--3.2CUEE从属子句
select empid,custid,SUM(qty) as sumqty
from dbo.Orders
group by cube(empid,custid);
```

（3）`ROLLUP`从属子句  
&emsp;&emsp;`ROLLUP`子句也是一种简略的方法，只不过它与`CUBE`不同，它强调输入成员之间存在一定的层次关系，从而生成让这种层次关系有意义的所有分组集。例如：`CUBE(a,b,c)`会生成 8 个可能的分组集，而`ROLLUP`则认为 3 个输入成员存在 a > b > c 的层次关系，所以只会生成4个分组集：(a,b,c)，(a,b)，(a)，()。  
&emsp;&emsp;下面我们假设想要按时间层次关系：订单年份 > 订单月份 > 订单日，以这样的关系来定义所有分组集，并未每个分组集返回其总订货量。可能我们用`GROUPING SETS`需要 4 行，然后使用`ROLLUP`却只需要一行：`group by rollup(YEAR(orderdate),MONTH(orderdate),DAY(orderdate));`  
&emsp;&emsp;完整SQL查询如下：
```sql
--3.3ROLLUP从属子句
select
  YEAR(orderdate) as orderyear,
  MONTH(orderdate) as ordermonth,
  DAY(orderdate) as orderday,
  SUM(qty) as sumqty
from dbo.Orders
group by rollup(YEAR(orderdate),MONTH(orderdate),DAY(orderdate));
```

&emsp;&emsp;执行结果如下图所示：
![ ](27.png)

（4）`GROUPING_ID`函数  
&emsp;&emsp;如果一个查询定义了多个分组集，还想把结果行和分组集关联起来，也就是说，为每个结果行标注它是和哪个分组集关联的。SQL Server 2008 中引入了一个`GROUPING_ID`函数，简化了关联结果行和分组集的处理，可以容易地计算出每一行和哪个分组集相关联。  
&emsp;&emsp;例如，继续上面的例子，我们想要将 empid，custid 作为输入：
```sql
select 
  grouping_id(empid,custid) as groupingset,
  empid, custid, SUM(qty) as sumqty
from dbo.Orders
group by cube(empid,custid);
```
&emsp;&emsp;执行结果中会出现 groupingset 为 0，1，2，3 ，分别代表了 empid，custid 的4个可能的分组集 (empid,custid)，(empid)，(custid)，() 。

---

# 数据修改
## 插入与删除数据
### 看我花式插入数据
&emsp;&emsp;① `INSERT VALUES`语句 ：这个语句恐怕我们再熟悉不过了吧，在任何一本数据库的书上面都可以看到这个语句的身影。
```sql
INSERT INTO dbo.Orders(orderid, orderdate, empid, custid)
  VALUES(10001, '20090212', 3, 'A');
```

&emsp;&emsp;需要了解的是，前面也提到过，SQL Server 2008 增强了`VALUES`语句的功能，允许在一条语句中指定由逗号分隔开的多行记录。例如下面的语句向 Orders 中插入了 4 行数据：
```sql
INSERT INTO dbo.Orders
  (orderid, orderdate, empid, custid)
VALUES
  (10003, '20090213', 4, 'B'),
  (10004, '20090214', 1, 'A'),
  (10005, '20090213', 1, 'C'),
  (10006, '20090215', 3, 'C');
```

&emsp;&emsp;② `INSERT SELECT`语句 ：将一组由`SELECT`查询返回的结果行插入到目标表中。
```sql
INSERT INTO dbo.Orders(orderid, orderdate, empid, custid)
  SELECT orderid, orderdate, empid, custid
  FROM TSQLFundamentals2008.Sales.Orders
  WHERE shipcountry = 'UK';
```

&emsp;&emsp;③ `INSERT EXEC`语句：将存储过过程或动态 SQL 批处理返回的结果集插入目标表。  
&emsp;&emsp;下面的示例演示了如何执行存储过程 usp_getorders 并将结果插入到 Orders 表中：
```sql
INSERT INTO dbo.Orders(orderid, orderdate, empid, custid)
  EXEC TSQLFundamentals2008.Sales.usp_getorders @country = 'France';
```

&emsp;&emsp;④ `SELECT INTO`语句：它会创建一个目标表，并用查询返回的结果来填充它。需要注意的是：**它不是一个标准的 SQL 语句（即不是 ANSI SQL 标准的一部分），不能用这个语句向已经存在的表中插入数据。**
```sql
--保证目标表不存在
IF OBJECT_ID('dbo.Orders', 'U') IS NOT NULL DROP TABLE dbo.Orders;

SELECT orderid, orderdate, empid, custid
INTO dbo.Orders
FROM TSQLFundamentals2008.Sales.Orders;
```

&emsp;&emsp;⑤ `BULK INSERT`语句：用于将文件中的数据导入一个已经存在的表，需要制定目标表、源文件以及一些其他的选项。  
&emsp;&emsp;下面的栗子演示了如何将文件 "C:\testdata\orders.txt" 中的数据容量插入（bulk insert）到 Orders 表，同时还指定了文件类型为字符格式，字段终止符为逗号，行终止符为换行符（\t）：
```sql
BULK INSERT dbo.Orders FROM 'C:\testdata\orders.txt'
  WITH 
    (
       DATAFILETYPE    = 'char',
       FIELDTERMINATOR = ',',
       ROWTERMINATOR   = '\n'
    );
```

### 看我花式删除数据
&emsp;&emsp;① `DELETE`语句：标准 SQL 语句，大家最常见的用法。
```sql
DELETE FROM dbo.Orders
WHERE orderdate < '20070101';
```

&emsp;&emsp;② `TRUNCATE`语句：不是标准的 SQL 语句，永于删除表中的所有行，不需要过滤条件。
> &emsp;&emsp;**Tips**：`TRUNCATE`与`DELETE`在性能上差异巨大，对一个百万行级记录的表，`TRUNCATE`几秒内就可以解决，而`DELETE`可能需要几分钟。因为`TRUNCATE`会以最小模式记录日志，而`DELETE`则以完整模式记录日志。所以，各位，谨慎使用`TRUNCATE`。因此，我们可以创建一个虚拟表（Dummy Table），让虚拟表包含一个指向产品表的外键，这样就可以保护产品表了。

&emsp;&emsp;③ 基于联接的`DELETE`：也不是标准 SQL 语句，可以根据另一个表中相关行的属性定义的过滤器来删除表中的数据行。  
&emsp;&emsp;例如，下面语句用以删除美国客户下的订单：
```sql
DELETE FROM O
FROM dbo.Orders AS O
  JOIN dbo.Customers AS C
    ON O.custid = C.custid
WHERE C.country = N'USA';
```

&emsp;&emsp;当然，如果要使用标准 SQL 语句，也可以采用下面的方式：
```sql
DELETE FROM dbo.Orders
WHERE EXISTS
  (SELECT *
   FROM dbo.Customers AS C
   WHERE Orders.custid = C.custid
     AND C.country = N'USA');
```

## 更新与合并数据
### 花式更新数据
&emsp;&emsp;① `UPDATE`语句：不解释了，大家都在用  
&emsp;&emsp;下面来看两个不一样的栗子，第一个是关于同时操作的性质。看看下面的`UPDATE`语句：
```sql
UPDATE dbo.T1
  SET col1 = col1 + 10, col2 = col1 + 10;
```

&emsp;&emsp;假设 T1 表中的 col1 列为 100，col2 列为 200。在计算后是多少呢？  
&emsp;&emsp;答案揭晓：col=110, col=110。  
&emsp;&emsp;再来看一个栗子，假设我们要实现两个数的交换该怎么做？我们可能迫不及待的说出临时变量。然而，在 SQL 中所有赋值表达式好像都是同时计算的，解决这个问题就不需要临时变量了。
```sql
UPDATE dbo.T1
  SET col1 = col2, col2 = col1;
```

&emsp;&emsp;② 基于联接的`UPDATE`语句：同样不是 SQL 标准语法，联接在此与基于联接的`DELETE`一样是起到过滤作用。
```sql
UPDATE OD
  SET discount = discount + 0.05
FROM dbo.OrderDetails AS OD
  JOIN dbo.Orders AS O
    ON OD.orderid = O.orderid
WHERE custid = 1;
```

&emsp;&emsp;同样，要使用标准 SQL 语法的话，可以用子查询替代联接：
```sql
UPDATE dbo.OrderDetails
  SET discount = discount + 0.05
WHERE EXISTS
  (SELECT * FROM dbo.Orders AS O
   WHERE O.orderid = OrderDetails.orderid
     AND custid = 1);
```

&emsp;&emsp;③ 赋值`UPDATE`：这是 T-SQL 特有的语法，可以对表中的数据进行更新的同时为变量赋值。你不需要使用单独的`UPDATE`和`SELECT`语句，就能完成同样的任务。  
&emsp;&emsp;假设我们有一个表 Sequence，它只有一列 val，全是序号数字。我们可以通过赋值`UPDATE`得到一个新的序列值：
```sql
DECLARE @nextval AS INT;
UPDATE Sequence SET @nextval = val = val + 1;
SELECT @nextval;
```

### 新玩法：合并数据
&emsp;&emsp;SQL Server 2008 引入了一个叫做`MERGE`的语句，它能在一条语句中根据逻辑条件对数据进行不同的修改操作（`INSERT`/`UPDATE`/`DELETE`）。`MERGE`语句是 SQL 标准的一部分，而 T-SQL 版本的`MERGE`语句也增加了一些非标准的扩展。  
&emsp;&emsp;下面我们看看如何合并，首先我们准备两张表 Customers 和 CustomersStage：
```sql
--merge data
USE tempdb;

IF OBJECT_ID('dbo.Customers', 'U') IS NOT NULL DROP TABLE dbo.Customers;
GO

CREATE TABLE dbo.Customers
(
  custid      INT         NOT NULL,
  companyname VARCHAR(25) NOT NULL,
  phone       VARCHAR(20) NOT NULL,
  address     VARCHAR(50) NOT NULL,
  CONSTRAINT PK_Customers PRIMARY KEY(custid)
);

INSERT INTO dbo.Customers(custid, companyname, phone, address)
VALUES
  (1, 'cust 1', '(111) 111-1111', 'address 1'),
  (2, 'cust 2', '(222) 222-2222', 'address 2'),
  (3, 'cust 3', '(333) 333-3333', 'address 3'),
  (4, 'cust 4', '(444) 444-4444', 'address 4'),
  (5, 'cust 5', '(555) 555-5555', 'address 5');

IF OBJECT_ID('dbo.CustomersStage', 'U') IS NOT NULL DROP TABLE dbo.CustomersStage;
GO

CREATE TABLE dbo.CustomersStage
(
  custid      INT         NOT NULL,
  companyname VARCHAR(25) NOT NULL,
  phone       VARCHAR(20) NOT NULL,
  address     VARCHAR(50) NOT NULL,
  CONSTRAINT PK_CustomersStage PRIMARY KEY(custid)
);

INSERT INTO dbo.CustomersStage(custid, companyname, phone, address)
VALUES
  (2, 'AAAAA', '(222) 222-2222', 'address 2'),
  (3, 'cust 3', '(333) 333-3333', 'address 3'),
  (5, 'BBBBB', 'CCCCC', 'DDDDD'),
  (6, 'cust 6 (new)', '(666) 666-6666', 'address 6'),
  (7, 'cust 7 (new)', '(777) 777-7777', 'address 7');

-- Query tables
SELECT * FROM dbo.Customers;

SELECT * FROM dbo.CustomersStage;
```

&emsp;&emsp;执行结果如下图所示：
![ ](28.png)

&emsp;&emsp;现在我们想要增加还不存在的客户，并更新已经存在的客户。源表：CustomersStage，目标表：Customers。
```sql
MERGE INTO dbo.Customers AS TGT
USING dbo.CustomersStage AS SRC
  ON TGT.custid = SRC.custid
WHEN MATCHED THEN
  UPDATE SET
    TGT.companyname = SRC.companyname,
    TGT.phone = SRC.phone,
    TGT.address = SRC.address
WHEN NOT MATCHED THEN 
  INSERT (custid, companyname, phone, address)
  VALUES (SRC.custid, SRC.companyname, SRC.phone, SRC.address);
```
&emsp;&emsp;谓词条件：`TGT.custid=SRC.custid`用于定义什么样的数据是匹配的，什么样的数据是不匹配的。

> &emsp;&emsp;**Tips**：**`MERGE`语句必须以分号结束**，而对于 T-SQL 中的大多数其他语句来说是可选的。但是，推荐遵循最佳实践，以分号结束。

## 高级数据更新方法
&emsp;&emsp;① 通过表表达式修改数据
```sql
-- 基于联接的UPDATE
UPDATE OD
  SET discount = discount + 0.05
FROM dbo.OrderDetails AS OD
  JOIN dbo.Orders AS O
    ON OD.orderid = O.orderid
WHERE custid = 1;
-- 基于表表达式（这里是CTE）的UPDATE
WITH C AS
(
  SELECT custid, OD.orderid,
    productid, discount, discount + 0.05 AS newdiscount
  FROM dbo.OrderDetails AS OD
    JOIN dbo.Orders AS O
      ON OD.orderid = O.orderid
  WHERE custid = 1
)
UPDATE C
  SET discount = newdiscount;
```

&emsp;&emsp;② 带有`TOP`选项的数据更新
```sql
-- 删除前50行
DELETE TOP(50) FROM dbo.Orders;
-- 更新前50行
UPDATE TOP(50) dbo.Orders
  SET freight = freight + 10.00;
-- 基于CTE删除前50行
WITH C AS
(
  SELECT TOP(50) *
  FROM dbo.Orders
  ORDER BY orderid
)
DELETE FROM C;
-- 基于CTE更新前50行
WITH C AS
(
  SELECT TOP(50) *
  FROM dbo.Orders
  ORDER BY orderid DESC
)
UPDATE C
  SET freight = freight + 10.00;
```

## OUTPUT 子句
&emsp;&emsp;在某些场景中，我们希望能够从修改过的行中返回数据，这时就可以使用`OUTPUT`子句。SQL Server 2005 引入了`OUTPUT`子句，通过在修改语句中添加`OUTPUT`子句，就可以实现从修改语句中返回数据的功能。  
&emsp;&emsp;① 带有`OUTPUT`的`INSERT`语句
```sql
INSERT INTO dbo.T1(datacol)
  OUTPUT inserted.keycol, inserted.datacol
    SELECT lastname
    FROM TSQLFundamentals2008.HR.Employees
    WHERE country = N'USA';
```

&emsp;&emsp;② 带有`OUTPUT`的`DELETE`语句
```sql
DELETE FROM dbo.Orders
  OUTPUT
    deleted.orderid,
    deleted.orderdate,
    deleted.empid,
    deleted.custid
WHERE orderdate < '20080101';
```

&emsp;&emsp;③ 带有`OUTPUT`的`UPDATE`语句
```sql
UPDATE dbo.OrderDetails
  SET discount = discount + 0.05
OUTPUT
  inserted.productid,
  deleted.discount AS olddiscount,
  inserted.discount AS newdiscount
WHERE productid = 51;
```

&emsp;&emsp;④ 带有`OUTPUT`的`MERGE`语句
```sql
MERGE INTO dbo.Customers AS TGT
USING dbo.CustomersStage AS SRC
  ON TGT.custid = SRC.custid
WHEN MATCHED THEN
  UPDATE SET
    TGT.companyname = SRC.companyname,
    TGT.phone = SRC.phone,
    TGT.address = SRC.address
WHEN NOT MATCHED THEN 
  INSERT (custid, companyname, phone, address)
  VALUES (SRC.custid, SRC.companyname, SRC.phone, SRC.address)
OUTPUT $action, inserted.custid,
  deleted.companyname AS oldcompanyname,
  inserted.companyname AS newcompanyname,
  deleted.phone AS oldphone,
  inserted.phone AS newphone,
  deleted.address AS oldaddress,
  inserted.address AS newaddress;
```

&emsp;&emsp;以上`MERGE`语句使用`OUTPUT`子句返回被修改过的行的新旧版本的值。对于`INSERT`操作不存在旧版本的值，因此所有 deleted 列的值都返回 *NULL* 。`$action`函数会告诉我们输出行是`UPDATE`还是由`INSERT`操作生成的。
![ ](29.png)

---

# 事务和并发
## 事务
### 事务的概念
&emsp;&emsp;事务是作为单个工作单元而执行的一系列操作，比如查询和修改数据等。  
&emsp;&emsp;事务是数据库并发控制的基本单位，一条或者一组语句要么全部成功，对数据库中的某些数据成功修改；要么全部不成功，数据库中的数据还原到这些语句执行之前的样子。

> &emsp;&emsp;比如网上订火车票，要么你定票成功，余票显示就减一张；要么你定票失败获取取消订票，余票的数量还是那么多。不允许出现你订票成功了，余票没有减少或者你取消订票了，余票显示却少了一张的这种情况。这种不被允许出现的情况就要求购票和余票减少这两个不同的操作必须放在一起，成为一个完整的逻辑链，这样就构成了一个事务。

### 事务的 ACID 特性
* **原子性（Atomicity）**：  
&emsp;&emsp;事务的原子性是指一个事务中包含的一条语句或者多条语句构成了一个完整的逻辑单元，这个逻辑单元具有不可再分的原子性。这个逻辑单元要么一起提交执行全部成功，要么一起提交执行全部失败。  
* **一致性（Consistency）**：  
&emsp;&emsp;可以理解为数据的完整性，事务的提交要确保在数据库上的操作没有破坏数据的完整性，比如说不要违背一些约束的数据插入或者修改行为。一旦破坏了数据的完整性，SQL Server 会回滚这个事务来确保数据库中的数据是一致的。  
* **隔离性（Isolation）**：  
&emsp;&emsp;与数据库中的事务隔离级别以及锁相关，多个用户可以对同一数据并发访问而又不破坏数据的正确性和完整性。但是，并行事务的修改必须与其它并行事务的修改相互独立，隔离。 但是在不同的隔离级别下，事务的读取操作可能得到的结果是不同的。  
* **持久性（Durability）**：  
&emsp;&emsp;数据持久化，事务一旦对数据的操作完成并提交后，数据修改就已经完成，即使服务重启这些数据也不会改变。相反，如果在事务的执行过程中，系统服务崩溃或者重启，那么事务所有的操作就会被回滚，即回到事务操作之前的状态。

> &emsp;&emsp;在极端断电或者系统崩溃的情况下，一个发生在事务未提交之前，数据库应该记录了这个事务的 "ID" 和部分已经在数据库上更新的数据。供电恢复数据库重新启动之后，这时完成全部撤销和回滚操作。如果在事务提交之后的断电，有可能更改的结果没有正常写入磁盘持久化，但是有可能丢失的数据会通过事务日志自动恢复并重新生成以写入磁盘完成持久化。

### 如何定义事务
（1）显示定义：以`BEGIN TRAN`开始，提交的话则`COMMIT`提交事务，否则以`ROLLBACK`回滚事务。
```sql
--定义事务
BEGIN TRAN;
  INSERT INTO dbo.T1(keycol, col1, col2) VALUES(4,101,'C');
  INSERT INTO dbo.T1(keycol, col1, col2) VALUES(4,201,'X');
COMMIT TRAN;
```

（2）隐式定义：SQL Server 中默认把每个单独的语句作为一个事务。  
&emsp;&emsp;换句话说，SQL Server 默认在执行完每个语句之后就自动提交事务。当然，我们可以通过`IMPLICIT_TRANSACTIONS`会话选项来改变 SQL Server 处理默认事务的方式，该选项默认情况下是 *OFF* 。如果将其设置为 *ON* ，那么就不必用`BEGIN TRAN`语句来表明事务开始，但仍然需要以`COMMIT`或`ROLLBACK`来标明事务完成。

## 锁定和阻塞
### 锁
（1）锁是什么鬼？  
&emsp;&emsp;锁是事务获取的一种控制资源，用于保护数据资源，防止其他事务对数据进行冲突的或不兼容的访问。

（2）锁模式及其兼容性  
&emsp;&emsp;主要有两种主要的锁模式 —— **排它锁（Exclusive Lock）** 和 **共享锁（Shared Lock）**。  
&emsp;&emsp;当试图修改数据时，事务会为所依赖的数据资源请求排它锁，一旦授予，事务将一直持有排它锁，直至事务完成。在事务执行过程中，其他事务就不能再获得该资源的任何类型的锁。  
&emsp;&emsp;当试图读取数据时，事务默认会为所依赖的数据资源请求共享锁，读操作一完成，就立即释放共享锁。在事务执行过程中，其他事务仍然能够获得该资源的共享锁。
![ ](30.png)

（3）可锁定资源的类型  
&emsp;&emsp;SQL Server 可以锁定不同类型或粒度的资源，这些资源类型包括 RID 或 KEY（行），PAGE（页）、对象（例如：表）及数据库等。

### 阻塞
（1）阻塞是个什么鬼？  
&emsp;&emsp;如果一个事务持有某一数据资源上的锁，而另一事务请求相同资源上的不兼容的锁，则对新锁的请求将被阻塞，发出请求的事务进入等待状态。默认情况下，被阻塞的请求会一直等待，直到原来的事务释放相关的锁。

> &emsp;&emsp;只要能够在合理的时间范围内满足请求，系统中的阻塞就是正常的。但是，如果一些请求等待了太长时间，可能就需要手工排除阻塞状态，看看能采取什么措施来防止这样长时间的延迟。

（2）近距离观测阻塞  
&emsp;&emsp;Step1. 打开两个独立的查询窗口，这里称之为 Connection A，Connection B

&emsp;&emsp;Step2. 在 Connection A 中运行以下代码（这里 productid=2 的 unitprice 本来为 19）
```sql
BEGIN TRAN;
  UPDATE Production.Products SET unitprice=unitprice+1.00
  WHERE productid=2;
```
&emsp;&emsp;为了更新这一行，会话必须先获得一个排它锁，如果更新成功，SQL Server 会向会话授予这个锁。

&emsp;&emsp;Step3. 在 Connection B 中运行以下代码
```sql
SELECT productid, unitprice
FROM Production.Products
WHERE productid=2;
```

&emsp;&emsp;默认情况下，该会话需要一个共享锁，但因为共享锁和排它锁是不兼容的，所以该会话被阻塞，进入等待状态。
![ ](31.png)

（3）如何检测阻塞  
&emsp;&emsp;假设我们的系统里边出现了阻塞，而且被阻塞了很长时间，如何去检测和排除呢？  
&emsp;&emsp;① 继续上例，打开一个新的会话，称之为 Connection C，查询动态管理视图（DMV）`sys.dm_tran_locks`：
```sql
-- Lock info
SELECT -- use * to explore
  request_session_id            AS spid,
  resource_type                 AS restype,
  resource_database_id          AS dbid,
  DB_NAME(resource_database_id) AS dbname,
  resource_description          AS res,
  resource_associated_entity_id AS resid,
  request_mode                  AS mode,
  request_status                AS status
FROM sys.dm_tran_locks;
```

&emsp;&emsp;② 运行上面的代码，可以得到以下输出：
![ ](32.png)

&emsp;&emsp;③ 每个会话都有唯一的服务器进程标识符（SPID），可以通过查询`@@SPID`函数来查看会话 ID。另外，当前会话的 SPID 还可以在查询窗口的标题栏中找到。
![ ](33.png)

&emsp;&emsp;④ 在前面查询的输出中，可以观察到进程 53 正在等待请求 TSQLFundamental2008 数据库中一个行的共享锁。但是，进程 52 持有同一个行上的排它锁。沿着 52 和 53 的所层次结构向上检查：（查询`sys.dm_exec_connections`的动态管理视图，筛选阻塞链中涉及到的那些 SPID）
```sql
-- Connection info
SELECT -- use * to explore
  session_id AS spid,
  connect_time,
  last_read,
  last_write,
  most_recent_sql_handle
FROM sys.dm_exec_connections
WHERE session_id IN(52, 53);
```

&emsp;&emsp;查询结果输出如下：
![ ](34.png)

&emsp;&emsp;⑤ 借助交叉联接，和`sys.dm_exec_sql_text`表函数生成查询结果：
```sql
-- SQL text
SELECT session_id, text 
FROM sys.dm_exec_connections
  CROSS APPLY sys.dm_exec_sql_text(most_recent_sql_handle) AS ST 
WHERE session_id IN(52, 53);
```

&emsp;&emsp;查询结果如下，我们可以达到阻塞链中涉及到的每个联接最后调用的批处理代码：
![ ](35.png)
&emsp;&emsp;以上就显示了进程 53 正在等待的执行代码，因为这是该进程最后执行的一个操作。对于阻塞进程来说，通过这个例子能够看到是哪条语句导致了问题。

（4）如何解除阻塞  
&emsp;&emsp;① 设置超时时间  
&emsp;&emsp;首先取消掉原来 Connection B 中的查询，然后执行以下代码：这里我们限制会话等待释放锁的时间为 5 秒：
```sql
-- Session B
SET LOCK_TIMEOUT 5000;

SELECT productid, unitprice
FROM Production.Products
WHERE productid=2;
```

&emsp;&emsp;然后 5 秒之后我们可以看到以下执行结果：
![ ](36.png)
&emsp;&emsp;注意：锁定超时不会引发事务回滚。

&emsp;&emsp;② `KILL`掉引起阻塞的进程  
&emsp;&emsp;在 Connection C 中执行以下语句，终止 SPID=52 中的更新事务而产生的效果，于是 SPID=52 中的事务的回滚，同时释放排它锁。
```sql
--KILL SPID=52
KILL 52;
```

&emsp;&emsp;这时再在 Connection B 中执行查询，便可以查到回滚后的结果（仍然是 19）：
![ ](37.png)

## 隔离级别
&emsp;&emsp;隔离级别用于决定如何控制并发用户读写数据的操作。前面说到，读操作默认使用共享锁，写操作需要使用排它锁。对于操作获得的锁，以及锁的持续时间来说，虽然不能控制写操作的处理方式，但可以控制读操作的处理方式。作为对读操作的行为进行控制的一种结果，也会隐含地影响写操作的行为方式。  
&emsp;&emsp;为此，可以在会话级别上用会话选项来设置隔离级别，也可以在查询级别上用表提示（Table Hint）来设置隔离级别。  
&emsp;&emsp;在 SQL Server 中，可以设置的隔离级别有 6 个：`READ UNCOMMITED`（未提交读）、`READ COMMITED`（已提交读）、`REPEATABLE READ`（可重复读）、`SERIALIZEABLE`（可序列化）、`SNAPSHOT`（快照）和`READ COMMITED SNAPSHOT`（已经提交读隔离）。最后两个`SNAPSHOT`和`READ COMMITED SNAPSHOT`是在 SQL Server 2005 中引入的。  
&emsp;&emsp;要设置整个会话级别的隔离级别，可以使用以下语句：
```sql
SET TRANSACTION ISOLATION LEVEL <isolation name>;
```

&emsp;&emsp;也可以使用表提示来设置查询级别的隔离级别：
```sql
SELECT ... FROM <table> WITH <isolation name>;
```

### READ UNCOMMITED 未提交读
&emsp;&emsp;未提交读是最低的隔离级别，读操作不会请求共享锁。换句话说，在该级别下的读操作正在读取数据时，写操作可以同时对这些数据进行修改。

&emsp;&emsp;同样，使用两个会话来模拟：  
&emsp;&emsp;Step1. 在 Connection A 中运行以下代码，更新产品 2 的单价，为当前值（19.00）增加 1.00，然后查询该产品：
```sql
-- Connection A
BEGIN TRAN;

UPDATE Production.Products
SET unitprice = unitprice + 1.00
WHERE productid = 2;

SELECT productid, unitprice
FROM Production.Products
WHERE productid = 2;
```
![ ](38.png)

&emsp;&emsp;Step2. 在 Connection B 中运行以下代码，首先设置隔离级别为未提交读，再查询产品 2 所在的记录：
```sql
-- Connection B
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SELECT productid, unitprice
FROM Production.Products
WHERE productid = 2;
```

&emsp;&emsp;因为这个读操作不用请求共享锁，因此不会和其他事务发生冲突，该查询返回了如下图所示的修改后的状态，即使这一状态还没有被提交：
![ ](39.png)

&emsp;&emsp;Step3. 在 Connection A 中运行以下代码回滚事务：
```sql
ROLLBACK TRAN;
```

> &emsp;&emsp;这个回滚操作撤销了对产品 2 的更新，这时它的价格被修改回了 19.00，但是读操作此前获得的 20.00 再也不会被提交了。这就是**脏读**的一个实例！

![ ](40.png)

### READ COMMITED 已提交读
&emsp;&emsp;刚刚说到，未提交到会引起脏读，能够防止脏读的最低隔离级别是已提交读，这也是所有 SQL Server 版本默认使用的隔离级别。如其名称所示，这个隔离级别只允许读取已经提交的修改，它要求读操作必须获得共享锁才能操作，从而防止读取未提交的修改。

&emsp;&emsp;继续使用两个会话来模拟：  
&emsp;&emsp;Step1. 在 Connection A 中运行以下代码，更新产品 2 的价格，再查询显示价格：
```sql
BEGIN TRAN;

UPDATE Production.Products
SET unitprice = unitprice + 1.00
WHERE productid = 2;

SELECT productid, unitprice
FROM Production.Products
WHERE productid = 2;
```
![ ](41.png)

&emsp;&emsp;Step2. 再在 Connection B 中运行以下代码，这段代码将会话的隔离级别设置为已提交读，再查询产品 2 所在的行记录：
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SELECT productid, unitprice
FROM Production.Products
WHERE productid = 2;
```

&emsp;&emsp;这时该会话语句会被阻塞，因为它需要获取共享锁才能进行读操作，而它与会话 A 的写操作持有的排它锁相冲突。这里因为我设置了默认会话阻塞超时时间，所以出现了以下输出：
![ ](42.png)

&emsp;&emsp;Step3. 在 Connection A 中运行以下代码，提交事务：
```sql
COMMIT TRAN;
```

&emsp;&emsp;Step4. 回到 Connection B，此时会得到以下输出：
![ ](43.png)

> &emsp;&emsp;在已提交读级别下，不会读取脏数据，只能读取已经提交过的修改。但是，该级别下，其他事务可以在两个读操作之间更改数据资源，读操作因而可能每次得到不同的取值。这种现象被称为**不可重复读**。

### REPEATABLE READ 可重复读
&emsp;&emsp;如果想保证在事务内进行的两个读操作之间，其他任何事务都不能修改由当前事务读取的数据，则需要将隔离级别升级为可重复读。在该级别下，十五中的读操作不但需要获得共享锁才能读数据，而且获得的共享锁将一直保持到事务完成为止。换句话说，在事务完成之前，没有其他事务能够获得排它锁以修改这一数据资源，由此来保证实现可重复的读取。  
&emsp;&emsp;Step1. 为了重新演示可重复读的示例，首先需要将刚刚的测试数据清理掉，在 Connection A 和 B 中执行以下代码：
```sql
-- Clear Test Data
UPDATE Production.Products
SET unitprice = 19.00
WHERE productid = 2;
```

&emsp;&emsp;Step2. 在 Connection A 中运行以下代码，将会话的隔离级别设置为可重复读，再查询产品 2 所在的行记录：
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
```

![ ](44.png)
&emsp;&emsp;这时该会话仍然持有产品 2 上的共享锁，因为在该隔离级别下，共享锁要一直保持到事务结束为止。

&emsp;&emsp;Step3. 在 Connection B 中尝试对产品 2 这一行进行修改：
```sql
UPDATE Production.Products
  SET unitprice = unitprice + 1.00
WHERE productid = 2;
```

&emsp;&emsp;这时该会话已被阻塞，因为修改操作锁请求的排它锁与前面会话授予的共享锁有冲突。换句话说，如果读操作是在未提交读或已提交读级别下运行的，那么事务此时将不再持有共享锁，Connection B 尝试修改改行的操作应该能够成功。  
&emsp;&emsp;同样，由于我设置了超时释放时间，因此会有以下输出：
![ ](45.png)

&emsp;&emsp;Step4. 回到 Connection A，运行以下代码，再次查询产品 2 所在的行，提交事务：
```sql
  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;

COMMIT TRAN;
```

&emsp;&emsp;这时的返回结果仍然与第一次相同：
![ ](46.png)

&emsp;&emsp;Step5. 这时再执行 Connection B 中的更新语句，便能够正常获得排它锁了，于是执行成功，价格变为了 20.00。

> &emsp;&emsp;可重复读隔离级别不仅可以**防止不可重复读**，另外还能**防止丢失更新**。丢失更新是指两个事务读取了同一个值，然后基于最初读取的值进行计算，接着再更新该值，就会发生丢失更新的问题。这是因为在可重复读隔离级别下，两个事务在第一次读操作之后都保留有共享锁，所以其中一个都不能成功获得为了更新数据而需要的排它锁。但是，**负面影响就是会导致死锁**。  
> &emsp;&emsp;在可重复读级别下运行的事务，读操作获得的共享锁将一直保持到事务结束。因此可以保证在事务中第一次读取某些行后，还可以重复读取这些行。但是，事务只锁定查询第一次运行时找到的那些行，而不会锁定查询结果范围外的其他行。因此，在同一事务进行第二次读取之前，如果其他事务插入了新行，而且新行也能满足读操作额查询过滤条件，那么这些新行也会出现在第二次读操作返回的结果中。这些新行称之为幻影，这种读操作也被称为**幻读**。

### SERIALIZEABLE 可序列化
&emsp;&emsp;为了避免刚刚提到的幻读，需要将隔离级别设置为可序列化。可序列化级别的处理方式与可重复读类似：读操作需要获得共享锁才能读取数据并一直保留到事务结束，不同之处在于在可序列化级别下，读操作不仅锁定了满足查询条件的那些行，还锁定了可能满足查询条件的行。换句话说，如果其他事务试图增加能够满足操作的查询条件的新行，当前事务就会阻塞这样的操作。

&emsp;&emsp;同样，继续来模拟：  
&emsp;&emsp;Step1. 在 Connection A 中运行代码，设置隔离级别为可序列化，再查询产品分类等于 1 的所有产品：
```sql
-- Connection A
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

BEGIN TRAN

  SELECT productid, productname, categoryid, unitprice
  FROM Production.Products
  WHERE categoryid = 1;
```
![ ](47.png)

&emsp;&emsp;Step2. 在 Connection B 中运行代码，尝试插入一个分类等于 1 的新产品：
```sql
-- Connection B
INSERT INTO Production.Products
    (productname, supplierid, categoryid,
     unitprice, discontinued)
  VALUES('Product ABCDE', 1, 1, 20.00, 0);
```

&emsp;&emsp;这时，该操作会被阻塞。因为在可序列化级别下，前面的读操作不仅锁定了满足查询条件的那些行，还锁定了可能满足查询条件的行。  
&emsp;&emsp;同样，由于我设置了超时释放时间，因此会有以下输出：
![ ](48.png)

&emsp;&emsp;Step3. 回到 Connection A，运行以下代码，再次查询分类 1 的产品，最后提交事务：
```sql
  SELECT productid, productname, categoryid, unitprice
  FROM Production.Products
  WHERE categoryid = 1;

COMMIT TRAN;
```

&emsp;&emsp;Step4. 回到 Connection B，这时 Connection B 就已经获得了等候已久的排它锁，插入了新行。
```sql
INSERT INTO Production.Products
    (productname, supplierid, categoryid,
     unitprice, discontinued)
  VALUES('Product ABCDE', 1, 1, 20.00, 0);

SELECT productid, productname, categoryid, unitprice
FROM Production.Products
WHERE categoryid = 1;
```
![ ](49.png)

&emsp;&emsp;Step5. 为了后面的演示，运行以下代码清理测试数据：
```sql
-- Cleanup
DELETE FROM Production.Products
WHERE productid > 77;

DBCC CHECKIDENT ('Production.Products', RESEED, 77);
```

### SNAPSHOT 快照
&emsp;&emsp;首先解释一下什么是快照？事务已经提交的行的上一个版本存在`tempdb`数据库中，这是 SQL Server 引入的一个新功能。  
&emsp;&emsp;以这种行版本控制技术为基础，SQL Server 增加了两个新的隔离级别：`SNAPSHOT`和`READ COMMITED SNAPSHOT`。如果启用任何一种基于快照的隔离级别，`DELETE`和`UPDATE`语句在做出修改前都会把行的当前版本复制到`tempdb`数据库中；`INSERT`语句则不会，因为这时还没有行的旧版本。  
&emsp;&emsp;在`SNAPSHOPT`（快照）隔离级别下，当读取数据时，可以保证读操作**读取的行是事务开始时可用的最后提交的版本**。

&emsp;&emsp;下面来模拟一下该隔离级别下的场景：  
&emsp;&emsp;Step1. 还是打开两个会话窗口，在其中一个执行以下代码，设置隔离级别为`SNAPSHOT`：
```sql
-- Allow SNAPSHOT isolation in the database
ALTER DATABASE TSQLFundamentals2008 SET ALLOW_SNAPSHOT_ISOLATION ON;
```

&emsp;&emsp;Step2. 在 Connection A 中运行以下代码，更新产品 2 的价格，然后再查询该产品的价格：
```sql
-- Connection A
BEGIN TRAN;

  UPDATE Production.Products
    SET unitprice = unitprice + 1.00
  WHERE productid = 2;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
```
![ ](50.png)

&emsp;&emsp;Step3. 在 Connection B 中运行以下代码，设置隔离级别为`SNAPSHOT`，并查询产品 2 的价格：
```sql
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
```

&emsp;&emsp;这时的返回结果如下所示，可以看到这个结果是在该事务启动时可用的最后提交的版本。
![ ](51.png)

&emsp;&emsp;Step4. 回到 Connection A 提交这一修改的行：
```sql
COMMIT TRAN;
```

&emsp;&emsp;Step5. 在 Connection B 中运行以下代码，再次读取数据，然后提交事务：
```sql
  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
  
COMMIT TRAN;
```

&emsp;&emsp;然后我们会得到跟之前一样的结果，奇了个怪了：
![ ](52.png)

&emsp;&emsp;但是如果我们再次在 Connection B 中运行以下完整语句：
```sql
BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
  
COMMIT TRAN;
```

&emsp;&emsp;这时结果便会同步，这个事务开始时可用的上一个提交的版本是价格 = 20.00
![ ](53.png)
&emsp;&emsp;为什么两个事务得到结果会不同？这是因为快照清理线程每隔一分钟运行一次，现在由于没有事务需要为价格 = 20.00 的那个行版本了，所以清理线程下一次运行时会将这个行版本从`tempdb`数据库中删除掉。

&emsp;&emsp;最后，为了下一次演示，清理测试数据：
```sql
-- Clear Test Data
UPDATE Production.Products
SET unitprice = 19.00
WHERE productid = 2;
```

> &emsp;&emsp;这一隔离级别使用的不是共享锁，而是行版本控制。如前所述，不论修改操作（主要是更新和删除数据）是否在某种基于快照的隔离级别下的会话执行，快照隔离级别都会带来性能上的开销。

&emsp;&emsp;另外，在`SNAPSHOT`快照级别下，可以通过检查的行版本，检测出更新冲突。它能判断出在快照事务的一次读操作和一次写操作之间是否有其他事务修改过数据。如果 SQL Server 检测到在读取和写入操作之间有另一个事务修改了数据，则会让事务因失败而终止，并返回以下错误信息：
![ ](54.png)

&emsp;&emsp;冲突检测完整实例如下：
```sql
---------------------------------------------------------------------
-- Conflict Detection 冲突检测实例
---------------------------------------------------------------------

-- Connection A, Step 1
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;

BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;

-- Connection A, Step 2
  UPDATE Production.Products
    SET unitprice = 20.00
  WHERE productid = 2;
  
COMMIT TRAN;

-- Cleanup
UPDATE Production.Products
  SET unitprice = 19.00
WHERE productid = 2;

-- Connection A, Step 1
BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;

-- Connection B, Step 1
UPDATE Production.Products
  SET unitprice = 25.00
WHERE productid = 2;

-- Connection A, Step 2
  UPDATE Production.Products
    SET unitprice = 20.00
  WHERE productid = 2;

-- Cleanup
UPDATE Production.Products
  SET unitprice = 19.00
WHERE productid = 2;

-- Close all connections
```

### READ COMMITED SNAPSHOT 已经提交读隔离
&emsp;&emsp;已提交读隔离也是基于行版本控制，但与快照不同之处在于：在已提交读级别下，读操作读取的数据行不是食物启动之前最后提交的版本，而是语句启动前最后提交的版本。  
&emsp;&emsp;此外，该级别不会像快照隔离级别一样进行更新冲突检测。这样一来，它就跟 SQL Server 默认的`READ COMMITED`级别非常类似了，只不过**读操作不用获得共享锁，当请求的资源被其他事务的排它锁锁定时，也不用等待**。

&emsp;&emsp;下面继续通过案例来模拟：  
&emsp;&emsp;Step1. 运行以下代码，设置隔离级别：
```sql
-- Turn on READ_COMMITTED_SNAPSHOT
ALTER DATABASE TSQLFundamentals2008 SET READ_COMMITTED_SNAPSHOT ON;
```

&emsp;&emsp;执行该查询需要一定的时间，并且要注意：要成功运行，当前连接必须是指定数据库的唯一连接，请关掉其他连接，只保留一个会话来执行。  
&emsp;&emsp;可以看到它跟我们之前设置隔离级别所使用的的语句不同，这个选项其实就是把默认的`READ COMMITED`的寒意变成了`READ COMMITED SNAPSHOT`。意味着打开这个选项时，除非显式地修改会话的隔离级别，否则`READ COMMITED SNAPSHOT`将成为默认的隔离级别。

&emsp;&emsp;Step2. 在 Connection A 中运行以下代码，更新产品 2 所在的行记录，再读取这一行记录，并且一直保持事务打开：
```sql
-- Connection A
USE TSQLFundamentals2008;

BEGIN TRAN;

  UPDATE Production.Products
    SET unitprice = unitprice + 1.00
  WHERE productid = 2;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
```
![ ](55.png)

&emsp;&emsp;Step3. 在 Connection B 中读取产品 2 所在的行记录，并一直保持事务打开：
```sql
-- Connection B
BEGIN TRAN;

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;
```

&emsp;&emsp;得到的结果是语句启动之前最后提交的版本（19.00）：
![ ](56.png)

&emsp;&emsp;Step4. 回到 Connection A，提交事务：
```sql
COMMIT TRAN;
```

&emsp;&emsp;Step5. 回到 Connection B，再次读取产品 2 所在的行，并提交事务：
```sql
  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;

COMMIT TRAN;
```

&emsp;&emsp;这时结果如下，可以看到跟`SNAPSHOT`不同，这次的结果是在语句执行之前最后提交的版本而不是事务执行之前最后提交的版本，因此得到了 20.00：
![ ](57.png)

> &emsp;&emsp;回想一下，这种现象是不是我们常听见的**不可重复读**？也就是说，该级别下，无法防止不可重复读问题。

&emsp;&emsp;最后，按照国际惯例，清理测试数据：
```sql
-- Clear Test Data
UPDATE Production.Products
SET unitprice = 19.00
WHERE productid = 2;
```

&emsp;&emsp;然后，关闭所有连接，然后在一个新的连接下运行以下代码，以禁用指定数据库的基于快照的隔离级别：（执行`ALTER DATABASE TSQLFundamentals2008 SET READ_COMMITTED_SNAPSHOT OFF;`这一句时可能需要花费一点时间，请耐心等候）
```sql
-- Make sure you're back in default mode
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Change database options to default
ALTER DATABASE TSQLFundamentals2008 SET ALLOW_SNAPSHOT_ISOLATION OFF;
ALTER DATABASE TSQLFundamentals2008 SET READ_COMMITTED_SNAPSHOT OFF;
```

### 隔离级别总结
&emsp;&emsp;下表总结了每种隔离级别能够解决各种逻辑一致性的问题，以及隔离级别是否会检测更新冲突，是否使用了行版本控制。
![ ](58.png)
![ ](59.png)

&emsp;&emsp;这时再回顾以下各个问题的描述及结果，我们来看另一个表：
![ ](60.png)

## 死锁
### 死锁是个什么鬼？
&emsp;&emsp;死锁是指一种**进程之间互相永久阻塞的状态**，可能涉及到两个或者多个进程。两个进程发生死锁的例子是：进程 A 阻塞了进程 B，进程 B 又阻塞了进程 A。在任何一种情况下，SQL Server 都可以检测到死锁，并选择终止其中一个事务以干预死锁状态。如果 SQL Server 不干预，那么死锁涉及到的进程将会永远保持死锁状态。
![ ](61.png)

&emsp;&emsp;默认情况下，SQL Server 会选择终止做过的操作最少的事务，因为这样可以让回滚开销降低到最低。当然，在 SQL Server 2005 及之后的版本中，可以通过将会话选项`DEADLOCK_PRIORITY`设置为范围（-10 到 10）之间的任一整数值。

### 死锁实例
&emsp;&emsp;仍然打开三个会话：Connection A、B 和 C：  
&emsp;&emsp;Step1. 在 Connection A 中更新 Products 表中产品 2 的行记录，并保持事务一直打开：
```sql
-- Connection A
USE TSQLFundamentals2008;

BEGIN TRAN;

  UPDATE Production.Products
    SET unitprice = unitprice + 1.00
  WHERE productid = 2;
```
&emsp;&emsp;这时 Connection A 对产品表的产品 2 请求了排它锁。

&emsp;&emsp;Step2. 在 Connection B 中更新 OrderDetails 表中产品 2 的订单明细，并保持事务一直打开：
```sql
-- Connection 2
BEGIN TRAN;

  UPDATE Sales.OrderDetails
    SET unitprice = unitprice + 1.00
  WHERE productid = 2;
```
&emsp;&emsp;这时 Connection A 对订单明细表的产品 2 请求了排它锁。

&emsp;&emsp;Step3. 回到 Connection A 中，执行以下语句，请求查询产品 2 的订单明细记录：
```sql
-- Connection A

  SELECT orderid, productid, unitprice
  FROM Sales.OrderDetails
  WHERE productid = 2;

COMMIT TRAN;
```
&emsp;&emsp;由于此时实在默认的`READ COMMITED`隔离级别下运行的，所以 Connection A 中的事务需要一个共享锁才能读数据，因此这里会一直阻塞住。但是，此时并没有发生死锁，而只是发生了阻塞。

&emsp;&emsp;Step4. 回到 Connection B 中，执行以下语句，尝试在 Products 表查询产品 2 的记录：
```sql
-- Connection 2

  SELECT productid, unitprice
  FROM Production.Products
  WHERE productid = 2;

COMMIT TRAN;
```

&emsp;&emsp;这里由于这个请求和 Connection A 中的事务在同一个资源上持有的排它锁发生了冲突，于是相互阻塞发生了死锁。SQL Server 通常会在几秒钟之内检测到死锁，并从这两个进程中选择一个作为牺牲品，终止其事务。所以我们还是得到了以下结果：
![ ](62.png)

&emsp;&emsp;Step5. 刚刚提到了 SQL Server 会选择一个作为牺牲品，我们回到 Connection A 会看到以下的错误信息提示：
![ ](63.png)

&emsp;&emsp;在这个例子中，由于两个事务进行的工作量差不多一样，所以任何一个事务都有可能被终止。（前面提到，如果没有手动设置优先级，那么 SQL Server 会选择工作量较小的一个事务作为牺牲品）另外，解除死锁需要一定的系统开销，因为这个过程会涉及撤销已经执行过的处理。

> &emsp;&emsp;显然，事务处理的时间越长，持有锁的时间也就越长，死锁的可能性也就越大。应该尽量**保持事务简短**，把逻辑上可以属于同一工作单元的操作移到事务之外。

### 避免死锁
（1）**改变访问资源的顺序可以避免死锁**  
&emsp;&emsp;继续上面的例子，Connection A 先访问 Products 表中的行，然后访问 OrderDetails 表中的行；Connection B 先访问 OrderDetails 表中的行，然后访问 Products 表中的行。  
&emsp;&emsp;这时如果我们改变一下访问顺序：两个事务按照同样的顺序来访问资源，则不会发生这种类型的死锁。
> &emsp;&emsp;通过交换其中一个事务的操作顺序，就可以避免发生这种类型的死锁（假设交换顺序不必改变程序的逻辑）。

（2）**良好的索引设计也可以避免死锁**  
&emsp;&emsp;如果查询筛选条件缺少良好的索引支持，也会造成死锁。例如，假设 Connection B 中的事务有两条语句要对产品 5 进行筛选，Connection A 中的事务要对产品 2 进行处理，那么他们就不应该有任何冲突。但是，如果在表的 productid 列上如果没有索引来支持查询筛选，那么 SQL Server 就必须扫描（并锁定）表中的所有行，这样当然会导致死锁。
> &emsp;&emsp;总之，良好的索引设计将有助于减少这种没有真正的逻辑冲突的死锁。

&emsp;&emsp;最后，按照国际惯例清理掉测试数据：
```sql
-- Cleanup
UPDATE Production.Products
  SET unitprice = 19.00
WHERE productid = 2;

UPDATE Sales.OrderDetails
  SET unitprice = 19.00
WHERE productid = 2
  AND orderid >= 10500;

UPDATE Sales.OrderDetails
  SET unitprice = 15.20
WHERE productid = 2
  AND orderid < 10500;
```

---

# 可编程对象
## 变量与批处理
（1）变量：`DECLARE`+`SET`/`SELECT`  
&emsp;&emsp;`DECLARE`语句可以声明一个或多个变量，然后使用`SET`/`SELECT`语句可以把一个变量设置成指定的值。  
&emsp;&emsp;① `SET`语句每次只能针对一个变量进行操作
```sql
--set方式
declare @i as int
set @i=10;

--SQL Server 2008可以在同一语句同时声明和初始化变量
declare @i as int = 10;
```

&emsp;&emsp;② `SELECT`语句允许从同一行中获得的多个值分配给多个变量。
```sql
--select方式
declare @firstname as nvarchar(20), @lastname as nvarchar(40);

select
  @firstname = firstname,
  @lastname = lastname
from hr.Employees
where empid=3;

select @firstname as firstname, @lastname as lastname;
```

&emsp;&emsp;`SET`语句比复制`SELECT`语句更加安全，因为它要求使用标量子查询来从表中提取数据。如果在运行时，标量子查询返回了多个值，则查询会失败。例如下面的代码在运行时会报错：
```sql
--set比select语句更安全
declare @empname as nvarchar(61);

set @empname = (select firstname + N' '+ lastname
                from hr.Employees
                where mgrid=2);
                
select @empname as empname;
```
![ ](64.png)

（2）批处理  
&emsp;&emsp;客户端应用程序发送到 SQL Server 的一组单条或多条 T-SQL 语句，SQL Server 将批处理语句作为单个可执行的单元。
![ ](65.png)

&emsp;&emsp;下面是一个批处理的示例，但要注意的是如果批处理中存在语法错误，整个批处理是不会提交到 SQL Server 执行的。
```sql
-- A Batch as a Unit of Parsing
-- Valid batch
PRINT 'First batch';
USE TSQLFundamentals2008;
GO
-- Invalid batch
PRINT 'Second batch';
SELECT custid FROM Sales.Customers;
SELECT orderid FOM Sales.Orders; -- 这一句有语法错误，故整个批处理不能提交到SQL Server执行
GO
-- Valid batch
PRINT 'Third batch';
SELECT empid FROM HR.Employees;
GO
```

> &emsp;&emsp;**Tips**：批处理和事务不同，事务是工作的原子工作单元，而一个批处理可以包含多个事务，一个事务也可以在多个批处理中的某些部分提交。当事务在执行中被取消或者回滚时，SQL Server 会撤销自事务开始以来的部分活动，而不考虑批处理是从哪里开始的。

## 流程控制
（1）`IF`...`ELSE`  
&emsp;&emsp;这个大家应该都知道，但是需要注意的是：T-SQL 使用的是三值逻辑，当条件取值为 *FALSE* 或 *UNKNOWN* 时，都可以激活`ELSE`语句块。如果条件取值可能为 *FALSE* 或 *UNKNOWN* （例如，涉及到 *NULL* 值），而且对每种情况需要进行不同的处理时，必须用`IS NULL`谓词对 *NULL* 值进行显式地测试。  
&emsp;&emsp;下面的`IF-ELSE`代码演示了：如果今天是一个月的第一天，则对数据库进行完整备份；如果今天是一个月的最后一天，则对数据库进行差异备份（所谓差异备份，就是指只保存上一次完整备份以来做过的更新）。
```sql
IF DAY(CURRENT_TIMESTAMP) = 1
BEGIN
  PRINT 'Today is the first day of the month.';
  PRINT 'Starting a full database backup.';
  BACKUP DATABASE TSQLFundamentals2008
    TO DISK = 'C:\Temp\TSQLFundamentals2008_Full.BAK' WITH INIT;
  PRINT 'Finished full database backup.';
END
ELSE
BEGIN
  PRINT 'Today is not the first day of the month.'
  PRINT 'Starting a differential database backup.';
  BACKUP DATABASE TSQLFundamentals2008
    TO DISK = 'C:\Temp\TSQLFundamentals2008_Diff.BAK' WITH INIT;
  PRINT 'Finished differential database backup.';
END
GO
```
&emsp;&emsp;这里假设备份的文件路径目录 C:Temp 已经存在。

（2）`WHILE`：不解释了，各位应该都懂。
```sql
DECLARE @i AS INT;
SET @i = 1;
WHILE @i <= 10
BEGIN
  PRINT @i;
  SET @i = @i + 1;
END;
GO
```

## 游标
&emsp;&emsp;T-SQL 中支持一种叫做游标的对象，可以用它来**处理查询返回的结果集中的各行，以指定的顺序一次只处理一行**。这种处理方式与使用基于集合的查询相反，普通的查询是把集合作为一个整体来处理，不依赖任何顺序。  
&emsp;&emsp;换句话说，使用游标，就像是用鱼竿钓鱼，一次只能勾到一条鱼一样。而使用集合，就像用渔网捕鱼，一次能捕到整整一网鱼。因此，使用游标的场景我们应该多多斟酌。一般来说，如果按固定顺序一次处理一行的游标方式涉及到的数据访问要比基于集合的方式少得多，则使用游标会更加有效，前一篇提到的连续聚合就是这样的一个例子。

&emsp;&emsp;如何使用游标呢？
![ ](66.png)

&emsp;&emsp;下面来看看一个实例，它使用游标来计算 CustOrders 视图中每个客户每个月的连续总订货量（连续聚合案例）：
```sql
-- Example: Running Aggregations
SET NOCOUNT ON;
USE TSQLFundamentals2008;

DECLARE @Result TABLE
(
  custid     INT,
  ordermonth DATETIME,
  qty        INT, 
  runqty     INT,
  PRIMARY KEY(custid, ordermonth)
);

DECLARE
  @custid     AS INT,
  @prvcustid  AS INT,
  @ordermonth DATETIME,
  @qty        AS INT,
  @runqty     AS INT;

DECLARE C CURSOR FAST_FORWARD /* read only, forward only */ FOR
  SELECT custid, ordermonth, qty
  FROM Sales.CustOrders
  ORDER BY custid, ordermonth;

OPEN C

FETCH NEXT FROM C INTO @custid, @ordermonth, @qty;

SELECT @prvcustid = @custid, @runqty = 0;

WHILE @@FETCH_STATUS = 0
BEGIN
  IF @custid <> @prvcustid
    SELECT @prvcustid = @custid, @runqty = 0;

  SET @runqty = @runqty + @qty;

  INSERT INTO @Result VALUES(@custid, @ordermonth, @qty, @runqty);
  
  FETCH NEXT FROM C INTO @custid, @ordermonth, @qty;
END

CLOSE C;

DEALLOCATE C;

SELECT 
  custid,
  CONVERT(VARCHAR(7), ordermonth, 121) AS ordermonth,
  qty,
  runqty
FROM @Result
ORDER BY custid, ordermonth;
GO
```

&emsp;&emsp;执行结果如下图所示：
![ ](67.png)

## 临时表
&emsp;&emsp;有时需要把数据临时保存到表中，而且在有些情况下，我们可能不太想要使用永久性的表。在这种情况下，使用临时表可能会更方便。  
（1）局部临时表：  
&emsp;&emsp;只对创建它的会话在创建级和对调用对战的内部级（内部的过程、函数、触发器等）是可见的，当创建会话从 SQL Server 实例断开时才会自动删除它。

&emsp;&emsp;创建临时局部表，只需要在命名时以单个`#`号作为前缀：
```sql
IF OBJECT_ID('tempdb.dbo.#MyOrderTotalsByYear') IS NOT NULL
  DROP TABLE dbo.#MyOrderTotalsByYear;
GO

SELECT
  YEAR(O.orderdate) AS orderyear,
  SUM(OD.qty) AS qty
INTO dbo.#MyOrderTotalsByYear
FROM Sales.Orders AS O
  JOIN Sales.OrderDetails AS OD
    ON OD.orderid = O.orderid
GROUP BY YEAR(orderdate);

SELECT Cur.orderyear, Cur.qty AS curyearqty, Prv.qty AS prvyearqty
FROM dbo.#MyOrderTotalsByYear AS Cur
  LEFT OUTER JOIN dbo.#MyOrderTotalsByYear AS Prv
    ON Cur.orderyear = Prv.orderyear + 1;
GO
```

（2）全局临时表：  
&emsp;&emsp;可以对其他所有会话都可见，当创建临时表的会话断开数据库的连接，而且也没有活动在引用全局临时表时，SQL Server 才会自动删除相应的全局临时表。

&emsp;&emsp;创建全局局部表，只需要在命名时以两个`#`号作为前缀：
```sql
-- Global Temporary Tables
CREATE TABLE dbo.##Globals
(
  id  sysname     NOT NULL PRIMARY KEY,
  val SQL_VARIANT NOT NULL
);
```

## 动态 SQL
&emsp;&emsp;SQL Server 允许用字符串来动态构造 T-SQL 代码的一个批处理，接着再执行这个批处理，这种功能叫做动态 SQL（Daynamic SQL）。

（1）使用`EXEC`（EXECUTE 的缩写）命令
```sql
-- Simple example of EXEC
DECLARE @sql AS VARCHAR(100);
SET @sql = 'PRINT ''This message was printed by a dynamic SQL batch.'';';
EXEC(@sql);
GO
```

（2）使用`sp_executesql`存储过程  
&emsp;&emsp;`sp_executesql`存储过程有两个输入参数和一个参数赋值部分：第一个参数需要指定包含想要运行的批处理代码的 Unicode 字符串，第二个参数是一个 Unicode 字符串，包含第一个参数中所有输入和输出参数的生命。接着为输入和输出参数指定取值，各参数之间用逗号分隔。
```sql
-- Simple example using sp_executesql
DECLARE @sql AS NVARCHAR(100);

SET @sql = N'SELECT orderid, custid, empid, orderdate
FROM Sales.Orders
WHERE orderid = @orderid;';

EXEC sp_executesql
  @stmt = @sql,
  @params = N'@orderid AS INT',
  @orderid = 10248;
GO
```

> **Tips**：  
> &emsp;&emsp;① `sp_executesql`存储过程在执行性能上比`EXEC`要好，因为它的参数化有助于重用缓存过的执行计划。
> &emsp;&emsp;② `sp_executesql`存储过程在安全上也比`EXEC`要好，它的参数化也可以不必受 SQL 注入的困扰。

## 例程：用户定义函数、存储过程与触发器
（1）用户定义函数：封装计算的逻辑处理，有可能需要基于输入的参数，并返回结果。  
&emsp;&emsp;下面的示例创建了一个用户定义函数 dbo.fn_age，对于给定出生日期和事件日期，这个函数可以返回某个人在时间`日期当时的年龄：
```sql
IF OBJECT_ID('dbo.fn_age') IS NOT NULL DROP FUNCTION dbo.fn_age;
GO

CREATE FUNCTION dbo.fn_age
(
  @birthdate AS DATETIME,
  @eventdate AS DATETIME
)
RETURNS INT
AS
BEGIN
  RETURN
    DATEDIFF(year, @birthdate, @eventdate)
    - CASE WHEN 100 * MONTH(@eventdate) + DAY(@eventdate)
              < 100 * MONTH(@birthdate) + DAY(@birthdate)
           THEN 1 ELSE 0
      END
END
GO
```

（2）存储过程：封装 T-SQL 代码地服务器端例程，可以有输入和输出参数，可以返回多个查询的结果集。  
&emsp;&emsp;下面的示例创建了一个存储过程 usp_GetCustomerOrders，它接受一个客户 ID 和日期范围作为输入参数，返回 Orders 表中由指定客户在指定日期范围内所下的订单组成的结果集，同时也将受查询影响的行为作为输出参数。
```sql
IF OBJECT_ID('Sales.usp_GetCustomerOrders', 'P') IS NOT NULL
  DROP PROC Sales.usp_GetCustomerOrders;
GO

CREATE PROC Sales.usp_GetCustomerOrders
  @custid   AS INT,
  @fromdate AS DATETIME = '19000101',
  @todate   AS DATETIME = '99991231',
  @numrows  AS INT OUTPUT
AS
SET NOCOUNT ON;

SELECT orderid, custid, empid, orderdate
FROM Sales.Orders
WHERE custid = @custid
  AND orderdate >= @fromdate
  AND orderdate < @todate;

SET @numrows = @@rowcount;
GO

DECLARE @rc AS INT;

EXEC Sales.usp_GetCustomerOrders
  @custid   = 1, -- Also try with 100
  @fromdate = '20070101',
  @todate   = '20080101',
  @numrows  = @rc OUTPUT;

SELECT @rc AS numrows;
GO
```

> &emsp;&emsp;**Tips**：存储过程可以封装业务逻辑处理，更好地控制安全性（有助于避免 SQL 注入），提高执行性能（减少网络通信流量）。

（3）触发器：  
&emsp;&emsp;一种特殊的存储过程，只要特定事件发生，就会调用触发器，运行它的代码。SQL Server 支持两种类型相关的触发器，分别是：DML 触发器 和 DDL 触发器。

&emsp;&emsp;下面的示例演示了一个简单的 DML 触发器，对插入到表的数据进行审核（插入到 Audit 审核表）。
```sql
CREATE TRIGGER trg_T1_insert_audit ON dbo.T1 AFTER INSERT
AS
SET NOCOUNT ON;

INSERT INTO dbo.T1_Audit(keycol, datacol)
  SELECT keycol, datacol FROM inserted;
GO
```

## 错误处理
&emsp;&emsp;T-SQL 代码中提供了一种成为 `TRY`...`CATCH` 的结构，在 SQL Server 2005 中引入的。
```sql
BEGIN TRY
  PRINT 10/2;
  PRINT 'No error';
END TRY
BEGIN CATCH
  PRINT 'Error';
END CATCH
GO
```
&emsp;&emsp;对于错误处理代码，在实际开发中，可以封装创建一个存储过程来重用错误代码。

---

# 参考资料
![[美] Itzik Ben-Gan 著，成保栋 译，《Microsoft SQL Server 2008 技术内幕：T-SQL 语言基础》](68.png)
