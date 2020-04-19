---

show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，熟悉MySQL的执行计划和CASE表达式

#### 请点击右侧选择使用的实验环境

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。

#### 概述

**执行计划**

我们知道，不管是哪种数据库，或者是哪种数据库引擎，在对一条SQL语句进行执行的过程中都会做很多相关的优化，对于查询语句，最重要的优化方式就是使用索引。而执行计划，就是显示数据库引擎对于SQL语句的执行的详细情况，其中包含了是否使用索引，使用什么索引，使用的索引的相关信息等。

**CASE表达式**

CASE表达式是一个流程控制结构，用在在SELECT、WHERE等语句中根据条件动态构造内容。

## 打开项目

#### 打开idea

打开idea代码开发工具

![1735-110-1.png](https://doc.shiyanlou.com/courses/1735/1207281/6f87a8c93937c3c51f6d4839559de710-0)

#### 打开SSQL-MySQL项目

打开SSQL-MySQL项目，在该课程中完成后续试验

![1735-110-13.png](https://doc.shiyanlou.com/courses/1735/1207281/40a9e7b6fbd5c3853dc09f69d0a06c86-0)

#### 打开lesson6_explainAndCase包

打开lesson6_explainAndCase packge，在该packge中完成后续课程。

![1735-160-1.png](https://doc.shiyanlou.com/courses/1735/1207281/68397f159f4e8f581e42d2e680ba7182-0)

## 查看和理解执行计划

#### 执行计划说明

| 字段          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 查询的执行顺序号                                             |
| select_type   | 查询类型                                                     |
| table         | 当前执行的表                                                 |
| type          | 查询使用了那种类型                                           |
| possible_keys | 显示可能应用在这张表中的索引，一个或多个。                   |
| key           | 实际使用的索引                                               |
| key_len       | 表示索引中使用的字节数                                       |
| ref           | 显示索引的那一列被使用了                                     |
| rows          | 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数 |
| filtered      | 存储引擎返回的数据在server层过滤后,剩下多少满足查询的记录数量的比例 |
| extra         | 包含不适合在其他列中显式但十分重要的额外信息                 |

> **select_type**常见和常用的值有如下几种：
>
> SIMPLE、PRIMARY、SUBQUERY 、DERIVED、UNION 、UNION RESULT 从UNION表获取结果的SELECT
>
> **type**包含的类型包括如下几种，从最好到最差依次是：
>
> system > const > eq_ref > ref > range > index > all
>

#### 查看执行计划

查看select * from employee的执行计划

打开ExplainTest.java，修改第16行run1方法中的TODO

```java
stmt = conn.createStatement();
String sql3 = "explain select * from employee where ename ='Parto'";
rs = stmt.executeQuery(sql3);
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

右键ExplainAndCaseMainTest，选择Edit修改参数为explain

![1735-160-2.png](https://doc.shiyanlou.com/courses/1735/1207281/2f6af365d654535be294e83cf3c5c717-0)

右键ExplainAndCaseMainTest，选择Run，运行代码

![1735-160-4.png](https://doc.shiyanlou.com/courses/1735/1207281/77bb75762b84ba37651230f85f55d780-0)

查看结果为：

| id   | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                                        |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| 1    | SIMPLE      | employee | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 6    | 16.67    | Using where with pushed condition (`mysqlTest`.`employee`.`ename` = 'Parto') |

> 在MySQL 5.7，可以查看select，delete，insert，replace和update语句的执行计划。

## 创建索引，改变执行计划

为表employee的列ename创建索引，再次查看执行计划

打开ExplainTest.java，修改第11行run2方法中的TODO

```java
stmt = conn.createStatement();
String sql = "alter table employee add index(ename)";
stmt.executeUpdate(sql);
String sql3 = "explain select * from employee where ename = 'Parto'";
rs = stmt.executeQuery(sql3);
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

右键ExplainAndCaseMainTest，选择Edit修改参数为alterExplain

![1735-160-2.png](https://doc.shiyanlou.com/courses/1735/1207281/2f6af365d654535be294e83cf3c5c717-0)

右键ExplainAndCaseMainTest，选择Run，运行代码

![1735-160-4.png](https://doc.shiyanlou.com/courses/1735/1207281/77bb75762b84ba37651230f85f55d780-0)

查看结果为：

| id   | select_type | table    | partitions | type | possible_keys | key   | key_len | ref   | rows | filtered | Extra |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ----- | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | employee | NULL       | ref  | ename         | ename | 515     | const | 1    | 100.00   | NULL  |

## Case表达式

MySQL CASE表达式是一个流程控制结构，用在在SELECT、WHERE等语句中根据条件动态构造内容。

MySQL的CASE表达式有2种形式，一种更像是编程语言当中的CASE语句，拿一个给定的值（变量）跟一系列特定的值作比较,称之为CASE类型。另一种则更像是编程语言中的if语句，当满足某些条件的时候取特定值，称之为IF类型。

#### case类型

此类型的语句结构如下：

```sql
CASE value
WHEN compare_value_1 THEN result_1
WHEN compare_value_2 THEN result_2
…
ELSE result END
```

此情况下，拿value与各个compare_value比较，相等时取对应的值，都不相等时取最后的result。

打开ExplainTest.java，修改第17行run1方法中的TODO

```java
stmt = conn.createStatement();
String sql3 = "SELECT ename,\n" +
    "    CASE ename\n" +
    "        WHEN 'Parto' THEN 'P'\n" +
    "        WHEN 'Georgi' THEN 'G'\n" +
    "        WHEN 'Chirs' THEN 'C'\n" +
    "        ELSE 'XX'\n" +
    "    END AS mark\n" +
    "FROM\n" +
    "    employee";
rs = stmt.executeQuery(sql3);
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

右键ExplainAndCaseMainTest，选择Edit修改参数为caseTest

![1735-160-2.png](https://doc.shiyanlou.com/courses/1735/1207281/2f6af365d654535be294e83cf3c5c717-0)

右键ExplainAndCaseMainTest，选择Run，运行代码

![1735-160-4.png](https://doc.shiyanlou.com/courses/1735/1207281/77bb75762b84ba37651230f85f55d780-0)

查看结果为：

​			Georgi	G	
​			Bezalel	XX	
​			Parto	P		
​			Chirs	C	
​			Kyoichi	XX	
​			Anneke	XX

#### IF类型

此类型的CASE表达式如下：

```sql
CASE
WHEN condition_1 THEN result_1
WHEN condition_2 THEN result_2
…
ELSE result END
```

此时自上而下根据condition判断，取对应的值，都不满足的时候取最后的result。

打开ExplainTest.java，修改第11行run2方法中的TODO

```java
stmt = conn.createStatement();
String sql3 = "SELECT \n" +
    "    *\n" +
    "FROM\n" +
    "    employee\n" +
    "ORDER BY (CASE\n" +
    "    WHEN empno IS NULL THEN age\n" +
    "    ELSE empno\n" +
    "END);";
rs = stmt.executeQuery(sql3);
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```
右键ExplainAndCaseMainTest，选择Edit修改参数为caseIfTest

![1735-160-2.png](https://doc.shiyanlou.com/courses/1735/1207281/2f6af365d654535be294e83cf3c5c717-0)

右键ExplainAndCaseMainTest，选择Run，运行代码

![1735-160-4.png](https://doc.shiyanlou.com/courses/1735/1207281/77bb75762b84ba37651230f85f55d780-0)

查看结果为：

​			10001	Georgi	48	
​			10002	Bezalel	21	
​			10003	Parto	33	
​			10004	Chirs	40	
​			10005	Kyoichi	23	
​			10006	Anneke	19	

