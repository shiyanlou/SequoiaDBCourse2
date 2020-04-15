---
show: step
version: 1.0
---

## 课程介绍

本课程将带领您了解与学习当下非常火热的大数据实时计算框架Flink，并分析一下Sequoiadb再实时计算场景的应用。

#### 请点击右侧选择使用的实验环境

#### 部署架构：
本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 Flink节点、1个引擎协调节点，1个编目节点与3个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：
* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为1.9.2。

## 打开项目

#### 打开IDEA

打开idea代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson1 packge
打开```com.sequoiadb.scdd.lesson1_intro```packge，在该package中完成本课程。

![1739-510-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/a88d97c8a83e607911d85fa999712382-0)


## Flink简介

#### Flink是什么

Apache Flink 是一个开源框架和分布式处理引擎，可用于在无边界和有边界数据流上进行有状态的计算。Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。

通俗来讲Flink是一个框架（为我们实现了大量复杂逻辑，让我们实现功能更加简单），也是一个分布式处理引擎（Flink支持分布式并行计算）；同时Flink可以做批处理（可以理解为有界的流）也可以做流处理；而整个计算过程中每个时间点的任务状态都可以被报存下来，一旦任务失败可以退回到某个时间点而不是全部重来一遍。Flink可以运行在Yarn, K8S, Apache Mesos或独立集群中，可适配到多种现有环境，其基于内存的计算并且可以部署任意规模的集群，小到个人PC虚拟机用来玩一玩，大到AWS的超大分布式集群处理海量应用数据。



#### Flink的应用场景

![1739-510-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/e97d07fb063c8e68e7935e6901d5561f-0)

从这里可以看到常见的项目分析流程，首先数据（可以是业务数据，日志，物联网，点击行为等）直接进入或经转一些存储设备后进入Flink；Flink通常用于事件驱动型应用处理，流批处理与ETL场景。其可运行在k8s，Yarn，Mesos等资源调度平台，状态可存储在HDFS，S3，NFS等存储平台；最终数据结果可落地到多种平台。

## Flink的特点

#### 为什么要用Flink

使用Flink的原因很多，最重要的有两个原因

- Flink EventTime的支持与灵活的窗口
- Exactly once语义保证

这里简单解释一下，后续会专门去了解这些概念的含义与其实现原理。EventTime是数据产生的时间，例如在日志收集系统中，日志A产生的时间是12:00整，而日志B产生的时间为12:01。但是由于日志发送网络波动等原因，导致系统在12:03收到了日志B，12:04收到了日志A，我们发现日志产生的顺序和我们收到日志的顺序是不一致的，但是我们想按照日志产生的顺序去处理日志，这个日志产生的时间在Flink中就叫做EventTime。而通常情况下在一个流上做一些统计操作是没有意义的，因为流没有尽头，所以Flink内置了多种窗口，各种窗口各种功能。而由于Flink支持状态管理，可以保证所有数据处理且仅处理一次，这就是Exactly once语义。

#### Flink抽象级别

![1739-510-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/8394551203320de27a40de2e3350d92d-0)

从上图中可以看到，Flink的核心（通常情况下我们称之为Runtime）可运行在常见的资源环境中，如本地JVM，集群和云平台中。其基础API可以看到分为用于流场景的DataStream与批场景的DataSet的，基于这两种API，Flink又抽象出Table Api与CEP和ML等高级接口，本次课程只演示DataStream API和Table API的使用。

#### Flink的执行流程

![1739-510-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/5509b69c586de4f3cff7ddac390cf55c-0)
这是Flink的工作流程，首先了解Flink中的基本角色

- JobManager 整个集群的Master，负责接收客户端的消息和分配调度集群资源和分发任务给TaskManager
- JobClient 负责向JobManager发送请求，在提交作业时负责将Flink Program组装为一个JobGraph发给JobManager
- TaskManager负责具体任务的执行，Task Slot是其拥有资源（内存）的单位
- Flink Program就是我们要编写的Flink程序， 在执行前会被映射成一个 Streaming Dataflow结构。在下图中可以看到Streaming Dataflow的具体结构，可以分为三种， 分别为Source、Transformation、Sink。Source表示的是数据的来源，Sink表示数据的落地，Transformation表示的是数据的一系列转化流程。其中的map、keyby等就是Flink程序中具体的转换算子。

![1739-510-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/775c3b6f72eceb152d101daba2c99f92-0)

## Flink Demo示例

为了帮助您更好的理解flink的工作原理及开发流程，本小节将展示一个Demo示例，一个经典案例单词统计，统计原始数据行中各个单词出现的次数。本案例仅做了解，算子的具体使用见下一小节。

#### 打开类

在当前工程包下打开类IntroDemoMain。

![1739-510-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/a33b303a8f34959f2bca2ae07ebc6ddd-0)



#### 获取执行环境

一个flink程序由Source，Transformation，Sink三部分组成。首先需要获取到Flink的流作业的执行环境，添加转换逻辑。

在当前类中找到environment方法，找到 TODO code 1。

![1739-510-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/6f8d37c1e639225dd360f1c469400358-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// 获取执行环境
env = StreamExecutionEnvironment.getExecutionEnvironment();
```

#### 使用Source获取DataStream

Source算子用于产生一个DataStream。

在当前类中找到source方法，找到 TODO code 2。

![1739-510-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/70d53c17390cbd9e57fde3eed307547b-0)

将下列代码粘贴到 TODO code 2区间内。

```java
// 通过RandomSource生成一些随机的数据行
dataSource = env.addSource(new RandomSource());
```

#### Transformation的使用

Transformation可以对数据做转换操作，代码中的算子使用规则详见下一小节，此处仅做演示。

在当前类中找到transformate方法，找到 TODO code 3。

![1739-510-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/d0224bfa01c602e43e1a396420850ee9-0)

将下列代码粘贴到 TODO code 3区间内。

```java
// 转换算子
SingleOutputStreamOperator<String> flatMapData = lineData.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String s, Collector<String> collector) throws Exception {
        Arrays.stream(s.split(" ")).forEach(collector::collect);
    }
});
// 过滤算子
SingleOutputStreamOperator<String> filterData = flatMapData.filter(s -> !s.equals("java"));
// 转换算子
SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = filterData.map(new MapFunction<String, Tuple2<String, Integer>>() {
    @Override
    public Tuple2<String, Integer> map(String s) throws Exception {
        return Tuple2.of(s, 1);
    }
});
// 分组聚合算子
sumData = mapData.keyBy(0).sum(1);
```

#### Sink算子的使用

使用Sink将结果输出到控制台。此处使用的print方法实则调用了一个ConsoleSink，会将结果sink到控制台。

在当前类中找到sink方法，找到 TODO code 4。

![1739-510-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/5c066bcb15d49c6c196d625b80e7578d-0)

将下列代码粘贴到 TODO code 4区间内。

```java
sumData.print();
```

#### 执行流作业

上述代码仅仅只是定义了一个流的转换逻辑，如果想让该流作业执行，还需要一个调用一个执行函数。

在当前类中找到exec方法，找到 TODO code 5。

![1739-510-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/beaff442291e00a599c5bf20614af7f8-0)

将下列代码粘贴到 TODO code 5区间内。

```java
// 参数为当前作业的名字
env.execute("flink intro demo");
```

#### 运行程序

通过在当前类上右键单击 > 左键单击Run 'IntroDemoMain.main' 运行该Flink程序。

![1739-510-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/92cad176fcb88a7485c6a3bc93740c55-0)

#### 执行结果

统计结果如下图。

![1739-510-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/597843c31ef8a551bc1bc19b019d374b-0)





## Sequoiadb在流场景中的应用

在处理大数据问题时，用户主要会面临三个V的挑战，也就是“海量数据”、“多样化数据”以及“时效性数据”这三种特性。使用SequoiaDB能够从动态数据类型解决多样化数据的问题，能够以水平切分的方式解决海量数据的问题，并以其高性能和高可扩展性满足时效性数据处理的需求。

在很多情况下，大数据应用需要NoSQL与Hadoop技术相结合以满足三个V的全部需求，以搭建能够承载批量分析与实时查询的混合大数据平台。作为国内首家通过Cloudera技术认证的NewSQL数据库产品企业，SequoiaDB与Hadoop不论在功能、性能、安全性还是稳定性上均通过了国际领先的Hadoop企业的官方认证，能够完全满足企业级用户与互联网用户对大数据的需求。

## 总结

本小节讲述了flink的使用场景，flink的执行流程，一个flink程序的结构。

