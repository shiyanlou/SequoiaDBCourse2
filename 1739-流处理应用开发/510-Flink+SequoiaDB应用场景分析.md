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
课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为1.9.2




## Flink简介

#### Flink是什么

Apache Flink 是一个开源框架和分布式处理引擎，可用于在无边界和有边界数据流上进行有状态的计算。Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。

这是官方的说辞，通俗点来讲就是Flink是一个框架（为我们实现了大量复杂逻辑，让我们实现功能更加简单）也是一个分布式处理引擎（Flink支持分布式并行计算）；同时Flink可以做批处理（可以理解为有界的流）也可以做流处理；而整个计算过程中每个时间点的任务详情都可以被报存下来，一旦任务失败可以退回到某个时间点而不是全部重来一遍。Flink可以运行再Yarn, K8S, Apache Mesos或独立集群中，可适配到多种现有环境中，基于内存的计算并且可以部署任意规模的集群，小到个人PC虚拟机用来玩一玩，大到阿里巴巴和AWS的超大分布式集群处理海量应用数据。



#### Flink的应用场景

![1739-510-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/e97d07fb063c8e68e7935e6901d5561f-0)

从这里可以看到常见的项目分析流程，首先数据（可以是业务数据，日志，物联网，点击行为等）直接进入或经转一些存储设备后进入Flink；Flink通常用于事件驱动型应用处理，流批处理与ETL场景。其可运行在k8s，Yarn，Mesos等资源调度平台，状态可存储在HDFS，S3，NFS等存储平台；最终数据结果可落地到多种平台。

#### Flink Demo示例

此处会先展示一个经典案例wordCount，先帮助您了解一下Flink的开发，下面请使用IDEA打开工程包"lesson1_intro"，找到IntroDemoMain运行主函数即可。此处只做演示，后续会有详细的开发课程。

结果如下图：

![1739-510-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/597843c31ef8a551bc1bc19b019d374b-0)



## Flink的特点

#### 为什么要用Flink

使用Flink的原因很多，最重要的有两个原因

- Flink EventTime的支持与灵活的窗口
- Exactly once语义保证

这里简单解释一下，后续会专门去了解这些概念的含义与其实现原理。EventTime就是数据产生的时间，例如在日志收集系统中，日志A产生的时间是12:00整，而日志B产生的时间为12:01。但是由于日志发送网络波动等原因，导致系统在12:03收到了日志B，12:04收到了日志A，我们发现日志产生的顺序和我们收到日志的顺序是不一致的，但是我们想按照日志产生的顺序去处理日志，这个日志产生的时间在Flink中就叫做EventTime。而通常情况下在一个流上做一些统计操作是没有意义的，因为流没有尽头，所以Flink内置了多种窗口，各种窗口各种功能。而由于Flink支持状态管理，可以保证所有数据处理且仅处理一次，这就是Exactly once语义。

#### Flink抽象级别

![1739-510-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/8394551203320de27a40de2e3350d92d-0)

从上图中可以看到，Flink的核心（通常情况下我们称之为Runtime）可运行在常见的资源环境中，如本地JVM，集群和云平台中。其基础API可以看到分为用于流场景的DataStream与批场景的DataSet的，基于这两种API，Flink又抽象出Table Api与CEP和ML等高级接口，本次课程只演示DataStream API和Table API的使用。

#### Flink的执行流程

![1739-510-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/5509b69c586de4f3cff7ddac390cf55c-0)
这是Flink的工作流程，首先了解Flink中的基本角色

- JobManager 整个集群的Master，负责接收客户端的消息和分配调度集群资源和分发任务给TaskManager
- JobClient 负责向JobManager发送请求，在提交作业时负责将Flink Program组装为一个JobGraph发给JobManager
- TaskManager负责具体任务的执行，Task Slot是其拥有资源（内存）的单位
- Flink Program就是我们要编写的Flink程序， 在执行前会被映射成一个 Streaming Dataflow结构。在下图中可以看到Streaming Dataflow的具体结构，可以分为三种， 分别为Source、Transformation、Sink。Source表示的是数据的来源，Sink表示数据的落地，Transformation表示的是数据的一系列转化流程。其中的map、keyby等就是Flink程序中具体的转换算子

![1739-510-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/775c3b6f72eceb152d101daba2c99f92-0)



## Sequoiadb在流场景中的应用

## Flink UI的认识
