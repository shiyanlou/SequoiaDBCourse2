---
show: step
version: 1.0
---

## 课程介绍

本实验将带领您了解与学习Flink Table API与Flink SQL。

Flink Table是Flink中的高级API, Table API将大大降低开发Flink程序的难度。本实验将使用Flink Table Api与Flink SQL来实现流作业的逻辑。

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

#### 打开IDEA

打开IDEA代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开flink-developer项目
打开flink-developer项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开lesson6 packge
打开```com.sequoiadb.flink.scdd.lesson6_table```，在该package中完成本课程。

![1739-560-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/8da2d09a94a1c75ba2c70342ec16c7f3-0)

#### 认识依赖

查看pom.xml文件，认识下列依赖。本案例新增了flink table的驱动包。
![1739-560-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/d66701bcb93d7343fb94b9269a243b3c-0)



## Table的简介

#### Table是什么

table是一个逻辑概念，其映射具体的DataStream或DataSet。可以通过sql操作table来达到操作具体的DataStream和DataSet。

#### Table环境

table的使用需要依赖于table的执行环境，table的执行环境可以通过现有的流环境进行创建

#### 如何创建一个表

- 从现有的StreamData中转换表

- 通过tableEnv 表描述器注册表

- 通过DDL（建表语句）创建表

#### TableDataStream模式

在flink中，模式分为Append, Retract和Upsert，Append表示仅有查询。Retract和Upsert将带有修改，每条数据均增加了一个Boolean字段。在Retract模式中，当该字段为false时，表示该条数据需要被删除。

## DataStream与表的转换

本例使用Flink Table实现word count。演示从DataStream转换Table，经中间转换过程后将在Table转换为DataStream，最后输出结果到控制台。

#### 打开类

在当前包下，打开类```CreateTableFromDataStreamMain```

![1739-560-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/93610689ea1667f04c9db3e463ba04c6-0)

#### 常见SQL算子

| SQL算子 | 用处       |
| ------- | ---------- |
| groubBy | 分组       |
| select  | 查询       |
| as      | 重命名字段 |
| where   | 数据过滤   |

#### 从已有的DataStream中创建Table

本案例中已存在一个DataStream<Tuple2<String, Integer>>，格式为（'单词', 1）。tbEnv.fromDataStream函数接收两个参数，分别为DataStream与一个字符串，表示字段名，多个字段用逗号分隔。

在当前类中找到createTableFromDataStream方法，找到 TODO code 1。

![1739-560-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/8d6d4a2772416779f23777f280d2198f-0)

将下列代码粘贴到 TODO code 1区间内。

```java
table = tbEnv.fromDataStream(wordData, "name, num");
```

#### SQL算子的使用

SQL算子的用途与标准sql中关键字一致。

在当前类中找到select方法，找到 TODO code 2。

![1739-560-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/32dbe649c789ce563ab671432d7dc739-0)

将下列代码粘贴到 TODO code 2区间内。

```java
/**
 * 等同于sql中的 
 * select word, sum(num)
 * from 
 *  ( select name as word, num 
 *   from 当前表 )
 * where word != 'java'
 * group by word 
 */
resultTable = initTable.as("word, num")         // 重命名字段
    .where("word != 'java'")                    // where算子过滤
    .groupBy("word")                            // 通过groupby 聚合
    .select("word, sum(num)");                  // 求和
```

#### Table转换为DataStream

当对table查询之后，向输出到控制台则需要将Table转换为DataStream

- 要点一：在此处需要传入一个TypeInformation，描述一个具体Flink的对象类型，Flink会将Table中的记录封装为该对象，此处为Tuple2<String, Integer>类型，当类型带有泛型时需要借助TypeHint对象获取。
- 要点二：由于使用了groupby算子，返回时必须使用toRetractStream。
- 要点三：toRetractStream返回一个RetractStream对象，实则就是一个在每个时间上均带有布尔类型的的DataStream。该布尔值为true时表示当前事件需要被删除。

在当前类中找到converTable2DataStream方法，找到 TODO code 3。

![1739-560-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/90b44f48ee4fcec3d10c43f936484de5-0)

将下列代码粘贴到 TODO code 3区间内。

```java
dataStream = tbEnv.toRetractStream(table, TypeInformation.of(
    new TypeHint<Tuple2<String, Integer>>() {}));
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableFromDataStreamMain.main()' 运行该Flink程序。

![1739-560-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/937f1de18e6772d9f0887caabb65432a-0)

查看结果。

![1739-560-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/81b61de6b2094ddd79e5fbd1b92c059b-0)

## 通过表描述器注册表

本例将通过描述器创建Source表与Sink表，实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

在当前包下，打开类```CreateTableByConnectTableSourceMain```

![1739-560-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/d8bfee1316715692b09b738ba8269f2e-0)

#### 通过描述器创建一个Source表

在当前类中找到createSourceTable方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
tbEnv.connect(
  new Sdb()
    .hosts("localhost:11810")                          // sdb 的连接地址
    .username("sdbadmin")                              // 用户名
    .password("sdbadmin")                              // 密码
    .collectionSpace("VIRTUAL_BANK")                   // 集合空间
    .collection("TRANSACTION_FLOW")                    // 集合
    .timestampField("create_time")                     // 流标识字段名
).withFormat(
  new Bson()                                           // 使用Bson数据格式
    .deriveSchema()                                    // 自动映射同名数据字段
    .failOnMissingField()                              // 当获取不到某个字段值时任务失败
).withSchema(
  new Schema()                                         // 定义table的结构
    .field("account", Types.STRING)					   // 账户
    .field("trans_name", Types.STRING)				   // 交易名
    .field("money", Types.BIG_DEC)					   // 交易金额
    .field("create_time", Types.SQL_TIMESTAMP)		   // 交易时间
).inAppendMode()
.registerTableSource("TRANSACTION_FLOW");              // 注册为一个数据来源表
```

#### 通过描述器创建一个Sink表

在当前类中找到createSinkTable方法，找到 TODO code 2。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/4bf3b08d1de0ef68b97a65fa3e221744-0)

将下列代码粘贴到 TODO code 2区间内。

```java
tbEnv.connect(
  new Sdb() 
    .hosts("localhost:11810")                          // sdb 的连接地址
    .username("sdbadmin")                              // 用户名
    .password("sdbadmin")                              // 密码
    .collectionSpace("VIRTUAL_BANK")                   // 集合空间
    .collection("LESSON_6_CONNECT")                    // 集合
).withFormat(
  new Bson()                                           // 使用Bson数据格式
    .deriveSchema()                                    // 自动映射同名数据字段
    .failOnMissingField()                              // 当获取不到某个字段值时任务失败
).withSchema(
  new Schema()                                         // 定义table的结构
    .field("total_sum", Types.BIG_DEC)
    .field("trans_name", Types.STRING)
).inUpsertMode()
    .registerTableSink("LESSON_6_CONNECT");             // 注册为一个数据来源表
```

#### 编写统计SQL

编写sql统计结果并将结果输出到巨杉数据库，统计每种交易的交易总额。

在当前类中找到select方法，找到 TODO code 3。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/19e2bda1a605f3f4bde299edf5ad3e0c-0)

将下列代码粘贴到 TODO code 3区间内。

```java
tbEnv.sqlUpdate(
    "INSERT INTO LESSON_6_CONNECT " +
    "SELECT " +
        "SUM(money) AS `total_sum`, " +
        "trans_name " +
    "FROM TRANSACTION_FLOW " +
    "GROUP BY " +
   		"trans_name");
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableByConnectTableSourceMain.main()' 运行该Flink程序。

![1739-560-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/954f646639b519256fc2b7262402357f-0)

通过SAC查看结果数据，结果在VIRTUAL_BANK.LESSON_6_CONNECT集合下。



## 通过DDL创建表

本案例通过DDL创建Flink流表。实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

打开类CreateTableByDDLMain

![1739-560-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/f9164f40d2b38d658d8d7c5708dba55a-0)

#### 创建source表

通过DDL创建flink source表。

在当前类中找到createSourceTable方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE TRANSACTION_FLOW (" +
    "  account STRING, " +                                 // 账户号
    "  trans_name STRING, " +                              // 交易名称
    "  money DECIMAL(10, 2), " +                           // 交易金额
    "  create_time TIMESTAMP(3)" +                         // 交易世家
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +                 // 连接介质类型
    "  'connector.hosts' = 'localhost:11810', " +          // 连接地址
    "  'connector.username' = 'sdbadmin', " +              // 用户名
    "  'connector.password' = 'sdbadmin', " +              // 密码
    "  'connector.collection-space' = 'VIRTUAL_BANK', " +  // 集合空间名
    "  'connector.collection' = 'TRANSACTION_FLOW', " +    // 集合名
    "  'connector.timestamp-field' = 'create_time', " +    // 流标识字段
    "  'format.type' = 'bson', " +                         // 数据类型 bson
    "  'format.derive-schema' = 'true', " +                // 自动映射同名字段
    "  'format.fail-on-missing-field' = 'true', " +   // 当某个字段获取不到时任务失败
    "  'update-mode' = 'append'" +                    // append模式
    ")");
```

#### 创建sink表

通过DDL创建flink sink表。

在当前类中找到createSinkTable方法，找到 TODO code 2。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/4bf3b08d1de0ef68b97a65fa3e221744-0)

将下列代码粘贴到 TODO code 2区间内。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE LESSON_6_DDL (" +
    "  trans_name STRING, " +                           // 交易名称
    "  `total_sum` DECIMAL(10, 2)" +                    // 交易总额
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +
    "  'connector.hosts' = 'localhost:11810', " +
    "  'connector.username' = 'sdbadmin', " +
    "  'connector.password' = 'sdbadmin', " +
    "  'connector.collection-space' = 'VIRTUAL_BANK', " +
    "  'connector.collection' = 'LESSON_6_DDL', " +
    "  'format.type' = 'bson', " +
    "  'format.derive-schema' = 'true', " +
    "  'format.fail-on-missing-field' = 'true', " +
    "  'update-mode' = 'upsert'" +                      // upsert模式，该模式可执行聚合语句
    ")");
```

#### 编写查询SQL

执行统计，统计每种交易的交易总额。

在当前类中找到select方法，找到 TODO code 3。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/19e2bda1a605f3f4bde299edf5ad3e0c-0)

将下列代码粘贴到 TODO code 3区间内。

```java
 tbEnv.sqlUpdate(
     "INSERT INTO LESSON_6_DDL " +
     "SELECT " +
         "SUM(money) AS `total_sum` " +
     	 "trans_name, " +
     "FROM TRANSACTION_FLOW " +
     "GROUP BY " +
     	"trans_name");
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableByDDLMain.main()' 运行该Flink程序。

![1739-560-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/702cef0700359287d448cbee0e0aab34-0)

通过SAC查看结果数据，结果在VIRTUAL_BANK.LESSON_6_DDL集合下。

## TableAPI中Watermark与Window的使用

打开类ExecuteSqlWithWatermakerAndWindowMain

![1739-560-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/042069c9598290fb4feb8903dc14a470-0)

#### 使用描述器中定义一个使用EventTime和Watermark

使用描述器定义一个使用EventTime和Watermark的source表。

在当前类中找到createSourceTable方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// 通过描述器连接表
tbEnv.connect(
   new Sdb()
    .hosts("localhost:11810")                               // sdb 的连接地址
    .username("sdbadmin")                                   // 用户名
    .password("sdbadmin")                                   // 密码
    .collectionSpace("VIRTUAL_BANK")                        // 集合空间
    .collection("TRANSACTION_FLOW")                         // 集合
    .timestampField("create_time")                          // 流标识字段名
).withFormat(
   new Bson()                           // 使用Bson数据格式, 当使用rowtime时必须显示指定format
    .bsonSchema(                        // Bson序列化器允许使用一个json串表示BsonFormat
        "{" +
            "account: 'string', " +
            "trans_name: 'string', " +
            "money: 'decimal', " +
            "create_time: 'timestamp'" +
        "}")
    .failOnMissingField()                       // 当获取不到某个字段值时抛出异常
).withSchema(
   new Schema()                                 // 定义table的结构
    .field("account", Types.STRING)             // 账户号
    .field("trans_name", Types.STRING)          // 交易名称，例如：结息，取款等
    .field("money", Types.BIG_DEC)              // 交易金额
    .field("create_time", Types.SQL_TIMESTAMP)  // 交易的时间
    .field("rowtime", Types.SQL_TIMESTAMP)      // EventTime字段
    .rowtime(
       new Rowtime()
        .timestampsFromField("create_time")     // 从字段中提取时间戳
        .watermarksPeriodicAscending()          // 设置watermark生成规则
    )
).inAppendMode()                                
.registerTableSource("LESSON_6_SQL");
```

#### Flink SQL中的函数

- TUMBLE_START()

  该函数表示获取翻滚窗口的开始时间，其中第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。如TUMBLE_START(rowtime, INTERVAL '5' SECOND)表示使用rowtime字段作为事件时间戳，获取窗口大小为5秒的翻滚窗口的窗口开始时间。此函数必须在GROUP BY TUMBLE(...) 才可以使用且TUMBLE_START函数的参数需要与TUMBLE函数参数完全一致。

- TUMBLE()

   窗口划分函数，表示使用翻滚窗口。第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。

- DATA_FORMAT() 

  该方法可以将时间戳格式化为固定格式的时间字符串。接收两个参数，第一个参数为一个Timestamp类型的字段名，为待转换的时间戳字段，第二个参数为格式化的字符串。

#### 编写SQL

执行统计，统计每种交易的交易总额。

在当前类中找到select方法，找到 TODO code 2。

![1739-560-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/542a6ee56b6da51cb1736ecdedfd7b3a-0)

将下列代码粘贴到 TODO code 2区间内。

```java
// 执行sql 数据统计
tbEnv.sqlUpdate(
    "INSERT INTO LESSON_6_SQL ( " +
    "SELECT " +
        "trans_name, " +
        "SUM(money) AS total_sum, " +
        "DATA_FORMAT(TUMBLE_END(`rowtime`, INTERVAL '5' SECOND), " +
    				"'HH:mm:ss') AS win_time " +
    "FROM TRANSACTION_FLOW " +
    "GROUP BY " +
        "TUMBLE(`rowtime`, INTERVAL '5' SECOND), " +
        "trans_name )"
);
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'ExecuteSqlWithWatermakerAndWindowMain.main()' 运行该Flink程序。

![1739-560-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/4896c1688098596aa7559ef4fc86b3d4-0)

通过SAC查看结果数据，结果在VIRTUAL_BANK.LESSON_6_SQL集合下。