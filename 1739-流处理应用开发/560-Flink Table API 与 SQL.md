---
show: step
version: 1.0
---

## 课程介绍

本实验将带领您了解与学习 Flink Table API 与 Flink SQL。

Flink Table 是 Flink 中的高级 API, Table API 将大大降低开发Flink程序的难度。本实验将使用 Flink Table API 与 Flink SQL 来实现流作业的逻辑。

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1 个 Flink 节点、1 个引擎协调节点，1 个编目节点与 3 个数据节点。

![1739-510-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/a8fa9ed16eda4d9d3ef1f521c7dabdeb-0)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本，Flink版本为 1.9.2。

本实验中使用了 flink-connect-sequoiadb 依赖（Flink 连接 SequoiaDB 驱动包），该依赖来自巨杉开源社区。

* [下载地址](https://github.com/chaochaoc/flink-connector-sequoiadb/)

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![1739-510-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/c5a12bc733b440ce265298eb3cc4a715-0)

#### 打开 scdd-flink 项目
打开 scdd-flink 项目，在该课程中完成本试验。

![1739-510-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/2b68951cb04a44566d0a7219ede54005-0)

#### 打开 lesson6 packge
打开 com.sequoiadb.lesson.flink.lesson6_table ，在该 package 中完成本课程。

![1739-560-00001.png](https://doc.shiyanlou.com/courses/1739/1207281/d9ac2d8b7f74f7fed908551c04e4ef6d-0)

#### 认识依赖

打开 pom.xml 文件，认识依赖。

![1739-520-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/9b4833b8e0bc2160d90625911973ed4b-0)

本案例新增了 Flink Table 的驱动包。

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

![1739-560-00003.png](https://doc.shiyanlou.com/courses/1739/1207281/d7b32cd9daaeb7de0135c3301909c1bc-0)

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

![1739-560-00008.png](https://doc.shiyanlou.com/courses/1739/1207281/cfc89fac14d9e8f6f14c14756ac88021-0)

将下列代码粘贴到 TODO code 1区间内。

```java
table = tbEnv.fromDataStream(wordData, "name, num");
```

#### SQL 算子的使用

SQL 算子的用途与标准sql中关键字一致。

在当前类中找到 select 方法，找到 TODO code 2。

![1739-560-00009.png](https://doc.shiyanlou.com/courses/1739/1207281/b4677408557dd40313997bf67b6ad933-0)

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

![1739-560-00010.png](https://doc.shiyanlou.com/courses/1739/1207281/9f323afc171e9e56a0cd255b658ac41a-0)

将下列代码粘贴到 TODO code 3区间内。

```java
dataStream = tbEnv.toRetractStream(table, TypeInformation.of(
    new TypeHint<Tuple2<String, Integer>>() {}));
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableFromDataStreamMain.main()' 运行该 Flink 程序。

![1739-560-00011.png](https://doc.shiyanlou.com/courses/1739/1207281/6bbbdd54be8487835091af979f4a7322-0)

查看结果。

![1739-560-00012.png](https://doc.shiyanlou.com/courses/1739/1207281/55a4b46011b5ebfcf0facdda51edeee7-0)

## 通过表描述器注册表

本例将通过描述器创建 Source 表与 Sink 表，实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

在当前包下，打开类 CreateTableByConnectTableSourceMain 

![1739-560-00004.png](https://doc.shiyanlou.com/courses/1739/1207281/12826533fd38450196b4c0179e24fdbf-0)

#### 通过描述器创建一个 Source 表

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/cfd849d76dff56c152a418a8325a671a-0)

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

![1739-560-00014.png](https://doc.shiyanlou.com/courses/1739/1207281/2e8504672439a12fecaf8a46c91439d2-0)

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

![1739-560-00015.png](https://doc.shiyanlou.com/courses/1739/1207281/8e7324917ce37f3921d500e485d0ecc3-0)

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

![1739-560-00005.png](https://doc.shiyanlou.com/courses/1739/1207281/71c5938a1ecf6268a8c97703ee3660fe-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_CONNECT 集合下。

通过浏览器打开 localhost:8000 进入SequoiaDB SAC管理界面。

![1739-540-00049.png](https://doc.shiyanlou.com/courses/1739/1207281/b4c3578fcb61d5b65d87b2fc084f7a05-0)

点击数据菜单选择 "SequoiaDB" 分布式存储。

![1739-540-00050.png](https://doc.shiyanlou.com/courses/1739/1207281/4e240fc768dd2c562e1f1ad7c5e68600-0)

选择集合选项卡， 在搜索栏输入集合空间名 "VIRTUAL_BANK" ，查找该集合空间下的所有集合。

![1739-560-00023.png](https://doc.shiyanlou.com/courses/1739/1207281/92c5204482abf40ee31401742534cffc-0)

选中集合 "VIRTUAL_BANK.LESSON_6_CONNECT" 点击右侧的 "浏览数据"，可以看到当前集合中的所有数据。

![1739-560-00024.png](https://doc.shiyanlou.com/courses/1739/1207281/ace404bcb5c1f55401ad8898e3cba7ea-0)

## 通过 DDL 创建表

本案例通过 DDL 创建 Flink 流表。实现从巨杉数据库读入数据，经 Flink 统计后实时写入结果到巨杉数据库。统计一个交易流水表中的总交易额。

#### 打开类

打开类 CreateTableByDDLMain

![1739-560-00006.png](https://doc.shiyanlou.com/courses/1739/1207281/71c5938a1ecf6268a8c97703ee3660fe-0)

#### 创建 Source 表

通过 DDL 创建 Flink Source 表。

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00019.png](https://doc.shiyanlou.com/courses/1739/1207281/c968ddd8227ffe9eecd70a65a24e0c0d-0)

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

![1739-560-00020.png](https://doc.shiyanlou.com/courses/1739/1207281/371431aad08d2f54c49c301f9fd10359-0)

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

![1739-560-00021.png](https://doc.shiyanlou.com/courses/1739/1207281/2c849f8a89a7c3d5375543d7df2355e4-0)

将下列代码粘贴到 TODO code 3区间内。

```java
 tbEnv.sqlUpdate(
     "INSERT INTO LESSON_6_DDL " +
     "SELECT " +
         "trans_name, " +
         "SUM(money) AS `total_sum` " +
     "FROM TRANSACTION_FLOW " +
     "GROUP BY " +
     	"trans_name");
```

#### 执行当前作业

通过在当前类文件上右键 > Run 'CreateTableByDDLMain.main()' 运行该 Flink 程序。

![1739-560-00017.png](https://doc.shiyanlou.com/courses/1739/1207281/972eb681725f8c68a894c6a6b937f740-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_DDL 集合下。

通过浏览器打开 localhost:8000 进入SequoiaDB SAC管理界面。

![1739-540-00049.png](https://doc.shiyanlou.com/courses/1739/1207281/b4c3578fcb61d5b65d87b2fc084f7a05-0)

点击数据菜单选择 "SequoiaDB" 分布式存储。

![1739-540-00050.png](https://doc.shiyanlou.com/courses/1739/1207281/4e240fc768dd2c562e1f1ad7c5e68600-0)

选择集合选项卡， 在搜索栏输入集合空间名 "VIRTUAL_BANK" ，查找该集合空间下的所有集合。

![1739-560-00025.png](https://doc.shiyanlou.com/courses/1739/1207281/24aa8965ff2e2f7cf7b1861c86f5f8fe-0)

选中集合 "VIRTUAL_BANK.LESSON_6_DDL" 点击右侧的 "浏览数据"，可以看到当前集合中的所有数据。

![1739-560-00026.png](https://doc.shiyanlou.com/courses/1739/1207281/28767f91758cddabc6b25a097a763076-0)

## Table API 中 Watermark 与 Window 的使用

打开类 ExecuteSqlWithWatermakerAndWindowMain

![1739-560-00007.png](https://doc.shiyanlou.com/courses/1739/1207281/6b45ef42369e2125919eddb6168b47b2-0)

#### 使用描述器中定义一个使用 EventTime 和 Watermark

使用描述器定义一个使用 EventTime 和 Watermark 的 Source 表。

在当前类中找到 createSourceTable 方法，找到 TODO code 1。

![1739-560-00022.png](https://doc.shiyanlou.com/courses/1739/1207281/e5e8fee33b633dec50236b09dd8edec9-0)

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
.registerTableSource("TRANSACTION_FLOW");
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

![1739-560-00016.png](https://doc.shiyanlou.com/courses/1739/1207281/25bbe70a395a238ba5f7a2f960cdffce-0)

将下列代码粘贴到 TODO code 2区间内。

```java
// Execute sql data statistics
tbEnv.sqlUpdate(
    "INSERT INTO LESSON_6_SQL ( " +
    "SELECT " +
        "trans_name, " +
        "SUM(money) AS total_sum, " +
    	"TUMBLE_END(`rowtime`, INTERVAL '5' SECOND) as `timestamp`, " +
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

![1739-560-00018.png](https://doc.shiyanlou.com/courses/1739/1207281/be8fc8acb0cfe1e6a89d93a7444eb0a3-0)

通过 SAC 查看结果数据，结果在 VIRTUAL_BANK.LESSON_6_SQL 集合下。

通过浏览器打开 localhost:8000 进入SequoiaDB SAC管理界面。

![1739-540-00049.png](https://doc.shiyanlou.com/courses/1739/1207281/b4c3578fcb61d5b65d87b2fc084f7a05-0)

点击数据菜单选择 "SequoiaDB" 分布式存储。

![1739-540-00050.png](https://doc.shiyanlou.com/courses/1739/1207281/4e240fc768dd2c562e1f1ad7c5e68600-0)

选择集合选项卡， 在搜索栏输入集合空间名 "VIRTUAL_BANK" ，查找该集合空间下的所有集合。

![1739-560-00027.png](https://doc.shiyanlou.com/courses/1739/1207281/916fff7511e8486026d51f0ad1829fab-0)

选中集合 "VIRTUAL_BANK.LESSON_6_SQL" 点击右侧的 "浏览数据"，可以看到当前集合中的所有数据。

![1739-560-00028.png](https://doc.shiyanlou.com/courses/1739/1207281/ff8ef65a1b5fd01ff7763ad009be3d4f-0)

## 总结

本小节是对 Flink Table 和 Flink SQL的学习，学习如何从现有的 DataStream 中创建 Table，如何通过描述器注册 Table，通过 DDL 注册 Table，以及如何在 Flink Table API 中使用 Watermark。

**知识点**

- Flink Table API  的了解
- Flink Table 常见的三种创建方式
- Flink SQL 中的常见函数及语法的使用
- Flink Table API 中Watermark 的使用