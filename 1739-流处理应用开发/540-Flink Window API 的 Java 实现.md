---
show: step
version: 1.0
---

## 课程介绍

本实验将带领学习Window，Flink的Time以及watermark机制。

本实验中使用了flink-connect-sequoiadb依赖，该依赖由巨杉开源社区提供。

* [下载地址](https://github.com/chaochaoc/flink-connect-sequoiadb)

#### 打开idea

打开idea代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson4 packge
打开```com.sequoiadb.flink.scdd.lesson4_window```，在该package中完成本课程。

![1739-540-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/fc0819b8e1c521dff7cd9c578e453398-0)

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

#### 打开类

在当前包下，打开类```TumblingCountWindowMain```

![1739-540-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/e3287d6c7d800c2f108b940ba4000c7e-0)

#### 原始数据的了解

本案例将使用一个在SequoiaDB中已存在的交易流水表，本例中只关心以下两个字段。

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

SequoiadbSource可以非常容易地从Sequoiadb中读取一个流。

```java
 // 构建连接Option
 SequoiadbOption option = SequoiadbOption.bulider()
 .host("localhost:11810")
 .username("sdbadmin")
 .password("sdbadmin")
 .collectionSpaceName("VIRTUAL_BANK")
 .collectionName("TRANSACTION_FLOW")
 .build();
 // 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"timestamp"构建流）
 env.addSource(new SequoiadbSource(option, "create_time"));
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳类型。

#### Map算子的使用

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem<BSONObject>，返回一个DataStream<Tuple2<Double, Integer>>。在本实验中，流中的每条数据均有实际意义，flink中将其称为一个事件。

```java
return dataStream.map(new MapFunction<BSONObject, Tuple2<Double, Integer>>() {
    /**
     * 在每个事件上调用一次
     * @param object 原始事件
     * @return 转换后的事件
     * @throws Exception
     */
    @Override
    public Tuple2<Double, Integer> map(BSONObject object) throws Exception {
        // 此处将事件中的money字段抽取出来，1表示当前事件中包含1笔交易
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
    /**
     * 聚合操作
     * @param t1 流上的其中一个事件
     * @param t2 流上的另一个事件
     * @return 合并后的事件
     * @throws Exception
     */
    @Override
    public Tuple2<Double, Integer> reduce(Tuple2<Double, Integer> t1, Tuple2<Double, Integer> t2) throws Exception {
        // 此处将统计总交易额和总交易量
        return Tuple2.of(t1.f0 + t2.f0, t1.f1 + t2.f1);
    }
});
```

### 运行作业

- 通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/83617d1fa1ab77f38247868bd0cd7b17-0)

- 查看结果。



## Tumbling Time Window的实现

#### 打开类

在当前包下，打开类```TumblingTimeWindowMain```

![1739-540-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/491d0cda3aca588ab10545802d44986f-0)

#### 原始数据的了解

本案例中使用到了以下三个字段

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

#### 添加Watermark

向流中添加watermark

```java
return transData.assignTimestampsAndWatermarks(new AssignerWithPeriodicWatermarks<BSONObject>() {
    // 延迟时间 (ms)
    private final static int maxOutOfOrderness = 3000;
    private long maxTimestamp = 0L;
    /**
     * 获取当前数据中的rowtime
     * @param object 当前数据行
     * @param timestamp 上一条数据的时间戳
     * @return 当前时间戳
     */
    @Override
    public long extractTimestamp(BSONObject object, long timestamp) {
        int currentTimestamp = ((BSONTimestamp) object.get("timestamp")).getTime();
        if (maxTimestamp < currentTimestamp) maxTimestamp = currentTimestamp;
        return currentTimestamp;
    }

    /**
     * 获取watermark
     * @return watermark对象
     */
    @Nullable
    @Override
    public Watermark getCurrentWatermark() {
        return new Watermark(maxTimestamp - maxOutOfOrderness);
    }
});
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
return dataStream.keyBy(new KeySelector<Tuple3<String, Double, Integer>, String>() {
    @Override
    public String getKey(Tuple3<String, Double, Integer> t) throws Exception {
        return t.f0;
    }
});
```

#### 在keyedStream上使用window

```java
return keyedStream.window(SlidingEventTimeWindows.of(Time.seconds(5), Time.seconds(2)));
```

#### 聚合求和

```java
return windowedStream.process(new ProcessWindowFunction<Tuple3<String, Double, Integer>, Result, String, TimeWindow>() {
    /**
      * @param s key
      * @param context 上下文对象，本算子的精华
      * @param iterable 当前window中的事件引用
      * @param collector 事件收集器
      * @throws Exception
      */
    @Override
    public void process(String s, Context context, Iterable<Tuple3<String, Double, Integer>> iterable, Collector<Result> collector) throws Exception {
        double sum = 0;
        int count = 0;
        Iterator<Tuple3<String, Double, Integer>> iterator = iterable.iterator();
        while (iterator.hasNext()) {
            Tuple3<String, Double, Integer> next = iterator.next();
            count += next.f2;
            sum += next.f1;
        }
        collector.collect(new Result(s, sum, count, new java.sql.Time(context.window().getEnd())));
    }
});
```

#### 将元组转换为BsonObject

```java
 return dataStream.map(new MapFunction<Result, BSONObject>() {
     @Override
     public BSONObject map(Result result) throws Exception {
         BasicBSONObject object = new BasicBSONObject();
         object.append("count", result.getCount());
         object.append("money", result.getMoney());
         object.append("trans_name", result.getTransName());
         object.append("time", result.getWindowTime());
         return object;
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

