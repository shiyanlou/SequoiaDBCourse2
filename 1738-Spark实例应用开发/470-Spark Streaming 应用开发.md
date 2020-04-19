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

如图所示找到 pom.xml 文件：

![1738-410-pom](https://doc.shiyanlou.com/courses/1738/1207281/2096e77f8ff05283b1b51e9f5182b861-0)

在 pom.xml 中可以找到当前实验需要用到的 Maven 依赖：

![1738-470-06](https://doc.shiyanlou.com/courses/1738/1207281/35d7b9dc1a5dcdc2bbcddf9799a6a143-0)



## Spark Streaming 统计端口输入的单词数

程序将监听本地端口 6789，并通过 Spark Streaming 获取每间隔 10 秒内 6789 端口输入的单词并统计单词数量。

#### 打开 WordCount 类

如图所示找到 com.sequoiadb.lesson.spark.lesson7_sparkstreaming.wordCount 类：

![1738-470-07](https://doc.shiyanlou.com/courses/1738/1207281/ae7bce8e8ee892843cc8d43744baef53-0)

#### 程序代码

  ```java
// Configure master and appname of spark
// Master must be local[n], n> 1(1 thread receives data and n-1 threads process data)
// local [*] means using available threads to process data
SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("streaming word count");
// Create sparkcontext
JavaSparkContext javaSparkContext = new JavaSparkContext(sparkConf);
// Create streamingcontext
// "Durations" means the time intervals calculated for the stream
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
