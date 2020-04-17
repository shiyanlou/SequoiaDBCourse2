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

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 flink-developer 项目
打开 flink-developer 项目，在该课程中完成本试验。

![1739-510-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/99b152f08db639b9d163676a09b7102e-0)

#### 打开 lesson6 packge
打开 com.sequoiadb.flink.scdd.lesson6_table ，在该 package 中完成本课程。

![1739-560-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/8da2d09a94a1c75ba2c70342ec16c7f3-0)

#### 认识依赖

查看 pom.xml 文件，认识下列依赖。本案例新增了flink table的驱动包。
![1739-560-00002.png](https://doc.shiyanlou.com/courses/1739/1207281/d66701bcb93d7343fb94b9269a243b3c-0)



## Table 的简介

#### Table 是什么

Table 是一个逻辑概念，其映射具体的 DataStream 或 DataSet 。可以通过 sql 操作 Table 来达到操作具体的DataStream和DataSet。

#### Table 环境

Table 的使用需要依赖于table的执行环境，table的执行环境可以通过现有的流环境进行创建

#### 如何创建一个表

- 从现有的 StreamData 中转换表

- 通过 TableEnv 表描述器注册表

- 通过 DDL（建表语句）创建表

#### TableDataStream 模式

在 flink中，模式分为 Append, Retract 和 Upsert，Append 表示仅有查询。Retract 和 Upsert 将带有修改，每条数据均增加了一个Boolean字段。在 Retract 模式中，当该字段为 false 时，表示该条数据需要被删除。

## DataStream 与表的转换

本例使用 Flink Table 实现 word count。演示从 DataStream转换 Table，经中间转换过程后将在 Table 转换为DataStream，最后输出结果到控制台。

#### 打开类

在当前包下，打开类 CreateTableFromDataStreamMain 

![1739-560-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/93610689ea1667f04c9db3e463ba04c6-0)

#### 常见 SQL 算子

| SQL算子 | 用处       |
| ------- | ---------- |
| groubBy | 分组       |
| select  | 查询       |
| as      | 重命名字段 |
| where   | 数据过滤   |

#### 从已有的 DataStream 中创建 Table

本案例中已存在一个 DataStream<Tuple2<String, Integer>>，格式为（'单词', 1）。tbEnv.fromDataStream 函数接收两个参数，分别为 DataStream 与一个字符串，表示字段名，多个字段用逗号分隔。

在当前类中找到 createTableFromDataStream 方法，找到 TODO code 1。

![1739-560-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/8d6d4a2772416779f23777f280d2198f-0)

将下列代码粘贴到 TODO code 1区间内。

```java
table = tbEnv.fromDataStream(wordData, "name, num");
```

#### SQL 算子的使用

SQL 算子的用途与标准sql中关键字一致。

在当前类中找到 select 方法，找到 TODO code 2。

![1739-560-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/32dbe649c789ce563ab671432d7dc739-0)

将下列代码粘贴到 TODO code 2区间内。

```java
/**
 * Equivalent to sql
 * select word, sum(num)
 * from 
 *  ( select name as word, num 
 *   from "current table" )
 * where word != 'java'
 * group by word 
 */
resultTable = initTable.as("word, num")         // Rename field
    .where("word != 'java'")                    // where operator filtering
    .groupBy("word")                            // Aggregate by groupby
    .select("word, sum(num)");                  // Sum
```

#### Table 转换为 DataStream

当对table查询之后，向输出到控制台则需要将Table转换为DataStream

- 要点一：在此处需要传入一个 TypeInformation，描述一个具体 Flink 的对象类型，Flink 会将 Table 中的记录封装为该对象，此处为 Tuple2<String, Integer> 类型，当类型带有泛型时需要借助 TypeHint 对象辅助获取。
- 要点二：由于使用了 groupby 算子，返回时必须使用 toRetractStream。
- 要点三：toRetractStream 返回一个 RetractStream 对象，实则就是一个在每个时间上均带有布尔类型的的DataStream。该布尔值为 false 时表示当前事件需要被删除。

在当前类中找到 converTable2DataStream 方法，找到 TODO code 3。

![1739-560-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/90b44f48ee4fcec3d10c43f936484de5-0)

将下列代码粘贴到 TODO code 3区间内。

```java
dataStream = tbEnv.toRetractStream(table, TypeInformation.of(
    new TypeHint<Tuple2<String, Integer>>() {}));
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableFromDataStreamMain.main()' 运行该 Flink 程序。

![1739-560-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/937f1de18e6772d9f0887caabb65432a-0)

查看结果。

![1739-560-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/81b61de6b2094ddd79e5fbd1b92c059b-0)

## 通过表描述器注册表

本例将通过描述器创建 Source 表与 Sink 表，实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

在当前包下，打开类 CreateTableByConnectTableSourceMain 

![1739-560-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/d8bfee1316715692b09b738ba8269f2e-0)

#### 通过描述器创建一个 Source 表

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
tbEnv.connect(
  new Sdb()
    .version("3.4")									   // Version of sdb
    .hosts("localhost:11810")                          // Connection address of sdb
    .username("sdbadmin")                              // Username
    .password("sdbadmin")                              // Password
    .collectionSpace("VIRTUAL_BANK")                   // CollectionSpace
    .collection("TRANSACTION_FLOW")                    // Collection
    .timestampField("create_time")                     // Stream Timestamp field
).withFormat(
  new Bson()                                           // Use Bson data format
    .deriveSchema()                                    //  Map data fields with the same name automatically
    .failOnMissingField()                              // When a field value cannot be obtained, the task fails
).withSchema(
  new Schema()                                         // Define the structure of the table
    .field("account", Types.STRING)					   // Account
    .field("trans_name", Types.STRING)				   // Transaction name
    .field("money", Types.BIG_DEC)					   // Transaction amount
    .field("create_time", Types.SQL_TIMESTAMP)		   // Transaction hour
).inAppendMode()
.registerTableSource("TRANSACTION_FLOW");              // Register as a data source table
```

#### 通过描述器创建一个 Sink 表

在当前类中找到 createSinkTable 方法，找到 TODO code 2。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/4bf3b08d1de0ef68b97a65fa3e221744-0)

将下列代码粘贴到 TODO code 2区间内。

```java
tbEnv.connect(
  new Sdb() 
    .version("3.4")									   // Version of sdb
    .hosts("localhost:11810")                          // Connection address of sdb
    .username("sdbadmin")                              // Username
    .password("sdbadmin")                              // Password
    .collectionSpace("VIRTUAL_BANK")                   // CollectionSpace
    .collection("LESSON_6_CONNECT")                    // Collection
).withFormat(
  new Bson()                                           // Use Bson data format
    .deriveSchema()                                    //  Map data fields with the same name automatically
    .failOnMissingField()                              // When a field value cannot be obtained, the task fails
).withSchema(
  new Schema()                                         // Define the structure of the table
    .field("total_sum", Types.BIG_DEC)
    .field("trans_name", Types.STRING)
).inUpsertMode()
    .registerTableSink("LESSON_6_CONNECT");             // Register as a data source table
```

#### 编写统计 SQL

编写 sql 统计结果并将结果输出到巨杉数据库，统计每种交易的交易总额。

在当前类中找到 select 方法，找到 TODO code 3。

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

通过在当前类文件上右键 > Run 'CreateTableByConnectTableSourceMain.main()' 运行该 Flink 程序。

![1739-560-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/954f646639b519256fc2b7262402357f-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_CONNECT 集合下。



## 通过 DDL 创建表

本案例通过 DDL 创建 Flink 流表。实现从巨杉数据库读入数据，经 Flink 统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

打开类 CreateTableByDDLMain

![1739-560-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/f9164f40d2b38d658d8d7c5708dba55a-0)

#### 创建 Source 表

通过 DDL 创建 Flink Source 表。

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE TRANSACTION_FLOW (" +
    "  account STRING, " +                                 // Account number
    "  trans_name STRING, " +                              // Name of transaction
    "  money DECIMAL(10, 2), " +                           // Transaction amount
    "  create_time TIMESTAMP(3)" +                         // Transaction time
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +                 // Connection media type
    "  'connector.version' = '3.4', " +					   // Version of SequoiaDB
    "  'connector.hosts' = 'localhost:11810', " +          // Connection address
    "  'connector.username' = 'sdbadmin', " +              // Username
    "  'connector.password' = 'sdbadmin', " +              // Password
    "  'connector.collection-space' = 'VIRTUAL_BANK', " +  // CollectionSpace
    "  'connector.collection' = 'TRANSACTION_FLOW', " +    // CollectionName
    "  'connector.timestamp-field' = 'create_time', " +    // Stream Timestamp field
    "  'format.type' = 'bson', " +                         // Data type bson
    "  'format.derive-schema' = 'true', " +                //  Map data fields with the same name automatically
    "  'format.fail-on-missing-field' = 'true', " +   // When a field cannot be obtained, the task fails
    "  'update-mode' = 'append'" +                    // append mode
    ")");
```

#### 创建 Sink 表

通过 DDL 创建 Flink Sink 表。

在当前类中找到 createSinkTable 方法，找到 TODO code 2。

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/4bf3b08d1de0ef68b97a65fa3e221744-0)

将下列代码粘贴到 TODO code 2区间内。

```java
tbEnv.sqlUpdate(
    "CREATE TABLE LESSON_6_DDL (" +
    "  trans_name STRING, " +                           // Transaction name
    "  `total_sum` DECIMAL(10, 2)" +                    // Transaction sum
    ") WITH (" +
    "  'connector.type' = 'sequoiadb', " +
    "  'connector.version' = '3.4', " +					// Version of SequoiaDB
    "  'connector.hosts' = 'localhost:11810', " +
    "  'connector.username' = 'sdbadmin', " +
    "  'connector.password' = 'sdbadmin', " +
    "  'connector.collection-space' = 'VIRTUAL_BANK', " +
    "  'connector.collection' = 'LESSON_6_DDL', " +
    "  'format.type' = 'bson', " +
    "  'format.derive-schema' = 'true', " +
    "  'format.fail-on-missing-field' = 'true', " +
    "  'update-mode' = 'upsert'" +                      // upsert mode, which can execute aggregate statements
    ")");
```

#### 编写查询 SQL

执行统计，统计每种交易的交易总额。

在当前类中找到 select 方法，找到 TODO code 3。

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

通过在当前类文件上右键 > Run 'CreateTableByDDLMain.main()' 运行该 Flink 程序。

![1739-560-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/702cef0700359287d448cbee0e0aab34-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_DDL 集合下。

## Table API 中 Watermark 与 Window 的使用

打开类 ExecuteSqlWithWatermakerAndWindowMain

![1739-560-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/042069c9598290fb4feb8903dc14a470-0)

#### 使用描述器中定义一个使用 EventTime 和 Watermark

使用描述器定义一个使用 EventTime 和 Watermark 的 Source 表。

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00013.png](https://doc.shiyanlou.com/courses/1739/1207281/04b23470bd4cc9f33a3f08a703e24f1a-0)

将下列代码粘贴到 TODO code 1区间内。

```java
// Connection table via descriptor
tbEnv.connect(
   new Sdb()
    .version("3.4")									        // Version of sdb
    .hosts("localhost:11810")                               // Connection address of sdb
    .username("sdbadmin")                                   // Username
    .password("sdbadmin")                                   // Password
    .collectionSpace("VIRTUAL_BANK")                        // CollectionSpace
    .collection("TRANSACTION_FLOW")                         // Collection
    .timestampField("create_time")                          // Stream Timestamp field
).withFormat(
   new Bson()                           // Use Bson data format, when using rowtime, users must display the specified format
    .bsonSchema(                        // Bson serializer allows BsonFormat to be represented using a json string
        "{" +
            "account: 'string', " +
            "trans_name: 'string', " +
            "money: 'decimal', " +
            "create_time: 'timestamp'" +
        "}")
    .failOnMissingField()                       // Exception thrown when a field value cannot be obtained
).withSchema(
   new Schema()                                 // Define the structure of the table
    .field("account", Types.STRING)             // Account
    .field("trans_name", Types.STRING)          // Transaction name, for example: interest settlement, withdrawal, and etc.
    .field("money", Types.BIG_DEC)              // Transaction amount
    .field("create_time", Types.SQL_TIMESTAMP)  // Transaction time
    .field("rowtime", Types.SQL_TIMESTAMP)      // EventTime field
    .rowtime(
       new Rowtime()
        .timestampsFromField("create_time")     // Extract timestamp from field
        .watermarksPeriodicAscending()          // Set watermark generation rules
    )
).inAppendMode()                                
.registerTableSource("LESSON_6_SQL");
```

#### Flink SQL 中的函数

- TUMBLE_START()

  该函数表示获取翻滚窗口的开始时间，其中第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。如TUMBLE_START(rowtime, INTERVAL '5' SECOND)表示使用rowtime字段作为事件时间戳，获取窗口大小为5秒的翻滚窗口的窗口开始时间。此函数必须在GROUP BY TUMBLE(...) 才可以使用且TUMBLE_START函数的参数需要与TUMBLE函数参数完全一致。

- TUMBLE()

   窗口划分函数，表示使用翻滚窗口。第一个参数表示事件的时间戳字段，第二个参数表示窗口的大小，使用INTETVAL指定一个时间间隔。

- DATA_FORMAT() 

  该方法可以将时间戳格式化为固定格式的时间字符串。接收两个参数，第一个参数为一个Timestamp类型的字段名，为待转换的时间戳字段，第二个参数为格式化的字符串。

#### 编写 SQL

执行统计，统计每种交易的交易总额。

在当前类中找到 select 方法，找到 TODO code 2。

![1739-560-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/542a6ee56b6da51cb1736ecdedfd7b3a-0)

将下列代码粘贴到 TODO code 2区间内。

```java
// Execute sql data statistics
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

通过在当前类文件上右键 > Run 'ExecuteSqlWithWatermakerAndWindowMain.main()' 运行该 Flink 程序。

![1739-560-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/4896c1688098596aa7559ef4fc86b3d4-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_SQL 集合下。