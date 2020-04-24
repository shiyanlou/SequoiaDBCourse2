---
show: step
version: 1.0
---

## 课程介绍

本实验为 Flink 流作业 Scala 版本的实现，与 Java 版的讲述相同，如果不感兴趣可以跳到下一小节。

本实验将带领您学习 Flink 的常用算子的使用，帮助您快速入门；同时学习如何将工程打包发布到集群环境。本实验采用经典案例 WordCount 单词统计进行演示。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 Flink节点、1个引擎协调节点，1个编目节点与3个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为 1.9.2。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 scdd-flink 项目

打开 scdd-flink 项目，在该课程中完成本试验。

![1739-510-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/2b68951cb04a44566d0a7219ede54005-0)

#### 打开 lesson3 packge

打开 com.sequoiadb.lesson.flink.lesson3_word_count，在该 package 中完成本课程。注意：该包位于 scala 源码包下。

![1730-530-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/0fd4e6295ec707993e09e044c0e24998-0)

#### 认识依赖

打开 pom.xml 文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/9b4833b8e0bc2160d90625911973ed4b-0)

本案例使用了 Flink 的 Runtime 依赖 flink-core 和流作业 Scala 开发依赖 flink-streaming-scala 包。

![1730-530-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/8a4039344d1aa5cbe1c4a1de4fef8aba-0)

## 查看原始数据格式

#### 打开类

在当前包下，打开类 WordCountMain。

![1730-530-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/e7952672e981d664fdc68fb81b7fa3f1-0)

#### 运行程序

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b527fd0f3f17108ffcdbd2dd726d56a7-0)

#### 查看结果

执行结果如下图。可以看到是一些数据行，每行有多个单词构成，此时如果想要统计每个单词出现的次数首先需要使用该算子对数据行进行切分成单个单词的数据行。

![1739-520-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/da5c2a4f975c9d36254f9cddd9476ca3-0)

## flatmap 算子

#### flatmap 算子的作用

flatmap 算子是 Transformation 的其中一种。该算子接收一个 DataStream 对象，返回一个 DataStream 对象，它在每个数据行上被调用一次，可以将一个数据行转换为多个数据行。

#### flatmap算子的使用

flatmap 算子中需传入一个函数或 FlatmapFunction 对象，简单的操作一般传入函数。

在当前类中找到 flatmap 方法，找到 TODO code 1。

![1730-530-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/66074b54e9c56eee316eba1454b72304-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
// "_" means each data row
flatmapData = dataStream.flatMap(_.split(" "))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b527fd0f3f17108ffcdbd2dd726d56a7-0)

可以看到在每个数据行上仅有一个单词。

![1739-520-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/be28584578c4520b5c2d20d42ed96652-0)

## filter 算子

#### filter 算子的作用

filter 算子是 Transformation 的其中一种。该算子在每个数据行上被调用一次，可以帮助去除掉某些数据行，参数是一个函数，该函数内部实现返回一个布尔类型，当其值为 false 时当前数据行被丢弃。

#### filter的使用

现在想把数据行中“java”单词去掉。

在当前类中找到 filter 方法，找到 TODO code 2。

![1730-530-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/3f66e4fb6aed19821cbfefdc098248f2-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
// Remove the word "java"
filterData = dataStream.filter(!_.equals("java"))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b527fd0f3f17108ffcdbd2dd726d56a7-0)

可以看到数据中已经没有“java”单词了。

![1739-520-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/473bb94278cfcff09c763af4dec5ff32-0)

## map 算子

#### map 算子的作用

map 算子也是 Transformation 的其中一种。map算子同样在每个数据行上被调用一次。值得注意的是与 flatmap 算子不同，map 算子在一个数据行上的调用中仅能输出一个新的数据行，而 flatmap 可以输出多行（包含零）。

#### map算子的使用

本实验中使用了 Scala 中的元组类型，用一对小括号表示，可以理解为能保存不同数据类型的列表。同时在 map 算子的输出结果中添加了一个整数1，表示当前记录的单词数。

在当前类中找到 map 方法，找到 TODO code 3。

![1730-530-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/b944a8e6b1211f6280b35e6dcc666e6f-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
// Convert data into tuples. 1 means there is a word in the current data row.
mapData = dataStream.map((_, 1))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b527fd0f3f17108ffcdbd2dd726d56a7-0)

可以看到每个数据行上都是一个元组，包含一个单词和1

![1739-520-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/975df71ddf869638717272b792d48273-0)



## keyBy 与 sum 算子

keyBy 与 sum 均为 Transformation 的一种。

#### keyBy 算子的作用

keyBy 算子可以通过指定 key 对数据进行分组，类似于 sql中的“group by”。值得注意的是，使用 keyBy 算子之后我们将得到一个 KeyedStream 对象，可以肯定的是无法在 keyBy 之后再次使用 keyBy。

#### sum 算子的作用

sum 算子接收一个 KeyedStream，可以对指定的字段进行求和操作，类似 sql “sum()”函数。

#### 实现单词数的统计

在 DataStream 的泛型为元组时，可以通过下标索引进行 keyBy 与 sum，当前实验使用第一个字段进行分组，对第二个字段进行求和。

在当前类中找到 sum 方法，找到 TODO code 4。

![1730-530-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/bb8085cb74ec1085393190530ccf4c25-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
// Users can group by the first field (words) in the tuple, and sum the second field (number of words).
sumData = dataStream.keyBy(0).sum(1)
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/b527fd0f3f17108ffcdbd2dd726d56a7-0)

可以看到单词统计的结果。

![1739-520-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/e86809b9ec06de067b157e0fed498ef1-0)

## reduce 算子（可选）

#### reduce 算子的作用

reduce 算子定义任意两个数据行合并为一个的数据行的逻辑。其内部实现 reduce 方法，该方法有两个参数，代表两条数据，在该方法中需要实现两条数据的聚合规则。

#### reduce 算子的使用

上述示例中使用了 sum 进行求和，但是如果有较为复杂的需求（如求平均值等）则必须使用 reduce 算子，此处同样使用 reduce 算子实现求和逻辑。

在当前类中找到 reduce 方法，找到 TODO code 5。

![1730-530-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/bfe6c441cda36101d2d4c501260544d6-0)

将下列代码粘贴到 TODO code 5区间内。

```scala
// x and y respectively represent two pieces of data. The output is the words in x, and the number is the sum of the words in x and y.
sumData = keyedData.reduce((x, y) => (x._1, x._2 + y._2))
```

>Note:
>
>sum，reduce 等算子都属于聚合类算子，其必须使用在 KeyedStream 之上。

## Flink 作业的执行（可选）

#### 运行环境

在 IDEA 中运行 main 函数，实则就是在本地启动了一个 Flink 环境，并向本地环境提交了当前作业。而通常情况下完成任务的编写之后，会将该任务提交到集群环境。

#### 项目打包

点击 maven 侧边栏中的 package 打包。

![1739-520-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/818235d78cdcfc4ffffe654cf621f74b-0)

打包成功后 jar 包会在当前项目目录的 target 目录下。

![1739-520-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/16c046a2a4611d6170dd2a7595a781de-0)

#### 提交到集群环境


通过浏览器打开 localhost:9091进入FlinkUI，默认端口8081，实验环境由于端口冲突改为了9091。

可以通过UI界面 > submit new job > add new  首先上传本地 jar 包。 

![1739-520-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/8e6df7ea80e5358c21e5f3a115ad60d7-0)

上传成功后，选择刚刚上传好的 jar。

![1739-520-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/8483eeb5b276e5322275cba39410d2d7-0)

添加入口类的引用（如下），点击 submit 提交当前作业。

```xml
com.sequoiadb.lesson.flink.lesson3_word_count.WordCountMain
```

![1739-520-00024.png](https://doc.shiyanlou.com/courses/1739/1207281/1ac844cc1599ef05d63aa2372877a6b8-0)

任务成功提交后，发现已经在运行，并且可以在 UI 界面上看到程序的 Dataflow。

![1739-520-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/d62689be889e598eb78ddd1685e036fe-0)

在对应的 Task Manager 中可以查看到当前作业的执行结果。

![1739-520-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/fe9f48d360016724607850fdb35387f9-0)

## Flink 工程打包与参数的获取（可选）

编写的程序在提交到集群后的 jar 如果想修改某些参数，需要重新打包。但是这很明显大大增加了不必要的工作量，Flink 同样支持动态参数的获取，下面来改造一下吧。

#### 参数获取

首先可以在 main 函数的 TODO code 6添加下列代码。

```scala
// Transfer args to ParameterTool, and the ParameterTool can help us parse parameters
val tool = ParameterTool.fromArgs(args)
// Get an integer value by name. 10 is the default value, and the default value is enabled if the parameter is not found.
val lineNum = tool.getInt("lineNum", 10)
```

lineNum 便是入的函数，需要通过 RandomSource 的构造器传入该值。

```scala
// Modify the method to get data
val lineData: DataStream[String] = env.addSource(new RandomSource(lineNum))
```

接下来将 jar 重新上传到集群，在提交作业时，在参数行添加参数。

![1739-520-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/2838685e3213c8f792a2d7e04c5d9d33-0)

## 总结

本次实验讲述了在 Scala 语言中 Flink 中的常见算子的使用，以及如何提交作业到集群环境。

**知识点**

- 常见算子的作用
- 常见算子的使用
- 聚合求和的实现
- Flink UI 的简单使用
