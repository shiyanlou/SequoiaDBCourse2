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

打开 IDEA 代码开发工具。

![1738-410-07](https://doc.shiyanlou.com/courses/1738/1207281/5fd8d1853074d843bc97ac1cb8b0b581-0)

#### 打开 SCDD-Spark 项目

选择 Spark 课程项目

![1738-410-08](https://doc.shiyanlou.com/courses/1738/1207281/a7ab357431f711205346b87965a988ba-0)

#### 项目结构

项目结构以及目录文件规划如下图所示：

![1738-410-09](https://doc.shiyanlou.com/courses/1738/1207281/9bb783bcad1c10701a6c63219a8d0147-0)

#### Maven 依赖

当前课程使用到的 Maven 依赖如下：

![1738-410-10](https://doc.shiyanlou.com/courses/1738/1207281/6051f6b91a19df45bb674dd7fe1a8e0a-0)

## Hive jdbc 代码

#### 打开 HiveUtil 类

如图找到 com.sequoiadb.lesson.spark.base.util.HiveUtil 类，打开类准备编写代码

![1738-410-11](https://doc.shiyanlou.com/courses/1738/1207281/4326c67698d61128a54a4804b0e165cf-0)

#### 创建 JDBC 连接

使用 Hive JDBC 驱动创建 Hive 的 JDBC 连接代码如下：

```java
// 初始化连接
Connection connection = null;
try {
    // 获取 jdbc 驱动类
    Class.forName("org.apache.hive.jdbc.HiveDriver");
    // 获取 jdbc 连接
    connection = DriverManager.getConnection(
            "jdbc:hive2://sdbserver1:10000/default",// url
            "sdbadmin",// Hive on Spark 用户名
            ""// Hive on Spark 密码（默认未开启鉴权）
    );
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} catch (SQLException e) {
    e.printStackTrace();
}
// 返回 jdbc 连接
return connection;
```

将创建 JDBC 连接代码粘贴至 HiveUtil 类 getConnection 方法的 `TODO -- lesson1_sample:code1` 注释处（65 行），粘贴后效果如下：

![1738-410-12](https://doc.shiyanlou.com/courses/1738/1207281/a446d8decbf2dacf4dced8fc12595f64-0)

#### JDBC 创建数据库对象

JDBC 创建数据库对象的代码内容如下：

```java
// 获取 jdbc 连接
Connection connection = getConnection();
Statement statement = null;
try {
    // 装载sql语句
    statement = connection.createStatement();
    // 提交sql语句
    statement.execute(sql);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // 释放资源
    releaseSource(null, statement, connection);
}
```

将 JDBC 创建数据库对象代码粘贴至HiveUtil 类 doDDL方法的 `TODO -- lesson1_sample:code2` 注释处（54 行），粘贴后效果如下：

![1738-410-13](C:\Users\14620\Desktop\Spark开发课程\图片\lesson1\1738-410-13.png)

#### JDBC 操作数据库记录

JDBC 提交数据库记录操作语句代码内容如下：

```java
// 获取 jdbc 连接
Connection connection = getConnection();
Statement statement = null;
try {
    // 装载sql语句
    statement = connection.createStatement();
    // 提交sql语句
    statement.executeUpdate(sql);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // 释放资源
    releaseSource(null, statement, connection);
}
```

将 JDBC 操作数据库记录语句粘贴至 HiveUtil 类 doDML 方法的 `TODO -- lesson1_sample:code3` 注释处（43行），粘贴后效果如下：

![1738-410-14](https://doc.shiyanlou.com/courses/1738/1207281/7837bf0f64a0e7106478fdf3909750f9-0)

#### JDBC 查询数据库记录结果集

JDBC 查询数据库记录代码如下：

```java
// 获取 jdbc 连接
Connection connection = getConnection();
ResultSet resultSet = null;
Statement statement = null;
try {
    // 装载sql语句
    statement = connection.createStatement();
    // 提交sql语句获取查询结果集
    resultSet = statement.executeQuery(sql);
    // 格式化打印返回结果集
    ResultFormat.printResultSet(resultSet);
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    // 释放资源
    releaseSource(resultSet, statement, connection);
}
```

将 JDBC 查询数据库记录的语句粘贴至 HiveUtil 类 doDQL 方法的 `TODO -- lesson1_sample:code4` 注释处（ 32 行），粘贴后效果如下：

![1738-410-15](https://doc.shiyanlou.com/courses/1738/1207281/4654fdcef8d17f7f17a9b4aee308e9cd-0)

#### 释放 JDBC 资源

releaseSource() 为 HiveUtil 的一个公用方法，用于释放各种 JDBC 操作中使用到的 resultset、statement 和 connection 等资源。代码内容如下：

```java
// 释放 resultset
if (null != resultSet) {
    try {
        resultSet.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
// 释放 statement
if (null != statement) {
    try {
        statement.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
// 释放 connection
if (null != connection) {
    try {
        connection.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

将释放 jdbc 资源代码粘贴至 HiveUtil 类 releaseSource() 方法的 `!TODO -- lesson1_sample:code5` 注释处（21 行），粘贴后效果如下图所示：

![1738-410-16](https://doc.shiyanlou.com/courses/1738/1207281/3e1ce2edbee15c61ca99e0d79c758ad8-0)

## 样例程序

#### 打开 JdbcSample 类

如图所示打开 com.sequoiadb.lesson.spark.lesson1_sample.JdbcSample 类

![1738-410-17](https://doc.shiyanlou.com/courses/1738/1207281/9011d918220fff8e7947de39f0d8c68a-0)

#### JDBC 访问 Hive on Spark 样例

样例程序中调用了 HiveUtil 类中的方法，通过 JDBC 向 Hive on Spark 提交 SQL 语句。程序代码内容如下：

```java
// 初始化表
String dropTable =
        "DROP TABLE " +
                "IF " +
                "EXISTS jdbc_sample";
// 调用HiveUtil类doDDL()方法通过jdbc执行drop语句
HiveUtil.doDDL(dropTable);
// 创建表
String createTable =
        "CREATE TABLE jdbc_sample ( id INT, val VARCHAR ( 10 ) )";
// 调用HiveUtil类doDDL()方法通过jdbc执行建表语句
System.out.println("正在创建表……");
HiveUtil.doDDL(createTable);
// 插入数据
String insertDate =
        "INSERT INTO jdbc_sample " +
                "VALUES " +
                "( 1, \"abc\" )";
// 调用HiveUtildoDML()方法通过jdbc执行插入语句
System.out.println("正在写入记录……");
HiveUtil.doDML(insertDate);
// 查询结果集
String getResultSet =
        "SELECT " +
                "id, " +
                "val  " +
                "FROM " +
                "jdbc_sample";
// 通过HiveUtil类doDQL()方法通过jdbc获得结果集
System.out.println("正在查询记录……");
HiveUtil.doDQL(getResultSet);
```

将运行样例代码粘贴至 JdbcSample  类 sample 方法的 `!TODO -- lesson1_sample:code6` 注释处（20 行）。粘贴后效果如下图所示：

![1738-410-18](https://doc.shiyanlou.com/courses/1738/1207281/1b19b5c6a64a52c28c155e04be0410a0-0)

## 运行样例

#### 运行程序

右键点击 SampleMainTest 类，选择 `Run` 运行主函数：

![1738-410-19](https://doc.shiyanlou.com/courses/1738/1207281/05ba1cfaaf96207aed32a4def121aaf7-0)

#### 运行结果

程序运行结果如下图所示：

![1738-410-20](https://doc.shiyanlou.com/courses/1738/1207281/ced8b116cc695d4f65e8d81288fafa0d-0)

## 总结

通过本实验，我们对 Spark 技术架构和工作原理有了大致的了解，以及 Spark 是如何和 SequoiaDB 分布式存储集群交互工作的。此外，通过简单的实践练习我们可以通过 JDBC访问 Hive on Spark 并提交 SQL 任务，后续的若干章节会根据本章节的基础继续展开。
