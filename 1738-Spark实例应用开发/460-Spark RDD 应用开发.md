---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 的 RDD、DataSet 等有关概念，通过程序实现 word count 来简要说明 Spark RDD 操作。本章之前都是通过 JDBC 访问 Hive on Spark 的方式实现 Spark 和 数据源的交互，本章将通过 DataSet 读写 MySQL 实例的数据表的例子展示如何通过 Spark SQL 操作数据集的方式访问 MySQL 实例。

#### 实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* SequoiaDB version: 3.4
* SequoiaSQL-MySQL version: 3.4
* JDK version "1.8.0_172"
* IntelliJ IDEA Community Version: 2019.3.4
* Spark version: 2.4.3

#### 知识点

**RDD**

Resilient Distributed Dataset（弹性分布式数据集），是 Spark 的基本数据模型。通过 RDD 可以加强对数据以下几个方面的控制：

* 直接控制数据的共享
* 指定数据存储到硬盘或内存
* 控制数据的分区方法
* 控制数据集上进行的操作

**DataFrame**

DataFrame 是一种以 RDD 为基础的分布式数据集。和 RDD 相比，DataFrame 除了记录数据内容以外，还记录了数据的结构：

![1738-460-01](https://doc.shiyanlou.com/courses/1738/1207281/103159c31d74ee7026f6316ee1fb259b-0)

因此，Spark 在使用 DataFrame 时可以根据数据的 Schema 信息进行针对性的优化，提高运行效率。

**DataSet**

DataFrame 也可以叫 Dataset[Row] ，每一行的类型是 Row，不进行解析。而 Dataset 中，每一行是什么类型是不一定的。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具

![1738-460-02](https://doc.shiyanlou.com/courses/1738/1207281/da25b7d1777ca6eb909a4afa05c95fe7-0)

#### 打开 SCDD-Spark 项目

选择 Spark 课程项目

![1738-460-03](https://doc.shiyanlou.com/courses/1738/1207281/24aaf448f76d9e2e61a59a6dc44aa507-0)

#### 打开当前实验的 Package

如图所示找到当前实验使用的程序所在 Package

![1738-460-04](https://doc.shiyanlou.com/courses/1738/1207281/368eebd5c824a5f029eb3b546b98abcc-0)

#### Maven 依赖

在 pom.xml 中可以找到当前实验需要用到的 Maven 依赖：

![1738-460-05](https://doc.shiyanlou.com/courses/1738/1207281/72ddb27afd9118ddef82ca7aa3d56d39-0)



## RDD 实现 word count

程序将读取 txt 文件中的单词内容生成 RDD，并通过 RDD 的转换、合并等操作计算文本中出现的单词数量。txt 文件内容如下图所示：

![1738-460-06](https://doc.shiyanlou.com/courses/1738/1207281/264cd37a16e9df1b28ea05ef07b1caae-0)

#### 打开 RDDWordCount  类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.RDDWordCount 类：

![1738-460-07](https://doc.shiyanlou.com/courses/1738/1207281/ab74a6c6c542fc3499d75ffbd6701a25-0)

#### 程序代码

```java
// 创建 SparkContext
SparkConf conf = new SparkConf().setAppName("wordcount").setMaster("local[*]");
JavaSparkContext sc = new JavaSparkContext(conf);
// 读取文件生成 RDD
JavaRDD<String> lines = sc.textFile("src/main/resources/txt/words.txt");
// 将 JavaRDD 转化为键值对，key 为单词，value 为1
JavaPairRDD<String, Integer> pairs = lines.mapToPair(s -> new Tuple2(s, 1));
// key 值相同的 pair 合并（value 为 1 求和计数）
JavaPairRDD<String, Integer> counts = pairs.reduceByKey((a, b) -> a + b);
// 打印结果
System.out.println(counts.collect());
```

将上述代码粘贴至 RDDWordCount 类 RDDWordCount 方法的 TODO -- lesson6_rdd:code1 注释处（24 行）：

![1738-460-08](https://doc.shiyanlou.com/courses/1738/1207281/1077df56ec8e053cdb83733e91774429-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-09](https://doc.shiyanlou.com/courses/1738/1207281/489c9cb5aae86e649e1b75ee16e56851-0)

* 编辑主函数参数为 rddwordcount

  ![1738-460-10](https://doc.shiyanlou.com/courses/1738/1207281/49469dd2cd345c2c9cf25d8c915957c1-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-11](https://doc.shiyanlou.com/courses/1738/1207281/63ee2a98da8fc84982ed02dfbbcb17cc-0)

* 运行结果如下：

  ![1738-460-12](https://doc.shiyanlou.com/courses/1738/1207281/93b4cbc12388ee1d0f24b2e96b7de115-0)

## Spark SQL 实现 word count

程序将写有单词的 txt 文件读取为 RDD，并自定义 schema 将其转化成为 DataSet，利用 Spark SQL 的特性将具有结构的数据集创建成为临时表，通过 group by 的方式分组 count 出各个单词的数量。

#### 打开 SqlWordCount 类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.SqlWordCount 类：

![1738-460-13](https://doc.shiyanlou.com/courses/1738/1207281/80b5a138387d32aa0c99614d13e933b7-0)

#### 程序代码

```java
// 创建 SparkSession
SparkSession spark = SparkSession.builder().master("local[*]").appName("Spark").getOrCreate();
// 读取文件生成 RDD 后转化成 JavaRDD
JavaRDD<Row> rows = spark.read().text("src/main/resources/txt/words.txt").toJavaRDD();
// 创建存储字段类型的 ArrayList
ArrayList<StructField> fields = new ArrayList<StructField>();
// 创建名为 word 的字段类型为 StringType
StructField wordField = DataTypes.createStructField("word", DataTypes.StringType, true);
// 将字段添加到 ArrayList 中
fields.add(wordField);
// 通过保存了字段名和字段类型的的 ArrayList 创建 schema
StructType schema = DataTypes.createStructType(fields);
// 为从文件读取到的 RDD 指定 shema 使其具有表结构
Dataset<Row> wordCount = spark.createDataFrame(rows, schema);
// 将 DataSet 创建为 临时表
wordCount.createOrReplaceTempView("wordcount");
// 临时表分组查询实现单词数统计
Dataset<Row> result = spark.sql("select word,count(0) as count from wordcount group by word");
// 打印记录
result.show();
```

将上述代码粘贴至 SqlWordCount 类 countWord 方法的 TODO -- lesson6_rdd:code2 注释处（28 行）：

![1738-460-14](https://doc.shiyanlou.com/courses/1738/1207281/24d767bd0a0969af6c2258f80482014e-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-15](https://doc.shiyanlou.com/courses/1738/1207281/bffafe4a59c7e7939f1c4a7f979f5909-0)

* 编辑主函数参数为 sqlwordcount

  ![1738-460-16](https://doc.shiyanlou.com/courses/1738/1207281/26216dc59fbafe16bc655d96cf0e565b-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-17](https://doc.shiyanlou.com/courses/1738/1207281/e9ddfe74749b4e7c8791bed5e036767b-0)

* 运行结果如下：

  ![1738-460-18](https://doc.shiyanlou.com/courses/1738/1207281/83cf696c0501c5b6ef82acf3a62f7f82-0)

## 通过 DataSet 读写 MySQL 实例表

程序将 MySQL 实例的 employee 表读成 DataSet，将其创建成为临时表后分组查询统计男女职工的人数，并将统计结果保存为新的 DataSet。最后将保存有统计结果的 DataSet 写入到 MySQL 实例的新表中。

#### 打开 TableOperation 类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.TableOperation 类：

![1738-460-19](https://doc.shiyanlou.com/courses/1738/1207281/112bff919da30ce93dbb6267c277e924-0)

#### 创建 SparkSession

```java
// 创建 SparkSession
private static final SparkSession sparkSession = SparkSession.builder().master("local[*]").getOrCreate();
// 全局 Dataset 便于在不同函数中分别使用
private static Dataset<Row> countBySex = null;
private static Dataset<Row> employee = null;
```

将上述代码粘贴至 TableOperation 类的 TODO -- lesson6_rdd:code3 注释处（65 行）：

![1738-460-20](https://doc.shiyanlou.com/courses/1738/1207281/88dedcef9c77bd5a1fd747c6697af2af-0)

#### 读取 employee 表

```java
// 从 MySQL 表创建数据集
employee = sparkSession.read()
        .format("jdbc")//使用 jdbc 连接
        .option("url", "jdbc:mysql://localhost:3306/sample?useSSL=false")// MySQL 实例 url
        .option("dbtable", "sample.employee")// 源表的库名和表名
        .option("user", "root")// 用户名
        .option("password", "root")// 密码
        .load();
// 打印表结构
employee.printSchema();
// 打印结果集（部分）
employee.show();
```

将上述代码粘贴至 TableOperation 类 readTable 方法的 TODO -- lesson6_rdd:code4 注释处（45 行）：

![1738-460-21](https://doc.shiyanlou.com/courses/1738/1207281/9fc16ee344a6724ec18f74d906c233b5-0)

#### 创建临时表

```java
// 将 Spark SQL 读取到的数据集 employee 创建为临时表
employee.createOrReplaceTempView("employee");
// 通过 sparksession 执行 sql 语句进行分组查询
countBySex = sparkSession.sql("select sex,count(1) as num from employee group by sex");
// 打印统计表结构
countBySex.printSchema();
// 打印统计表数据
countBySex.show();
```

将上述代码粘贴至 TableOperation 类 tmpOperation 方法的 TODO -- lesson6_rdd:code5 注释处（34 行）：

![1738-460-22](https://doc.shiyanlou.com/courses/1738/1207281/52301ada76382612ab3b33ee513adda2-0)

#### 将统计结果集写入 MySQL 实例表

```java
// 删除已有 MySQL 实例表
MySQLUtil.dropTable("sexcount");
// 将统计后的数据集写入到 MySQL 实例
countBySex.write()
        .format("jdbc")//使用 jdbc 连接
        .option("url", "jdbc:mysql://sdbserver1:3306/sample?useSSL=false")// MySQL 实例 url
        .option("dbtable", "sample.sexcount")// 源表的库名和表名
        .option("user", "root")// 用户名
        .option("password", "root")// 密码
        .save();
// 打印 MySQL 实例表结构
MySQLUtil.getData("desc sexcount");
// 打印MySQL 实例表结果集
MySQLUtil.getData("select * from sexcount");
// 关闭 SparkSession
sparkSession.close();
```

将上述代码粘贴至 TableOperation 类 writeTable 方法的 TODO -- lesson6_rdd:code6 注释处（23 行）：

![1738-460-23](https://doc.shiyanlou.com/courses/1738/1207281/fc1789a0ca2b1330ed47171cb95c9d52-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-24](https://doc.shiyanlou.com/courses/1738/1207281/33c27b7fd421e4af27df43bf19580654-0)

* 编辑主函数参数为 tableoperation

  ![1738-460-25](https://doc.shiyanlou.com/courses/1738/1207281/adfdb0bd6896196d1966590733bc0c75-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-26](https://doc.shiyanlou.com/courses/1738/1207281/3493016e6f0f84ad89ca798e1925e91e-0)

* 运行结果如下：

  ![1738-460-27](https://doc.shiyanlou.com/courses/1738/1207281/af3e959e6ed3b42488628170ef723f5f-0)

## 总结

通过本课程的学习，可以了解 Spark 中 RDD、DataFrame 和 DataSet 的区别和联系。在实验中使用 RDD 和 DataSet 分别实现 word count 来展示的 Spark RDD 的简单操作，并通过 DataSet 读写 MySQL 实例展示了通过 Spark SQL 是如何与 MySQL 实例交互的。
