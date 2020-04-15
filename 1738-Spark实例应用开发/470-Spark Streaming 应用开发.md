---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 另一个热门组件——Spark Streaming。并通过 word count 的例子简单展示 Spark Streaming 的用法。

#### 实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* SequoiaDB version: 3.4
* SequoiaSQL-MySQL version: 3.4
* JDK version "1.8.0_172"
* IntelliJ IDEA Community Version: 2019.3.4
* Spark version: 2.4.3

#### 知识点

![1738-470-01](https://doc.shiyanlou.com/courses/1738/1207281/0ac1cf65cb1592f164e8d192beda79e3-0)

Spark  Streaming 是 core Spark API 的一个拓展组件，可以实现实时数据流的可拓展、高吞吐和可容错流处理。上游数据可以来自于 Kafka、Flume、S3 等多种数据源，同时可以支持类似 map、reduce、join 等复杂的算法和高级语言处理数据流，最终量处理过后的数据推送至文件系统、数据库或 Dashboards。

![1738-470-02](https://doc.shiyanlou.com/courses/1738/1207281/e27456ccdd41ab26289b5b714b5cef82-0)

Spark Streaming 在工作过程中，实时地接收输入的数据流，并将数据分成批次，然后由Spark引擎进行处理，以生成批次的最终结果流。Spark Streaming 提供被称为 离散流 或 DStream 的高级抽象，在交给 Spark 集群处理时，DStream 表示为 RDD 序列。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具

![1738-470-03](https://doc.shiyanlou.com/courses/1738/1207281/bc1af44ba494781b1ad66820d670c2bc-0)

#### 打开 SCDD-Spark 项目

选择 Spark 课程项目

![1738-470-04](https://doc.shiyanlou.com/courses/1738/1207281/cbcfa7406656330373ad8cb0e65fd45a-0)

#### 打开当前实验的 Package

如图所示找到当前实验使用的程序所在 Package

![1738-470-05](https://doc.shiyanlou.com/courses/1738/1207281/ea7604b2cd9611a29e1956d34aed9b80-0)

#### Maven 依赖

在 pom.xml 中可以找到当前实验需要用到的 Maven 依赖：

![1738-470-06](https://doc.shiyanlou.com/courses/1738/1207281/35d7b9dc1a5dcdc2bbcddf9799a6a143-0)



## Spark Streaming 统计端口输入的单词数

程序将监听本地端口 6789，并通过 Spark Streaming 获取每间隔 10 秒内 6789 端口输入的单词并统计单词数量。

#### 打开 WordCount 类

如图所示找到 com.sequoiadb.lesson.spark.lesson7_sparkstreaming.wordCount 类：

![1738-470-07](https://doc.shiyanlou.com/courses/1738/1207281/ae7bce8e8ee892843cc8d43744baef53-0)

#### 程序代码

  ```java
// 配置 spark 的 master 和 appname
// master必须为local[n],n>1,表示一个线程接收数据，n-1个线程处理数据
// local[*] 意为使用可用线程处理数据
SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("streaming word count");
// 创建 sparkcontext
JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
// 创建 streamingcontext
// Durations 为流计算的时间间隔
JavaStreamingContext javaStreamingContext = new JavaStreamingContext(javaSparkContext, Durations.seconds(10));
// 创建流通过 socket 获取指定端口输入(nc -lk 6789)
JavaReceiverInputDStream<String> lines =
        javaStreamingContext.socketTextStream("sdbserver1", 6789);
// 创建匹配样式指定为空格
Pattern SPACE = Pattern.compile(" ");
// 将端口输入的每行按照 Pattern 切分为单词
JavaDStream<String> words = lines.flatMap(x -> Arrays.asList(SPACE.split(x)).iterator());
// 将单词转化成键值对（key：单词，value：1），便于合并
JavaPairDStream<String, Integer> pairs = words.mapToPair(s -> new Tuple2<>(s, 1));
// 合并相同的单词计数
JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey((i1, i2) -> i1 + i2);
// 将统计信息（10秒内）打印到控制台
wordCounts.print();
try {
    // 开启流计算
    javaStreamingContext.start();
    // 等待结束
    javaStreamingContext.awaitTermination();
} catch (InterruptedException e) {
    e.printStackTrace();
}
  ```

将上述代码粘贴至 WordCount 类 wordCount 方法 的 TODO -- lesson7_sparkstreaming:code1 注释处（32 行）：

![1738-470-08](https://doc.shiyanlou.com/courses/1738/1207281/b6c86e1dd9766893c802a8ba15bbe0eb-0)

#### 运行程序

* 点击 Terminal 在命令行输入 nc -lk 6789 打开端口准备输入单词

  ![1738-470-09](https://doc.shiyanlou.com/courses/1738/1207281/4fb77dd9aea07e4276c68335df6aef51-0)

* 右键点击 WordCountMainTest 类选择 `Run` 主函数

  ![1738-470-10](https://doc.shiyanlou.com/courses/1738/1207281/615e68129524097a4c37749bf3ee2609-0)

* 点击 IDEA 下方 `Terminal`，在 nc -lk  6789 下连续输入多个单词

  ![1738-470-11](https://doc.shiyanlou.com/courses/1738/1207281/025664e8a2638fd8227121b9618536e0-0)

* 点击 IDEA 下方 `Run` 查看运行结果

  ![1738-470-12](https://doc.shiyanlou.com/courses/1738/1207281/e3a20017a109782b6727e420faead7a4-0)

## 总结

通过本课程的学习，可以了解到 Spark Streaming 的特性和工作原理。在实验中监听 socket 端口的消息并通过 Spark Streaming 进行实时的流计算统计指定时间间隔内的单词数，对 Spark Streaming 的应用可以有初步的了解。在实际应用中 Spark Streaming 经常和 Kafka 等消息队列工具一起使用，以应对实时流计算的场景。
