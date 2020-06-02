---
show: step
version: 1.0 

---

## 课程介绍

本课程将介绍 Spark 的 RDD、DataSet 等有关概念，通过 scala 程序实现 word count 来简要说明 Spark RDD 操作。本章之前都是通过 SQL API 的方式实现 Spark 和 SequoiaDB 的交互，本章将通过简单的例子展示如何通过 RDD 实现 Spark 与 SequoiaDB 的交互。

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

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的系统环境为 Ubuntu 16.04.6 LTS 版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。Spark 计算引擎为 2.4.3 版本。

## 打开项目

#### 打开 IDEA

1）打开 IDEA 代码开发工具。

![1738-410-07](https://doc.shiyanlou.com/courses/1738/1207281/72397a857808ab74f01b042f07ea0a27-0)

#### 打开 SCDD-Spark 项目

2）打开 Spark 实验的项目，在该项目中完成所有实验步骤。

![1738-410-08](https://doc.shiyanlou.com/courses/1738/1207281/5497d93bf19ee0442b4ae79b4cd8d39a-0)

#### Maven 依赖

实验环境中已经在 Maven 本地仓库添加了实验所需的依赖。当前实验使用到的 jar 包依赖以及依赖说明如下：

* spark-core_2.11-2.1.1.jar

  Spark 核心 jar 包

* spark-sql_2.11-2.1.1.jar

  Spark SQL 组件 jar 包

* spark-sequoiadb_2.11-3.2.4.jar

  SequoiaDB 的 Spark 连接驱动

* sequoiadb-driver-3.2.1.jar

  SequoiaDB Java 驱动

* fastjson-1.2.58.jar

  JSON 工具 jar 包

pom.xml 文件位置：

![1738-410-pom文件位置](https://doc.shiyanlou.com/courses/1738/1207281/822fa966b397b80c9eabbf0472eb52c4-0)

当前实验中使用到的 Maven 依赖如下：

![1738-460-maven1](https://doc.shiyanlou.com/courses/1738/1207281/e52d8d1471e3ea4db14081776ad479fd-0)

![1738-460-maven2](https://doc.shiyanlou.com/courses/1738/1207281/c9d72bc998cd4b5929ee95e509cf8271-0)

![1738-410-maven2](https://doc.shiyanlou.com/courses/1738/1207281/3d62b4c63338baa02ce4337f029ff166-0)

![1738-460-maven4](https://doc.shiyanlou.com/courses/1738/1207281/73495601a2f1cc48aca7e2a276c87fc7-0)

![1738-430-maven3](https://doc.shiyanlou.com/courses/1738/1207281/b849db56b5651754fad5ff0e3115d1ac-0)

> **说明**
>
> spark-sequoiadb_2.11-3.2.4.jar 可以在 [SequoiaDB 巨杉数据库下载中心](http://download.sequoiadb.com/cn/driver) 获取。

#### 打开当前实验的 Package

3）如图所示找到当前实验程序所在 Package，在该 Package 中完成后续实验步骤：

![1738-460-package](https://doc.shiyanlou.com/courses/1738/1207281/3415a3a31133a75480a6c49e2d93845d-0)

## RDD word count

#### 概述

在进行本实验步骤前，需要了解 Spark 的数据模型：

* RDD

  Resilient Distributed Dataset（弹性分布式数据集），是 Spark 的基本数据模型。

* DataFrame

  DataFrame 是一种以 RDD 为基础的分布式数据集。和 RDD 相比，DataFrame 除了记录数据内容以外，还记录了数据的结构：

  ![1738-460-01](https://doc.shiyanlou.com/courses/1738/1207281/103159c31d74ee7026f6316ee1fb259b-0)

  因此，Spark 在使用 DataFrame 时可以根据数据的 Schema 信息进行针对性的优化，提高运行效率

* DataSet

  DataFrame 也可以叫 Dataset[Row] ，每一行的类型是 Row，不进行解析。而 Dataset 中，每一行是什么类型是不一定的。

当前实验将读取读取 txt 文件中的单词内容生成 RDD，并通过 RDD 的转换、合并等操作计算文本中出现的单词数量。txt 文件位置以及内容如下图所示：

![1738-460-txt](https://doc.shiyanlou.com/courses/1738/1207281/23881c0a94af75d3e123f4aa05b67557-0)

#### 操作步骤

1）打开 WordCount object。

![1738-460-打开object](https://doc.shiyanlou.com/courses/1738/1207281/3003a197c66cf1d9f0b6cc60a53183dd-0)

2）复制创建 SparkSession 代码。SparkSession 为 Spark SQL DataSet API 的入口。在创建 SparkSession 时需要指定 master（当前实验环境中 Spark 为单机部署故设置为 local）、appName（自定义 app 名）、以及有关的 config（sequoiadb.host 为必须配置，指定为 SequoiaDB 的主机名和协调节点名）。

```scala
// Create SparkSession
sparkSession = SparkSession.builder()
  .master("local[*]") // Specify master: local [*] means using available threads for calculation
  .appName("word count") // Specify the app name
  .config("spark.driver.allowMultipleContexts", true) // Configuration allows multiple SparkContext
  .config("sequoiadb.host", "sdbserver1:11810") // Configure the host used by Spark to access SequoiaDB
  .getOrCreate()
```

3）将创建 SparkSession 代码粘贴至 WordCount object 方法的 TODO code 1 注释区间内。

![1738-460-TODO1](https://doc.shiyanlou.com/courses/1738/1207281/9f21c5bbf0ad314495689b76be82ccb7-0)

粘贴后完整代码如图：

![1738-460-DONE1](https://doc.shiyanlou.com/courses/1738/1207281/6fb7a3cec4fa4ceb641f78461b141e6b-0)

4）复制通过 RDD 统计单词数代码。程序中将文件读取为 RDD，经过 map、reduceby 等操作转化成新的保存有单词个数统计信息的 RDD。运行程序后会将最终 RDD 打印到控制台。

```scala
// Read RDD from file
var wordsRDD = sparkSession.sparkContext.textFile("src/main/resources/txt/words.txt")
// Convert RDD to key-value pair; key is word and value is 1
val wordsPairRDD = wordsRDD.map(f => (f, 1))
// Combine elements with the same key name in wordsPairRDD
wordsCountPair = wordsPairRDD.reduceByKey(_ + _)
```

5）将通过 RDD 统计单词数代码粘贴至 WordCount object 中 wordCount 方法的 TODO code 2 注释区间内。

![1738-460-TODO2](https://doc.shiyanlou.com/courses/1738/1207281/5df4556396454368ebd2dc7dfb2344ae-0)

粘贴后完整代码如图：

![1738-460-DONE2](https://doc.shiyanlou.com/courses/1738/1207281/2be3eb7bfe7707d4e527c18cf16f774b-0)

6）右键点击 WordCountMainTest object，选择 Create/Edit WordCountMainTest.main() 编辑主函数参数。

![1738-460-编辑1](https://doc.shiyanlou.com/courses/1738/1207281/732112c752489330b752398c155e5070-0)

7）编辑主函数参数为 wordcount。

![1738-460-参数1](https://doc.shiyanlou.com/courses/1738/1207281/cddeafb334ab356d1ca8f3868fda30af-0)

8）右键点击 WordCountMainTest object，选择 Run WordCountMainTest.main() 运行程序。

![1738-460-运行1](https://doc.shiyanlou.com/courses/1738/1207281/89cb45388364c2838e80e0dd2dbde376-0)

9）查看运行结果。

![1738-460-结果1](https://doc.shiyanlou.com/courses/1738/1207281/b39ad6aafae1a3e2a19bf91e1b7ef0b9-0)

## 将 RDD 写入 SequoiaDB

#### 概述

当前实验步骤中将介绍如何将 RDD 写入到 SequoiaDB 的集合中。

#### 操作步骤

1）打开 WordCount object。

![1738-460-打开object](https://doc.shiyanlou.com/courses/1738/1207281/3003a197c66cf1d9f0b6cc60a53183dd-0)

2）复制将 RDD 写入 SequoiaDB 代码。程序中将获取上一实验步骤中保存有单词数信息的 RDD，将其转换成适合 SequoiaDB 存储的 BSON 类型格式后，调用 SequoiaDB 的 Spark 连接驱动中的方法将 BSON 类型的 RDD 写入到 SequoiaDB 的集合中。运行程序时会调用封装好的方法打印 SequoiaDB 集合中的数据。

```scala
// Convert RDD into a format that suitable for SequoiaDB storage (BSON)
var wordsCountBSON = wordsCountPair.map(f => {
  var record: BSONObject = new BasicBSONObject()// Create BSONObject
    .append("word", f._1.asInstanceOf[String])// Add word information to BSONObject
    .append("count", f._2)// Add the number of words to BSONObject
  record// Return record
})
// Write converted RDD to SequoiaDB
wordsCountBSON.saveToSequoiadb(
  "sdbserver1:11810", // Specify the coord node
  "sample", // Specify collection space
  "wordcount"// Specify collection
)
```

3）将复制的代码粘贴至 WordCount object 中 writeCollection 方法的 TODO code 3 注释区间内。

![1738-460-TODO3](https://doc.shiyanlou.com/courses/1738/1207281/888d75c28960c89ab71a35ab9b0f2097-0)

粘贴后完整代码如图：

![1738-460-DONE3](https://doc.shiyanlou.com/courses/1738/1207281/197d43e589f57a08c7e9080c307b73d8-0)

4）右键点击 WordCountMainTest object，选择 Create/Edit WordCountMainTest.main() 编辑主函数参数。

![1738-460-编辑1](https://doc.shiyanlou.com/courses/1738/1207281/732112c752489330b752398c155e5070-0)

5）编辑主函数参数为 savetosdb。

![1738-460-参数2](https://doc.shiyanlou.com/courses/1738/1207281/e8b5d9e0f58d110e932b075aecd534c7-0)

6）右键点击 WordCountMainTest object，选择 Run WordCountMainTest.main() 运行程序。

![1738-460-运行1](https://doc.shiyanlou.com/courses/1738/1207281/89cb45388364c2838e80e0dd2dbde376-0)

7）查看运行结果。

![1738-460-结果2](https://doc.shiyanlou.com/courses/1738/1207281/2a809eb30eef1f603aa44e4bbb167bce-0)

## 读取 SequoiaDB 集合为 RDD

#### 概述

当前实验步骤将演示如何将 SequoiaDB 中的记录读取为 RDD。

#### 操作步骤

1）打开 WordCount object。

![1738-460-打开object](https://doc.shiyanlou.com/courses/1738/1207281/3003a197c66cf1d9f0b6cc60a53183dd-0)

2）复制读取 SequoiaDB 集合为 RDD 代码。程序中调用 SequoiaDB 的 Spark 连接驱动中的方法读取 SequoiaDB 集合为 RDD，并指定集合中需要的字段将其转换成新的 RDD。运行程序时将最终 RDD 到控制台。

```scala
// Read data from SequoiaDB to RDD
val sdbRDD = sparkSession.sparkContext.loadFromSequoiadb(
  "sample", // Collection space
  "wordcount"// Collection
)
// Reassemble the word and count information in RDD into a new RDD
sdbPairRDD = sdbRDD.map(f => (f.get("word"), f.get("count")))
```

3）将复制的代码粘贴至 WordCount object 中 readCollection 方法的 TODO code 4 注释区间内。

![1738-460-TODO4](https://doc.shiyanlou.com/courses/1738/1207281/5253abe04754cc39a4502831b9732f97-0)

粘贴后完整代码如图：

![1738-460-DONE4](https://doc.shiyanlou.com/courses/1738/1207281/49d25f6cee98d057288633f81dc09e8a-0)

4）右键点击 WordCountMainTest object，选择 Create/Edit WordCountMainTest.main() 编辑主函数参数。

![1738-460-编辑1](https://doc.shiyanlou.com/courses/1738/1207281/732112c752489330b752398c155e5070-0)

5）编辑主函数参数为 loadfromsdb。

![1738-460-参数3](https://doc.shiyanlou.com/courses/1738/1207281/0a10edf320b7dfafcb00de6045fe4d90-0)

6）右键点击 WordCountMainTest object，选择 Run WordCountMainTest.main() 运行程序。

![1738-460-运行1](https://doc.shiyanlou.com/courses/1738/1207281/89cb45388364c2838e80e0dd2dbde376-0)

7）查看运行结果。

![1738-460-结果3](https://doc.shiyanlou.com/courses/1738/1207281/9eb85970af18d4c3e4ce8dbc7ef45ac9-0)

## 总结

通过本课程的学习，可以了解 Spark 中 RDD、DataFrame 和 DataSet 的区别和联系。在实验中使用 RDD 实现了简单的 word count 来展示的 Spark RDD 的简单操作，并通过从 SequoiaDB 读写 RDD 的例子展示了如何使用 Spark 的 RDD 和 SequoiaDB 交互。
