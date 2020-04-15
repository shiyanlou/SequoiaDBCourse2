---
show: step
version: 1.0
---

## 课程介绍

本实验为Flink Window API Scala版本的实现，与Java版的讲述相同，如果不感兴趣可以跳到下一小节。

本实验将带领您了解与学习Flink中Window，Time以及watermark机制。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 Flink节点、1个引擎协调节点，1个编目节点与3个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为1.9.2。

本实验中使用了flink-connect-sequoiadb依赖（flink连接sequoiadb驱动包），该依赖来自巨杉开源社区。

- [下载地址](https://github.com/chaochaoc/flink-connect-sequoiadb)

#### 打开IDEA

打开IDEA代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson5 packge
打开com.sequoiadb.flink.scdd.lesson5_window，在该package中完成本课程。注意：包在scala源码包下。

![1739-550-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/7fd36371db75fbdaacc0754081477385-0)

#### 认识依赖

打开pom.xml文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/c8177f5490e581cd3a59c689b65f9143-0)

本案例新增了flink连接sequoiadb的驱动包。
![1739-540-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/6719e761e20edcdf9205b15252856610-0)

## Window简介

#### window是什么

Windows是处理无限流的核心。Windows将流分成有限大小的“桶”，可以在其上应用计算。window会按一定的规则将一个数据流进行切分成一个个小部分，可以在这些小部分上做批计算，以满足我们的多种需求。

#### 为什么要使用window

在实际使用Flink时，可能需要去统计一定范围的指标（如：每分钟进入Flink的数据量）。这种情况下使用整个流上的平均数是一个不太合理的选择，所以要将数据按照一定规则进行切分（也就是分成不同的桶）。在每个window上进行统计操作是更符合实际需要的。

#### window的使用

window在Flink程序中使用时可以分为两类。第一类是keyed上，第二类是non-keyed Stream。即用在keyBy算子之后，使用window(...)方法进行分桶，non-keyed在DataStream使用windowAll(...)方法进行分桶。

## Window的划分规则

flink内部提供了三种window，分布是Tumbling Windows（翻滚窗口）、Sliding Windows（滑动窗口）、Session Windows（会话窗口）。

#### 翻滚窗口

翻滚窗口会将数据流切分成不重叠的窗口，每一个事件只能属于一个窗口。窗口的划分只受控于窗口的大小。
![1739-540-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/c848a3b17c2b516f31b917091aa3cffc-0)

#### 滑动窗口

滑动窗口和翻滚窗口类似，区别在于：滑动窗口可以有重叠的部分。受控于窗口的大小与滑动步长。

![1739-540-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/418b1ccc4f62116aa686664aa6d50aed-0)

#### 会话窗口

会话窗口不重叠，没有固定的开始和结束时间。当较长时间没有数据输入时窗口结束。

![1739-540-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/860e8fee3c9bf459fef816d959c59f59-0)



## Tumbling Count Window的实现

本案例通过Tumbling Count Window统计一个交易流水中每100次交易中的总交易额。

#### 打开类

在当前包下，打开类TumblingCountWindowMain

![1739-550-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/5c243a2aa1d4d9d90091cbdba0eecd92-0)

#### 原始数据的了解

本案例将使用一个在SequoiaDB中已存在的交易流水表，本例中只关心以下两个字段。

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

SequoiadbSource可以非常容易地从Sequoiadb中读取一个流。

在当前类中找到source方法，找到 TODO code 1。

![1739-550-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/b1f5e0cb6b13a60abac31491d08a79d7-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
// 构建连接Option
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("localhost:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("VIRTUAL_BANK")
      .collectionName("TRANSACTION_FLOW")
      .build
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
resultData = env.addSource(new SequoiadbSource(option, "create_time"));
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳类型。

#### 查看原始数据格式

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/991bd84db57188a4e374b4609f955ef9-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/0e2a071608abcb5c16effccba29a284f-0)



#### map算子的使用

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem[BSONObject]，返回一个DataStream[(String, Double, Int)]。

在当前类中找到map方法，找到 TODO code 2。

![1739-550-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/7c307d768c068194262680ef3f5b6bca-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = transData.map(obj => (obj.get("money")
         .asInstanceOf[BSONDecimal].toBigDecimal.doubleValue(), 1))
```

#### 查看原始数据格式

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/991bd84db57188a4e374b4609f955ef9-0)

执行结果如下图，可以看到一个元组，包含交易额和1。

![1739-540-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/1e19ed6e0e99eccdb350b21138c05e7b-0)

#### Window划分

使用windowAll对流上数据进行分桶，此处使用翻滚计数窗口，窗口长度为100条，该算子返回一个AllWindowedStream[(Double, Integer), GlobalWindow]对象，表示Window中的数据类型，以及window的引用，在CountWindow中引用是一个全局的window对象。

在当前类中找到windowAll方法，找到 TODO code 3。

![1739-550-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/edc9c04e5fd0b831ed865469172e26fa-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
resultData = moneyData.countWindowAll(100)
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/991bd84db57188a4e374b4609f955ef9-0)

执行结果如下图，可以看到每个window中的数据。

![1739-540-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/2c78a36653a1cae1ab491acee0c4daa4-0)

#### 聚合结果

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple2<Double, Integer>，分别表示总金额和总交易量。

在当前类中找到reduce方法，找到 TODO code 4。

![1739-550-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/a1d29a9fd4399cc630a052884fc2c5c4-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = windowData.reduce((x, y) => (x._1 + y._1, x._2 + y._2))
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/991bd84db57188a4e374b4609f955ef9-0)

查看结果，可以得到每100次的交易额。

![1739-540-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/8f50992a6a7522e48c4156c30c52b931-0)

## Tumbling Time Window的实现

本案例通过Tumbling Time Window统计一个交易流水中每5秒中，每种交易的总交易额，总交易量。

#### 打开类

在当前包下，打开类TumblingTimeWindowMain

![1739-550-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b5316a2c19616f82447e9bb10ba941f7-0)

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
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
resultData = env.addSource(new SequoiadbSource(option, "crate_time"))
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/3af41173188cb264483df34b4c36455a-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/0e2a071608abcb5c16effccba29a284f-0)

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 2。

![1739-550-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/3d7d1761d9281632700108a5ea4da211-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = transData.map(obj => {
    (obj.get("trans_name").asInstanceOf[String], obj.get("money").
     asInstanceOf[BSONDecimal].toBigDecimal.doubleValue, 1)
})
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/3af41173188cb264483df34b4c36455a-0)

执行结果如下图，可以看到转换后的元组数据。

![1739-540-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/588a0f94eea2b2a8d890a37b93ca1381-0)

#### 分组

keyBy算子通过元组的第一个字段（交易名“trans_name”）进行分组，keyBy返回一个KeyedStream[(String, Double, Integer), String]对象，泛型中包含数据行和一个分组字段值。

在当前类中找到keyBy方法，找到 TODO code 3。

![1739-550-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/343fdef34aaea4e8b6dca13e90b8e91e-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
resultData = moneyData.keyBy(_._1)
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/3af41173188cb264483df34b4c36455a-0)

执行结果如下图，可以看到keyBy后的数据。

![1739-540-00029.png](https://doc.shiyanlou.com/courses/1739/1207281/331cfda4cca86782818d94365cd58ae3-0)

#### 在keyedStream上使用window

本案例使用时间进行划分窗口，窗口大小为5秒。

在当前类中找到window方法，找到 TODO code 4。

![1739-550-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/f54ea085d6e2ee35decde25ce57c2f70-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
resultData = keyedData.timeWindow(Time.seconds(5))
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/3af41173188cb264483df34b4c36455a-0)

执行结果如下图，可以看到每个window内的数据。

![1739-540-00030.png](https://doc.shiyanlou.com/courses/1739/1207281/8e1c76bc372d1b9f2960a83acb3725f2-0)

#### 聚合求和

通过聚合算子求出每个时间窗口中的交易名称，总交易额，总交易量，以及每个window的结束时间。

在当前类中找到reduce方法，找到 TODO code 5。

![1739-550-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/ef89430124f9ebe01b5195980dff4906-0)

将下列代码粘贴到 TODO code 5区间内。

```scala
resultData = value.apply(new WindowFunction[(String, Double, Int),
        (String, Double, Int, java.sql.Time), String, TimeWindow] {
    /**
     * 在每个window中执行一次
     *
     * @param key    分组字段值
     * @param window 当前window对象
     * @param input  当前window中所有数据的迭代器
     * @param out    返回结果收集器
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

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-550-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/3af41173188cb264483df34b4c36455a-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00031.png](https://doc.shiyanlou.com/courses/1739/1207281/e97c49caac3f754cf21f9784f18e094c-0)

## Sliding Count Window的实现

本案例使用Sliding Count Window统计一个交易流水中每中交易类型中100次交易的总交易额。

#### 打开类

在当前包下，打开类```SlidingCountWindowMain```

![1739-550-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/123ce3493dcadaaaae15c7c537c51256-0)

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
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
resultData = env.addSource(new SequoiadbSource(option, "crate_time"))
```

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 2。

![1739-550-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/39d976f96abdf39614a0f835017692e8-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = value.map(obj => {
    Trans(obj.get("trans_name").asInstanceOf[String], 
          obj.get("money").asInstanceOf[Double], 1)
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
resultData = keyedData.countWindow(100, 50);
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
     * 在窗口满足条件时执行
     * @param key 分组字段
     * @param window 全局的window引用
     * @param input 当前window中所有数据集的引用
     * @param out 结果收集器
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
// 构建连接Option
val option = SequoiadbOption.bulider
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("LESSON_5_COUNT")
    .build
streamSink = value.addSink(new SequoiadbSink(option))
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'SlidingCountWindowMain.main()' 运行该Flink程序。

![1739-550-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/e9fd5b252c73c7ee09f44a787ee63dd1-0)

通过SAC查看结果数据。结果在VIRTUAL_BANK.LESSON_5_COUNT。

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

本案例使用Sliding Time Window统计一个交易流水中每5秒中，每种交易的总交易额，总交易量。本例使用EventTime，且使用Watermark解决数据延迟问题。

#### 打开类

在当前包下，打开类SlidingTimeWindowWithWatermarkerMain

![1739-540-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/85d9ad373ea2ab9c45ff9f15c838dd39-0)

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
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
resultData = env.addSource(new SequoiadbSource(option, "crate_time"))
```

#### 添加Watermark

向流中添加watermark。

在当前类中找到watermark方法，找到 TODO code 2。

![1739-550-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/285a9ac61572378e30b4f723e8e1ca2d-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
resultData = value.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks[BSONObject] {
    // 最大的乱序时间
    private val maxOutOfOrderness: Long = 5000
    private var maxTimestamp: Long = 0

    /**
     * 返回一个watermark
     *
     * @return
     */
     override def getCurrentWatermark: Watermark = {
     	new Watermark(maxTimestamp - maxOutOfOrderness)
     }
    
    /**
     * 抽取当前数据的时间戳
     *
     * @param t 当前的数据
     * @param l 上一条数据的时间戳
     * @return 当前数据的时间戳
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
     * window 聚合方法，每个window调用一次
     * @param key 分组字段值
     * @param context 上下文对象，本算子的精华
     * @param elements 当前window中的事件引用
     * @param out 事件收集器
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
        // 构建BsonObject对象
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

#### 查看数据的结果

通过在当前类文件上右键 > Run 'SlidingTimeWindowWithWatermarkerMain.main()' 运行该Flink程序。

![1739-550-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/7cdd28c024571a2d1a3cd4a4c02ccc92-0)

通过SAC查看结果数据。结果在VIRTUAL_BANK.LESSON_5_TIME集合下。

