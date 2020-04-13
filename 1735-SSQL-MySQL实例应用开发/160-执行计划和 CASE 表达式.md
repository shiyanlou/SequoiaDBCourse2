---
show: step
version: 1.0 
---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，熟悉MySQL的执行计划和SQL表达式

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。

## 查看和理解执行计划

查看执行计划

```sql
explain select * from employee;
```

在MySQL 5.7，可以查看select，delete，insert，replace和update语句的执行计划。

输出字段说明

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
> 1.SIMPLE 简单的select查询，查询中不包含子查询或者UNION
>
> 2.PRIMARY 查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY
>
> 3.SUBQUERY 在select或where列表中包含了子查询
>
> 4.DERIVED 在from列表中包含的子查询被标记为DERIVED(衍生),MySQL会递归执行这些子查询，把结果放在临时表中
>
> 5.UNION 若第二个SELECT出现在UNION之后，则被标记为UNION;若UNION包含在FROM子句的子查询中，外层SELECT将被标记为：DERIVED
>
> 6.UNION RESULT 从UNION表获取结果的SELECT
>
> **type**包含的类型包括如下几种，从最好到最差依次是：
>
> system > const > eq_ref > ref > range > index > all
>
> 1.system 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计
> 2.const 表示通过索引一次就找到了，const用于比较primary key 或者unique索引。因为只匹配一行数据，所以很快。如将主键置于where列表中，MySQL就能将该查询转换为一个常量。
> 3.eq_ref 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描
>
> 4.ref 非唯一性索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体。 
>
> 5.range 只检索给定范围的行，使用一个索引来选择行，key列显示使用了哪个索引，一般就是在你的where语句中出现between、< 、>、in等的查询，这种范围扫描索引比全表扫描要好，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引。
>
> 6.index Full Index Scan，Index与All区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘读取的）
>
> 7.all Full Table Scan 将遍历全表以找到匹配的行 

## 创建索引，改变执行计划



#### SQL表达式

#### 简单表达式

#### 复合表达式

#### Case表达式
