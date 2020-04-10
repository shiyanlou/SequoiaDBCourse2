# 1 Spark 概述

## 1.1课程介绍

本课程将介绍 Spark 的技术架构，以及通过 Spark SQL 访问 SequoiaDB 外部数据源的原理等有关知识，并通过简单的实验介绍如何通过 jdbc 访问 Spark SQL。

### 1.1.1实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* SequoiaDB version: 5.0
* SequoiaSQL-MySQL version: 3.4
* openjdk version "1.8.0_242"
* IntelliJ IDEA Community Version: 2019.3.4
* spark version: 2.4.3

## 1.2 知识点

### 1.2.1 Spark 简介

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165410.png" alt="spark-logo-trademark" style="zoom: 80%;" />

Spark 是加州大学伯克利分校AMP实验室开发的通用大数据处理框架。Spark 在 2013 年 6 月进入 Apache 成为孵化项目，8 个之后成为了 Apache 的顶级项目，很快 Spark 就成为了社区的热门，围绕 Spark 推出了 Spark SQL、Spark Streaming、MLlib、GraphX 和 SparkR 等丰富的组件。

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165411.png" alt="spark-stack" style="zoom:67%;" />

Spark 使用 Scala 语言实现，具有易用性的特点，除 Scala 以外，Spark 还提供了 Java、Python、R 甚至是 SQL 的 API 。Spark 还具有很强的适应性，可以在 Hadoop，Apache Mesos，Kubernetes，standalone 模式或是云端中运行。

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165412.png" alt="spark-runs-everywhere" style="zoom:80%;" />

此外，Spark 还可以访问各种数据源。

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165413.png" alt="largest-open-source-apache-spark" style="zoom:80%;" />

### 1.2.2 Spark 集群模式工作原理

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165415.png" alt="访问SequoiaDB" style="zoom: 67%;" />

在集群中，Spark应用以独立的进程集合的方式运行，并由主程序（driver program）中的 SparkContext  对象进行统一的调度。当需要在集群上运行时，SparkContext 会连接到几个不同类的 ClusterManager（集群管理器）上（Spark  自己的 Standalone/Mesos/YARN）, 集群管理器将给各个应用分配资源。连接成功后，Spark  会请求集群各个节点的Executor（执行器），它是为应用执行计算和存储数据的进程的总称。之后，Spark会将应用提供的代码（应用已经提交给  SparkContext 的 JAR 或 Python 文件）交给 executor。最后，由SparkContext 发送tasks提供给 executor 执行（多线程）。

### 1.2.3 Spark SQL 简介

Spark SQL 是 Spark 用于处理结构化数据的的模块。通过 Spark SQL 可以：

* 在 Java、Python 和 R 程序中通过标准 SQL 查询结构化数据；
* 使用通用方式访问各种数据源，甚至可以跨数据源连接数据；
* 访问现有的 Hive MetaStore；
* 使用标准的 JDBC 和 ODBC 连接方式访问数据源

### 1.2.4 Spark + SequoiaSQL-MySQL + SequoiaDB

<img src="https://raw.githubusercontent.com/AarZK/picstore/master/20200406165429.png" alt="image-20200404231656020" style="zoom:67%;" />

Spark 具有访问多种外部数据源的特性。在 SequoiaDB 分布式存储架构中，Spark 可以像访问 MySQL 数据库那样访问 SequoiaDB 分布式存储的 MySQL 实例，也可以通过 SequoiaDB 的 Spark 连接器直接访问底层的 SequoiaDB 存储集群。

## 1.3 JDBC 连接 Spark SQL

### 1.3.1 Maven 工程介绍

* 打开 SCDD-Spark 工程

  ![image-20200404231817998](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165430.png)

* 工程结构

  ![cluster-overview](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165414.png)

* 当前实验使用到的 Maven 依赖

  ```xml
          <dependency>
              <!-- hive 的 jdbc 连接依赖 -->
              <groupId>org.apache.hive</groupId>
              <artifactId>hive-jdbc</artifactId>
              <version>1.2.1</version>
          </dependency>
  ```

### 1.3.2 jdbc 代码

* step1：定义 jdbc 连接信息

  ```java
      // jdbc连接信息（Spark SQL默认未开启鉴权）
      private static final String url = "jdbc:hive2://192.168.1.254:10000/sample";
      private static final String username = "sdbadmin";
      private static final String password = "";
  ```

  将 jdbc 连接信息粘贴至 `！TODO --lesson1_sample:step1` 标签处

  ![image-20200404210526454](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165416.png)

* step2: 创建 jdbc 连接

  ```java
          Connection connection = null;
          try {
              // 获取 jdbc 驱动类
              Class.forName("org.apache.hive.jdbc.HiveDriver");
              // 创建 jdbc 连接
              connection = DriverManager.getConnection(url, username, password);
          } catch (ClassNotFoundException e) {
              e.printStackTrace();
          } catch (SQLException e) {
              e.printStackTrace();
          }
          // 返回 jdbc 连接
          return connection;
  ```

  将创建 jdbc 连接代码粘贴至 `!TODO -- lesson1_sample:step2` 标签处

  ![image-20200404230234370](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165426.png)

* step3：jdbc 创建数据库对象

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

  将 jdbc 创建数据库对象代码粘贴至 `！TODO -- lesson1_sample:step3` 标签处

  ![image-20200404215018708](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165417.png)

* step4：jdbc 操作数据库记录

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

  将 jdbc 操作数据库记录语句粘贴至 `!TODO -- lesson1_sample:step4` 标签处

  ![image-20200404220107388](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165418.png)

* step5：jdbc 查询数据库记录结果集

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
          } catch (SQLException e) {
              e.printStackTrace();
          } finally {
              // 释放资源
              releaseSource(null, statement, connection);
          }
          // 返回查询结果集
          return resultSet;
  ```

  将 jdbc 查询数据库记录的语句粘贴至 `!TODO -- lesson1_sample:step5` 标签处

  ![image-20200404221502305](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165420.png)

* step6：释放 jdbc 资源

  ```java
          // 释放 resultset
          if (resultSet != null) {
              try {
                  resultSet.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          // 释放 statement
          if (statement != null) {
              try {
                  statement.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
          // 释放 connection
          if (connection != null) {
              try {
                  connection.close();
              } catch (SQLException e) {
                  e.printStackTrace();
              }
          }
  ```

  将释放 jdbc 资源代码粘贴至 `!TODO -- lesson1_sample:step6` 标签处

  ![image-20200404220431994](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165419.png)

### 1.3.3 程序样例

* step7：

  ```java
          // 初始化表
          String dropTable =
                  "DROP TABLE\n" +
                          "IF\n" +
                          "\tEXISTS jdbc_sample";
          // 调用HiveUtil类doDDL()方法通过jdbc执行drop语句
          HiveUtil.doDDL(dropTable);
          // 创建表
          String createTable =
                  "CREATE TABLE jdbc_sample ( id INT, val VARCHAR ( 10 ) )";
          // 调用HiveUtil类doDDL()方法通过jdbc执行建表语句
          HiveUtil.doDDL(createTable);
          // 插入数据
          String insertDate =
                  "INSERT INTO jdbc_sample\n" +
                          "VALUES\n" +
                          "\t( 1, \"abc\" )";
          // 调用HiveUtildoDML()方法通过jdbc执行插入语句
          HiveUtil.doDML(insertDate);
          // 查询结果集
          String getResultSet =
                  "SELECT\n" +
                          "\tid,\n" +
                          "\tval \n" +
                          "FROM\n" +
                          "\tjdbc_sample";
          // 通过HiveUtil类doDQL()方法通过jdbc获得结果集
          ResultSet resultSet = HiveUtil.doDQL(getResultSet);
          // 调用自定义的结果集打印机格式化控制台输出
          try {
              ResultFormat.printResultSet(resultSet);
          } catch (SQLException e) {
              e.printStackTrace();
          }
  ```

  将运行样例代码粘贴至 `!TODO -- lesson1_sample:step7` 标签处

  ![image-20200404222935086](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165422.png)

### 1.3.4 运行样例

* 运行程序

  右键 Run 当前样例程序

  ![image-20200404222230999](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165421.png)

* 获取返回结果

  ![image-20200404231032908](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165428.png)

### 1.3.4 查看 Spark 任务

* 浏览器打开 `localhost:8080` 可以打开 Spark 任务显示平台

* ![image-20200404232237113](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165431.png)

  ![image-20200404230920894](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165427.png)

  主页面及相关提示信息说明如下:

  ![image-20200404225924761](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165424.png)

* 查看Spark任务

  点击正在运行的 thriftserver

  ![image-20200404223229475](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165423.png)

  可以看到当前thriftserver处理的任务及任务进展

  ![image-20200404230117678](https://raw.githubusercontent.com/AarZK/picstore/master/20200406165425.png)

## 1.4 总结

通过本实验，我们对 Spark 技术架构和工作原理有了大致的了解，以及 Spark 是如何和 SequoiaDB 分布式存储集群交互工作的。此外，通过简单的实践练习我们可以通过 jdbc 访问 Spark 并提交 SQL 任务，后续的若干章节会根据本章节的基础继续展开。
