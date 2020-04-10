## Window的概念与使用window的原因

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
![1585720341163](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1585720341163.png)

#### 滑动窗口

滑动窗口和翻滚窗口类似，区别在于：滑动窗口可以有重叠的部分。受控于窗口的大小于滑动步长。

![1585720351990](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1585720351990.png)

#### 会话窗口

会话窗口不重叠，没有固定的开始和结束时间。当较长时间没有数据输入时窗口结束。

![1585720376281](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1585720376281.png)



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

```java
 // 构建连接Option
 SequoiadbOption option = SequoiadbOption.bulider()
 .host("192.168.0.111:11810")
 .username("sdbadmin")
 .password("sdbadmin")
 .collectionSpaceName("test")
 .collectionName("test7")
 .build();
 // 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"timestamp"构建流）
 env.addSource(new SequoiadbSource(option, "timestamp"));
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳。

> 知识拓展
>
> SequoiadbSource读取数据采用小批的形式，由Flink完成全局排序。
>
> 批次的大小采用时间间隔控制，构建该对象时可传入一个int表示批次大小，默认为10s.

#### Map算子的使用

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem<BSONObject>，返回一个DataStream<Tuple2<Double, Integer>>.

```java
return transData.map(new MapFunction<BSONObject, Tuple2<Double, Integer>>() {
    @Override
    public Tuple2<Double, Integer> map(BSONObject object) throws Exception {
    	return Tuple2.of(((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
    }
});
```

#### Window划分

使用windowAll对流上数据进行分桶，此处使用翻滚计数窗口，窗口长度为100条，该算子返回一个AllWindowedStream<Tuple2<Double, Integer>, GlobalWindow>对象，表示Window中的数据类型，以及window的引用，在CountWindow中引用是一个全局的window对象。

```java
return moneyData.countWindowAll(100);
```

#### 聚合结果

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple2<Double, Integer>，分别表示总金额和总交易量

```java
return dataStream.reduce(new ReduceFunction<Tuple2<Double, Integer>>() {
    @Override
    public Tuple2<Double, Integer> reduce(Tuple2<Double, Integer> t1, Tuple2<Double, Integer> t2) throws Exception {
    return Tuple2.of(t1.f0 + t2.f0, t1.f1 + t2.f1);
    }
});
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
SequoiadbOption option = SequoiadbOption.bulider()
.host("192.168.0.111:11810")
.username("sdbadmin")
.password("sdbadmin")
.collectionSpaceName("test")
.collectionName("test7")
.build();
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
return env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 类型转换

通过map算子获取到交易名，交易金额

```java
 return dataStream.map(new MapFunction<BSONObject, Tuple3<String, Double, Integer>>() {
	@Override
	public Tuple3<String, Double, Integer> map(BSONObject object) throws Exception {
    	return Tuple3.of(object.get("trans_name").toString(), ((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
    }
});
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, String>对象，泛型中包含数据行和一个分组字段值

```java
return dataStream.keyBy(new KeySelector<Tuple3<String, Double, Integer>, String>() {
    /**
     * 分组函数，使用KeySelector 可以显示获取到分组字段的类型
     * @param t 分组前的数据集
     * @return 分组字段值
     * @throws Exception
     */
    @Override
    public String getKey(Tuple3<String, Double, Integer> t) throws Exception {
        return t.f0;
    }
});
```

#### 在keyedStream上使用window

```java
return keyedData.timeWindow(Time.seconds(5));
```

#### 聚合求和

```java
return windowData.apply(new WindowFunction<Tuple3<String, Double, Integer>,
                Tuple4<String, Double, Integer,java.sql.Time>, String, TimeWindow>() {
	/**
     * 在每个window中执行一次 
     * @param key 分组字段值
     * @param timeWindow 当前window对象
     * @param iterable 当前window中所有数据的迭代器
     * @param collector 返回结果收集器
     * @throws Exception
     */
      @Override
      public void apply(String key, TimeWindow timeWindow, Iterable<Tuple3<String, Double, Integer>> iterable, Collector<Tuple4<String, Double, Integer,java.sql.Time>> collector) throws Exception {
          double sum = 0;
          int count = 0;
          Iterator<Tuple3<String, Double, Integer>> iterator = iterable.iterator();
          while (iterator.hasNext()) {
              Tuple3<String, Double, Integer> next = iterator.next();
              sum += next.f1;
              count += next.f2;
          }
          collector.collect(Tuple4.of(key, sum, count, new java.sql.Time(timeWindow.getEnd())));
      }
});
```

## Sliding Count Window的实现

请使用C$SlidingCountWindowMain完成当前演示，统计一个交易流水中每100次交易中的总交易额。

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数

```java
// 构建连接Option
SequoiadbOption option = SequoiadbOption.bulider()
  .host("192.168.0.111:11810")
  .username("sdbadmin")
  .password("sdbadmin")
  .collectionSpaceName("test")
  .collectionName("test7")
  .build();
return env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 类型转换

通过map算子获取到交易名，交易金额

```java
return transData.map(new MapFunction<BSONObject, Tuple3<String, Double, Integer>>() {
	@Override
    public Tuple3<String, Double, Integer> map(BSONObject object) throws Exception {
      return Tuple3.of(object.get("trans_name").toString(),((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
      }
});
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, Tuple>对象，泛型中包含数据行和一个Tuple类型的分组字段值

```java
return moneyData.keyBy(0);
```

#### 在keyedStream上使用window

```java
return keyedData.countWindow(100, 50);
```

#### 聚合求和

```java
return countWindow.apply(new WindowFunction<Tuple3<String, Double, Integer>, Tuple2<String, Double>, Tuple, GlobalWindow>() {
     /**
      * 在窗口满足条件时执行，类似于flatMap算子
      * @param tuple 分组字段值，由于使用了下标进行分组，无法获取到具体的数据类型，故此处使用Tuple抽象表示
      * @param globalWindow 全局的window引用
      * @param iterable 当前window中所有数据集的引用
      * @param collector 结果收集器
      * @throws Exception
      */
    @Override
    public void apply(Tuple tuple, GlobalWindow globalWindow, Iterable<Tuple3<String, Double, Integer>> iterable,
                      Collector<Tuple2<String, Double>> collector) throws Exception {
        double sum = 0;
        Iterator<Tuple3<String, Double, Integer>> iterator = iterable.iterator();
        while (iterator.hasNext()) {
            sum += iterator.next().f1;
        }
        collector.collect(Tuple2.of(tuple.getField(0), sum));
    }
});
```

#### 将元组转换为BsonObject

```java
return dataStream.map(new MapFunction<Tuple2<String, Double>, BSONObject>() {
    @Override
    public BSONObject map(Tuple2<String, Double> value) throws Exception {
        BasicBSONObject obj = new BasicBSONObject();
        obj.append("trans_name", value.f0);
        obj.append("money", value.f1);
        return obj;
    }
});
```

#### 通过SequoiadbSink完成sink函数

```java
SequoiadbOption option = SequoiadbOption.bulider()
     .host("192.168.0.111:11810")
     .username("sdbadmin")
     .password("sdbadmin")
     .collectionSpaceName("test")
     .collectionName("test7")
     .build();
return dataStream.addSink(new SequoiadbSink(option));
```

## Flink中的Time和Watermark

#### Flink 时间概念

Flink 在流程序中支持不同的时间概念，下图为各个时间在整个流处理中的位置。

![1586437217324](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1586437217324.png)

- Processing time（处理时间），指正在执行相应操作时当前系统的时间。

- Event time（事件时间），事件时间是每个事件在其生产设备上发生的时间。

- Ingestion time（摄取时间），摄取时间是事件进入Flink的时间，在使用该时间时可以自动分配时间戳和自动生成watermark（水位线）。

#### Watermark的概念

Watermark（水位线）是Flink中衡量事件时间进度的机制。也是用于处理乱序事件的手段。Watermark是流的一部分，它维护一个时间戳，作为流中特殊的事件穿插在其中。它宣布事件的达到时间，这意味着当遇到Watermark时将认为晚于其内部时间戳的事件已经全部到达。

![1586427611519](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1586427611519.png)

而在分布式环境中，当多个上级算子生成不同的Watermark时，window算子将采用最小的一个。

![1586427832943](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1586427832943.png)

在window中，watermark的作用可从下图看出，当watermark的值大于或等于window结束时间时将触发window操作（当然当前window中必须有数据存在）。

![1586440190277](C:\Users\chac\Desktop\实验楼FLINK课程设计\004\assets\1586440190277.png)

#### 如何生成Watermark

生成watermark可以在DataStream使用assignTimestampsAndWatermarks函数创建watermark，其内部实现了多种机制，下面是两种Watermark的生成方式的接口

- AssignerWithPeriodicWatermarks可以每隔一段时间向事件流中插入一个watermark，间隔时间可通过ExecutionConfig.setAutoWatermarkInterval(...)指定，默认100ms
- AssignerWithPunctuatedWatermarks每个事件上都可以生成一个watermark，返回null时表示不生成

#### Watermark和SlidingTimeWindow的使用

