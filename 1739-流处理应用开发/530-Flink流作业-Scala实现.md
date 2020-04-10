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
    <!--flink 流处理scala版，本地环境运行需要-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
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

点击图中（1）或（2）位置让idea将jar包自动引入到当前项目环境中

![1585564635592](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585564635592.png)

## 通过flatmap算子解析数据

#### flatmap算子的作用

flatmap算子就是上一小节讲到的Transformation的其中一种，它可以将一行数据转换为多行数据。

首先执行WordCountMain的主函数查看一下原始数据的格式

![1585575485899](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585575485899.png)

可以看到是一些数据行，每行有多个单词构成，首先想到的就是将其切分为单个的单词。

#### flatmap算子的使用

注释上一步的打印操作，添加flatmap转换逻辑

```scala
val flatMapData: DataStream[String] = lineData.flatMap(_.split(" "))
// 打印出来看一下,执行下一步前为了避免干扰请注释或删除
flatMapData.print();
```

flatmap算子中需要传递一个函数，"_"表示原始数据。

#### 查看数据的结果

可以看到已经变成了一个一个的单词

![1585617105439](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585617105439.png)

## filter算子实现数据的过滤

#### filter算子的作用

fliter算子可以帮助我们去除掉某些数据行，该内部需要一个函数，函数返回一个布尔类型，当其值为false时当前数据行被丢弃

#### filter的使用

现在我们想把数据行中“java”单词去掉

```scala
val filterData: DataStream[String] = flatMapData.filter(!_.equals("java"))
// 打印出来看一下，看过之后就可以注释或删除掉
filterData.print();
```

## 实现Map算子的转换逻辑

#### map算子的作用

map算子可以将原来的数据做一定转换之后输出新的一条数据

#### map算子的使用

这个地方使用了一个在Flink中的新的数据类型——元组，在scala中用小括号表示，可以理解为能保保存不同数据类型的列表。我们在元组中将每条记录添加了一个整数1，表示当前记录的单词数。

```scala
val mapData: DataStream[(String, Int)] = filterData.map((_, 1))
```



## keyBy与sum算子

#### keyBy算子的作用

keyBy算子可以通过指定key对数据进行分组，类似于sql中的“group by”. 值得注意的是，使用keyBy算子之后我们将得到一个KeyedStream对象，表示我们无法在keyBy之后再次使用keyBy.

#### sum算子的作用

sum算子可以对指定的字段进行求和操作，类似sql “sum()”函数。

#### 实现单词数的统计

```scala
val sumData: DataStream[(String, Int)] = mapData.keyBy(0).sum(1)
```

本示例中使用了第一个字段(单词)进行分组，第二个字段(单词个数进行求和)。

![1585703949687](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585703949687.png)

## reduce算子

#### reduce算子的作用

reduce算子中传入一个函数，函数接收两个同类型参数，表示其中两条数据，返回一个与参数同类型的值，表示两条数据的聚合结果，其中可以自定义逻辑。

#### reduce算子的使用

上述示例中使用了sum进行求和，但是如果有较为复杂的需求（如求平均值等）则必须使用reduce算子，此处同样使用reduce算子实现求和逻辑。

```scala
// 只做演示使用，此处与sum算子效果一致，只实现一种即可
val reduceData: DataStream[(String, Int)] = mapData.keyBy(0).reduce((x, y) => (x._1, x._2 + y._2))
```

>Note:
>
>sum,reduce等算子都属于聚合类算子，其必须使用在keyby之后

## Flink作业的执行

#### 本地环境运行

我们在IDEA中运行main函数的话，实则就是在本地启动了一个flink环境，并向本地环境提交了当前作业。而通常情况下我们完成任务的编写之后，会将该任务提交到集群环境。

#### 项目打包

点击maven侧边栏中的package打包

![1585704634656](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585704634656.png)

打包成功后包会在我们的项目目录的target目录下

![1585795442094](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585795442094.png)

#### 提交到集群环境

我们可以通过UI界面 > submit new job > add new(上传jar包) > 选择jar > 添加入口类 > submit(提交任务)

![1585705179987](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585705179987.png)
发现任务已经成功提交，并且已经在运行，可以在界面上看到程序的执行结果

![1585705373859](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585705373859.png)

![1585705488307](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585705488307.png)

## Flink工程打包与参数的获取

编写的程序在提交到集群后的jar如果想修改某些参数，需要重新打包。但是这很明显大大增加了不必要的工作量，Flink同样支持动态参数的获取，下面来改造一下吧。

- 首先可以在main函数的开头添加下列代码

```scala
// 使用参数解析
val tool = ParameterTool.fromArgs(args)
// 获取参数值，如果没有获取到则取 10
val lineNum = tool.getInt("lineNum", 10)
```

- lineNum便是我们传入的函数，我们需要通过RandomSource的构造器传入该值
```java
// 修改获取数据的写法
val lineData: DataStream[String] = env.addSource(new RandomSource(lineNum))
```

- 接下来重新提交集群，红色区域便是传入的参数。

![1585706389687](C:\Users\chac\Desktop\实验楼FLINK课程设计\002\assets\1585706389687.png)

