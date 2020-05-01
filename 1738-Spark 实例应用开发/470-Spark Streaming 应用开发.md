---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 另一个热门组件——Spark Streaming ，并通过 word count 的例子简单展示 Spark Streaming 的用法。

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

课程使用的系统环境为 Ubuntu 16.04.6 LTS 版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。Spark 计算引擎为 3.4 版本。

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

* spark-streaming_2.11-2.4.3.jar

  Spark Streaming 组件 jar 包


pom.xml 文件位置：

![1738-410-pom文件位置](https://doc.shiyanlou.com/courses/1738/1207281/822fa966b397b80c9eabbf0472eb52c4-0)

当前实验中使用到的 Maven 依赖如下：

![1738-460-maven1](https://doc.shiyanlou.com/courses/1738/1207281/e52d8d1471e3ea4db14081776ad479fd-0)

![1738-470-maven2](https://doc.shiyanlou.com/courses/1738/1207281/a233222c8c883e31c4d6cdff59d7921c-0)

#### 打开当前实验的 Package

3）如图所示找到当前实验程序所在 Package，在该 Package 中完成后续实验步骤：

![1738-470-打开package](https://doc.shiyanlou.com/courses/1738/1207281/df399aa82ab1fcd8d71cdf90a48b5f15-0)

## Spark Streaming 持续统计端口输入的单词数

#### 概述

![1738-470-01](https://doc.shiyanlou.com/courses/1738/1207281/0ac1cf65cb1592f164e8d192beda79e3-0)

Spark  Streaming 是 Spark 的一个拓展组件，可以实现实时数据流的可拓展、高吞吐和可容错流处理。上游数据可以来自于 Kafka、Flume、S3 等多种数据源，同时可以支持类似 map、reduce、join 等复杂的算法和高级语言处理数据流，最终量处理过后的数据推送至文件系统、数据库或 Dashboards。

![1738-470-02](https://doc.shiyanlou.com/courses/1738/1207281/e27456ccdd41ab26289b5b714b5cef82-0)

Spark Streaming 在工作过程中，实时地接收输入的数据流，并将数据分成批次，然后由Spark引擎进行处理，以生成批次的最终结果流。Spark Streaming 提供被称为 离散流 或 DStream 的高级抽象，在交给 Spark 集群处理时，DStream 表示为 RDD 序列。

Spark Streaming 常被运用于实时流处理场景。

本实验将简单演示如何通过 Spark Streaming 持续捕获指定端口输入的单词，将一定时间间隔内的单词及统计数量生成 RDD 并将其打印到控制台。

#### 操作步骤

1）打开 WordCount 类。

![1738-470-打开类](https://doc.shiyanlou.com/courses/1738/1207281/8900bda62b971c1cb99603c7b934b707-0)

2）复制通过 Spark Streaming 统计指定端口输入单词数的程序代码。程序中将创建 StreamingContext 监听 sdbserver1 的 6789 端口，每间隔 10 秒收集一次时间段内端口输入的单词数生成 JavaDStream（相当于一个每次统计单词数的 RDD 的集合），和实验 6 中 RDD 的转化类似，可以将 JavaDStream 通过 map、reduce 等操作转化成最终的 JavaPairDStream（相当于保存有单词统计信息键值对 RDD 的集合）。程序运行期间会持续监听 6789 端口，持续打印 10 时间间隔内端口输入的单词数。

```java
// Configure master and appname of spark
// Master must be local [n], n> 1, indicating that one thread receives data and n-1 threads process data
// local [*] means using available threads to process data
SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("streaming word count");
// Create sparkcontext
JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
// Create streamingcontext
// Durations are the time intervals calculated for the stream
JavaStreamingContext javaStreamingContext = new JavaStreamingContext(javaSparkContext, Durations.seconds(10));
// Creating stream gets the specified port input (nc -lk 6789) through socket.
JavaReceiverInputDStream<String> lines =
        javaStreamingContext.socketTextStream("sdbserver1", 6789);
// Create matching style specified as spaces
Pattern SPACE = Pattern.compile(" ");
// Divide each line of the port input into words according to the Pattern
JavaDStream<String> words = lines.flatMap(x -> Arrays.asList(SPACE.split(x)).iterator());
// Words are converted into key-value pairs (key: words, value: 1) for merging easily.
JavaPairDStream<String, Integer> pairs = words.mapToPair(s -> new Tuple2<>(s, 1));
// Merge the same word count
JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((i1, i2) -> i1 + i2);
// Print statistics (within 10 seconds) to the console
wordCounts.print();
try {
    // Start the stream computing
    javaStreamingContext.start();
    // Wait for the end
    javaStreamingContext.awaitTermination();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

3）将代码粘贴至 WordCount 类的 wordCount 

![1738-470-TODO1](https://doc.shiyanlou.com/courses/1738/1207281/d08a49770ff89dd685cb1466d367a93a-0)

粘贴后代码如下图所示（截图中已将 try 代码块折叠）：

![1738-470-DONE1](https://doc.shiyanlou.com/courses/1738/1207281/d04031291d348191237bfc7c8f070777-0)

4）点击 IDEA 控制台下方的 Terminal ，输入 nc -lk 6789 回车后等待输入单词。

![1738-470-打开端口](https://doc.shiyanlou.com/courses/1738/1207281/a31f6ef518e3cfdbf62cac2164757a1a-0)

4）右键点击 WordCountMainTest 类，选择 Run WordCountMainTest.main() 运行程序。

![1738-470-运行1](https://doc.shiyanlou.com/courses/1738/1207281/a08a7f842cdc4a00bf8274b46b713a68-0)

5）点击 IDEA 控制台下方的 Terminal ，在 6789 端口下连续输入多个单词。

![1738-470-输入单词](https://doc.shiyanlou.com/courses/1738/1207281/85a55316bd032577d423d0b065fe862c-0)

6）点击 IDEA 控制台下方的 Run，查看 Spark Streaming 统计单词数结果。

![1738-470-运行结果](https://doc.shiyanlou.com/courses/1738/1207281/4e9ddbad29febd22a0bdd2d22fa591e4-0)

7）点击 IDEA 控制台左侧红色方格关闭 Spark Streaming 程序。

![1738-470-关闭程序](https://doc.shiyanlou.com/courses/1738/1207281/d5d850641c118fc603a2f98080d0274c-0)

## 总结

通过本课程的学习，可以了解到 Spark Streaming 的特性和工作原理。在实验中监听 socket 端口的消息并通过 Spark Streaming 进行实时的流计算统计指定时间间隔内的单词数，对 Spark Streaming 的应用可以有初步的了解。在实际应用中 Spark Streaming 经常和 Kafka 等消息队列工具一起使用，以应对实时流计算的场景。
