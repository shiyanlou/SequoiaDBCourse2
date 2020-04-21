---
show: step
version: 1.0
---

## 课程介绍

本实验将带领您学习flink的常用算子的使用，帮助您快速入门；同时学习如何将工程打包发布到集群环境。本实验采用经典案例WordCount 单词统计进行演示。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 Flink节点、1个引擎协调节点，1个编目节点与3个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink 版本为1.9.2。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 flink-developer 项目
打开 flink-developer 项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开 lesson2 packge
打开 com.sequoiadb.scdd.lesson2_word_count ，在该 package 中完成本课程。

![1739-520-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/c6df278d4353c03ac992972f49d311d1-0)


#### 认识依赖

打开 pom.xml 文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/c8177f5490e581cd3a59c689b65f9143-0)

本案例使用了 flink 的 runtime 依赖 flink-core 和流作业开发依赖 flink-streaming-java 包。
![1739-520-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/874953cf1510ccc9a14a15d2fd0f689b-0)

## 查看原始数据格式

#### 打开类

在当前包下，打开类 WordCountMain

![1739-520-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/bc95c345870df4ee91a4921858025104-0)

#### 运行程序

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序

![1739-520-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/3bd650d98a39ad86a74825302015f618-0)

#### 查看结果

执行结果如下图。可以看到是一些数据行，每行有多个单词构成，此时如果想要统计每个单词出现的次数首先需要使用该算子对数据行进行切分成单个单词的数据行。

![1739-520-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/da5c2a4f975c9d36254f9cddd9476ca3-0)

## flatmap 算子

#### flatmap 算子的作用

flatmap 算子是 Transformation 的其中一种。该算子接收一个 DataStream 对象，返回一个 DataStream 对象，它在每个数据行上被调用一次，可以将一个数据行转换为多个数据行。

#### flatmap 算子的使用

flatmap 算子中需要传递一个对象，该对象有两个泛型，分别为输入数据的类型及输出数据的类型，其有一个抽象方法 flatmap，用于实现转换的具体逻辑。

在当前类中找到 flatmap 方法，找到 TODO code 1。

![1739-520-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/65b6cb33f8dc4be13ef4e5494f2d2069-0)

将下列代码粘贴到 TODO code 1区间内。

```java
flatMapData = dataStreamSource.flatMap(new FlatMapFunction<String, String>() {
  /**
   * Execute once on each data row and it can output multiple data
   * @param s Raw data
   * @param collector Output result collector, which can send multiple data out through this object
   * @throws Exception
   */
  @Override
  public void flatMap(String s, Collector<String> collector) throws Exception {
      // Divide raw data into multiple words by spaces
      String[] strings = s.split(" ");
      for (int i = 0; i < strings.length; i++) {
          // Send each word as a new data row
          collector.collect(strings[i]);
      }
  }
});
```

> Note
>
> 此处请不要使用1.8中的函数式接口实现，由于其没有显式指定输出数据的类型会导致程序无法获取返回类型而抛出异常。

#### 查看数据的结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序

![1739-520-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/3bd650d98a39ad86a74825302015f618-0)

可以看到在每个数据行上仅有一个单词。

![1739-520-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/be28584578c4520b5c2d20d42ed96652-0)

## filter 算子

#### filter 算子的作用

filter 算子是 Transformation 的其中一种。该算子在每个数据行上被调用一次，可以帮助去除掉某些数据行，该内部实现返回一个布尔类型，当其值为 false 时当前数据行被丢弃。

#### filter 的使用

现在想把数据行中 “java” 单词去掉。

在当前类中找到 filter 方法，找到 TODO code 2。

![1739-520-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/8b75e2619afd10d3012c0be5ca3df0a1-0)

将下列代码粘贴到 TODO code 2区间内。

```java
// Filter the word "java"
filterData = dataStream.filter(new FilterFunction<String>() {
    /**
     * Execute on each data row
     * @param s Data row
     * @return Return false, and the current data row is discarded
     * @throws Exception
     */
    @Override
    public boolean filter(String s) throws Exception {
        return !s.equals("java");
    }
});
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1739-520-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/3bd650d98a39ad86a74825302015f618-0)

可以看到数据中已经没有“java”单词了。

![1739-520-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/473bb94278cfcff09c763af4dec5ff32-0)

#### 拓展提高（可选）

本步骤为可选，由于在filter算子中，输入的数据类型与输出的数据类型一致，则该算子中可以使用函数式的写法。如果有兴趣，可将filter函数中修改为下列代码块后重新执行当前程序。

```java
filterData = dataStream.filter(i -> !i.equals("java"));
```


## map算子

#### map算子的作用

map算子也是Transformation的其中一种。map算子同样在每个数据行上被调用一次。值得注意的是与flatmap算子不同，map算子在一个数据行上的调用中仅能输出一个新的数据行，而flatmap可以输出多行（包含零）。

#### map算子的使用

本实验中使用了一个在Flink中的新的数据类型，Tuple(元组)可以理解为能保存不同数据类型的列表。同时在map算子的输出结果中添加了一个整数1，表示当前记录的单词数。

在当前类中找到map方法，找到 TODO code 3。

![1739-520-00019.png](https://doc.shiyanlou.com/courses/1739/1207281/36524684f1caa4c8b94f15621454751f-0)

将下列代码粘贴到 TODO code 3区间内。

```java
mapData = dataStream.map(new MapFunction<String, Tuple2<String, Integer>>() {
    /**
     * Be called on each data row
     * @param s Original data
     * @return Converted data
     * @throws Exception
     */
    @Override
    public Tuple2<String, Integer> map(String s) throws Exception {
        return Tuple2.of(s, 1);
    }
});
```


#### 查看数据的结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序。

![1739-520-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/3bd650d98a39ad86a74825302015f618-0)


可以看到每个数据行上都是一个 Tuple2，包含一个单词和1。

![1739-520-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/975df71ddf869638717272b792d48273-0)



## keyBy 与 sum 算子

keyBy 与 sum 均为 Transformation 的一种。

#### keyBy 算子的作用

keyBy 算子可以通过指定 key 对数据进行分组，类似于 SQL 中的 “group by” 。值得注意的是，使用 keyBy 算子之后我们将得到一个 KeyedStream 对象，表示将无法在 keyBy 之后再次使用 keyBy。

#### sum算子的作用

sum 算子接收一个 KeyedStream，可以对指定的字段进行求和操作，类似 SQL中的 “sum()”函数。

#### 实现单词数的统计

在 DataStream 的泛型为 Tuple 时，可以通过下标索引进行 keyBy 与 sum，当前实验使用第一个字段进行分组，对第二个字段进行求和。

在当前类中找到 sum 方法，找到 TODO code 4。

![1739-520-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/cf2eb7863f07d910a08d0f0b2ff901aa-0)

将下列代码粘贴到 TODO code 4区间内。

```java
// When the generic type of DataStream is Tuple, users can directly sum keyBy through the subscript index.
sumData = tupleData.keyBy(0).sum(1);
```

#### 查看数据的结果

通过在当前类文件上右键 > Run 'WordCountMain' 运行该 Flink 程序

![1739-520-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/3bd650d98a39ad86a74825302015f618-0)


可以看到单词统计的结果。

![1739-520-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/e86809b9ec06de067b157e0fed498ef1-0)

## reduce 算子（可选）

#### reduce 算子的作用

reduce 算子定义任意两个数据行合并为一个的数据行的逻辑。其内部实现 reduce 方法，该方法有两个参数，代表两条数据，在该方法中需要实现两条数据的聚合规则。

#### reduce 算子的使用

上述示例中使用了 sum 进行求和，但是如果有较为复杂的需求（如求平均值等）则必须使用 reduce 算子，此处同样使用 reduce 算子实现求和逻辑。

在当前类中找到 reduce 方法，找到 TODO code 5。

![1739-520-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/25043e1d851d5f57cfdadb42441a5b83-0)

将下列代码粘贴到 TODO code 5区间内。

```java
// The following code is only for demonstration. It has the same effect as the sum operator, and implementing one is fine.
sumData = keyedData.reduce((t1, t2) -> Tuple2.of(t1.f0, t1.f1 + t2.f1));
```

>Note:
>
>sum，reduce 等算子都属于聚合类算子，其必须使用在 KeyedStream 之上。

## Flink 作业的执行（可选）

#### 运行环境

在IDEA中运行main函数，实则就是在本地启动了一个flink环境，并向本地环境提交了当前作业。而通常情况下完成任务的编写之后，会将该任务提交到集群环境。

#### 项目打包

点击maven侧边栏中的package打包。

![1739-520-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/63114ba459947ae84b120802661be94c-0)

打包成功后 jar 包会在当前项目目录的 target 目录下。

![1739-520-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/5c71f5421c834b0c51693d4f3a03726e-0)

#### 提交到集群环境

通过浏览器打开 localhost:9091进入FlinkUI，默认端口8081，实验环境由于端口冲突改为了9091。

可以通过UI界面 > submit new job > add new  首先上传本地 jar 包。 

![1739-520-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/065a67dd2edca2ff0d97f1df1c0a6242-0)

上传成功后，选择刚刚上传好的 jar。

![1739-520-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/c79098a0c0b369cc233e84f323abcd95-0)

添加入口类的引用（如下），点击 submit 提交当前作业。

```
com.sequoiadb.flink.scdd.lesson2_word_count.WordCountMain
```

![1739-520-00024.png](https://doc.shiyanlou.com/courses/1739/1207281/cf55c8e6662a34c69965a2f51d15f936-0)

任务成功提交后，发现已经在运行，并且可以在 UI 界面上看到程序的 Dataflow。

![1739-520-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/d62689be889e598eb78ddd1685e036fe-0)

在对应的 Task Manager 中可以查看到当前作业的执行结果。

![1739-520-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/fe9f48d360016724607850fdb35387f9-0)

## Flink 工程参数的获取（可选）

编写的程序在提交到集群后的jar如果想修改某些参数，需要重新打包。但是这很明显大大增加了不必要的工作量，Flink 同样支持动态参数的获取，下面来改造一下。

#### 参数获取

首先可以在 main 函数的开头 TODO code 6添加下列代码。

```java
// Transfer args to ParameterTool, and the ParameterTool can help us parse parameters
ParameterTool tool = ParameterTool.fromArgs(args);
// Get an integer value by name. 10 is the default value, and the default value is enabled if the parameter is not found
int lineNum = tool.getInt("lineNum", 10);
```

lineNum 便是入的函数，需要通过 RandomSource 的构造器传入该值，此处修改当前类main函数中添加 Source的代码部分。

```java
// Modify the method to get data, and add the construction parameter lineNum
DataStreamSource<String> lineData = env.addSource(new RandomSource(lineNum));
```

接下来将 jar 重新上传到集群，在提交作业时，在参数行添加参数。

![1739-520-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/d9787373b031a95f80d0933c993fc02a-0)

## 总结

本次实验讲述了 Flink 中的常见算子的使用，以及如何提交作业到集群环境。

**知识点**

- 常见算子的作用
- 常见算子的使用
- 聚合求和的实现
- Flink UI 的简单使用