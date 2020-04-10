---
show: step
version: 1.0 
---

## 1课程介绍

本课程将介绍 Spark 的技术架构，以及通过 Spark SQL 访问 SequoiaDB 外部数据源的原理等有关知识，并通过简单的实验介绍如何通过 jdbc 访问 Spark SQL。

#### 1.1实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* jdk version "1.8.0_242"
* SequoiaDB version: 5.0
* SequoiaSQL-MySQL version: 3.4
* spark version: 2.4.3
* IntelliJ IDEA Community Version: 2019.3.4

## 2 知识点

#### 2.1 Spark 简介

![1738-410-01](https://doc.shiyanlou.com/courses/1738/1207281/efd6b5049d176e5d5218e2c97f569412-0)

Spark 是加州大学伯克利分校AMP实验室开发的通用大数据处理框架。Spark 在 2013 年 6 月进入 Apache 成为孵化项目，8 个之后成为了 Apache 的顶级项目，很快 Spark 就成为了社区的热门，围绕 Spark 推出了 Spark SQL、Spark Streaming、MLlib、GraphX 和 SparkR 等丰富的组件。

![1738-410-02](https://doc.shiyanlou.com/courses/1738/1207281/22f9c4ffc35617c67cd74bf7b3442c7d-0)

Spark 使用 Scala 语言实现，具有易用性的特点，除 Scala 以外，Spark 还提供了 Java、Python、R 甚至是 SQL 的 API 。Spark 还具有很强的适应性，可以在 Hadoop，Apache Mesos，Kubernetes，standalone 模式或是云端中运行。

![1738-410-03](https://doc.shiyanlou.com/courses/1738/1207281/e299e0e052d994f0007e3a3950bb3c46-0)

此外，Spark 还可以访问各种数据源。

![1738-410-04](https://doc.shiyanlou.com/courses/1738/1207281/dd07c822301331de05f709c01fb71ba1-0)

#### 2.2 Spark 集群模式工作原理

![1738-410-05](https://doc.shiyanlou.com/courses/1738/1207281/622e8e88eda39e756d96e58fe9fd5ea1-0)

在集群中，Spark应用以独立的进程集合的方式运行，并由主程序（driver program）中的 SparkContext  对象进行统一的调度。当需要在集群上运行时，SparkContext 会连接到几个不同类的 ClusterManager（集群管理器）上（Spark  自己的 Standalone/Mesos/YARN）, 集群管理器将给各个应用分配资源。连接成功后，Spark  会请求集群各个节点的Executor（执行器），它是为应用执行计算和存储数据的进程的总称。之后，Spark会将应用提供的代码（应用已经提交给  SparkContext 的 JAR 或 Python 文件）交给 executor。最后，由SparkContext 发送tasks提供给 executor 执行（多线程）。

#### 2.3 Spark + SequoiaSQL-MySQL + SequoiaDB

![1738-410-06](https://doc.shiyanlou.com/courses/1738/1207281/19d04ba47ed6c6579724d464c280528c-0)

Spark 具有访问多种外部数据源的特性。在 SequoiaDB 分布式存储架构中，Spark 可以像访问 MySQL 数据库那样访问 SequoiaDB 分布式存储的 MySQL 实例，也可以通过 SequoiaDB 的 Spark 连接器直接访问底层的 SequoiaDB 存储集群。

#### 2.4 Hive on Spark

Hive on spark 是一个Hive的发展计划，由 Cloudera 发起，由 Intel、MapR 等公司共同参与的开源项目，其目的是把 Spark 作为 Hive 的一个计算引擎，将 Hive 的查询作为 Spark 的任务提交到 Spark 集群上进行计算。

Hive on spark 可以通过 Hive jdbc 的方式进行操作，本系列实验也将围绕这一方式进行展开。

## 3 Maven 工程介绍

* #### 打开 SCDD-Spark 工程

  ![1738-410-07](https://doc.shiyanlou.com/courses/1738/1207281/8af3109f14cdde63375c463dc0a985d0-0)

* #### 工程结构

  ![1738-410-08](https://doc.shiyanlou.com/courses/1738/1207281/c3f2631ce168fdce625762fcb21498f3-0)

* #### 当前实验使用到的 Maven 依赖

  ```xml
          <dependency>
              <!-- hive 的 jdbc 连接依赖 -->
              <groupId>org.apache.hive</groupId>
              <artifactId>hive-jdbc</artifactId>
              <version>1.2.1</version>
          </dependency>
  ```

## 4 Hive jdbc 代码

* #### 打开 HiveUtil 类

  ![1738-410-09](https://doc.shiyanlou.com/courses/1738/1207281/646917523d9ea29d4777f5cf6168727f-0)

* #### 定义 jdbc 连接信息

  ```java
      // hive on spark url
      private static final String url = "jdbc:hive2://192.168.1.254:10000/sample";
      // jdbc 用户名及密码（Spark SQL默认未开启鉴权）
      private static final String username = "sdbadmin";
      private static final String password = "";
  ```

  将 jdbc 连接信息粘贴至 `！TODO --lesson1_sample:step1` 标签处

  ![1738-410-10](https://doc.shiyanlou.com/courses/1738/1207281/48958bf5a56d4a7d5126e582f4b46808-0)

* #### 创建 jdbc 连接

  ```java
          // 初始化连接
          Connection connection = null;
          try {
              // 获取 jdbc 驱动类
              Class.forName("org.apache.hive.jdbc.HiveDriver");
              // 获取 jdbc 连接
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

  ![1738-410-11](https://doc.shiyanlou.com/courses/1738/1207281/b4a78881dbc29f968498d6ecb16c508a-0)

* #### jdbc 创建数据库对象

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

  ![1738-410-12](https://doc.shiyanlou.com/courses/1738/1207281/6af60c8834460a4e89041482cde862a6-0)

* #### jdbc 操作数据库记录

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

  ![1738-410-13](https://doc.shiyanlou.com/courses/1738/1207281/496a2223790497d4a72e353a04fd472f-0)

* #### jdbc 查询数据库记录结果集

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

  ![1738-410-14](https://doc.shiyanlou.com/courses/1738/1207281/287588fc3fa41d197d4e8d11b73a9eb9-0)

* #### 释放 jdbc 资源

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

  ![1738-410-15](https://doc.shiyanlou.com/courses/1738/1207281/98c863721175f2feb204e83d1ab39cb8-0)

## 5 程序样例

* #### 打开 JdbcSample 类

  ![1738-410-16](https://doc.shiyanlou.com/courses/1738/1207281/2c61b57d9378043a7bc7c3eca37e50e3-0)

* #### 样例程序内容

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

  将运行样例代码粘贴至 `!TODO -- lesson1_sample:step7` 标签处

  ![1738-410-17](https://doc.shiyanlou.com/courses/1738/1207281/7ca32825a207645f069e2da92927c135-0)

## 6 运行样例

* #### 编辑调用程序入参

  右键点击 Entry 类，选择编辑主函数

  ![1738-410-18](https://doc.shiyanlou.com/courses/1738/1207281/ae208a779dac05ad300db5157bc99a58-0)

* #### 配置运行时参数为 `lesson1 sample`

  ![1738-410-19](https://doc.shiyanlou.com/courses/1738/1207281/1567ccc6332a35aa063bcaa0a1cbb2ea-0)
  
* #### 右键点击 Entry 类选择执行程序

  ![1738-410-19](https://doc.shiyanlou.com/courses/1738/1207281/decbbf4d11f6d2262b636eb81414cec7-0)
  
* #### 查看运行结果

  ![1738-410-19](https://doc.shiyanlou.com/courses/1738/1207281/10821c88e2993b06e7371c67d4d6aacb-0)

## 7 总结

通过本实验，我们对 Spark 技术架构和工作原理有了大致的了解，以及 Spark 是如何和 SequoiaDB 分布式存储集群交互工作的。此外，通过简单的实践练习我们可以通过 jdbc 访问 Hive on Spark 并提交 SQL 任务，后续的若干章节会根据本章节的基础继续展开。
