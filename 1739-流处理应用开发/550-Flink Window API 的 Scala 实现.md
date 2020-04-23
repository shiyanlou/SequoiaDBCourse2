---
show: step
version: 1.0
---

## 课程介绍

本实验为 Flink Window API Scala 版本的实现，与 Java 版的讲述相同，如果不感兴趣可以跳到下一小节。

本实验将带领您了解与学习 Flink 中 Window，Time 以及 Watermark 机制。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 Flink节点、1 个引擎协调节点，1 个编目节点与 3 个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink 版本为 1.9.2。

本实验中使用了 flink-connect-sequoiadb 依赖（ Flink 连接 SequoiaDB 驱动包），该依赖来自巨杉开源社区。

- [下载地址](https://github.com/chaochaoc/flink-connector-sequoiadb/)

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 scdd-flink 项目
打开 scdd-flink 项目，在该课程中完成本试验。

![1739-510-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/2b68951cb04a44566d0a7219ede54005-0)

#### 打开 lesson5 packge
打开 com.sequoiadb.lesson.flink.lesson5_window，在该 package 中完成本课程。注意：包在 scala 源码包下。

![1739-550-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/0f42e9ee90ae9f503a5e223ad6490b3a-0)

#### 认识依赖

打开 pom.xml 文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/9b4833b8e0bc2160d90625911973ed4b-0)

本案例新增了 Flink 连接 SequoiaDB 的驱动包。
![1739-540-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/6719e761e20edcdf9205b15252856610-0)

## Window 简介

#### Window 是什么

Window 是处理无限流的核心。Window 将流分成有限大小的“桶”，可以在其上应用计算。Window 会按一定的规则将一个数据流进行切分成一个个小部分，可以在这些小部分上做批计算，以满足我们的多种需求。

#### 为什么要使用 Window

在实际使用 Flink 时，可能需要去统计一定范围的指标（如：每分钟进入 Flink 的数据量）。这种情况下使用整个流上的平均数是一个不太合理的选择，所以要将数据按照一定规则进行切分（也就是分成不同的桶）。在每个Window 上进行统计操作是更符合实际需要的。

#### Window 的使用

Window 在 Flink 程序中使用时可以分为两类。第一类keyed Window，第二类是non-keyed Window。即用在keyBy 算子之后，使用 window(...) 方法进行分桶，non-keyed在 DataStream 使用 windowAll(...) 方法进行分桶。

## Window的划分规则

Flink内部提供了三种 Window，分别是 Tumbling Window（翻滚窗口）、Sliding Window（滑动窗口）、Session Window（会话窗口）。

#### 翻滚窗口

翻滚窗口会将数据流切分成不重叠的窗口，每一个事件只能属于一个窗口。窗口的划分只受控于窗口的大小。
![1739-540-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/c848a3b17c2b516f31b917091aa3cffc-0)

#### 滑动窗口

滑动窗口和翻滚窗口类似，区别在于：滑动窗口可以有重叠的部分。受控于窗口的大小与滑动步长。

![1739-540-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/418b1ccc4f62116aa686664aa6d50aed-0)

#### 会话窗口

会话窗口不重叠，没有固定的开始和结束时间。当较长时间没有数据输入时窗口结束。

![1739-540-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/860e8fee3c9bf459fef816d959c59f59-0)



## Tumbling Count Window 的实现

本案例通过 Tumbling Count Window 统计一个交易流水中每100次交易中的总交易额。

#### 打开类

在当前包下，打开类 TumblingCountWindowMain

![1739-550-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/9ca285e74807d675beff7b525ba679dd-0)

#### 原始数据的了解

本案例将使用一个在 SequoiaDB 中已存在的交易流水表，本例中只关心以下两个字段。

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource 的使用

SequoiadbSource 可以非常容易地从 SequoiaDB 中读取一个流。

在当前类中找到 source 方法，找到 TODO code 1。

![1739-550-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/641e1dc7d1dfd1da81f33e2221da7f95-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
// Build the connection Option
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("localhost:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("VIRTUAL_BANK")
      .collectionName("TRANSACTION_FLOW")
      .build
// Add a data source to the current environment (SequoiadbSource needs to build a stream through the time field "create_time")
resultData = env.addSource(new SequoiadbSource(option, "create_time"));
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳类型。

#### 查看原始数据格式

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5102af5d6523eb20f3acd79587eae4fd-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/e23bf77cd113f104628361d07e00ac68-0)



#### map算子的使用

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem[BSONObject]，返回一个DataStream[(String, Double, Int)]。

在当前类中找到 map 方法，找到 TODO code 2。

![1739-550-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/956666ea9d26dfa0d21037597e136f76-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = transData.map(obj => (obj.get("money")
         .asInstanceOf[BSONDecimal].toBigDecimal.doubleValue(), 1))
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该 Flink 程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5102af5d6523eb20f3acd79587eae4fd-0)

执行结果如下图，可以看到一个元组，包含交易额和1。

![1739-540-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/0ade0cf2f5ee1cd09976d4b6126f110c-0)

#### Window划分

使用windowAll对流上数据进行分桶，此处使用翻滚计数窗口，窗口长度为100条，该算子返回一个AllWindowedStream[(Double, Integer), GlobalWindow]对象，表示Window中的数据类型，以及window的引用，在CountWindow中引用是一个全局的window对象。

在当前类中找到windowAll方法，找到 TODO code 3。

![1739-550-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/afe8ee26d1f61766f47407496b6aa33f-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
resultData = moneyData.countWindowAll(100)
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5102af5d6523eb20f3acd79587eae4fd-0)

执行结果如下图，可以看到每个window中的数据。

![1739-540-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/2c78a36653a1cae1ab491acee0c4daa4-0)

#### 聚合计算

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple2<Double, Integer>，分别表示总金额和总交易量。

在当前类中找到reduce方法，找到 TODO code 4。

![1739-550-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/e5ed5956860ff6a6a65da71004b76b11-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = windowData.reduce((x, y) => (x._1 + y._1, x._2 + y._2))
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5102af5d6523eb20f3acd79587eae4fd-0)

查看结果，可以得到每100次的交易额。

![1739-540-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/db766aedb59b4e37e52ac9b9a32adb78-0)

## Tumbling Time Window的实现

本案例通过Tumbling Time Window统计一个交易流水中每5秒中，每种交易的总交易额，总交易量。

#### 打开类

在当前包下，打开类TumblingTimeWindowMain

![1739-550-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b41d46fa3eacac24dbf8686d94076fb7-0)

#### 原始数据的了解

本案例中使用到了以下三个字段

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| trans_name  | String    | 交易名称   |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数。

在当前类中找到source方法，找到 TODO code 1。

![1739-550-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/00e84d44017757a4922d1cd4d4931fd5-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
val option: SequoiadbOption = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("TRANSACTION_FLOW")
    .build
// Add a data source to the current environment (SequoiadbSource needs to build a stream through the time field "create_time")
resultData = env.addSource(new SequoiadbSource(option, "create_time"))
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/509bd1736f064bc1c74da6184ac2d652-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-550-00030.png](https://doc.shiyanlou.com/courses/1739/1207281/d43200d12361e58746491728bbab069e-0)

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 2。

![1739-550-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/a0ad1df2501c14d73114ec330bf5d9b5-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = transData.map(obj => {
    (obj.get("trans_name").asInstanceOf[String], obj.get("money").
     asInstanceOf[BSONDecimal].toBigDecimal.doubleValue, 1)
})
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/509bd1736f064bc1c74da6184ac2d652-0)

执行结果如下图，可以看到转换后的元组数据。

![1739-540-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/f4d425616da28b8b44427cc623ffe276-0)

#### 分组

keyBy算子通过元组的第一个字段（交易名“trans_name”）进行分组，keyBy返回一个KeyedStream[(String, Double, Integer), String]对象，泛型中包含数据行和一个分组字段值。

在当前类中找到keyBy方法，找到 TODO code 3。

![1739-550-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/4461bc04452b5a44e66b079561d3c4bc-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
resultData = moneyData.keyBy(_._1)
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/509bd1736f064bc1c74da6184ac2d652-0)

执行结果如下图，可以看到keyBy后的数据。

![1739-540-00029.png](https://doc.shiyanlou.com/courses/1739/1207281/7152003910cf3119484a2c464e06e00c-0)

#### 在keyedStream上使用window

本案例使用时间进行划分窗口，窗口大小为5秒。

在当前类中找到window方法，找到 TODO code 4。

![1739-550-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/94f2024eda9310e3d39967bc853411b4-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = keyedData.timeWindow(Time.seconds(5))
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/509bd1736f064bc1c74da6184ac2d652-0)

执行结果如下图，可以看到每个window内的数据。

![1739-540-00030.png](https://doc.shiyanlou.com/courses/1739/1207281/4428b3fb3dec83b8b305c8dc96aad420-0)

#### 聚合求和

通过聚合算子求出每个时间窗口中的交易名称，总交易额，总交易量，以及每个window的结束时间。

在当前类中找到reduce方法，找到 TODO code 5。

![1739-550-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/48b09cf13ec023604d516ef71a3007fb-0)

将下列代码粘贴到 TODO code 5区间内。

```scala
resultData = value.apply(new WindowFunction[(String, Double, Int),
        (String, Double, Int, java.sql.Time), String, TimeWindow] {
    /**
     * Execute once in each window
     *
     * @param key    Group field value
     * @param window Current window object
     * @param input  Iterator of all data in the current window
     * @param out    Returned result collector
     */
    override def apply(key: String, window: TimeWindow, 
            input: Iterable[(String, Double, Int)],
            out: Collector[(String, Double, Int, sql.Time)]): Unit = {
        var sum: Double = 0
        var count: Int = 0
        input.foreach(item => {
           sum += item._2
           count += item._3
        })
        out.collect((key, sum, count, new java.sql.Time(window.getEnd)))
    }
})
```

#### 查看结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/509bd1736f064bc1c74da6184ac2d652-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00031.png](https://doc.shiyanlou.com/courses/1739/1207281/3639aa7bd79ed91a36ce90cc93b08d50-0)

## Sliding Count Window 的实现

本案例使用 Sliding Count Window 统计一个交易流水中每中交易类型中100次交易的总交易额。

#### 打开类

在当前包下，打开类 SlidingCountWindowMain 

![1739-550-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/4a447b3868833a9f8d0178f13d3a044d-0)

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数。

在当前类中找到source方法，找到 TODO code 1。

![1739-550-00031.png](https://doc.shiyanlou.com/courses/1739/1207281/6aa28f38f6d9ff26f732333e311bd744-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
val option: SequoiadbOption = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("TRANSACTION_FLOW")
    .build
// Add a data source to the current environment (SequoiadbSource needs to build a stream through the time field "create_time")
resultData = env.addSource(new SequoiadbSource(option, "create_time"))
```

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 2。

![1739-550-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/39d976f96abdf39614a0f835017692e8-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = value.map(obj => {
      Trans(obj.get("trans_name").asInstanceOf[String],
        obj.get("money").asInstanceOf[BSONDecimal].toBigDecimal.doubleValue(), 1)
    })
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, Tuple>对象，泛型中包含数据行和一个Tuple类型的分组字段值。

在当前类中找到keyBy方法，找到 TODO code 3。

![1739-550-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/65292609e32a101e637f772ab62fda59-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
resultData = value.keyBy("name")
```

#### 在keyedStream上使用window

案例中使用Sliding Count Window，窗口大小100，滑动步长50。

在当前类中找到countWindow方法，找到 TODO code 4。

![1739-550-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/21709dd0cc9d2a144c6b8500461b9489-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = value.countWindow(100, 50)
```

#### 聚合求和

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple3<String, Double, Integer>，分别表示交易名称，总金额和总交易量。

在当前类中找到reduce方法，找到 TODO code 5。

![1739-550-00019.png](https://doc.shiyanlou.com/courses/1739/1207281/5f75ca3ced1d70ca44c7353f597d87fa-0)

将下列代码粘贴到 TODO code 5区间内。

```scala
resultData = value.apply(new WindowFunction[Trans, (String, Double), 
                                            Tuple, GlobalWindow] {
    /**
     * Execute when the window meets the conditions
     * @param key Group field
     * @param window Global window reference
     * @param input References to all data sets in the current window
     * @param out Result collector
     */
    override def apply(key: Tuple, window: GlobalWindow, input: Iterable[Trans],
                       out: Collector[(String, Double)]): Unit = {
        var sum: Double = 0
            input.foreach(sum += _.money)
            out.collect((key.getField[String](0), sum))
    }
})
```

#### 将元组转换为BsonObject

将元组转换为BSONObject。

在当前类中找到toBson方法，找到 TODO code 6。

![1739-550-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/210d11a5f430deae6684b99abdbd5e00-0)

将下列代码粘贴到 TODO code 6区间内。

```scala
resultData = value.map(item => {
    val nObject = new BasicBSONObject
    nObject.append("trans_name", item._1)
    nObject.append("total_sum", item._2)
    nObject
})
```

#### 通过SequoiadbSink完成sink函数

在当前类中找到sink方法，找到 TODO code 7。

![1739-550-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/8a7de5d001d13856577c8b4255856b56-0)

将下列代码粘贴到 TODO code 7区间内。

```scala
// Build the connection Option
val option = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("LESSON_5_COUNT")
    .build
streamSink = value.addSink(new SequoiadbSink(option))
```

#### 查看结果

通过在当前类文件上右键 > Run 'SlidingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/f1b7752ee3790dcd28f86fd27e7a7daa-0)

通过SAC查看结果数据。结果在VIRTUAL_BANK.LESSON_5_COUNT。

通过浏览器打开 localhost:8000 进入SequoiaDB SAC管理界面。

![1739-540-00049.png](https://doc.shiyanlou.com/courses/1739/1207281/b4c3578fcb61d5b65d87b2fc084f7a05-0)

点击数据菜单选择 "SequoiaDB" 分布式存储。

![1739-540-00050.png](https://doc.shiyanlou.com/courses/1739/1207281/4e240fc768dd2c562e1f1ad7c5e68600-0)

选择集合选项卡， 在搜索栏输入集合空间名 "VIRTUAL_BANK" ，查找该集合空间下的所有集合。

![1739-540-00051.png](https://doc.shiyanlou.com/courses/1739/1207281/8418058d02df0fa3122891e5c24d712c-0)

选中集合 "VIRTUAL_BANK.LESSON_5_COUNT" 点击右侧的 "浏览数据"，可以看到当前集合中的所有数据。

![1739-550-00033.png](https://doc.shiyanlou.com/courses/1739/1207281/4ef13996083075a9ca719bb92947a56e-0)

## Flink中的Time和Watermark

#### Flink 时间概念

Flink 在流程序中支持不同的时间概念，下图为各个时间在整个流处理中的位置。

![1739-540-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/75bb63af679fe5acefdb4056f364be31-0)

- Processing time（处理时间），指正在执行相应操作时当前系统的时间。

- Event time（事件时间），事件时间是每个事件在其生产设备上发生的时间。

- Ingestion time（摄取时间），摄取时间是事件进入Flink的时间，在使用该时间时可以自动分配时间戳和自动生成watermark（水位线）。

#### Watermark的概念

Watermark（水位线）是Flink中衡量事件时间进度的机制。也是用于处理乱序事件的手段。Watermark是流的一部分，它维护一个时间戳，作为流中特殊的事件穿插在其中。它宣布事件的达到时间，这意味着当遇到Watermark时将认为晚于其内部时间戳的事件已经全部到达。

![1739-540-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/fe11a3482860ce0ca2210541df1c0f47-0)

而在分布式环境中，当多个上级算子生成不同的Watermark时，window算子将采用最小的一个。

![1739-540-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/084bd88bce6705d90628b57123e0ee6a-0)

在window中，watermark的作用可从下图看出，当watermark的值大于或等于window结束时间时将触发window操作（当然当前window中必须有数据存在）。

![1739-540-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/81094ea976c12aadfcb859953b7809c2-0)

#### 如何生成Watermark

生成watermark可以在DataStream使用assignTimestampsAndWatermarks函数创建watermark，其内部实现了多种机制，下面是两种Watermark的生成方式的接口

- AssignerWithPeriodicWatermarks可以每隔一段时间向事件流中插入一个watermark，间隔时间可通过ExecutionConfig.setAutoWatermarkInterval(...)指定，默认100ms
- AssignerWithPunctuatedWatermarks每个事件上都可以生成一个watermark，返回null时表示不生成

## Watermark和SlidingTimeWindow的使用

本案例使用 Sliding Time Window 统计一个交易流水中每5秒中，每种交易的总交易额，总交易量。本例使用 EventTime，且使用 Watermark 解决数据延迟问题。

#### 打开类

在当前包下，打开类 SlidingTimeWindowWithWatermarkerMain

![1739-550-00029.png](https://doc.shiyanlou.com/courses/1739/1207281/5f32a1afbc9a752e36eb3a3cac7d3623-0)

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数。

在当前类中找到source方法，找到 TODO code 1。

![1739-550-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/b1f5e0cb6b13a60abac31491d08a79d7-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
val option: SequoiadbOption = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("TRANSACTION_FLOW")
    .build
// Add a data source to the current environment (SequoiadbSource needs to build a stream through the time field "create_time")
resultData = env.addSource(new SequoiadbSource(option, "create_time"))
```

#### 添加Watermark

向流中添加watermark。

在当前类中找到watermark方法，找到 TODO code 2。

![1739-550-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/285a9ac61572378e30b4f723e8e1ca2d-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = value.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks[BSONObject] {
    // Maximum out-of-order time
    private val maxOutOfOrderness: Long = 5000
    private var maxTimestamp: Long = 0

    /**
     * Return a watermark
     *
     * @return
     */
     override def getCurrentWatermark: Watermark = {
     	new Watermark(maxTimestamp - maxOutOfOrderness)
     }
    
    /**
     * Extract the timestamp of the current data
     *
     * @param t Current data
     * @param l Timestamp of the previous data
     * @return Timestamp of the current data
     */
    override def extractTimestamp(t: BSONObject, l: Long): Long = {
        val currentTimestamp: Long = t.get("create_time")
            .asInstanceOf[BSONTimestamp].getTime
        maxTimestamp = if (maxTimestamp > currentTimestamp) maxTimestamp 
            else currentTimestamp
        currentTimestamp
	}
})
```

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 3。

![1739-550-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/99724fec3692d2324b1abb5621d4065c-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
 resultData = value.map(obj => {
     (obj.get("trans_name").asInstanceOf[String], obj.get("money")
      .asInstanceOf[BSONDecimal].toBigDecimal.doubleValue, 1)
 })
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个 KeyedStream[(String, Double, Int), String]对象，泛型中包含数据行和一个Tuple类型的分组字段值。

在当前类中找到keyBy方法，找到 TODO code 4。

![1739-550-00027.png](https://doc.shiyanlou.com/courses/1739/1207281/0f635dbbdc0f84be9cdd272198a5539c-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = value.keyBy(_._1)
```

#### 在keyedStream上使用window

在当前类中找到window方法，找到 TODO code 5。

![1739-550-00025.png](https://doc.shiyanlou.com/courses/1739/1207281/8c9f60ad31838ceef630419b360a72f8-0)

将下列代码粘贴到 TODO code 5区间内。

```scala
resultData = value.window(SlidingEventTimeWindows.of(Time.seconds(5), Time.seconds(2)))
```

#### 聚合求和

在当前类中找到reduce方法，找到 TODO code 6。

![1739-550-00026.png](https://doc.shiyanlou.com/courses/1739/1207281/cb64880218b6d7122c88b9a82500aed4-0)

将下列代码粘贴到 TODO code 6区间内。

```scala
resultData = value.process(new ProcessWindowFunction[(String, Double, Int), 
                                                     BSONObject, String, TimeWindow] {
    /**
     * window Aggregation method, call once per window
     * @param key Group field value
     * @param context Context objects, the essence of this operator
     * @param elements Event reference in current window
     * @param out Event collector
     */
    override def process(key: String, context: Context, 
                         elements: Iterable[(String, Double, Int)],
                         out: Collector[BSONObject]): Unit = {
        var sum: Double = 0
        var count: Int = 0
        elements.foreach(i => {
            sum += i._2
            count += i._3
        })
        // Construct a BsonObject object
        val nObject = new BasicBSONObject
        nObject.append("trans_name", key)
        nObject.append("total_sum", sum)
        nObject.append("count", count)
        out.collect(nObject)
    }
})
```

#### 通过SequoiadbSink完成sink函数

在当前类中找到sink方法，找到 TODO code 7。

![1739-550-00027.png](https://doc.shiyanlou.com/courses/1739/1207281/8401c6aa441b14be2d1b933046836f7b-0)

将下列代码粘贴到 TODO code 7区间内。

```scala
val option = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("LESSON_5_TIME")
    .build
value.addSink(new SequoiadbSink(option))
```

#### 查看结果

通过在当前类文件上右键 > Run 'SlidingTimeWindowWithWatermarkerMain.main()' 运行该Flink程序。

![1739-550-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/48db0bd8f633896ef17f2b0ce4ee2796-0)

通过SAC查看结果数据。结果在VIRTUAL_BANK.LESSON_5_TIME集合下。

通过浏览器打开 localhost:8000 进入 SequoiaDB SAC 管理界面。

![1739-540-00049.png](https://doc.shiyanlou.com/courses/1739/1207281/b4c3578fcb61d5b65d87b2fc084f7a05-0)

点击数据菜单选择 "SequoiaDB" 分布式存储。

![1739-540-00050.png](https://doc.shiyanlou.com/courses/1739/1207281/4e240fc768dd2c562e1f1ad7c5e68600-0)

选择集合选项卡， 在搜索栏输入集合空间名 "VIRTUAL_BANK" ，查找该集合空间下的所有集合。

![1739-550-00034.png](https://doc.shiyanlou.com/courses/1739/1207281/28ae9b5073dd0ecbb2e2e8f99506920c-0)

选中集合 " VIRTUAL_BANK.LESSON_5_TIME" 点击右侧的 "浏览数据"，可以看到当前集合中的所有数据。

![1739-550-00035.png](https://doc.shiyanlou.com/courses/1739/1207281/e5155197b75b40bba6c752c654eb45f7-0)



## 总结

本小节为 Flink 学习提升篇，讲述了 Flink 的时间概念与 Window 的概念及如何用 Scala语言实现，Watermark 机制的了解与使用。

**知识点**

- Window 的概念及 Flink 中提供的 Window 是按照什么规则划分的
- Time 的概念
- 多种 Window 的使用
- Watermark 的使用