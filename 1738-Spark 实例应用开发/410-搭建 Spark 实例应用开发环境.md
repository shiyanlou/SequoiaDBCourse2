---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 的技术架构，并在已经部署了 Spark 和 SequoiaDB 的环境中，通过简单的实验实现使用 JDBC 访问 Spark 操作 SequoiaDB 中的数据。

#### 请点击右侧选择使用的实验环境

#### 部署架构

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：

* 1 个 SequoiaSQL-MySQL 数据库实例节点
* 1 个引擎协调节点
* 1 个编目节点
* 3 个数据节点

![1738-410-06](https://doc.shiyanlou.com/courses/1738/1207281/ff2754d609aba12340efeb27ce0645bb-0)

在当前实验的部署架构中，Spark 通过 MySQL 实例间接访问 SequoiaDB 存储集群，也可以通过 SequoiaDB 的 Spark 连接驱动直接访问底层的 SequoiaDB 存储集群。

详细了解 SequoiaDB 巨杉数据库系统架构：

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1561381722-edition_id-304)

#### 实验环境

课程使用的系统环境为 Ubuntu 16.04.6 LTS 版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。Spark 计算引擎为 2.4.3 版本。

## 打开项目

#### 打开 IDEA

1）打开 IDEA 代码开发工具。

![1738-410-07](https://doc.shiyanlou.com/courses/1738/1207281/72397a857808ab74f01b042f07ea0a27-0)

#### 打开 SCDD-Spark 项目

2）打开 Spark 实验的项目，在该项目中完成所有实验步骤。

![1738-410-08](https://doc.shiyanlou.com/courses/1738/1207281/5497d93bf19ee0442b4ae79b4cd8d39a-0)

#### 项目结构

3）项目结构如下图所示：

![1738-410-项目结构](https://doc.shiyanlou.com/courses/1738/1207281/fd4d3109d9905204bbe1cfdfcfb0a3be-0)

#### Maven 依赖

当前实验会使用到 hive-jdbc-1.2.1.spark.jar 和 spark-sequoiadb_2.11-3.2.4.jar 两个 jar 包依赖。其中 hive-jdbc-1.2.1.spark.jar 为 Hive JDBC 驱动包，用于在 Java 程序中通过 JDBC 访问 Spark SQL；spark-sequoiadb_2.11-3.2.4.jar 为 SequoiaDB 的 Spark 连接驱动包，用于在 Spark SQL 中创建 SequoiaDB 集合的映射。实验环境中已经在 Maven 本地仓库添加了实验所需的依赖。

pom.xml 文件位置：

![1738-410-pom文件位置](https://doc.shiyanlou.com/courses/1738/1207281/9792106157a176f82acdaebf04961568-0)

当前实验中使用到的 Maven 依赖如下：

![1738-410-maven1](https://doc.shiyanlou.com/courses/1738/1207281/b0f437f18a0a276bbf404da17fef9a2c-0)

![1738-410-maven2](https://doc.shiyanlou.com/courses/1738/1207281/0b93cf135364f978faeba469c3cca528-0)

> **说明**
>
> spark-sequoiadb_2.11-3.2.4.jar 可以在 [SequoiaDB 巨杉数据库下载中心](http://download.sequoiadb.com/cn/driver) 获取。

#### 打开 Package

4）如图所示找到当前实验程序所在 Package，在该 Package 中完成后续实验步骤：

![1738-410-实验package](https://doc.shiyanlou.com/courses/1738/1207281/aa98e5edd8835f19a8c8d2aa2380be79-0)

## **Spark 简介**

![1738-410-01](https://doc.shiyanlou.com/courses/1738/1207281/0f9515037aa252fe897fe6e48f7f5ab1-0)

Spark 是加州大学伯克利分校AMP实验室开发的通用大数据处理框架。Spark 在 2013 年 6 月进入 Apache 成为孵化项目，8 个月之后成为了 Apache 的顶级项目，很快 Spark 就成为了社区的热门，围绕 Spark 推出了 Spark SQL、Spark Streaming、MLlib、GraphX 和 SparkR 等丰富的组件。

![1738-410-02](https://doc.shiyanlou.com/courses/1738/1207281/e37fd6e7f082ad243ceea6faa9f53675-0)

Spark 使用 Scala 语言实现，具有易用性的特点，除 Scala 以外，Spark 还提供了 Java、Python、R 甚至是 SQL 的 API 。Spark 还具有很强的适应性，可以在 Hadoop，Apache Mesos，Kubernetes，standalone 模式或是云端中运行，可以访问 Hadoop、HBASE 等多种数据源。

![1738-410-03](https://doc.shiyanlou.com/courses/1738/1207281/69f18a05ffc82f5a1ddcb60f3757e5ce-0)

Spark SQL 是 Spark 用于处理结构化数据的组件。可以通过 SQL API 和 DataSet API 两种方式和 Spark SQL 进行交互。在 Spark 应用开发系列课程中将围绕 Spark 的 Spark SQL 组件和 SequoiaDB 交互进行讲解。在本课程的前 5 个实验将介绍使用 Spark SQL 的 SQL API 操作 SequoiaDB 中的数据；通过 DataSet API 和 Spark SQL 交互操作 SequoiaDB 中的数据将在实验 6 中进行介绍。

## Spark 访问 SequoiaDB 样例

#### 概述

由于 Spark SQL 使用的 Thrift JDBC server 和 Hive 的 HiveServer2 是一致的，因此可以通过 Hive JDBC 访问 Spark SQL。Spark SQL 和 SequoiaDB 交互主要是通过在 Spark SQL 中创建 SequoiaDB 集合的映射表实现的，通过 Hive JDBC 提交读写映射表的 SQL 语句，即可达到操作 SequoiaDB 存储集群中的数据的目的。

当前实验简单展示了如何通过 JDBC 操作 Spark SQL 以及在 Spark SQL 中创建 SequoiaDB 集合的映射表，在程序中会向映射表中插入一条记录，并查询映射表中的记录打印到控制台。

实验仅对通过 JDBC 实现 Spark SQL 与 SequoiaDB 交互进行简单演示，具体实现细节将在后续的课程中进行介绍。

#### 操作步骤

1）打开 JdbcSample 类

![1738-410-打开JdbcSample](https://doc.shiyanlou.com/courses/1738/1207281/9eb13b34461a50f8f62709c267811d78-0)

2）复制程序代码。程序中会创建 JDBC 连接，并通过 JDBC 在 Spark SQL 中创建 jdbc_sample 集合的映射表，向映射表中插入一条记录后查询 jdbc_sample 记录打印到控制台，最终关闭 JDBC 的连接资源。 

```java
// Call the predefined SdbUtil class to create a collection space and collection
SdbUtil.initCollectionSpace("sample");
SdbUtil.initCollection("sample", "jdbc_sample");
// Load Hive JDBC driver
Class.forName("org.apache.hive.jdbc.HiveDriver");
// Create a Hive JDBC connection
Connection connection = DriverManager.getConnection(
        "jdbc:hive2://sdbserver1:10000/default",// Hive JDBC connection url
        "sdbadmin",// Hive JDBC connection user name
        ""// Hive JDBC connection password (authentication is not enabled by default)
);
// Create Statement
Statement statement = connection.createStatement();
// Drop the existing table
String dropTable = "DROP TABLE IF EXISTS jdbc_sample";
// Execute the SQL statement of drop table
statement.execute(dropTable);
// Create a mapping table
String mapping =
        "CREATE TABLE jdbc_sample ( id INT, val VARCHAR ( 10 ) )" +
                "USING com.sequoiadb.spark " +
                "OPTIONS(" +
                "host 'sdbserver1:11810'," +
                "collectionspace 'sample'," +
                "collection 'jdbc_sample'" +
                ")";
// Execute the SQL statement of create a mapping table
statement.execute(mapping);
// Insert record
String insert = "INSERT INTO jdbc_sample VALUES ( 1, 'SequoiaDB' )";
// Execute the SQL statement of insert record
statement.executeUpdate(insert);
// Query record
String query = "SELECT * FROM jdbc_sample";
// Execute the SQL statement of query record to get result set
ResultSet resultSet = statement.executeQuery(query);
// Call the predefined result set to beautify the utility class and print the result set
ResultFormat.printResultSet(resultSet);
// Release JDBC sources
resultSet.close();
statement.close();
connection.close();
```

> **说明**
>
> SdbUtil 为已经封装好的 SequoiaDB 工具类，在当前实验中用于创建实验使用的集合空间和集合；ResultFormat 工具类为已经封装好的查询结果集打印类，用于美化结果集的打印。封装的工具类在此不做赘述。

3）将程序代码粘贴至 JdbcSample 类 sample 方法的 TODO code 1 注释区间内。

![1738-410-TODO1](https://doc.shiyanlou.com/courses/1738/1207281/6b97e7daab37bc3d6a3a9103e78fbbfc-0)

> **说明**
>
> 粘贴方法如下：
>
> * 点击代码框右上角的 copy 图标
>
> * 选择实验界面右边的 “剪切板”
>
>   ![paste1](https://doc.shiyanlou.com/courses/1738/1207281/7745e7378b70a60ad6073262f05762ec-0)
>
> * 在弹出的“在线环境剪切板”中粘贴复制的代码内容
>
>   ![paste2](https://doc.shiyanlou.com/courses/1738/1207281/6b477101feb04b1db73e8f893ba3b334-0)
>
> * 在实验环境中到对应的位置粘贴
>
>   ![paste3](https://doc.shiyanlou.com/courses/1738/1207281/14482e482cde033e4f78cca144abdcee-0)

4）右键点击 JdbcSample 类，选择 Run 'JdbcSample.main()' 运行样例程序。

![1738-410-运行样例](https://doc.shiyanlou.com/courses/1738/1207281/3936c68ac9ac00cc118d0059d0f5078e-0)

5）查看运行结果

![1738-410-运行结果](https://doc.shiyanlou.com/courses/1738/1207281/613198a8b61bdf238613d9296f1d2596-0)

## 总结

本课程简要介绍了 Spark 的技术架构和基础特性，并通过实验演示了使用 JDBC 在 Spark SQL 建立与 SequoiaDB 的映射并操作存储在 SequoiaDB 中的数据。如何使用 JDBC 访问 Spark SQL 以及如何在 Spark SQL 中创建 SequoiaDB 集合的映射表进行数据操作的实现细节将在后续的实验中展开介绍。
