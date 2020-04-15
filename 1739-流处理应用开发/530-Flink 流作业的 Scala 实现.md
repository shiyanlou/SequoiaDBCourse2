---
show: step
version: 1.0
---

## 课程介绍

本实验为Scala版本的实现，与Java版的讲述相同，如果不感兴趣可以跳到下一小节。

本实验示例模拟了一个不断产生一个单词串的源头,需要使用Flink通过一些处理逻辑最终统计出每个单词出现的次数。

本实验将带领您学习flink的常用算子的使用，帮助您快速入门；同时学习如何将工程打包发布到集群环境。



#### 请点击右侧选择使用的实验环境

## 打开项目

#### 打开IDEA

打开IDEA代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson3 packge
打开```com.sequoiadb.scdd.lesson3_word_count```，在该package中完成本课程。注意：该包位于scala源码包下。

![1730-530-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/4f9788c0136df45e1312bc4cd911acaf-0)


#### 认识依赖

打开pom.xml文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/c8177f5490e581cd3a59c689b65f9143-0)

本案例使用了flink的runtime依赖flink-core和流作业Scala开发依赖flink-streaming-scala包。

![1730-530-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/8a4039344d1aa5cbe1c4a1de4fef8aba-0)

## flatmap算子

#### flatmap算子的作用

flatmap算子是Transformation的其中一种。该算子接收一个DataStream对象，返回一个DataStream对象，它在每个数据行上被调用一次，可以将一个数据行转换为多个数据行。

#### 查看原始数据格式

在当前包下，打开类```WordCountMain```

![1730-530-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/ac7f94c25e627be26f98111e28c39431-0)

通过在当前类文件上右键 > Run 'WordCountMain' 运行该Flink程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/984b8f0bd0d930cb4e4fa0313be4e3be-0)

执行结果如下图。

![1739-520-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/c4f49f737c7ddb0a52e56d679f40b93f-0)

可以看到是一些数据行，每行有多个单词构成，此时如果想要统计每个单词出现的次数首先需要使用该算子对数据行进行切分成单个单词的数据行。

#### flatmap算子的使用

flatmap算子中需传入一个函数或FlatmapFunction对象，简单的操作一般传入函数。

在当前类中找到flatmap方法，找到 TODO code 1。

![1730-530-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/a502cc2ef6797c2275030630bb0b1f90-0)

将下列代码粘贴到 TODO code 1区间内。

```scala
// "_"为每个数据行
flatmapData = dataStream.flatMap(_.split(" "))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该Flink程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/984b8f0bd0d930cb4e4fa0313be4e3be-0)

可以看到在每个数据行上仅有一个单词。

![1739-520-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/cb7cb4d2f65581057b8f4650d37b7a42-0)

## filter算子

#### filter算子的作用

filter算子是Transformation的其中一种。该算子在每个数据行上被调用一次，可以帮助去除掉某些数据行，参数是一个函数，该函数内部实现返回一个布尔类型，当其值为false时当前数据行被丢弃。

#### filter的使用

现在想把数据行中“java”单词去掉。

在当前类中找到filter方法，找到 TODO code 2。

![1730-530-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/7d00631a1d984d45d36ffbb01407ce10-0)

将下列代码粘贴到 TODO code 2区间内。

```scala
// 去除单词"java"
filterData = dataStream.filter(!_.equals("java"))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该Flink程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/984b8f0bd0d930cb4e4fa0313be4e3be-0)

可以看到数据中已经没有“java”单词了。

![1739-520-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/cb7cb4d2f65581057b8f4650d37b7a42-0)

## map算子

#### map算子的作用

map算子也是Transformation的其中一种。map算子同样在每个数据行上被调用一次。值得注意的是与flatmap算子不同，map算子在一个数据行上的调用中仅能输出一个新的数据行，而flatmap可以输出多行（包含零）。

#### map算子的使用

本实验中使用了Scala中的元组类型，用一对小括号表示，可以理解为能保存不同数据类型的列表。同时在map算子的输出结果中添加了一个整数1，表示当前记录的单词数。

在当前类中找到map方法，找到 TODO code 3。

![1730-530-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/bd337314ead497d72cbe6e4ae304c539-0)

将下列代码粘贴到 TODO code 3区间内。

```scala
// 将数据转化为元组，1表示当前数据行有一个单词
mapData = dataStream.map((_, 1))
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该Flink程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/984b8f0bd0d930cb4e4fa0313be4e3be-0)

可以看到每个数据行上都是一个Tuple，包含一个单词和1

![1739-520-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/69bf7d925bc6e8ecf950f5bc63d9c822-0)



## keyBy与sum算子

keyBy与sum均为Transformation的一种。

#### keyBy算子的作用

keyBy算子可以通过指定key对数据进行分组，类似于sql中的“group by”。值得注意的是，使用keyBy算子之后我们将得到一个KeyedStream对象，表示我们无法在keyBy之后再次使用keyBy。

#### sum算子的作用

sum算子接收一个KeyedStream，可以对指定的字段进行求和操作，类似sql “sum()”函数。

#### 实现单词数的统计

在DataStream的泛型为元组时，可以通过下标索引进行keyBy与sum，当前实验使用第一个字段进行分组，对第二个字段进行求和。

在当前类中找到sum方法，找到 TODO code 4。

![1730-530-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/ff2b5f93e4a39b2c6d86e613cbb96d99-0)

将下列代码粘贴到 TODO code 4区间内。

```scala
// 此处通过元组中第一个字段（单词）进行分组，第二个字段（单词数）进行求和
sumData = dataStream.keyBy(0).sum(1)
```

#### 查看结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该Flink程序。

![1730-530-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/984b8f0bd0d930cb4e4fa0313be4e3be-0)

可以看到单词统计的结果。

![1739-520-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5c0e1096418b2c32e3d09b69190be4e5-0)

## reduce算子（可选）

#### reduce算子的作用

reduce算子定义任意两个数据行合并为一个的数据行的逻辑。其内部实现reduce方法，该方法有两个参数，代表两条数据，在该方法中需要实现两条数据的聚合规则。

#### reduce算子的使用

上述示例中使用了sum进行求和，但是如果有较为复杂的需求（如求平均值等）则必须使用reduce算子，此处同样使用reduce算子实现求和逻辑。

在当前类中找到reduce方法，找到 TODO code 5。

![1730-530-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/57c28e7242836a179c6a772bedf76db6-0)

将下列代码粘贴到 TODO code 5区间内。

```java
// x和y分别表示两条数据，输出结果为 x中的单词，个数为x与y中的单词总和
sumData = keyedData.reduce((x, y) => (x._1, x._2 + y._2))
```

>Note:
>
>sum,reduce等算子都属于聚合类算子，其必须使用在KeyedStream之上。

## Flink作业的执行（可选）

#### 运行环境

在IDEA中运行main函数，实则就是在本地启动了一个flink环境，并向本地环境提交了当前作业。而通常情况下完成任务的编写之后，会将该任务提交到集群环境。

#### 项目打包

点击maven侧边栏中的package打包。

![1739-520-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/37946ad7e0012704490e2d0bde233908-0)

打包成功后jar包会在当前项目目录的target目录下。

![1739-520-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/eeaa23a35f2e41e8dfc49f78de5613a6-0)

#### 提交到集群环境

我们可以通过UI界面 > submit new job > add new(上传jar包) > 选择jar > 添加入口类 > submit(提交任务)。

![1739-520-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/e61441a7c28b896e9dc3923bd6d832b2-0)
发现任务已经成功提交，并且已经在运行，可以在界面上看到程序的执行结果。

![1739-520-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/3388299b06e7b517e58e93925c9e1879-0)

![1739-520-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/85b316e7d239a486ff553efa5cc41c7a-0)

## Flink工程打包与参数的获取（可选）

编写的程序在提交到集群后的jar如果想修改某些参数，需要重新打包。但是这很明显大大增加了不必要的工作量，Flink同样支持动态参数的获取，下面来改造一下吧。

#### 参数获取

- 首先可以在main函数的 TODO code 6添加下列代码。

```java
// 将args传入ParameterTool, ParameterTool可以帮助我们解析参数
val tool = ParameterTool.fromArgs(args)
// 通过名字获取一个整数值，10为默认值，如果参数未发现则启用默认值
val lineNum = tool.getInt("lineNum", 10)
```

- lineNum便是入的函数，需要通过RandomSource的构造器传入该值。

```scala
// 修改获取数据的写法
val lineData: DataStream[String] = env.addSource(new RandomSource(lineNum))
```

- 接下来重新提交集群，红色区域便是传入的参数。

![1739-520-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/133d00735186b728f871b9c9e26e4ab9-0)
