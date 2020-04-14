---
show: step
version: 1.0
---

## 课程介绍

本实验将带领学习Flink Table与Flink SQL。

Flink Table是Flink中的高级api, Table api将大大降低开发Flink程序的难度。本实验将使用Flink Table Api与Flink SQL来实现流作业的逻辑。

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

- groupby： 分组
- select: 查询
- as: 重命名字段
- where: 过滤

#### 从已有的DataStream中创建Table

本案例中已存在一个DataStream<Tuple2<String, Integer>>，格式为（'单词', 1）。现在将其转换为Table，请将createTableFromDataStream中粘贴下列代码段。tbEnv.fromDataStream函数接收两个参数，分别为DataStream与一个字符串，表示字段名，多个字段用逗号分隔。

```java
table = tbEnv.fromDataStream(wordData, "name, num");
```

#### SQL算子的使用

SQL算子的用途与标准sql中关键字一致。请将createTableFromDataStream中粘贴下列代码段。

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
- 要点二：由于使用了groupby算子，返回时必须使用toRetractStream，使用toAppendStream将会抛出异常。
- 要点三：toRetractStream返回一个RetractStream对象，实则就是一个在每个时间上均带有布尔类型的的DataStream。该布尔值为true时表示当前事件需要被删除。

```java
dataStream = tbEnv.toRetractStream(table, TypeInformation.of(
    new TypeHint<Tuple2<String, Integer>>() {}));
```

## 通过表描述器注册表

本例将通过描述器创建Source表与Sink表，实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

在当前包下，打开类```CreateTableByConnectTableSourceMain```

![1739-560-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/d8bfee1316715692b09b738ba8269f2e-0)

#### 通过描述器创建一个Source表

```java
tbEnv.connect(
  new Sdb()
    .hosts("192.168.0.111:11810")                      // sdb 的连接地址
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
    .field("account", Types.STRING)
    .field("trans_name", Types.STRING)
    .field("money", Types.BIG_DEC)
    .field("create_time", Types.SQL_TIMESTAMP)
).inAppendMode()
.registerTableSource("TRANSACTION_FLOW");              // 注册为一个数据来源表
```

#### 通过描述器创建一个Sink表

```java
tbEnv.connect(
  new Sdb()
    .hosts("192.168.0.111:11810")                      // sdb 的连接地址
    .username("sdbadmin")                              // 用户名
    .password("sdbadmin")                              // 密码
    .collectionSpace("VIRTUAL_BANK")                   // 集合空间
    .collection("TABLE_ANALYSIS")                      // 集合
).withFormat(
  new Bson()                                           // 使用Bson数据格式
    .deriveSchema()                                    // 自动映射同名数据字段
    .failOnMissingField()                              // 当获取不到某个字段值时任务失败
).withSchema(
  new Schema()                                         // 定义table的结构
    .field("sum", Types.BIG_DEC)
    .field("trans_name", Types.STRING)
).inUpsertMode()
    .registerTableSink("TABLE_ANALYSIS");              // 注册为一个数据来源表
```

#### 执行数据统计，并将结果输出到巨杉数据库，统计每种交易的交易总额

```java
tbEnv.sqlUpdate(
    "INSERT INTO TABLE_ANALYSIS " +
    "SELECT " +
        "SUM(money) AS `sum`, " +
        "trans_name " +
    "FROM TRANSACTION_FLOW " +
    "GROUP BY " +
   		"trans_name");
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableByConnectTableSourceMain.main()' 运行该Flink程序。

![1739-560-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/954f646639b519256fc2b7262402357f-0)

#### 通过SAC页面查看数据结果

## 通过DDL创建表

本案例通过DDL创建Flink流表。实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

打开类```CreateTableByDDLMain```

![1739-560-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/f9164f40d2b38d658d8d7c5708dba55a-0)

#### 创建source表

通过DDL创建flink source表。请在当前类的createSourceTable方法中粘贴下列代码段。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE test1 (" +
    "  account STRING, " +                              // 账户号
    "  trans_name STRING, " +                           // 交易名称
    "  money DECIMAL(10, 2), " +                        // 交易金额
    "  `timestamp` TIMESTAMP(3)" +                      // 交易世家
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +              // 连接介质类型
    "  'connector.hosts' = '192.168.0.111:11810', " +   // 连接地址
    "  'connector.username' = 'sdbadmin', " +           // 用户名
    "  'connector.password' = 'sdbadmin', " +           // 密码
    "  'connector.collection-space' = 'test', " +       // 集合空间名
    "  'connector.collection' = 'test7', " +            // 集合名
    "  'connector.timestamp-field' = 'timestamp', " +   // 流标识字段
    "  'format.type' = 'bson', " +                      // 数据类型 bson
    "  'format.derive-schema' = 'true', " +             // 自动映射同名字段
    "  'format.fail-on-missing-field' = 'true', " +     // 当某个字段获取不到时任务失败
    "  'update-mode' = 'append'" +                      // append模式
    ")");
```

#### 创建sink表

通过DDL创建flink sink表。请在当前类的createSinkTable方法中粘贴下列代码段。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE test2(" +
    "  trans_name STRING, " +                           // 交易名称
    "  `sum` DECIMAL(10, 2)" +                          // 交易总额
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +
    "  'connector.hosts' = '192.168.0.111:11810', " +
    "  'connector.username' = 'sdbadmin', " +
    "  'connector.password' = 'sdbadmin', " +
    "  'connector.collection-space' = 'test', " +
    "  'connector.collection' = 'test8', " +
    "  'format.type' = 'bson', " +
    "  'format.derive-schema' = 'true', " +
    "  'format.fail-on-missing-field' = 'true', " +
    "  'update-mode' = 'upsert'" +                      // upsert模式，该模式可执行聚合语句
    ")");
```

#### 编写查询SQL

执行统计，统计每种交易的交易总额。请在当前类的select方法中粘贴下列代码段。

```java
 tbEnv.sqlUpdate(
     "INSERT INTO SQL_ANALYSIS " +
     "SELECT " +
         "trans_name, " +
         "SUM(money) AS `sum` " +
     "FROM TRANSACTION_FLOW " +
     "GROUP BY " +
     	"trans_name");
```

#### 通过SAC页面查看数据结果

## TableAPI中Watermark与Window的使用

打开类```ExecuteSqlWithWatermakerAndWindowMain```

![1739-560-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/042069c9598290fb4feb8903dc14a470-0)

#### 使用描述器中定义一个使用EventTime和Watermark

使用描述器定义一个使用EventTime和Watermark的source表。请在当前类的createSourceTable方法中粘贴下列代码段。

```java
// 通过描述器连接表
tbEnv.connect(
   new Sdb()
    .hosts("192.168.0.111:11810")                           // sdb 的连接地址
    .username("sdbadmin")                                   // 用户名
    .password("sdbadmin")                                   // 密码
    .collectionSpace("test")                                // 集合空间
    .collection("test7")                                    // 集合
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
.registerTableSource("test1");
```

#### Flink SQL中的函数

- TUMBLE_START()

  该函数表示获取翻滚窗口的开始时间，其中第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。如TUMBLE_START(rowtime, INTERVAL '5' SECOND)表示使用rowtime字段作为事件时间戳，获取窗口大小为5秒的翻滚窗口的窗口开始时间。此函数必须在GROUP BY TUMBLE(...) 才可以使用且TUMBLE_START函数的参数需要与TUMBLE函数参数完全一致。

- TUMBLE()

   窗口划分函数，表示使用翻滚窗口。第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。

- DATA_FORMAT() 

  该方法可以将时间戳格式化为固定格式的时间字符串。接收两个参数，第一个参数为一个Timestamp类型的字段名，为待转换的时间戳字段，第二个参数为格式化的字符串。

#### 执行统计，统计每种交易的交易总额

```java
// 执行sql 数据统计
tbEnv.sqlUpdate(
    "INSERT INTO test2 ( " +
    "SELECT " +
    "trans_name, " +
    "SUM(money) AS `sum`, " +
    "TUMBLE_START(`rowtime`, INTERVAL '5' SECOND) AS win_start_time, " +
    "DATA_FORMAT(TUMBLE_END(`rowtime`, INTERVAL '5' SECOND), 'HH:mm:ss') AS format_win_end_time " +
    "FROM test1 " +
    "GROUP BY " +
    "TUMBLE(rowtime, INTERVAL '5' SECOND), " +
    "trans_name )"
);
```

#### 通过SAC页面查看数据结果