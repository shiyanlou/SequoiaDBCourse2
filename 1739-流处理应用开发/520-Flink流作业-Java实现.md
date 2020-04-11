---
show: step
version: 1.0
---

## 创建Flink-Maven项目

本实验示例模拟了一个不断产生一个单词串的源头,需要使用Flink通过一些处理逻辑最终统计出每个单词出现的次数.

#### 认识依赖

查看pom.xml文件，认识下列依赖

```xml
<dependencies>
    <!--flink runtime包，本地环境运行需要-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-core</artifactId>
        <version>${flink.version}</version>
        <scope>${flink.scope}</scope>
    </dependency>
    <!--flink 流处理java版，本地环境运行需要-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
        <version>${flink.version}</version>
        <scope>${flink.scope}</scope>
    </dependency>
    <!--flink sequoiadb 连接驱动包-->
    <dependency>
        <groupId>com.sequoiadb.flink</groupId>
        <artifactId>flink-connector-sequoiadb-${sequoiadb.version}_${scala.binary.version}</artifactId>
        <version>${flink.version}</version>
        <scope>${flink.scope}</scope>
    </dependency>
    <!--日志包-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.7</version>
        <scope>${flink.scope}</scope>
    </dependency>
</dependencies>
```


## flatmap算子

#### flatmap算子的作用

flatmap算子就是上一小节讲到的Transformation的其中一种，它可以将一行数据转换为多行数据。

首先执行WordCountMain的主函数查看一下原始数据的格式

![1739-520-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/c4f49f737c7ddb0a52e56d679f40b93f-0)

可以看到是一些数据行，每行有多个单词构成，首先想到的就是将其切分为单个的单词。

#### flatmap算子的使用

注释上一步的打印操作，添加flatmap转换逻辑

```java
SingleOutputStreamOperator<String> flatMapData = lineData.flatMap(new FlatMapFunction<String, String>() {
    @Override
    public void flatMap(String s, Collector<String> collector) throws Exception {
        String[] strings = s.split(" ");
        for (int i = 0; i < strings.length; i++) {
            collector.collect(strings[i]);
        }
    }
});
// 打印出来看一下,执行下一步前为了避免干扰请注释或删除
flatMapData.print();
```

flatmap算子中需要传递一个对象，该对象有一个抽象方法flatmap，参数分别为每行的数据"s"和接收返回结果的Collector<T>对象，泛型为返回记录的类型，这里我们也返回一个String。

>Note:
>
>此处请不要使用java1.8中的函数式接口实现，会导致程序无法获取返回类型。但是在返回类型可以确定的算子中可以使用这种写法，例如下面会讲到的filter算子，reduce算子都属于返回类型可以确定的算子

#### 查看数据的结果

可以看到已经变成了一个一个的单词

![1739-520-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/cb7cb4d2f65581057b8f4650d37b7a42-0)

## filter算子

#### filter算子的作用

fliter算子可以帮助我们去除掉某些数据行，该内部实现返回一个布尔类型，当其值为false时当前数据行被丢弃

#### filter的使用

现在我们想把数据行中“java”单词去掉

```java
SingleOutputStreamOperator<String> filterData = flatMapData.filter(new FilterFunction<String>() {
    @Override
    public boolean filter(String s) throws Exception {
    return !s.equals("java");
    }
});
// 打印出来看一下，看过之后就可以注释或删除掉
filterData.print();
```

## 实现Map算子的转换逻辑

#### map算子的作用

map算子可以将原来的数据做一定转换之后输出新的一条数据

#### map算子的使用

这个地方使用了一个在Flink中的新的数据类型，Tuple(元组)可以理解为能保保存不同数据类型的列表。我们在map中将每条记录添加了一个整数1，表示当前记录的单词数。

```java
SingleOutputStreamOperator<Tuple2<String, Integer>> mapData = filterData.map(new MapFunction<String, Tuple2<String, Integer>>() {
    @Override
    public Tuple2<String, Integer> map(String s) throws Exception {
        return Tuple2.of(s, 1);
    }
});
```



## keyBy与sum算子

#### keyBy算子的作用

keyBy算子可以通过指定key对数据进行分组，类似于sql中的“group by”. 值得注意的是，使用keyBy算子之后我们将得到一个KeyedStream对象，表示我们无法在keyBy之后再次使用keyBy.

#### sum算子的作用

sum算子可以对指定的字段进行求和操作，类似sql “sum()”函数。

#### 实现单词数的统计

```java
SingleOutputStreamOperator<Tuple2<String, Integer>> sumData = mapData.keyBy(0).sum(1);
```

本示例中使用了第一个字段(单词)进行分组，第二个字段(单词个数进行求和)。

![1739-520-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/5c0e1096418b2c32e3d09b69190be4e5-0)

## reduce算子

#### reduce算子的作用

reduce算子接收两个同类型参数，表示其中两条数据，返回一个与参数同类型的值，表示两条数据的聚合结果，其中可以自定义逻辑。

#### reduce算子的使用

上述示例中使用了sum进行求和，但是如果有较为复杂的需求（如求平均值等）则必须使用reduce算子，此处同样使用reduce算子实现求和逻辑。

```java
// 只做演示使用，此处与sum算子效果一致，只实现一种即可
SingleOutputStreamOperator<Tuple2<String, Integer>> reduceData = mapData.keyBy(0)
                .reduce((t1, t2) -> Tuple2.of(t1.f0, t1.f1 + t2.f1));
```

>Note:
>
>sum,reduce等算子都属于聚合类算子，其必须使用在keyby之后

## Flink作业的执行

#### 本地环境运行

我们在IDEA中运行main函数的话，实则就是在本地启动了一个flink环境，并向本地环境提交了当前作业。而通常情况下我们完成任务的编写之后，会将该任务提交到集群环境。

#### 项目打包

点击maven侧边栏中的package打包

![1739-520-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/37946ad7e0012704490e2d0bde233908-0)

打包成功后包会在我们的项目目录的target目录下

![1739-520-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/eeaa23a35f2e41e8dfc49f78de5613a6-0)

#### 提交到集群环境

我们可以通过UI界面 > submit new job > add new(上传jar包) > 选择jar > 添加入口类 > submit(提交任务)

![1739-520-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/e61441a7c28b896e9dc3923bd6d832b2-0)
发现任务已经成功提交，并且已经在运行，可以在界面上看到程序的执行结果

![1739-520-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/3388299b06e7b517e58e93925c9e1879-0)

![1739-520-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/85b316e7d239a486ff553efa5cc41c7a-0)

## Flink工程打包与参数的获取

编写的程序在提交到集群后的jar如果想修改某些参数，需要重新打包。但是这很明显大大增加了不必要的工作量，Flink同样支持动态参数的获取，下面来改造一下吧。

#### 参数获取

- 首先可以在main函数的开头添加下列代码

```java
// 将args传入ParameterTool, ParameterTool可以帮助我们解析参数
ParameterTool tool = ParameterTool.fromArgs(args);
// 通过名字获取一个整数值，10为默认值，如果参数未发现则启用默认值
int lineNum = tool.getInt("lineNum", 10);
```

- lineNum便是我们传入的函数，我们需要通过RandomSource的构造器传入该值
```java
// 修改获取数据的写法
DataStreamSource<String> lineData = env.addSource(new RandomSource(lineNum));
```

- 接下来重新提交集群，红色区域便是传入的参数。

![1739-520-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/133d00735186b728f871b9c9e26e4ab9-0)
