---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 的技术架构，以及通过 Spark SQL 访问 SequoiaDB 外部数据源的原理等有关知识，并通过简单的实验介绍如何通过 JDBC 访问 Spark SQL。

#### 实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* JDK version "1.8.0_172"
* SequoiaDB version: 3.4
* SequoiaSQL-MySQL version: 3.4
* Spark version: 2.4.3
* IntelliJ IDEA Community Version: 2019.3.4

#### 知识点

**Spark 简介**

![1738-410-01](https://doc.shiyanlou.com/courses/1738/1207281/0f9515037aa252fe897fe6e48f7f5ab1-0)

Spark 是加州大学伯克利分校AMP实验室开发的通用大数据处理框架。Spark 在 2013 年 6 月进入 Apache 成为孵化项目，8 个之后成为了 Apache 的顶级项目，很快 Spark 就成为了社区的热门，围绕 Spark 推出了 Spark SQL、Spark Streaming、MLlib、GraphX 和 SparkR 等丰富的组件。

![1738-410-02](https://doc.shiyanlou.com/courses/1738/1207281/e37fd6e7f082ad243ceea6faa9f53675-0)

Spark 使用 Scala 语言实现，具有易用性的特点，除 Scala 以外，Spark 还提供了 Java、Python、R 甚至是 SQL 的 API 。Spark 还具有很强的适应性，可以在 Hadoop，Apache Mesos，Kubernetes，standalone 模式或是云端中运行。

![1738-410-03](https://doc.shiyanlou.com/courses/1738/1207281/69f18a05ffc82f5a1ddcb60f3757e5ce-0)

此外，Spark 还可以访问各种数据源。

![1738-410-04](https://doc.shiyanlou.com/courses/1738/1207281/91401f4c77ec312eb15f9faef95f3396-0)

**Spark 集群模式工作原理**

![1738-410-05](https://doc.shiyanlou.com/courses/1738/1207281/e47c46e4b2ea1a76598667284f644dda-0)

在集群中，Spark应用以独立的进程集合的方式运行，并由主程序（driver program）中的 SparkContext  对象进行统一的调度。当需要在集群上运行时，SparkContext 会连接到几个不同类的 ClusterManager（集群管理器）上（Spark  自己的 Standalone/Mesos/YARN）, 集群管理器将给各个应用分配资源。连接成功后，Spark  会请求集群各个节点的Executor（执行器），它是为应用执行计算和存储数据的进程的总称。之后，Spark会将应用提供的代码（应用已经提交给  SparkContext 的 JAR 或 Python 文件）交给 executor。最后，由SparkContext 发送tasks提供给 executor 执行（多线程）。

**Spark + SequoiaSQL-MySQL + SequoiaDB**

![1738-410-06](https://doc.shiyanlou.com/courses/1738/1207281/ff2754d609aba12340efeb27ce0645bb-0)

Spark 具有访问多种外部数据源的特性。在 SequoiaDB 分布式存储架构中，Spark 可以像访问 MySQL 数据库那样访问 SequoiaDB 分布式存储的 MySQL 实例，也可以通过 SequoiaDB 的 Spark 连接器直接访问底层的 SequoiaDB 存储集群。

**Hive on Spark**

Hive on spark 是一个Hive的发展计划，由 Cloudera 发起，由 Intel、MapR 等公司共同参与的开源项目，其目的是把 Spark 作为 Hive 的一个计算引擎，将 Hive 的查询作为 Spark 的任务提交到 Spark 集群上进行计算。

Hive on spark 可以通过 Hive jdbc 的方式进行操作，本系列实验也将围绕这一方式进行展开。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具

![1738-410-07](https://doc.shiyanlou.com/courses/1738/1207281/5fd8d1853074d843bc97ac1cb8b0b581-0)

#### 打开 SCDD-Spark 项目

选择 Spark 课程项目

![1738-410-08](https://doc.shiyanlou.com/courses/1738/1207281/a7ab357431f711205346b87965a988ba-0)

#### 项目结构

项目结构以及目录文件规划如下图所示：

![1738-410-09](https://doc.shiyanlou.com/courses/1738/1207281/9bb783bcad1c10701a6c63219a8d0147-0)

#### Maven 依赖

如图所示找到 pom.xml 文件：

![1738-410-pom](https://doc.shiyanlou.com/courses/1738/1207281/2096e77f8ff05283b1b51e9f5182b861-0)

在 pom.xml 文件中可以找到当前课程使用到的 Maven 依赖：

![1738-410-10](https://doc.shiyanlou.com/courses/1738/1207281/6051f6b91a19df45bb674dd7fe1a8e0a-0)

## Hive JDBC 代码

编写通过 JDBC 连接 Hive on Spark 进行数据操作的代码，在后文的样例程序中会调用本节定义的方法。在之后的课程中使用到 HiveUtil 类时会调用已有的该类，代码内容和本节叙述一致，将不再赘述。

#### 打开 HiveUtil 类

如图找到 com.sequoiadb.lesson.spark.base.util.HiveUtil 类，打开类准备编写代码

![1738-410-11](https://doc.shiyanlou.com/courses/1738/1207281/cfd69ce99d15bc6b0605c697652d2b49-0)

#### 创建 JDBC 连接

使用 Hive JDBC 驱动创建 Hive 的 JDBC 连接代码如下：

```java
try {
    // Get JDBC driver class
    Class.forName("org.apache.hive.jdbc.HiveDriver");
    // Get JDBC connection
    connection = DriverManager.getConnection(
            "jdbc:hive2://sdbserver1:10000/default",// url
            "sdbadmin",// Hive on Spark Username
            ""// Hive on Spark password(Authentication is not enabled by default)
    );
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException e) {
    e.printStackTrace();
}
```

将创建 JDBC 连接代码粘贴至 HiveUtil 类 getConnection 方法的 TODO -- lesson1_sample:code1 注释处（69 行）：

![1738-410-12](https://doc.shiyanlou.com/courses/1738/1207281/5cc82cbd784601606c669d4f45b5ac42-0)

> **说明**
>
> 粘贴方法如下：
>
> * 点击代码框右上角的 copy 图标
>
> * 选择实验界面左边的 “剪切板”
>
>   ![paste1](https://doc.shiyanlou.com/courses/1738/1207281/7745e7378b70a60ad6073262f05762ec-0)
>
> * 在弹出的“在线环境剪切板”中粘贴复制的代码内容
>
>   ![paste2](https://doc.shiyanlou.com/courses/1738/1207281/6b477101feb04b1db73e8f893ba3b334-0)
>
> * 在实验环境中到对应和位置粘贴
>
>   ![paste3](https://doc.shiyanlou.com/courses/1738/1207281/14482e482cde033e4f78cca144abdcee-0)

#### JDBC 创建数据库对象

JDBC 创建数据库对象的代码内容如下：

```java
// Get JDBC connection
Connection connection = getConnection();
Statement statement = null;
try {
    // Load SQL statement
    statement = connection.createStatement();
    // Submit SQL statement
    statement.execute(sql);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // Release resource
    releaseSource(null, statement, connection);
}
```

将 JDBC 创建数据库对象代码粘贴至 HiveUtil 类 doDDL 方法的 TODO -- lesson1_sample:code2 注释处（56 行）：

![1738-410-13](https://doc.shiyanlou.com/courses/1738/1207281/ccf219c515863a87a36cf96906819147-0)

#### JDBC 操作数据库记录

JDBC 提交数据库记录操作语句代码内容如下：

```java
// Get JDBC connection
Connection connection = getConnection();
Statement statement = null;
try {
    // Load SQL statement
    statement = connection.createStatement();
    // Submit SQL statement
    statement.executeUpdate(sql);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // Release source
    releaseSource(null, statement, connection);
}
```

将 JDBC 操作数据库记录语句粘贴至 HiveUtil 类 doDML 方法的 TODO -- lesson1_sample:code3 注释处（45行）：

![1738-410-14](https://doc.shiyanlou.com/courses/1738/1207281/99f702bcef254ac740127494c83bdee2-0)

#### JDBC 查询数据库记录结果集

JDBC 查询数据库记录代码如下：

```java
// Get JDBC connection
Connection connection = getConnection();
ResultSet resultSet = null;
Statement statement = null;
try {
    // Load SQL statement
    statement = connection.createStatement();
    // Submit SQL statement to get query result set
    resultSet = statement.executeQuery(sql);
    // Format printing and return result set
    ResultFormat.printResultSet(resultSet);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    //  Release source
    releaseSource(resultSet, statement, connection);
}
```

将 JDBC 查询数据库记录的语句粘贴至 HiveUtil 类 doDQL 方法的 TODO -- lesson1_sample:code4 注释处（ 34 行）：

![1738-410-15](https://doc.shiyanlou.com/courses/1738/1207281/2c80a1291e95d90249d96e69f4cb2c34-0)

#### 释放 JDBC 资源

releaseSource() 为 HiveUtil 的一个公用方法，用于释放各种 JDBC 操作中使用到的 resultset、statement 和 connection 等资源：

```java
// Release resultset
if (null != resultSet) {
    try {
        resultSet.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
// Release statement
if (null != statement) {
    try {
        statement.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
// Release connection
if (null != connection) {
    try {
        connection.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

将释放 jdbc 资源代码粘贴至 HiveUtil 类 releaseSource 方法的 TODO -- lesson1_sample:code5 注释处（23 行）：

![1738-410-16](https://doc.shiyanlou.com/courses/1738/1207281/08620fbc6bf6a00420ef7256715ca3a7-0)

## 样例程序

样例程序将简单展示通过调用 HiveUtil 中的方法提交 SQL 语句的方式。

#### 打开 JdbcSample 类

如图所示打开 com.sequoiadb.lesson.spark.lesson1_sample.JdbcSample 类

![1738-410-17](https://doc.shiyanlou.com/courses/1738/1207281/2f65df33feacaf3fa902b1d9d68a4119-0)

#### JDBC 访问 Hive on Spark 样例

样例程序中调用了 HiveUtil 类中的方法，通过 JDBC 向 Hive on Spark 提交 SQL 语句。程序代码内容如下：

```java
// Init table
String dropTable =
        "DROP TABLE IF EXISTS jdbc_sample";
// Call the method of the HiveUtil class to execute the drop statement through JDBC
HiveUtil.doDDL(dropTable);
// Create table
String createTable =
        "CREATE TABLE jdbc_sample ( id INT, val VARCHAR ( 10 ) )";
// Call the method of the HiveUtil class to execute the create statement through jdbc
System.out.println("Creating table...");
HiveUtil.doDDL(createTable);
// Insert data
String insertDate =
        "INSERT INTO jdbc_sample VALUES ( 1, \"SequoiaDB\" )";
// Call the method of the HiveUtil class to execute the insert statement through jdbc
System.out.println("Writing record...");
HiveUtil.doDML(insertDate);
// Query result set
String getResultSet =
        "SELECT id,val FROM jdbc_sample";
// Call the method of the HiveUtil class to get the result set through jdbc
System.out.println("Query record...");
HiveUtil.doDQL(getResultSet);
```

将运行样例代码粘贴至 JdbcSample  类 sample 方法的 TODO -- lesson1_sample:code6 注释处（20 行）：

![1738-410-18](https://doc.shiyanlou.com/courses/1738/1207281/c4e0ac9f67908d5c74e1db9ec2e90bfe-0)

## 运行样例

#### 运行程序

右键点击 SampleMainTest 类，选择 Run 运行主函数：

![1738-410-19](https://doc.shiyanlou.com/courses/1738/1207281/05ba1cfaaf96207aed32a4def121aaf7-0)

#### 运行结果

程序运行结果如下图所示：

![1738-410-20](https://doc.shiyanlou.com/courses/1738/1207281/9998c4015f44b613202d911c1ea157cc-0)

## 总结

通过本实验，可以对 Spark 技术架构和工作原理有了大致的了解，以及 Spark 是如何和 SequoiaDB 分布式存储集群交互工作的。此外，通过简单的实践练习可以使用标准 JDBC 访问 Hive on Spark 并提交 SQL 任务，后续的若干章节会根据本章节的基础继续展开。
