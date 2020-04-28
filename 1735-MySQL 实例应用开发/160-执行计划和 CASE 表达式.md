---

show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，熟悉 MySQL 的执行计划和 CASE 表达式。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 MySQL 实例均为 3.4 版本。

## 打开项目

#### 打开 idea

打开 idea 代码开发工具

![1735-110-1.png](https://doc.shiyanlou.com/courses/1735/1207281/6f87a8c93937c3c51f6d4839559de710-0)

#### 打开 scdd-mysql 项目

打开 scdd-mysql 项目，在该课程中完成后续试验

![1587923287299](https://doc.shiyanlou.com/courses/1735/1207281/2e66fe621bc8196ead5a7141c8125db4-0)

#### 打开lesson6_explainandcase包

打开 lesson6_explainandcase packge，在该 packge 中完成后续课程

![1587923393235](https://doc.shiyanlou.com/courses/1735/1207281/69545b5f00569e8ced7e73c1f028af35-0)

## 查看和理解执行计划

我们知道，不管是哪种数据库，或者是哪种数据库引擎，在对一条 SQL 语句进行执行的过程中都会做很多相关的优化，对于查询语句，最重要的优化方式就是使用索引。而执行计划，就是显示数据库引擎对于 SQL 语句的执行的详细情况，其中包含了是否使用索引，使用什么索引，使用的索引的相关信息等。

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

**select_type **常见和常用的值有如下几种：

SIMPLE、PRIMARY、SUBQUERY 、DERIVED、UNION 、UNION RESULT  从 UNION 表获取结果的 SELECT

**type **包含的类型包括如下几种，从最好到最差依次是：

system > const > eq_ref > ref > range > index > all

#### 查看执行计划

查看 select * from employee 的执行计划

1）打开 ExplainTest.java

![1735-160-100.png](https://doc.shiyanlou.com/courses/1735/1207281/9ec03de3a77511ff6c2268a53ec148d2-0)

2）在 run1 方法中找到 TODO code 1

![1735-160-101.png](https://doc.shiyanlou.com/courses/1735/1207281/6f0df8090b504a1a930ad26e9fa73ac5-0)

3）将下方代码粘贴到 TODO code 1 区域内，查看 select * from employee 的执行计划

```java
// Create a Statement object to send SQL statements to the database
stmt = conn.createStatement();
// Write sql
String sql3 = "EXPLAIN SELECT * FROM employee WHERE ename ='Parto'";
// Execute sql
rs = stmt.executeQuery(sql3);
// Traverse the query results
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

代码粘贴结果如图所示：

![1735-160-1000.png](https://doc.shiyanlou.com/courses/1735/1207281/a91a6463ac9d24c4d77c694ab240c0b2-0)

4）修改参数，右键 ExplainAndCaseMainTest.java，选择 Edit ' ExplainAndCase...main()'

![1735-160-115.png](https://doc.shiyanlou.com/courses/1735/1207281/c3769b1054271871856a48d07950d855-0)

5）修改参数为 explain

![1735-160-102.png](https://doc.shiyanlou.com/courses/1735/1207281/4e7c1572d80a4a28d3b1fc8f17b9aafe-0)

6）执行代码，右键 ExplainAndCaseMainTest.java，选择 Run 'ExplainAndCase...main()'，运行代码

![1735-160-113.png](https://doc.shiyanlou.com/courses/1735/1207281/91e311e849e293d14879285d2ace1852-0)

7）查看结果

| id   | select_type | table    | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                                                        |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ---- | ------- | ---- | ---- | -------- | ------------------------------------------------------------ |
| 1    | SIMPLE      | employee | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 6    | 16.67    | Using where with pushed condition (`mysqlTest`.`employee`.`ename` = 'Parto') |

> 在 MySQL 5.7，可以查看 select，delete，insert，replace 和 update 语句的执行计划。

## 创建索引，改变执行计划

为表 employee 的列 ename 创建索引，再次查看执行计划。

1）打开 ExplainTest.java

![1735-160-103.png](https://doc.shiyanlou.com/courses/1735/1207281/8275599d562ba91722acae96f50d2ada-0)

2）在 run2 方法中找到 TODO code 2

![1735-160-104.png](https://doc.shiyanlou.com/courses/1735/1207281/4b66506e44bf4cff8401aff101dc0f9e-0)

3）将下方代码粘贴到 TODO code 2 区域内，为表 employee 的列 ename 创建索引，再次查看执行计划，发现执行计划改变

```java
// Create a Statement object to send SQL statements to the database
stmt = conn.createStatement();
// Write sql to modify index
String sql = "ALTER TABLE employee ADD INDEX(ename)";
// Execute sql
stmt.executeUpdate(sql);
// Write SQL to view the execution plan
String sql3 = "EXPLAIN SELECT * FROM employee WHERE ename = 'Parto'";
// Execute sql
rs = stmt.executeQuery(sql3);
// Traverse the query results
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

代码粘贴结果如图所示：

![1735-160-1001.png](https://doc.shiyanlou.com/courses/1735/1207281/be9a63a4ba92d4810527c6693db38b16-0)

4）修改参数，右键 ExplainAndCaseMainTest.java，选择 Edit 'ExplainAndCase...main()'

![1735-160-115.png](https://doc.shiyanlou.com/courses/1735/1207281/c3769b1054271871856a48d07950d855-0)

5）修改参数为 alterExplain

![1735-160-105.png](https://doc.shiyanlou.com/courses/1735/1207281/930834f84b44925bd8be6bebc485a8ec-0)

6）执行代码，右键 ExplainAndCaseMainTest.java，选择 Run 'ExplainAndCase...main()'，运行代码

![1735-160-113.png](https://doc.shiyanlou.com/courses/1735/1207281/91e311e849e293d14879285d2ace1852-0)

7）查看结果

| id   | select_type | table    | partitions | type | possible_keys | key   | key_len | ref   | rows | filtered | Extra |
| ---- | ----------- | -------- | ---------- | ---- | ------------- | ----- | ------- | ----- | ---- | -------- | ----- |
| 1    | SIMPLE      | employee | NULL       | ref  | ename         | ename | 515     | const | 1    | 100.00   | NULL  |

## CASE表达式

MySQL CASE 表达式是一个流程控制结构，用在在 SELECT、WHERE 等语句中根据条件动态构造内容。

MySQL 的 CASE 表达式有2种形式，一种更像是编程语言当中的 CASE 语句，拿一个给定的值（变量）跟一系列特定的值作比较,称之为 CASE 类型。另一种则更像是编程语言中的if语句，当满足某些条件的时候取特定值，称之为 IF 类型。

#### case类型

此类型的语句结构如下：

> CASE value
>
> WHEN compare_value_1 THEN result_1
>
> WHEN compare_value_2 THEN result_2
>
> …
>
> ELSE result END

此情况下，拿 value 与各个 compare_value 比较，相等时取对应的值，都不相等时取最后的 result。

1）打开 CaseTest.java

![1735-160-106.png](https://doc.shiyanlou.com/courses/1735/1207281/8a1dd6618fb9b35fda2061292a78ffc6-0)

2）在 run1 方法中找到 TODO code 1

![1735-160-107.png](https://doc.shiyanlou.com/courses/1735/1207281/6f819f33f5e945aacfd331379ae06f5c-0)

3）将下方代码粘贴到 TODO code 1 区域内，CASE 表达式，当 ename 为 ‘Parto’ 时，显示为 ‘P’，不符合 WHEN 条件的 ，显示为 ‘XX’

```java
// Create a Statement object to send SQL statements to the database
stmt = conn.createStatement();
// Write sql
String sql3 = "SELECT ename," +
    "    CASE ename" +
    "        WHEN 'Parto' THEN 'P'" +
    "        WHEN 'Georgi' THEN 'G'" +
    "        WHEN 'Chirs' THEN 'C'" +
    "        ELSE 'XX'" +
    "    END AS mark\n" +
    "FROM" +
    "    employee";
// Execute sql
rs = stmt.executeQuery(sql3);
// Traverse the query results
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

代码粘贴结果如图所示：

![1735-160-1002.png](https://doc.shiyanlou.com/courses/1735/1207281/27c3cb2fd3c4d1c90de88adc5c4200ea-0)

4）修改参数，右键 ExplainAndCaseMainTest.java，选择 Edit 'ExplainAndCase...main()'

![1735-160-115.png](https://doc.shiyanlou.com/courses/1735/1207281/c3769b1054271871856a48d07950d855-0)

4）修改参数为 caseTest

![1735-160-108.png](https://doc.shiyanlou.com/courses/1735/1207281/3fdcfa4f5beff758cc5f20993a300bd2-0)

6）执行代码，右键 ExplainAndCaseMainTest.java，选择 Run 'ExplainAndCase...main()'，运行代码

![1735-160-113.png](https://doc.shiyanlou.com/courses/1735/1207281/91e311e849e293d14879285d2ace1852-0)

7）查看结果

![1735-160-109.png](https://doc.shiyanlou.com/courses/1735/1207281/6b359abf43d45435dcf84aed550bdee2-0)

#### IF类型

此类型的 CASE 表达式如下：

> CASE
>
> WHEN condition_1 THEN result_1
>
> WHEN condition_2 THEN result_2
>
> …
>
> ELSE result END

此时自上而下根据 condition 判断，取对应的值，都不满足的时候取最后的 result 。

1）打开 CaseTest.java

![1735-160-106.png](https://doc.shiyanlou.com/courses/1735/1207281/8a1dd6618fb9b35fda2061292a78ffc6-0)

2）在 run2 方法中找到 TODO code 2

![1735-160-110.png](https://doc.shiyanlou.com/courses/1735/1207281/ec5d580130e237290e5429fc15ff6cd1-0)

3）将下方代码粘贴到 TODO code 2 区域内，查询表 employee，当 empno 字段为 null 时，按 age 排序，其余情况按 empno 排序

```java
// Create a Statement object to send SQL statements to the database
stmt = conn.createStatement();
// Write sql
String sql = "SELECT " +
    "    *" +
    "FROM" +
    "    employee\n" +
    "ORDER BY (CASE" +
    "    WHEN empno IS NULL THEN age" +
    "    ELSE empno\n" +
    "END);";
// Execute sql
rs = stmt.executeQuery(sql);
// Traverse the query results
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

代码粘贴结果如图所示：

![1735-160-1003.png](https://doc.shiyanlou.com/courses/1735/1207281/e1706664c74c1d24c21260fcbe3e1e67-0)

4）修改参数，右键 ExplainAndCaseMainTest.java，选择 Edit 'ExplainAndCase...main()'

![1735-160-115.png](https://doc.shiyanlou.com/courses/1735/1207281/c3769b1054271871856a48d07950d855-0)

5）修改参数为 caseIfTest

![1735-160-111.png](https://doc.shiyanlou.com/courses/1735/1207281/5fdf9ec88a87828d8c43248e671991d2-0)

6）执行代码，右键 ExplainAndCaseMainTest.java，选择 Run 'ExplainAndCase...main()'，运行代码

![1735-160-113.png](https://doc.shiyanlou.com/courses/1735/1207281/91e311e849e293d14879285d2ace1852-0)

7）查看结果

![1735-160-112.png](https://doc.shiyanlou.com/courses/1735/1207281/07b4f8133371392ced4117ff24d405da-0)

## 总结

通过本课程学会了如何查看和理解执行计划，以及 MySQL 中 两种类型 Case 表达式的使用。通过查看执行计划可以查看 SQL 语句的执行的详细情况，使用 CASE 表达式可以在 SELECT、WHERE 等语句中根据条件动态构造内容。

