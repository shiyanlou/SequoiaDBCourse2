---
show: step
version: 1.0
---

## 课程介绍

本实验将带领学习Window，Flink的Time以及watermark机制。

## 打开项目

#### 打开idea

打开idea代码开发工具

![桌面截图，指定需要选择的idea图标](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585561767462.png)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成后续试验。

![项目选择截图](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585561767462.png)

#### 打开lesson1 packge
打开```com.sequoiadb.scdd.lesson1_intro```packge，在该package中完成后续课程。

![packge选择截图](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585561767462.png)

## Window简介
#### window是什么

Windows是处理无限流的核心。Windows将流分成有限大小的“桶”，我们可以在其上应用计算。

通俗的讲就是按一定的规则将一个数据流进行切分成一个个小部分，可以在这些小部分上应用计算，以满足我们的多种需求。

#### 为什么要使用window

在实际使用Flink时，我们可能需要去统计不同时间段，或者一个固定数据范围的指标（如：每分钟进入Flink的数据量），这种情况下使用全局的平均数是一个不太合理的选择，所以要将数据按照一定规则进行切分（也就是官方的说辞：分成不同的桶）。在每个window上进行统计操作是更符合实际需要的。

#### window的使用

window式Flink程序的有两大类。第一类指的是Keyed流，第二类指的是non-keyed流。唯一的区别是keyed流是由keyBy算子生成的，使用window(...)方法进行分桶，non-keyed使用windowAll(...)方法进行分桶。

## Window的划分规则

window按照可以划分为Tumbling Windows（翻滚窗口）、Sliding Windows（滑动窗口）、Session Windows（会话窗口）、Global Windows（全局窗口，此类窗口需要自定义划分规则，不属于本次课程的讲述范围）

#### 翻滚窗口

翻滚窗口会将数据流切分成不重叠的窗口，每一个事件只能属于一个窗口。只受控于窗口的大小
![1739-540-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/c848a3b17c2b516f31b917091aa3cffc-0)

#### 滑动窗口

滑动窗口和翻滚窗口类似，区别在于：滑动窗口可以有重叠的部分。受控于窗口的大小于滑动步长。

![1739-540-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/418b1ccc4f62116aa686664aa6d50aed-0)

#### 会话窗口

会话窗口不重叠，没有固定的开始和结束时间。当较长时间没有数据输入时窗口结束。

![1739-540-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/860e8fee3c9bf459fef816d959c59f59-0)



## Tumbling Count Window的实现

请使用A$TumblingCountWindowMain完成当前演示，统计一个交易流水中每100条交易的总交易额。

#### 原始数据的了解

本例中我们只关心以下两个字段

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

本功能由巨杉开源社区提供，可以非常容易地从Sequoiadb中读取一个流。

```scala
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
env.addSource(new SequoiadbSource(option, "timestamp"))
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳。

> 知识拓展
>
> SequoiadbSource读取数据采用小批的形式，由Flink完成全局排序。
>
> 批次的大小采用时间间隔控制，构建该对象时可传入一个int表示批次大小，默认为10s.

#### map算子

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem[BSONObject]，返回一个DataStream[(Double, Integer)].

```scala
transData.map(obj => (obj.get("money").asInstanceOf[BSONDecimal].toBigDecimal.doubleValue(), 1))

```

#### Window划分

使用windowAll对流上数据进行分桶，此处使用翻滚计数窗口，窗口长度为100条，该算子返回一个AllWindowedStream[(Double, Integer)， GlobalWindow]对象，表示Window中的数据类型，以及window的引用，在CountWindow中引用是一个全局的window对象。

```scala
moneyData.countWindowAll(100)
```

#### 聚合结果

使用reduce对数据进行聚合求和，此处将的聚合结果为元组(Double, Integer)，分别表示总金额和总交易量

```scala
windowData.reduce((x, y) => (x._1 + y._1, x._2 + y._2))
```

## Tumbling Time Window的实现

请使用B$TumblingTimeWindowMain完成当前演示，统计一个交易流水中每种交易类型中每5秒的总交易额。
#### 原始数据的了解

本例中使用到了以下三个字段

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| trans_name  | String    | 交易名称   |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数

```java
// 构建连接Option
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 类型转换

通过map算子获取到交易名，交易金额

```scala
transData.map(obj => {
      (obj.get("trans_name").asInstanceOf[String], obj.get("money").asInstanceOf[BSONDecimal].toBigDecimal.doubleValue, 1)
    })
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream[(String, Double, Integer), String]对象，泛型中包含数据行和一个分组字段值

```scala
moneyData.keyBy(_._1)
```

#### 在keyedStream上使用window

```scala
keyedData.timeWindow(Time.seconds(5))
```

#### 聚合求和

```scala
value.apply(new WindowFunction[(String, Double, Int), (String, Double, Int, java.sql.Time), String, TimeWindow] {
    /**
     * 在每个window中执行一次
     * @param key 分组字段值
     * @param window 当前window对象
     * @param input 当前window中所有数据的迭代器
     * @param out 返回结果收集器
     */
    override def apply(key: String, window: TimeWindow, input: Iterable[(String, Double, Int)], out: Collector[(String, Double, Int, sql.Time)]): Unit = {
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

## Sliding Count Window的实现

请使用C$SlidingCountWindowMain完成当前演示，统计一个交易流水中每100次交易中的总交易额。

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数

```java
// 构建连接Option
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 类型转换

通过map算子获取到交易名，交易金额，此处使用了scala中的样例类Trans

```java
value.map(obj => {
      Trans(obj.get("trans_name").asInstanceOf[String], obj.get("money").asInstanceOf[Double], 1)
})
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream[Trans, Tuple]对象，泛型中包含数据行和一个Tuple类型的分组字段值

```scala
value.keyBy("name")
```

#### 在keyedStream上使用window

```java
// 返回一个Tuple 为分组字段，由于使用了下标进行分组，无法获取到具体的数据类型，故此处使用Tuple抽象表示
value.countWindow(100)
```

#### 聚合求和

```scala
value.apply(new WindowFunction[Trans, (String, Double), Tuple, GlobalWindow] {
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

```scala
value.map(item => {
    val nObject = new BasicBSONObject
    nObject.append("trans_name", item._1)
    nObject.append("money", item._2)
    nObject
})
```

#### 通过SequoiadbSink完成sink函数

```scala
val option = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
value.addSink(new SequoiadbSink(option))
```

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

请使用D$SlidingTimeWindowWithWatermarkerMain完成当前演示，使用EventTime完成需求。

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数

```scala
// 构建连接Option
val option: SequoiadbOption = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 添加Watermark

向流中插入watermark

```scala
value.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks[BSONObject] {
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
        val currentTimestamp: Long = t.get("timestamp").asInstanceOf[BSONTimestamp].getTime
        maxTimestamp = if (maxTimestamp > currentTimestamp) maxTimestamp else currentTimestamp
        currentTimestamp
      }
})
```



#### 类型转换

通过map算子获取到交易名，交易金额

```scala
value.map(obj => {
      (obj.get("trans_name").asInstanceOf[String], obj.get("money").asInstanceOf[BSONDecimal].toBigDecimal.doubleValue, 1)
    })
```

#### 分组

keyBy算子通过元组中第一个字段进行分组，keyBy返回一个KeyedStream[(String, Double, Int), String]对象，泛型中包含数据行和一个String类型的分组字段值

```scala
value.keyBy(_._1)
```

#### 在keyedStream上使用window

```scala
value.window(SlidingEventTimeWindows.of(Time.seconds(5), Time.seconds(2)));
```

#### 聚合求和

```scala
value.process(new ProcessWindowFunction[(String, Double, Int), Trans, String, TimeWindow] {
    /**
     * window 聚合方法，每个window调用一次
     * @param key 分组字段值
     * @param context 上下文对象，本算子的精华
     * @param elements 当前window中的事件引用
     * @param out 事件收集器
     */
    override def process(key: String, context: Context, elements: Iterable[(String, Double, Int)],
                         out: Collector[Trans]): Unit = {
    var sum: Double = 0
    var count: Int = 0
    elements.foreach(i => {
        sum += i._2
        count += i._3
    })
    out.collect(Trans(key, sum, count))
    }
})
```

#### 将元组转换为BsonObject

```scala
value.map(item => {
    val nObject = new BasicBSONObject
    nObject.append("trans_name", item.name)
    nObject.append("money", item.money)
    nObject.append("count", item.name)
    nObject.append("trans_name", item.name)
    nObject
})
```

#### 通过SequoiadbSink完成sink函数

```scala
val option = SequoiadbOption.bulider
      .host("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpaceName("test")
      .collectionName("test7")
      .build
value.addSink(new SequoiadbSink(option))
```

