---
show: step
version: 1.0
---

## 课程介绍

本课程将介绍当下非常火热的大数据实时计算框架 Flink ，并分析 SequoiaDB 巨杉数据库（之后简称 SequoiaDB ） 在实时计算场景的应用。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 Flink节点、1 个引擎协调节点，1 个编目节点与 3 个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

* [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为 1.9.2。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 scdd-flink 项目

打开 scdd-flink 项目，在该项目中完成本实验。

![1739-510-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/84d747adb87eaf46047241556ef88d8d-0)

#### 打开 lesson1 packge

打开包 com.sequoiadb.lesson.flink.lesson1_intro ，在该 package 中完成本课程。

![1739-510-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/33379a63a7db9a70550495c7d03cfb05-0)

## Flink 简介

#### Flink 是什么

Apache Flink 是一个开源框架和分布式处理引擎，可用于在无边界和有边界数据流上进行有状态的计算。Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。

Flink 是一个框架，也是一个分布式处理引擎（ Flink 支持分布式并行计算）；同时 Flink 可以做批处理（可以理解为有界的流）也可以做流处理；而整个计算过程中每个时间点的任务状态都可以被保存下来，一旦任务失败可以退回到某个时间点而不是全部重来一遍。Flink可以运行在 Yarn, K8S, Apache Mesos 或独立集群中，可适配到多种现有环境，其基于内存的计算并且可以部署任意规模的集群，小到个人PC虚拟机，大到 AWS 的超大分布式集群处理海量应用数据。

#### Flink的应用场景

![1739-510-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/e97d07fb063c8e68e7935e6901d5561f-0)

从这里可以看到常见的项目分析流程，首先数据（可以是业务数据，日志，物联网，点击行为等）直接进入或经转一些存储设备后进入 Flink ；Flink 通常用于事件驱动型应用处理，流批处理与 ETL 场景。其可运行在 K8S，Yarn，Mesos 等资源调度平台，状态可存储在 HDFS，S3，NFS 等存储平台；最终数据结果可落地到多种平台。

## Flink 的特点

#### 为什么要用 Flink

使用 Flink的原因很多，最重要的有以下几个原因。

- Flink EventTime 的支持
- 灵活的窗口
- Exactly once 语义保证

在 Flink 中，每条数据被称为一个 Event （时间）。所谓 EventTime 是便是数据产生的时间，例如在日志收集系统中，日志 A 产生的时间是 12:00 整，而日志 B 产生的时间为 12:01。但是由于日志发送过程中网络波动等原因，导致系统在 12:03 收到了日志 B，12:04 收到了日志 A，此时发现日志产生的顺序和收到日志的顺序是不一致的，而 Flink 支持通过事件产生的事件做处理。

通常情况下在一个流上做一些统计操作是没有意义的，因为流没有尽头，所以 Flink 内置了多种窗口，各种窗口可以按照不同的规则将数据流切分到多个不同的桶中，以满足各种需求。

Flink 支持状态管理，在任务出现异常时可以将任务回退到之前的某个事件点，此种模式可以保证所有数据处理且仅处理一次，也就是 Exactly once 语义。

#### Flink API 抽象级别

![1739-510-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/8394551203320de27a40de2e3350d92d-0)

从上图中可以看到，Flink 的 Core（也称之为 Runtime ）可运行在常见的资源环境中，如本地 JVM，集群和云平台中。其基础 API 可以看到分为用于流场景的 DataStream 与批场景的 DataSet，基于这两种 API，Flink 又抽象出 Table API 与 CEP 和 ML 等高级接口，本次课程只演示 DataStream API 和 Table API 的使用。

#### Flink 的执行流程

![1739-510-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/5509b69c586de4f3cff7ddac390cf55c-0)

这是 Flink 的工作流程，首先了解一下 Flink 中的基本角色。

- JobManager： 整个集群的 Master，负责接收客户端的消息和分配调度集群资源和分发任务给 TaskManager。
- JobClient： 负责向 JobManager 发送请求，在提交作业时负责将 Flink Program 组装为一个 JobGraph 发给 JobManager。
- TaskManager： 负责具体任务的执行，Task Slot 是其拥有资源（内存 ）的单位。
- Flink Program： 就是要编写的 Flink 程序， 在执行前会被映射成一个 Streaming Dataflow 结构。在下图中可以看到 Streaming Dataflow 的具体结构，可以分为三种， 分别为 Source、Transformation、Sink。Source 表示的是数据的来源，Sink 表示数据的落地，Transformation 表示的是数据的一系列转化流程。其中的 map、keyBy 等都是 Flink 程序中具体的 Transformation 算子。

![1739-510-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/775c3b6f72eceb152d101daba2c99f92-0)

## Flink Demo 示例

为了更好的理解 Flink 的工作原理及开发流程，本小节将展示一个 Demo 示例，一个经典案例单词统计，统计原始数据行中各个单词出现的次数。本案例主要学习 Flink 流程序的主要流程，算子的具体使用见下一小节。

一个 Flink 流程序中的主要流程为：获取流执行环境，使用 Source 添加数据源，使用 Transformation 算子对数据集进行转换操作，最终通过 Sink 将结果数据集输出到外部设备，流作业执行。

#### 打开类

在当前工程包下打开类 IntroDemoMain。

![1739-510-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/55be1976762224fb898c1a672547d98c-0)

#### 获取执行环境

一个 Flink 程序由 Source，Transformation，Sink 三部分组成。首先需要获取到 Flink 的流作业的执行环境。

1) 在当前类中找到 environment 方法，找到 TODO code 1。

![1739-510-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/0929b3a2eaa926acf17ae32f043da872-0)

2) 将下列代码粘贴到 TODO code 1 区间内，该代码实现获取流作业的执行环境。

```java
// Get the execution environment
env = StreamExecutionEnvironment.getExecutionEnvironment();
```
3) 粘贴代码后完整代码块如图所示。

![1739-510-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/d7c92eff4f7c6bbfb91830dae6c3be79-0)

> **说明**
>
> 粘贴方法如下：
>
> 1) 点击代码框右上角的 copy 图标。
>
> 2) 选择实验界面右边的 “剪切板”。
>
> ![1739-510-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/b60842a0c9d54018612e1dd416de2de2-0)
>
> 3) 在弹出的“在线环境剪切板”中粘贴复制的代码内容。
>
> ![1739-510-00019.png](https://doc.shiyanlou.com/courses/1739/1207281/f49b6f5673aea1b247782c104d891775-0)
>
> 4) 在实验环境中到对应的位置粘贴。
>
> ![1739-510-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/27f2a8ccf3a45b5758b9cdb1fed0ee98-0)

#### 使用Source获取DataStream

Source算子用于产生一个DataStream。

1) 在当前类中找到 source 方法，找到 TODO code 2。

![1739-510-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/7b713a9dbfebbc69b096f6f8d6997c18-0)

2) 将下列代码粘贴到 TODO code 2 区间内，当前代码段通过向流环境中添加数据源 RandomSource， 该数据源将每隔 1 秒向流中输入一条包含多个单词用空格连接的数据。

```java
// Generate some random data rows through RandomSource
dataSource = env.addSource(new RandomSource());
```

3) 粘贴代码后完整代码块如图所示。

![1739-510-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/8aab9c4a7c0954ce51b4e6fb256c827a-0)

#### Transformation的使用

Transformation可以对数据做转换操作，代码中的算子使用规则详见下一小节，此处仅做演示。

1) 在当前类中找到 transformate 方法，找到 TODO code 3。

![1739-510-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/a255639bcab9df82398483643aecdda5-0)

2) 将下列代码粘贴到 TODO code 3 区间内，该代码段实现将数据进行切分转换之后统计每个单词出现的次数。

```java
// Conversion the operator
SingleOutputStreamOperator<String> flatMapData = lineData.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String line, Collector<String> collector) throws Exception {
        for (String word : line.split(" ")) {
            collector.collect(word);
        }
    }
});
// Filter the operator 
SingleOutputStreamOperator<String> filterData = flatMapData.filter(s -> !s.equals("java"));
// Conversion the operator
SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = filterData.map(new MapFunction<String, Tuple2<String, Integer>>() {
    @Override
    public Tuple2<String, Integer> map(String s) throws Exception {
        return Tuple2.of(s, 1);
    }
});
// Group aggregation the operator
sumData = mapData.keyBy(0).sum(1);
```

3) 粘贴代码后完整代码块如图所示。

![1739-510-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/53608929b98744b9913249aca92719f4-0)

#### Sink算子的使用

使用Sink将结果输出到控制台。此处使用的 print 方法实则调用了一个 ConsoleSink，会将结果 sink 到控制台，或者说输出到标准输出。

1) 在当前类中找到 sink 方法，找到 TODO code 4，该代码段实现将数据结果输出到标准输出。

![1739-510-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/60d7ee96c9d74c42ef4398c46a9c7f4a-0)

2) 将下列代码粘贴到 TODO code 4 区间内。

```java
sumData.print();
```

3) 粘贴代码后完整代码块如图所示。

![1739-510-00024.png](https://doc.shiyanlou.com/courses/1739/1207281/8c74d66621f6a1677de8dd7b4a098b32-0)

#### 执行流作业

上述代码仅仅只是定义了一个流的转换逻辑，如果想让该流作业执行，还需要一个调用一个执行函数。

1) 在当前类中找到 exec 方法，找到 TODO code 5，该代码段将真正触发执行当前作业的数据处理流程。

![1739-510-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/3ae280970ea40ce4c03c7d86dce68106-0)

2) 将下列代码粘贴到 TODO code 5 区间内。

```java
// The parameter is the name of the current work
env.execute("flink intro demo");
```

3) 粘贴代码后完整代码块如图所示。

![1739-510-00025.png](https://doc.shiyanlou.com/courses/1739/1207281/c87c3e27c68c6fa0cc0712f776eadd0d-0)

#### 运行程序

通过在当前类上右键单击 > 左键单击 Run 'IntroDemoMain.main'  运行该 Flink 程序。

![1739-510-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/85a0bec4cf2695a1fda6074e47375710-0)

#### 执行结果

统计结果如下图。

![1739-510-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/163c12ef2d70f6ed685be00c38991234-0)

## SequoiaDB在流场景中的应用

SequoiaDB 是国内首个自主知识产权、自主研发的金融级分布式NewSQL 数据库。SequoiaDB 支持标准 SQL、事务操作、高并发、分布式、可扩展、与双引擎存储等特性，并已经作为商业化的数据库产品开源，SequoiaDB 目前在银行、证券、政府等领域都有广泛的应用，SequoiaDB 以其分布式、海量存储、高性能、可扩展等特性为个行业的客户提供了很好解决方案。

Flink 流数据处理的性能以及吞吐量在实际运用中，可以达到很高的处理性能和吞吐量，尤其是在处理基于 EventTime 的数据加工以及统计的场景下。其高性能和高容错机制在流数据处理的过程中能够既保证数据处理性能又可以保证数据处理的质量。

## 总结

本小节作为 Flink 的入门了解章节，讲述了 Flink 的使用场景以及 Flink 的执行流程。

SequoiaDB 数据库以其分布式、高性能、高可用的海量数据存储、查询、管理等特点能够完美胜任流处理场景的业务需求。

**知识点**

- Flink 的简介
- Flink 优势
- Flink 的组成结构
- Flink 作业的结构
- SequoiaDB 在流处理场景中的应用
