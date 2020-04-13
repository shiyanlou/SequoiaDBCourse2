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
* JDK version "1.8.0_242"
* IntelliJ IDEA Community Version: 2019.3.4
* Spark version: 2.4.3

#### 知识点

* **RDD**

  Resilient Distributed Dataset（弹性分布式数据集），是 Spark 的基本数据模型。通过 RDD 可以加强对数据的控制：

  1. 直接控制数据的共享
  2. 指定数据存储到硬盘或内存
  3. 控制数据的分区方法
  4. 控制数据集上进行的操作

* **DataFrame**

  DataFrame 是一种以 RDD 为基础的分布式数据集。和 RDD 相比，DataFrame 除了记录数据内容以外，还记录了数据的结构：

  ![1738-460-01](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-01.jpg)

  因此，Spark 在使用 DataFrame 时可以根据数据的 Schema 信息进行针对性的优化，提高运行效率。

* **DataSet**

  DataFrame 也可以叫 Dataset[Row] ，每一行的类型是 Row，不进行解析。而 Dataset 中，每一行是什么类型是不一定的。

## Maven 工程介绍

* 打开 SCDD-Spark 工程

  ![1738-450-02](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-02.jpg)

* 当前实验使用到的 Maven 依赖

  <img src="C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-maven.jpg" alt="1738-460-maven" style="zoom: 67%;" />

* 打开当前实验所在包

  ![1738-460-03](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-03.png)

## 程序代码

实验将通过 3 个不同的程序分别通过 RDD 的 reduceByKey 方法实现统计单词数，通过 Spark SQL 的方式新建临时表进行分组查询实现统计查询以及一个统计男女职员人数的例子简要介绍 Spark SQL 的方式读写 MySQL 实例的表。

#### RDD reduceBykey 实现 word count

程序将读取文件生成 Spark 的 RDD，通过对 RDD 的键值转化和合并等操作生成最终保存有单词以及单词数的 RDD。程序代码内容如下：

```java
// 创建 SparkSession
SparkSession sparkSession = SparkSession.builder().master("local[*]").getOrCreate();
// 读取文件生成 RDD
RDD<String> rdd = sparkSession.sparkContext().textFile("src/main/resources/txt/words.txt", 1);
// 将 RDD 转化为 JavaRDD
JavaRDD<String> javaRDD = rdd.toJavaRDD();
// 将 JavaRDD 转化为键值对，key 为单词，value 为1
JavaPairRDD<String, Integer> javaPairRDD = javaRDD.mapToPair(new PairFunction<String, String, Integer>() {
    public Tuple2<String, Integer> call(String word) {
        return new Tuple2<String, Integer>(word, 1);
    }
});
// key 值相同的 pair 合并（value 为 1 求和计数）
JavaPairRDD<String, Integer> wordCount = javaPairRDD.reduceByKey(new Function2<Integer, Integer, Integer>() {
    @Override
    public Integer call(Integer count1, Integer count2) throws Exception {
        return count1 + count2;
    }
});
// 打印结果
System.out.println(wordCount.collect());
```

将上述内容粘贴至 RDDWordCount 类 countWord 方法中的 `TODO -- lesson6_rdd:code1` 标签内（26 行），最终样式如下图所示：

![1738-460-04](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-04.png)

#### Spark SQL 实现 word count

程序将读取文件生成保存有所有单词的 RDD ，通过将 RDD 创建为临时表并分组查询的方式将不同单词以及单词个数保存在临时表中并打印。程序代码内容如下：

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

将上述代码粘贴至 SqlWordCount 类 countWord 方法中的 `TODO -- lesson6_rdd:code2` 标签内（28 行），最终样式如下图所示：

![1738-460-05](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-05.png)

#### 通过 RDD 读写 MySQL 实例表

程序将读取 MySQL实例的 employee 表将其转化成为 DataSet，将 DataSet 转化成为临时表后分组查询进行男女职员的人数统计，将最终的统计结果保存成 DataSet 再写入到 MySQL 实例 sexcount 表中。程序代码内容如下：

* ##### 定义 SparkSession 以及 DataSet

  ```java
  // 创建 SparkSession
  private static final SparkSession sparkSession = SparkSession.builder().master("local[*]").getOrCreate();
  // 全局 Dataset 便于在不同函数中分别使用
  private static Dataset<Row> countBySex=null;
  private  static Dataset<Row> employee=null;
  ```

  将上述代码粘贴至 TableOperation 类的 `TODO -- lesson6_rdd:code3` 标签处（62 行），最终样式如下图所示：

  ![1738-460-06](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-06.jpg)

* ##### 读取 employee 表为 DataSet

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

  将上述代码粘贴至 TableOperation 类 readTable 方法的 `TODO -- lesson6_rdd:code4` 标签处（45 行），最终样式如下图所示：

  ![1738-460-07](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-07.jpg)

* ##### 创建临时表分组查询

  ```java
  // 将 Spark SQL 读取到的数据集 employee 创建为临时表
  employee.createOrReplaceTempView("employee");
  // 通过 sparksession 执行 sql 语句进行分组查询
  countBySex =sparkSession.sql("select sex,count(1) as num from employee group by sex");
  // 打印统计表结构
  countBySex.printSchema();
  // 打印统计表数据
  countBySex.show();
  ```

  将上述代码粘贴至 TableOperation 类 tmpOperation 方法的 `TODO -- lesson6_rdd:code5` 标签处（34 行），最终样式如下图所示：

  ![1738-460-08](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-08.jpg)

* ##### 将统计表写入 MySQL 新表

  ```java
  // 删除已有 MySQL 实例表
  MySQLUtil.dropTable("sexcount");
  // 将统计表写入到 MySQL 实例
  countBySex.write()
          .format("jdbc")//使用 jdbc 连接
          .option("url", "jdbc:mysql://localhost:3306/sample?useSSL=false")// MySQL 实例 url
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

  将上述代码粘贴至 TableOperation 类 writeTable 方法的 `TODO -- lesson6_rdd:code6` 标签处（23 行），最终样式如下图所示：

  ![1738-460-09](C:\Users\14620\Desktop\Spark开发课程\图片\lesson6\1738-460-09.jpg)

## 运行程序

#### RDD reduceByKey 统计单词数

* 点击工具栏选择 `Run`

  

* 选择当前课程运行主类 RDDMainTest，编辑参数为 `rddwordcount`

  

* 点击运行 RDDMainTest

  

* 查看运行结果

  

#### Spark SQL 分组查询统计单词数

* 点击工具栏选择 `Run`

  

* 选择当前课程运行主类 RDDMainTest，编辑参数为 `sqlwordcount`

  

* 点击运行 RDDMainTest

  

* 查看运行结果

  

#### DataSet 读写 MySQL 表

* 点击工具栏选择 `Run`

  

* 选择当前课程运行主类 RDDMainTest，编辑参数为 `tableoperation`

  

* 点击运行 RDDMainTest

  

* 查看运行结果

  

## 总结

通过本课程的学习，可以了解 Spark 中 RDD、DataFrame 和 DataSet 的区别和联系。在实验中使用 RDD 和 DataSet 分别实现 word count 来展示的 Spark RDD 的简单操作，并通过 DataSet 读写 MySQL 实例展示了通过 Spark SQL 是如何与 MySQL 实例交互的。