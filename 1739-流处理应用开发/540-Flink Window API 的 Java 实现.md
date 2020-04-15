---
show: step
version: 1.0
---

## 课程介绍

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

* [下载地址](https://github.com/chaochaoc/flink-connect-sequoiadb)

## 打开项目

#### 打开IDEA

打开IDEA代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson4 packge
打开com.sequoiadb.flink.scdd.lesson4_window，在该package中完成本课程。

![1739-540-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/fc0819b8e1c521dff7cd9c578e453398-0)

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

![1739-540-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/e3287d6c7d800c2f108b940ba4000c7e-0)

#### 原始数据的了解

本案例将使用一个在SequoiaDB中已存在的交易流水表，本例中只关心以下两个字段。

| 字段名      | 字段类型  | 备注       |
| ----------- | --------- | ---------- |
| money       | Decimal   | 交易金额   |
| create_time | Timestamp | 交易的时间 |

#### SequoiadbSource的使用

SequoiadbSource可以非常容易地从Sequoiadb中读取一个流。

在当前类中找到source方法，找到 TODO code 1。

![1739-540-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/c87469f4e0aacf1bdab71b81c0558138-0)

将下列代码粘贴到 TODO code 1区间内。

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
 sourceData = env.addSource(new SequoiadbSource(option, "create_time"));
```

以上示例为SequoiadbSource的使用，需要构建一个Option，包含巨杉数据库的连接信息。而且由于数据库中录入数据无法像消息队列做到时间态的有序，其还需要一个时间字段名用于构建流，该字段值必须是时间戳类型。

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/83617d1fa1ab77f38247868bd0cd7b17-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/0e2a071608abcb5c16effccba29a284f-0)



#### map算子的使用

使用map算子对流上的数据类型进行转换，该方法中接收一个DataStrem<BSONObject>，返回一个DataStream<Tuple2<Double, Integer>>。

在当前类中找到map方法，找到 TODO code 2。

![1730-530-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/91129a76a83d77828c7e86d3f83acd79-0)

将下列代码粘贴到 TODO code 2区间内。

```java
resultData = dataStream.map(new MapFunction<BSONObject, 
                            Tuple2<Double, Integer>>() {
    /**
     * 在每个事件上调用一次
     * @param object 原始事件
     * @return 转换后的事件
     * @throws Exception
     */
    @Override
    public Tuple2<Double, Integer> map(BSONObject object) throws Exception {
        // 此处将事件中的money字段抽取出来，1表示当前事件中包含1笔交易
        return Tuple2.of(((BSONDecimal) object.get("money"))
                         .toBigDecimal().doubleValue(), 1);
    }
});
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/83617d1fa1ab77f38247868bd0cd7b17-0)

执行结果如下图，可以看到一个Tuple2，包含交易额和1。

![1739-540-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/1e19ed6e0e99eccdb350b21138c05e7b-0)



#### Window划分

使用windowAll对流上数据进行分桶，此处使用翻滚计数窗口，窗口长度为100条，该算子返回一个AllWindowedStream<Tuple2<Double, Integer>, GlobalWindow>对象，表示Window中的数据类型，以及window的引用，在CountWindow中引用是一个全局的window对象。

在当前类中找到window方法，找到 TODO code 3。

![1739-540-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/e9b6ea391397bd0f6af72394aced1fd8-0)

将下列代码粘贴到 TODO code 3区间内。

```java
resultData = dataStream.countWindowAll(100);
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/83617d1fa1ab77f38247868bd0cd7b17-0)

执行结果如下图，可以看到每个window中的数据。

![1739-540-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/2c78a36653a1cae1ab491acee0c4daa4-0)



#### 聚合结果

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple2<Double, Integer>，分别表示总金额和总交易量。

在当前类中找到reduce方法，找到 TODO code 4。

![1739-540-00019.png](https://doc.shiyanlou.com/courses/1739/1207281/6c8c2eda6828803ce912b411bd600e89-0)

将下列代码粘贴到 TODO code 4区间内。

```java
resultData = dataStream.reduce(new ReduceFunction<Tuple2<Double,
                               Integer>>() {
    /**
     * 聚合操作
     * @param t1 流上的其中一个事件
     * @param t2 流上的另一个事件
     * @return 合并后的事件
     * @throws Exception
     */
    @Override
    public Tuple2<Double, Integer> reduce(Tuple2<Double, Integer> t1, 
                 Tuple2<Double, Integer> t2) throws Exception {
        // 此处将统计总交易额和总交易量
        return Tuple2.of(t1.f0 + t2.f0, t1.f1 + t2.f1);
    }
});
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/83617d1fa1ab77f38247868bd0cd7b17-0)

查看结果，可以得到每100次的交易额。

![1739-540-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/8f50992a6a7522e48c4156c30c52b931-0)

## Tumbling Time Window的实现

本案例通过Tumbling Time Window统计一个交易流水中每5秒中，每种交易的总交易额，总交易量。

#### 打开类

在当前包下，打开类TumblingTimeWindowMain

![1739-540-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/491d0cda3aca588ab10545802d44986f-0)

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

![1739-540-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/c87469f4e0aacf1bdab71b81c0558138-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// 构建连接Option
SequoiadbOption option = SequoiadbOption.bulider()
 .host("localhost:11810")
 .username("sdbadmin")
 .password("sdbadmin")
 .collectionSpaceName("VIRTUAL_BANK")
 .collectionName("TRANSACTION_FLOW")
 .build();
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
sourceData = env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-540-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8828f84eb18a05a8d583c98863688eb9-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/0e2a071608abcb5c16effccba29a284f-0)

#### 类型转换

通过map算子获取到交易名，交易金额，将BsonObject转换为Tuple2。

在当前类中找到map方法，找到 TODO code 2。

![1739-540-00024.png](https://doc.shiyanlou.com/courses/1739/1207281/3cfc823aa5267fb7c36907f51fbf5827-0)

将下列代码粘贴到 TODO code 2区间内。

```java
resultData = dataStream.map(new MapFunction<BSONObject, 
                            Tuple3<String, Double, Integer>>() {
    /**
     * 在每个事件上执行
     * @param object 原始事件
     * @return
     * @throws Exception
     */
    @Override
    public Tuple3<String, Double, Integer> map(BSONObject object) 
        throws Exception {
        // 抽取出需要的字段
        return Tuple3.of(object.get("trans_name").toString(),                      			((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
    }
});
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-540-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8828f84eb18a05a8d583c98863688eb9-0)

执行结果如下图，可以看到转换后的Tuple数据。

![1739-540-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/588a0f94eea2b2a8d890a37b93ca1381-0)


#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, String>对象，泛型中包含数据行和一个分组字段值。

在当前类中找到keyBy方法，找到 TODO code 3。

![1739-540-00025.png](https://doc.shiyanlou.com/courses/1739/1207281/6bbb600d5a1ec69c5d37cf14202c4ee5-0)

将下列代码粘贴到 TODO code 3区间内。

```java
resultData = dataStream.keyBy(new KeySelector<Tuple3<String, 
                              Double, Integer>, String>() {
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

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-540-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8828f84eb18a05a8d583c98863688eb9-0)

执行结果如下图，可以看到keyBy后的数据。

![1739-540-00029.png](https://doc.shiyanlou.com/courses/1739/1207281/331cfda4cca86782818d94365cd58ae3-0)


#### 在keyedStream上使用window

本案例使用时间进行划分窗口，窗口大小为5秒。

在当前类中找到window方法，找到 TODO code 4。

![1739-540-00026.png](https://doc.shiyanlou.com/courses/1739/1207281/4837719b17666b59f1c3bd762e257d01-0)

将下列代码粘贴到 TODO code 4区间内。

```java
resultData = keyedData.timeWindow(Time.seconds(5));
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-540-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8828f84eb18a05a8d583c98863688eb9-0)

执行结果如下图，可以看到每个window内的数据。

![1739-540-00030.png](https://doc.shiyanlou.com/courses/1739/1207281/8e1c76bc372d1b9f2960a83acb3725f2-0)

#### 聚合求和

通过聚合算子求出每个时间窗口中的交易名称，总交易额，总交易量，以及每个window的结束时间。

在当前类中找到reduce方法，找到 TODO code 5。

![1739-540-00027.png](https://doc.shiyanlou.com/courses/1739/1207281/33034e7ec740e0506c9605fd65061815-0)

将下列代码粘贴到 TODO code 5区间内。

```java
resultData = windowData.apply(new WindowFunction<Tuple3<String, Double, Integer>,
        Tuple4<String, Double, Integer, java.sql.Time>, String, TimeWindow>() {
	/**
     * 在每个window中执行一次
     * @param key 分组字段值
     * @param timeWindow 当前window对象
     * @param iterable 当前window中所有事件
     * @param collector 返回结果收集器
     * @throws Exception
     */
     @Override
     public void apply(String key, TimeWindow timeWindow, 
                       Iterable<Tuple3<String, Double, Integer>> iterable,
                       Collector<Tuple4<String, Double, Integer, 
                       java.sql.Time>> collector) throws Exception {
         double sum = 0;
         int count = 0;
         Iterator<Tuple3<String, Double, Integer>> iterator = 
             iterable.iterator();
         // 遍历当前window中的所有事件
         while (iterator.hasNext()) {
             Tuple3<String, Double, Integer> next = iterator.next();
             sum += next.f1;
             count += next.f2;
         }
         // 向每个事件中添加事件所在Window的结束事件
         collector.collect(Tuple4.of(key, sum, count, 
                  new java.sql.Time(timeWindow.getEnd())));
     }
});
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'TumblingTimeWindowMain.main()' 运行该Flink程序。

![1739-540-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8828f84eb18a05a8d583c98863688eb9-0)

执行结果如下图，可以看到数据库中的原始数据。

![1739-540-00031.png](https://doc.shiyanlou.com/courses/1739/1207281/e97c49caac3f754cf21f9784f18e094c-0)


## Sliding Count Window的实现

本案例使用Sliding Count Window统计一个交易流水中每中交易类型中100次交易的总交易额。

#### 打开类

在当前包下，打开类SlidingCountWindowMain

![1739-540-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/28e3ed435f489b1574d4c102e1289c44-0)

#### SequoiadbSource的使用

通过SequoiadbSource完成soucre函数。

在当前类中找到source方法，找到 TODO code 1。

![1739-540-00032.png](https://doc.shiyanlou.com/courses/1739/1207281/d5fe58d4ad2dc2b183b2bcfdfc33c3e6-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// 构建连接Option
SequoiadbOption option = SequoiadbOption.bulider()
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("TRANSACTION_FLOW")
    .build();
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
dataSource = env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 类型转换

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 2。

![1739-540-00033.png](https://doc.shiyanlou.com/courses/1739/1207281/4257899fb7970d037a3a30c891740e75-0)

将下列代码粘贴到 TODO code 2区间内。

```java
resultData = transData.map(new MapFunction<BSONObject, 
                           Tuple3<String, Double, Integer>>() {
	@Override
    public Tuple3<String, Double, Integer> map(BSONObject object) 
        throws Exception {
      return Tuple3.of(object.get("trans_name").toString(),
         ((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
      }
});
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, Tuple>对象，泛型中包含数据行和一个Tuple类型的分组字段值。

在当前类中找到keyBy方法，找到 TODO code 3。

![1739-540-00034.png](https://doc.shiyanlou.com/courses/1739/1207281/dfdca6cc8018aea0d08da2826609cddf-0)

将下列代码粘贴到 TODO code 3区间内。

```java
resultData = moneyData.keyBy(0);
```

#### 在keyedStream上使用window

案例中使用Sliding Count Window，窗口大小100，滑动步长50。

在当前类中找到countWindow方法，找到 TODO code 4。

![1739-540-00035.png](https://doc.shiyanlou.com/courses/1739/1207281/cc9ead55c9c181b9d657c7bbd37e3e22-0)

将下列代码粘贴到 TODO code 4区间内。

```java
resultData = keyedData.countWindow(100, 50);
```

#### 聚合求和

使用reduce对数据进行聚合求和，此处将的聚合结果为Tuple3<String, Double, Integer>，分别表示交易名称，总金额和总交易量。

在当前类中找到reduce方法，找到 TODO code 5。

![1739-540-00036.png](https://doc.shiyanlou.com/courses/1739/1207281/c64d0aa9951671c7df4a115e91eaaca3-0)

将下列代码粘贴到 TODO code 5区间内。

```java
resultData = countWindow.apply(new WindowFunction<Tuple3<String, Double, Integer>, Tuple2<String, Double>, Tuple, GlobalWindow>() {
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

将元组转换为BSONObject。

在当前类中找到toBson方法，找到 TODO code 6。

![1739-540-00037.png](https://doc.shiyanlou.com/courses/1739/1207281/5b8cafbe8e97c0fb9f8859b318e5fc62-0)

将下列代码粘贴到 TODO code 6区间内。

```java
bsonData = dataStream.map(new MapFunction<Tuple2<String, Double>, BSONObject>() {
    @Override
    public BSONObject map(Tuple2<String, Double> value) throws Exception {
        BasicBSONObject obj = new BasicBSONObject();
        obj.append("trans_name", value.f0);
        obj.append("total_sum", value.f1);
        return obj;
    }
});
```

#### 通过SequoiadbSink完成sink函数

在当前类中找到sink方法，找到 TODO code 7。

![1739-540-00038.png](https://doc.shiyanlou.com/courses/1739/1207281/cecf4b26f507d2aad9c0dcfa4d300537-0)

将下列代码粘贴到 TODO code 7区间内。

```java
// 构建连接Option
SequoiadbOption option = SequoiadbOption.bulider()
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("LESSON_4_COUNT")
    .build();
streamSink = dataStream.addSink(new SequoiadbSink(option));
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'SlidingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00040.png](https://doc.shiyanlou.com/courses/1739/1207281/d37e874540ed0c7ee764a2be9454aa14-0)

通过SAC查看结果数据，结果在VIRTUAL_BANK.LESSON_4_COUNT集合下。

![1739-540-00039.png](https://doc.shiyanlou.com/courses/1739/1207281/f12fe61a1c93cb641e6de523a2de7805-0)

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

![1739-540-00032.png](https://doc.shiyanlou.com/courses/1739/1207281/d5fe58d4ad2dc2b183b2bcfdfc33c3e6-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// 构建连接Option
SequoiadbOption option = SequoiadbOption.bulider()
    .host("localhost:11810")
    .username("sdbadmin")
    .password("sdbadmin")
    .collectionSpaceName("VIRTUAL_BANK")
    .collectionName("TRANSACTION_FLOW")
    .build();
// 向当前环境中添加数据源（SequoiadbSource需要通过时间字段"create_time"构建流）
dataSource = env.addSource(new SequoiadbSource(option, "create_time"));
```

#### 添加Watermark

向流中添加watermark。

在当前类中找到watermark方法，找到 TODO code 2。

![1739-540-00041.png](https://doc.shiyanlou.com/courses/1739/1207281/e14f96f1a0c9f9f82a8f7c730aebe892-0)

将下列代码粘贴到 TODO code 2区间内。

```java
resultData = transData.assignTimestampsAndWatermarks(
    new AssignerWithPeriodicWatermarks<BSONObject>() {
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
        int currentTimestamp = ((BSONTimestamp) object.get("create_time")).getTime();
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

通过map算子获取到交易名，交易金额。

在当前类中找到map方法，找到 TODO code 3。

![1739-540-00042.png](https://doc.shiyanlou.com/courses/1739/1207281/d063688bb0ce603e67f3b6cae309b6b5-0)

将下列代码粘贴到 TODO code 3区间内。

```java
return transData.map(new MapFunction<BSONObject, Tuple3<String, Double, Integer>>() {
	@Override
    public Tuple3<String, Double, Integer> map(BSONObject object) throws Exception {
      return Tuple3.of(object.get("trans_name").toString(),((BSONDecimal) object.get("money")).toBigDecimal().doubleValue(), 1);
      }
});
```

#### 分组

keyBy算子通过“trans_name”进行分组，keyBy返回一个KeyedStream<Tuple3<String, Double, Integer>, Tuple>对象，泛型中包含数据行和一个Tuple类型的分组字段值。

在当前类中找到keyBy方法，找到 TODO code 4。

![1739-540-00043.png](https://doc.shiyanlou.com/courses/1739/1207281/2f14f4794f4e730959f88fb0a7b239a3-0)

将下列代码粘贴到 TODO code 4区间内。

```java
resultData = dataStream.keyBy(new KeySelector<Tuple3<String, Double, Integer>, 
                              String>() {
    @Override
    public String getKey(Tuple3<String, Double, Integer> t) throws Exception {
        return t.f0;
    }
});
```

#### 在keyedStream上使用window

此处使用了SlidingEventTimeWindow，窗口大小为5秒，滑动步长为2秒。

在当前类中找到window方法，找到 TODO code 5。

![1739-540-00044.png](https://doc.shiyanlou.com/courses/1739/1207281/5b97b02c1cd6928cbbe1be1a4b797915-0)

将下列代码粘贴到 TODO code 5区间内。

```java
resultData = keyedStream.window(SlidingEventTimeWindows.of(Time.seconds(5), Time.seconds(2)));
```

#### 聚合求和

本例在聚合时使用了process算子，该算子与apply作用一致，区别在于process中可以获取到上下文对象。

在当前类中找到reduce方法，找到 TODO code 6。

![1739-540-00045.png](https://doc.shiyanlou.com/courses/1739/1207281/f9f09a4fc2b376e30124ae71bbc9cebf-0)

将下列代码粘贴到 TODO code 6区间内。

```java
restultData = windowedStream.process(new ProcessWindowFunction<Tuple3<String, Double, Integer>, Result, String, TimeWindow>() {
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

将POJO转换为BSONObject。

在当前类中找到toBson方法，找到 TODO code 7。

![1739-540-00046.png](https://doc.shiyanlou.com/courses/1739/1207281/7ce471307afa5e95995e90a6de6bc8f2-0)

将下列代码粘贴到 TODO code 7区间内。

```java
resultData = dataStream.map(new MapFunction<Result, BSONObject>() {
     @Override
     public BSONObject map(Result result) throws Exception {
         BasicBSONObject object = new BasicBSONObject();
         object.append("count", result.getCount());
         object.append("total_sum", result.getTotalSum());
         object.append("trans_name", result.getTransName());
         object.append("win_time", result.getWindowTime());
         return object;
     }
});
```

#### 通过SequoiadbSink完成sink函数

在当前类中找到sink方法，找到 TODO code 8。

![1739-540-00047.png](https://doc.shiyanlou.com/courses/1739/1207281/4dfb6a66829323eecad34a25350333f5-0)

将下列代码粘贴到 TODO code 8区间内。

```java
SequoiadbOption option = SequoiadbOption.bulider()
     .host("localhost:11810")
     .username("sdbadmin")
     .password("sdbadmin")
     .collectionSpaceName("VIRTUAL_BANK")
     .collectionName("LESSON_4_TIME")
     .build();
streamSink = dataStream.addSink(new SequoiadbSink(option));
```
#### 查看数据的结果

通过在当前类文件上右键 > Run 'SlidingCountWindowMain.main()' 运行该Flink程序。

![1739-540-00048.png](https://doc.shiyanlou.com/courses/1739/1207281/a8e6d8f8475fc663a77a6820841f4a6f-0)

通过SAC查看结果数据，结果在VIRTUAL_BANK.LESSON_4_TIME集合下。
