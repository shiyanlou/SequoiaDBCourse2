---
show: step
version: 1.0 
---

## 课程介绍

本课程将介绍 Spark 另一个热门组件——Spark Streaming。并通过 word count 的例子简单展示了 Spark Streaming 的用法。

#### 实验环境

当前实验的系统和软件环境如下：

* Ubuntu 16.04.6 LTS
* SequoiaDB version: 3.4
* SequoiaSQL-MySQL version: 3.4
* JDK version "1.8.0_242"
* IntelliJ IDEA Community Version: 2019.3.4
* Spark version: 2.4.3

#### 知识点

![1738-470-01](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-01.png)

Spark  Streaming 是 core Spark API 的一个拓展组件，可以实现实时数据流的可拓展、高吞吐和可容错流处理。上游数据可以来自于 Kafka、Flume、S3 等多种数据源，同时可以支持类似 map、reduce、join 等复杂的算法和高级语言处理数据流，最终量处理过后的数据推送至文件系统、数据库或 Dashboards。

![1738-470-02](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-02.png)

Spark Streaming 在工作过程中，实时地接收输入的数据流，并将数据分成批次，然后由Spark引擎进行处理，以生成批次的最终结果流。Spark Streaming 提供被称为 离散流 或 DStream 的高级抽象，在交给 Spark 集群处理时，DStream 表示为 RDD 序列。

## Maven 工程介绍

* 打开 IDEA

  ![1738-470-03](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-03.png)

* 打开 SCDD-Spark 工程

  ![1738-470-04](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-04.png)

* 打开当前课程程序所在 package

  ![1738-470-05](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-05.png)

* 实验中使用到的 Maven 依赖说明

  ![1738-470-06](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-06.png)

  

## 程序代码

  程序将监听本地端口 6789，并通过 Spark Streaming 获取每间隔 10 秒内 6789 端口输入的单词并统计单词数量。程序代码内容如下：

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
          // 统计端口输入单词数
          JavaPairDStream<String, Integer> counts =
                  lines.flatMap(x -> Arrays.asList(x.split(" ")).iterator())
                          .mapToPair(x -> new Tuple2<String, Integer>(x, 1))
                          .reduceByKey((x, y) -> x + y);
  
          // 将统计信息（10秒内）打印到控制台
          counts.print();
          try {
              // 开启流计算
              javaStreamingContext.start();
              // 等待结束
              javaStreamingContext.awaitTermination();
          } catch (InterruptedException e) {
              e.printStackTrace();
          }
  ```

  将上述代码粘贴至 `TODO -- com.sequoiadb.lesson.spark.lesson7_sparkstreaming:code1` 标签处（28 行），最终样式如下图所示：

  ![1738-470-07](C:\Users\14620\Desktop\Spark开发课程\图片\lesson7\1738-470-07.png)

## 运行程序

* 右键点击 WordCountMainTest 类选择 `Edit/Create` 主函数

  

* 编辑主函数入参为 `wordcount`

  

* 右键点击 WordCountMainTest 类选择 `Run` 主函数

  

* 点击 IDEA 下方 `Terminal`，输入 `nc -lk 6789`，连续输入多个单词

  

* 点击 IDEA 下方 `Run` 查看运行结果

## 总结

通过本课程的学习，可以了解到 Spark Streaming 的特性和工作原理。在实验中监听 socket 端口的消息并通过 Spark Streaming 进行实时的流计算统计指定时间间隔内的单词数，对 Spark Streaming 的应用可以有初步的了解。在实际应用中 Spark Streaming 经常和 Kafka 等消息队列工具一起使用，以应对实时流计算的场景。