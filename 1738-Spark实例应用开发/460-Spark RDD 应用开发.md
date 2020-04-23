---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 的 RDD、DataSet 等有关概念，通过程序实现 word count 来简要说明 Spark RDD 操作。本章之前都是通过 SQL API 的方式实现 Spark 和 SequoiaDB 的交互，本章将通过简单的例子展示如何使用 Spark SQL 的 DataSet API 和 SequoiaSQL-MySQL 实例进行交互。

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

#### 实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* SequoiaDB version: 3.4
* SequoiaSQL-MySQL version: 3.4
* JDK version "1.8.0_172"
* IntelliJ IDEA Community Version: 2019.3.4
* Spark version: 2.4.3

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具

![1738-460-02](https://doc.shiyanlou.com/courses/1738/1207281/da25b7d1777ca6eb909a4afa05c95fe7-0)

#### 打开 SCDD-Spark 项目

选择 Spark 课程项目

![1738-460-03](https://doc.shiyanlou.com/courses/1738/1207281/24aaf448f76d9e2e61a59a6dc44aa507-0)

#### 打开当前实验的 Package

如图所示找到当前实验使用的程序所在 Package

![1738-460-04](https://doc.shiyanlou.com/courses/1738/1207281/f5e5baa583c84a7986af6b6185d6c25c-0)

#### Maven 依赖

如图所示找到 pom.xml 文件：

![pom](https://doc.shiyanlou.com/courses/1738/1207281/4474b7a73c5469e7315fc9a153d73ccc-0)

在 pom.xml 文件中可以找到当前实验使用到的 Maven 依赖：

![1738-460-05](https://doc.shiyanlou.com/courses/1738/1207281/72ddb27afd9118ddef82ca7aa3d56d39-0)



## RDD 实现 word count

程序将读取 txt 文件中的单词内容生成 RDD，并通过 RDD 的转换、合并等操作计算文本中出现的单词数量。txt 文件内容如下图所示：

![1738-460-06](https://doc.shiyanlou.com/courses/1738/1207281/264cd37a16e9df1b28ea05ef07b1caae-0)

#### 打开 RDDWordCount  类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.RDDWordCount 类：

![1738-460-07](https://doc.shiyanlou.com/courses/1738/1207281/b4532ac0a94a9538b0b50cb16f44406a-0)

#### 程序代码

```java
// Create SparkContext
SparkConf conf = new SparkConf().setAppName("wordcount").setMaster("local[*]");
JavaSparkContext sc = new JavaSparkContext(conf);
// Read file to generate RDD
JavaRDD<String> lines = sc.textFile("src/main/resources/txt/words.txt");
// Convert JavaRDD into key-value pairs. Key is word, and value is 1.
JavaPairRDD<String, Integer> pairs = lines.mapToPair(s -> new Tuple2(s, 1));
// The pair with the same key value is combined (the value is 1 and the sum is counted)
JavaPairRDD<String, Integer> counts = pairs.reduceByKey((a, b) -> a + b);
// Print the result
System.out.println(counts.collect());
```

将上述代码粘贴至 RDDWordCount 类 countWord 方法的 TODO code1 注释区间内：

![1738-460-08](https://doc.shiyanlou.com/courses/1738/1207281/d8b35031f88a90c89d71b87a1f8d18bd-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-09](https://doc.shiyanlou.com/courses/1738/1207281/e4084e1fd69f8345051730f975a418f5-0)

* 编辑主函数参数为 rddwordcount

  ![1738-460-10](https://doc.shiyanlou.com/courses/1738/1207281/214ce912c68e9c3de70fbd41d2f882cb-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-11](https://doc.shiyanlou.com/courses/1738/1207281/e005d3350e527c9cb9a457474741c1df-0)

* 运行结果如下：

  ![1738-460-12](https://doc.shiyanlou.com/courses/1738/1207281/689b73b0eac40d46b3a4e534aabff0c7-0)

## Spark SQL 实现 word count

程序将写有单词的 txt 文件读取为 RDD，并自定义 schema 将其转化成为 DataSet，利用 Spark SQL 的特性将具有结构的数据集创建成为临时表，通过 group by 的方式分组 count 出各个单词的数量。

#### 打开 SqlWordCount 类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.SqlWordCount 类：

![1738-460-13](https://doc.shiyanlou.com/courses/1738/1207281/c017e40ffbab17d262b37dc1cec9c627-0)

#### 程序代码

```java
// Create SparkSession
SparkSession spark = SparkSession.builder().master("local[*]").appName("Spark").getOrCreate();
// Read the file to generate RDD and convert it to JavaRDD
JavaRDD<Row> rows = spark.read().text("src/main/resources/txt/words.txt").toJavaRDD();
// Create an ArrayList that stores field types
ArrayList<StructField> fields = new ArrayList<StructField>();
// Create a field named word with StringType
StructField wordField = DataTypes.createStructField("word", DataTypes.StringType, true);
// Add this field to ArrayList
fields.add(wordField);
// Create a schema from an ArrayList that stores field names and field types
StructType schema = DataTypes.createStructType(fields);
// Specify the shema for the RDD read from the file, making it has a table structure
Dataset<Row> wordCount = spark.createDataFrame(rows, schema);
// Create DataSet as a temporary table
wordCount.createOrReplaceTempView("wordcount");
// Group query for temporary table to realize word count
Dataset<Row> result = spark.sql("SELECT word,count(0) AS count FROM wordcount GROUP BY word");
// Print the records
result.show();
```

将上述代码粘贴至 SqlWordCount 类 countWord 方法的 TODO code 2 注释区间内：

![1738-460-14](https://doc.shiyanlou.com/courses/1738/1207281/31cc2d56da5c30de1f934d3ef123d59c-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-15](https://doc.shiyanlou.com/courses/1738/1207281/bcf7028851fe9fcdb6bd9d74858f9fa2-0)

* 编辑主函数参数为 sqlwordcount

  ![1738-460-16](https://doc.shiyanlou.com/courses/1738/1207281/5ac0a1a2e55aee4ddb51b75ad7ebd94d-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-17](https://doc.shiyanlou.com/courses/1738/1207281/67fd16474719b3a6fd71dbfb79d094ee-0)

* 运行结果如下：

  ![1738-460-18](https://doc.shiyanlou.com/courses/1738/1207281/c99cfe5a246dc9c581e258f826580f12-0)

## 通过 DataSet 读写 MySQL 实例表

程序将 MySQL 实例的 employee 表读成 DataSet，将其创建成为临时表后分组查询统计男女职工的人数，并将统计结果保存为新的 DataSet。最后将保存有统计结果的 DataSet 写入到 MySQL 实例的新表中。

#### 打开 TableOperation 类

如图所示找到 com.sequoiadb.lesson.spark.lesson6_rdd.TableOperation 类：

![1738-460-19](https://doc.shiyanlou.com/courses/1738/1207281/abe4b22f10bffbc9bfcc84e1f2620c66-0)

#### 创建 SparkSession

```java
// Create SparkSession
private static final SparkSession sparkSession = SparkSession.builder().master("local[*]").getOrCreate();
// Global Dataset for using in different functions
private static Dataset<Row> countBySex = null;
private static Dataset<Row> employee = null;
```

将上述代码粘贴至 TableOperation 类的 TODO code 3 注释区间内：

![1738-460-20](https://doc.shiyanlou.com/courses/1738/1207281/60c869fa5001ddf8bbb5745114688dbe-0)

#### 读取 employee 表

```java
// Create a dataset from a MySQL table
employee = sparkSession.read()
        .format("jdbc")//Connect using jdbc
        .option("url", "jdbc:mysql://localhost:3306/sample?useSSL=false")// MySQL instance url
        .option("dbtable", "sample.employee")// Database name and table name of the source table
        .option("user", "root")// username
        .option("password", "root")// password
        .load();
// Print the structure of table
employee.printSchema();
// Print the result set (partial)
employee.show();
```

将上述代码粘贴至 TableOperation 类 readTable 方法的 TODO code 4 注释区间内：

![1738-460-21](https://doc.shiyanlou.com/courses/1738/1207281/6176ce0dd7b8b5b3fdccae6636322390-0)

#### 创建临时表

```java
// Create the data set employee read by Spark SQL as a temporary table
employee.createOrReplaceTempView("employee");
// Execute sql statement through sparksession
countBySex = sparkSession.sql("SELECT sex,count(1) AS num FROM employee GROUP BY sex");
// Print the structure of statistics table
countBySex.printSchema();
// Print the data of statistics table
countBySex.show();
```

将上述代码粘贴至 TableOperation 类 tmpOperation 方法的 TODO code 5 注释区间内：

![1738-460-22](https://doc.shiyanlou.com/courses/1738/1207281/b10b7091c67afc3ce8c2f9f5bb05e28d-0)

#### 将统计结果集写入 MySQL 实例表

```java
// Delete the existing MySQL instance table
MySQLUtil.dropTable("sexcount");
// Write the statistical data set to the MySQL instance
countBySex.write()
        .format("jdbc")//Connect using jdbc
        .option("url", "jdbc:mysql://sdbserver1:3306/sample?useSSL=false")// MySQL instance url
        .option("dbtable", "sample.sexcount")// Database name and table name of the source table
        .option("user", "root")// Username
        .option("password", "root")// Password
        .save();
// Print the structure of MySQL instance table
MySQLUtil.getData("desc sexcount");
// Print the result set of MySQL instance table
MySQLUtil.getData("select * from sexcount");
// Close SparkSession
sparkSession.close();
```

将上述代码粘贴至 TableOperation 类 writeTable 方法的 TODO code 6 注释区间内：

![1738-460-23](https://doc.shiyanlou.com/courses/1738/1207281/f877d50c6361f4cc18f2b9491c874ed5-0)

#### 运行程序

* 右键点击 RDDMainTest 类选择 Create/Edit 主函数

  ![1738-460-24](https://doc.shiyanlou.com/courses/1738/1207281/cd84332bb32a66da6d74908b03af8662-0)

* 编辑主函数参数为 tableoperation

  ![1738-460-25](https://doc.shiyanlou.com/courses/1738/1207281/ab48d7311f63d55f98e644ae4dc5fd19-0)

* 右键点击 RDDMainTest 类选择 Run 主函数

  ![1738-460-26](https://doc.shiyanlou.com/courses/1738/1207281/88e528e3319081ece5f68422682145ae-0)

* 运行结果如下：

  ![1738-460-27](https://doc.shiyanlou.com/courses/1738/1207281/57993866339a6b42174d1ebf1ea3347e-0)

## 总结

通过本课程的学习，可以了解 Spark 中 RDD、DataFrame 和 DataSet 的区别和联系。在实验中使用 RDD 和 DataSet 分别实现 word count 来展示的 Spark RDD 的简单操作，并通过 DataSet 读写 MySQL 实例展示了如何使用 Spark 的 DataSet API 和 SequoiaDB-MySQL 实例交互。
