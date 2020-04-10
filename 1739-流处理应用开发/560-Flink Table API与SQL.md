

# Flink Table API 与 SQL

flink table是flink中的高级api, 其table api将大大降低开发flink程序的难度

#### 创建项目

- 请打开maven工程```flink-table```

- 认识依赖

  ```xml
  <!--flink 核心包-->
  <dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-core</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--flink 流开发包-->
  <dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--flink table常规包，包含接口、工具等 -->
  <dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-table-common</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--flink table 开发包-->
  <dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--flink sql planner-->
  <dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-table-planner_${scala.binary.version}</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--flink sequoiadb 连接驱动包-->
  <dependency>
      <groupId>com.chaoc.flink</groupId>
      <artifactId>flink-connector-sequoiadb-${sequoiadb.version}_${scala.binary.version}</artifactId>
      <version>${flink.version}</version>
  </dependency>
  <!--日志包-->
  <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-log4j12</artifactId>
      <version>1.7.7</version>
  </dependency>
  ```

  

#### Table的简介

- table是什么

  table是一个逻辑概念，其映射具体的DataStream或DataSet。可以通过sql操作table来达到操作具体的DataStream和DataSet。

- table环境

  table的使用需要依赖于table的执行环境，table的执行环境可以通过现有的流环境进行创建

- 如何创建一个表 ,仅讲述使用较多的三种方法。

  从现有的StreamData中转换表

  通过tableEnv 表描述器注册表

  通过DDL（建表语句）创建表

- TableDataStream模式

  在flink中，模式分为Append, Retract和Upsert，Append表示仅有查询。Retract和Upsert将带有修改，每条数据均增加了一个Boolean字段。在Retract模式中，当该字段为false时，表示该条数据需要被删除。

#### DataStream与表的转换

上文中提到，可以通过现有的StreamData作业转换为table，当然也可以通过table转换为流

- 找到类```FlinkTableFromDataStreamMain```

  现在已经有一个存在的DataStream，本例演示将DataStream转换为Table，经中间转换过程后将Table转换为DataStream，最后打印到控制台。

- 常见算子

  groupby： 分组

  select: 查询

  as: 重命名字段

  where: 过滤

- 向类中添加如下代码

  api 的使用与标准sql极为相似，课程将不再做过多讲解。

  ```java
  // 通过现有的DataStream创建table
  Table nameTable = tbEnv.fromDataStream(wordData, "name, num");
  // 重命名字段
  Table wordTable = nameTable.as("word, num");
  // where算子过滤
  Table whereTable = wordTable.where("word != 'java'");
  // 通过groupby 聚合
  Table groupTable = whereTable.groupBy("word").select("word, sum(num)");
  ```

- 当对table查询之后，向输出到控制台则需要将Table转换为DataStream

  要点一：在此处需要传入一个TypeInformation，描述一个具体Flink的对象类型，Flink会将Table中的记录封装为该对象，此处为Tuple2<String, Integer>类型。

  要点二：由于使用了groupby算子，返回时必须使用toRetractStream，使用toAppendStream将会抛出异常。

  ```java
  // 将table转换为DataStream
  DataStream<Tuple2<Boolean, Tuple2<String, Integer>>> stream = tbEnv.toRetractStream(groupTable, TypeInformation.of(new TypeHint<Tuple2<String, Integer>>() {
  }));
  // 写入到console
  stream.print();
  ```

#### 通过tableEnv 表描述器注册表

本例将通过描述器创建Source表与Sink表，实现从巨杉数据库读入数据，经Flink统计后实时写入结果到巨杉数据库。

- 找到类```FlinkTableConnectTableSourceMain```

- 通过描述器创建一个Source表

  ```java
  // 通过描述器连接表
  tbEnv.connect(
      new Sdb()
      // sdb 的连接地址
      .hosts("192.168.0.111:11810")
      // 用户名
      .username("sdbadmin")
      // 密码
      .password("sdbadmin")
      // 集合空间
      .collectionSpace("test")
      // 集合，储存的是巨杉虚机银行一天的交易流水
      .collection("test7")
      // 流标识字段名
      .timestampField("timestamp")
  ).withFormat(
      // 使用Bson数据格式
      new Bson()
      // 自动映射同名数据字段
      .deriveSchema()
      // 当获取不到某个字段值时任务失败
      .failOnMissingField()
  ).withSchema(
      // 定义table的结构
      new Schema()
      // 账户号
      .field("account", Types.STRING)
      // 交易名称，例如：结息，取款等
      .field("trans_name", Types.STRING)
      // 交易金额
      .field("money", Types.BIG_DEC)
      // 交易的时间
      .field("timestamp", Types.SQL_TIMESTAMP)
  ).inAppendMode()
  // 注册为一个数据源表
  .registerTableSource("test1");
  ```

- 通过描述器创建一个Sink表

  ```java
  tbEnv.connect(
      new Sdb()
      .hosts("192.168.0.111:11810")
      .username("sdbadmin")
      .password("sdbadmin")
      .collectionSpace("test")
      .collection("test8")
  ).withFormat(
      new Bson()
      .deriveSchema()
      .failOnMissingField()
  ).withSchema(
      new Schema()
      // 交易总额
      .field("sum", Types.BIG_DEC)
      // 交易名称
      .field("trans_name", Types.STRING)
  ).inUpsertMode()
  // 通过Upsert模式注册为一个Sink表,该模式支持聚合操作
  .registerTableSink("test2");
  ```

- 执行数据统计，并将结果输出到巨杉数据库，统计每种交易的交易总额

  ```java
  // 执行sql 数据统计
  tbEnv.sqlUpdate("insert into test2 select sum(money) as `sum`, trans_name from test1 group by trans_name");
  ```

- 可以通过SAC页面查看数据结果

#### 通过DDL创建表

与上述需求一致，通过DDL实现。

- 找到类```FlinkTableDDLMain```

- 创建source表

  ```java
  tbEnv.sqlUpdate("CREATE TABLE test1 (" +
                  // 账户号
                  "  account STRING, " +
                  // 交易名称
                  "  trans_name STRING, " +
                  // 交易金额
                  "  money DECIMAL(10, 2), " +
                  // 交易世家
                  "  `timestamp` TIMESTAMP(3)" +
                  ") WITH (" +
                  // 连接介质类型
                  "  'connector.type' = 'sequoiadb', " +
                  // 连接介质版本
                  "  'connector.version' = '3.2.4', " +
                  // 连接地址
                  "  'connector.hosts' = '192.168.0.111:11810', " +
                  // 用户名
                  "  'connector.username' = 'sdbadmin', " +
                  // 密码
                  "  'connector.password' = 'sdbadmin', " +
                  // 集合空间名
                  "  'connector.collection-space' = 'test', " +
                  // 集合名
                  "  'connector.collection' = 'test7', " +
                  // 流标识字段
                  "  'connector.timestamp-field' = 'timestamp', " +
                  // 数据类型 bson
                  "  'format.type' = 'bson', " +
                  // 自动映射同名字段
                  "  'format.derive-schema' = 'true', " +
                  // 当某个字段获取不到时任务失败
                  "  'format.fail-on-missing-field' = 'true', " +
                  // append模式
                  "  'update-mode' = 'append'" +
                  ")");
  ```

- 创建sink表

  ```java
  tbEnv.sqlUpdate("CREATE TABLE test2(" +
                  // 交易名称
                  "  trans_name STRING, " +
                  // 交易总额
                  "  `sum` DECIMAL(10, 2)" +
                  ") WITH (" +
                  "  'connector.type' = 'sequoiadb', " +
                  "  'connector.version' = '3.2.4', " +
                  "  'connector.hosts' = '192.168.0.111:11810', " +
                  "  'connector.username' = 'sdbadmin', " +
                  "  'connector.password' = 'sdbadmin', " +
                  "  'connector.collection-space' = 'test', " +
                  "  'connector.collection' = 'test8', " +
                  "  'format.type' = 'bson', " +
                  "  'format.derive-schema' = 'true', " +
                  "  'format.fail-on-missing-field' = 'true', " +
                  // upsert模式，该模式执行聚合语句
                  "  'update-mode' = 'upsert'" +
                  ")");
  ```

- 执行统计，统计每种交易的交易总额

  ```java
  tbEnv.sqlUpdate("insert into test2 select trans_name, sum(money) as `sum` from test1 group by trans_name");
  ```

  

- 可以通过SAC页面查看数据结果

#### TableAPI中Watermark与Window的使用

- 找到类```FlinkTableWithWatermakerAndWindowExample```

  ```java
// 通过描述器连接表
tbEnv.connect(
    new Sdb()
    // sdb 的连接地址
    .hosts("192.168.0.111:11810")
    // 用户名
    .username("sdbadmin")
    // 密码
    .password("sdbadmin")
    // 集合空间
    .collectionSpace("test")
    // 集合
    .collection("test7")
    // 流标识字段名
    .timestampField("timestamp")
).withFormat(
    // 使用Bson数据格式, 当使用rowtime时必须显示指定format
    new Bson()
    .bsonSchema("{" +
                "account: 'string', " +
                "trans_name: 'string', " +
                "money: 'decimal', " +
                "timestamp: 'timestamp' " +
                "}")
    // 当获取不到某个字段值时任务失败
    .failOnMissingField()
).withSchema(
    // 定义table的结构
    new Schema()
    // 账户号
    .field("account", Types.STRING)
    // 交易名称，例如：结息，取款等
    .field("trans_name", Types.STRING)
    // 交易金额
    .field("money", Types.BIG_DEC)
    // 交易的时间
    .field("timestamp", Types.SQL_TIMESTAMP)
    .rowtime(
        new Rowtime()
        // 从字段中提取
        .timestampsFromField("timestamp")
        // 每条数据均提取
        .watermarksPeriodicAscending()
    )
).inAppendMode()
// 注册为一个数据源表
.registerTableSource("test1");
  ```

- 执行统计，统计每种交易的交易总额

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
