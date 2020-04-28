---
show: step
version: 1.0 


---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，熟悉并搭建 MySQL 开发环境

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 MySQL 实例均为 3.4 版本。

## 打开项目

#### 打开 idea

打开 idea 代码开发工具

![1735-110-1.png](https://doc.shiyanlou.com/courses/1735/1207281/6f87a8c93937c3c51f6d4839559de710-0)

#### 打开 scdd-mysql 项目

打开 scdd-mysql 项目，在该课程中完成后续试验

![1587923287299](https://doc.shiyanlou.com/courses/1735/1207281/2e66fe621bc8196ead5a7141c8125db4-0)

#### 打开 lesson1_environmentBuilding 包

打开 lesson1_environmentBuilding packge，在该 packge 中完成后续课程

![1587923393235](https://doc.shiyanlou.com/courses/1735/1207281/69545b5f00569e8ced7e73c1f028af35-0)

## 配置JDBC连接属性

在 idea 操作数据库，可以通过 JDBC 配置相关连接属性连接 MySQL 数据库。  

1）打开 JdbcDEV.java 类

![1735-110-120.png](https://doc.shiyanlou.com/courses/1735/1207281/c09afa774d77128ecb1c65329a610e5d-0)

2）在 run 方法中找到 TODO code 1

![1735-110-100.png](https://doc.shiyanlou.com/courses/1735/1207281/8e430ac3fd55a63cc6e22c95f8a81fc3-0)

3）将下方代码粘贴到 TODO code 1 区域内，使用 JDBC 配置连接信息，查询 employee 表

```java
// MySQL username
String user = "root";
// MySQL password
String password = "root";
// MySQL connection address
String url = "jdbc:mysql://sdbserver1:3306/mysqlTest?useSSL=false";
// Get connection object conn through configuration
Connection conn = DriverManager.getConnection(url, user, password);
// Create a Statement object to send SQL statements to the database
Statement stmt = conn.createStatement();
// Get the result set rs
ResultSet rs = stmt.executeQuery("SELECT * FROM employee");
boolean isHeaderPrint = false;
// Traverse the result set
while (rs.next()) {
    // Get the table structure
    ResultSetMetaData md = rs.getMetaData();
    // Get the number of rows
    int col_num = md.getColumnCount();
    if (!isHeaderPrint){
        // Traverse the database field names
        for (int i = 1; i  <= col_num; i++) {
            System.out.print(md.getColumnName(i) + "\t");
        }
        isHeaderPrint = true;
    }
    System.out.println();
    // Traverse each line to find the information
    for (int i = 1; i <= col_num; i++) {
        System.out.print(rs.getString(i) + "\t");
    }
}
// Close stmt and conn
stmt.close();
conn.close();
```

代码粘贴结果如图所示：

![1735-110-1000.png](https://doc.shiyanlou.com/courses/1735/1207281/8948471d7b73447a8ea2f8cd55dc3bee-0)

> **说明**
>
> 粘贴方法如下：
>
> - 点击代码框右上角的 copy 图标
> - 选择实验界面右边的 “剪切板”
>
> ![paste1](https://doc.shiyanlou.com/courses/1738/1207281/7745e7378b70a60ad6073262f05762ec-0)
>
> - 在弹出的“在线环境剪切板”中粘贴复制的代码内容
>
> ![paste2](https://doc.shiyanlou.com/courses/1738/1207281/6b477101feb04b1db73e8f893ba3b334-0)
>
> - 在实验环境中到对应的位置粘贴
>
> ![paste3](https://doc.shiyanlou.com/courses/1738/1207281/14482e482cde033e4f78cca144abdcee-0)

4）右键 EnvBuildingMainTest.java，选择 Edit 'EnvBuildingMai....main()'

![1735-110-118.png](https://doc.shiyanlou.com/courses/1735/1207281/d2a34472c4e021038d5f7443bcd368e3-0)

5）修改参数为 jdbcDEV 

![1735-110-102.png](https://doc.shiyanlou.com/courses/1735/1207281/b4a1b171602f99831497de1b8a562e8e-0)

6）再次右键 EnvBuildingMainTest.java 选择 Run 'EnvBuildingMai....main()' ，执行代码

![1735-110-119.png](https://doc.shiyanlou.com/courses/1735/1207281/ce564b103119e82ba8f6f1673a33c10b-0)

7）查看结果：

![1735-110-6.png](https://doc.shiyanlou.com/courses/1735/1207281/6bf4e7b063c2f2e01bda3bf5d938da79-0)

## 配置连接池

#### 什么是连接池？ 

数据库连接池（Database Connection Pooling）在程序初始化时创建一定数量的数据库连接对象并将其保存在一块内存区中，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个；释放空闲时间超过最大空闲时间的数据库连接以避免因为没有释放数据库连接而引起的数据库连接遗漏。

即在程序初始化的时候创建一定数量的数据库连接，用完归还，下一个继续使用。可以通过配置连接池的参数来控制连接池中的初始连接数、最小连接、最大连接、最大空闲时间。这些参数保证了访问数据库的数量在一定可控制的范围内，防止系统崩溃，使用户的体验好。

#### 为什么要用连接池？

数据库连接是一种**关键、有限且昂贵的**资源，创建和释放数据库连接是一个很耗时的操作，频繁地进行这样的操作将占用大量的性能开销，进而导致网站的响应速度下降，严重的时候可能导致服务器崩溃；数据库连接池可以节省系统许多开销。

#### C3P0连接池

C3P0 是一个开源的 JDBC 连接池，它实现了数据源与 JNDI 绑定，支持 JDBC3 规范和实现了 JDBC2 的标准扩展说明的 Connection 和 Statement 池的 DataSources 对象。

1）打开 C3P0 工具类 UtilsC3P0

![1735-110-121.png](https://doc.shiyanlou.com/courses/1735/1207281/d2fee49ff1d687da9f21880d168b77dc-0)

2）在 UtilsC3P0类的最下方 找到 TODO code 1

![1735-110-104.png](https://doc.shiyanlou.com/courses/1735/1207281/1aafd4dfca2c3905371268a0446086d5-0)

3）将下方代码粘贴到 TODO code 1 区域内，创建 C3P0 的 ComboPooledDataSource 对象，配置数据库连接信息

```java
private static ComboPooledDataSource dataSource=new ComboPooledDataSource();

static {
    try {
        // Set up the registered driver
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        //url
        dataSource.setJdbcUrl("jdbc:mysql://sdbserver1:3306/mysqlTest");
        // Database username
        dataSource.setUser("root");
        // Database password
        dataSource.setPassword("root");
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

代码粘贴结果如图所示：

![1735-110-1001.png](https://doc.shiyanlou.com/courses/1735/1207281/a9dd99f66876223bebecee5676e184ca-0)

4）在 getConnection 方法中找到 TODO code 2

![1735-110-106.png](https://doc.shiyanlou.com/courses/1735/1207281/38c00217fc3706f1cdbdc64ed2457488-0)

5）将下方代码粘贴到 TODO code 2 区域内，获取数据库连接

```java
try {
    // Get connection
    conn = dataSource.getConnection();
} catch (SQLException e) {
    throw new RuntimeException("database conn error"+e);
}
```

代码粘贴结果如图所示：

![1735-110-1002.png](https://doc.shiyanlou.com/courses/1735/1207281/f246b674640e20536714c3ee9071d28a-0)

6）在 close 方法中找到 TODO code 3

![1735-110-108.png](https://doc.shiyanlou.com/courses/1735/1207281/f25ec01797a964d324019b3afe31d1c7-0)

7）将下方代码粘贴到 TODO code 3 区域内，释放资源

```java
if (rs!=null){
    try {
        rs.close();// Return rs
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (stmt!=null){
    try {
        stmt.close();//Return stmt
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (conn!=null){
    try {
        conn.close();//Return conn
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

代码粘贴结果如图所示：

![1735-110-1003.png](https://doc.shiyanlou.com/courses/1735/1207281/c32d912a1db8c4bf64ca48e570067007-0)

## 验证连接池

1）打开验证连接池的 TestUtilsC3P0 类

![1735-110-122.png](https://doc.shiyanlou.com/courses/1735/1207281/3faf52569f0216bceb47dfbc8a24c7d8-0)

2）在 run 方法中找到 TODO code 1

![1735-110-111.png](https://doc.shiyanlou.com/courses/1735/1207281/7bbf5a2d1f1625a1e276ad9600bd584d-0)

3）将下方代码粘贴到 TODO code 1 区域内，验证连接池，查询 employee 表信息

```java
Connection conn = null;
Statement stmt = null;
ResultSet rs = null;
try {
    // 使用 C3P0 工具类获得 conn
    conn = UtilsC3P0.getConnection();
    System.out.println(conn);
    // 获得执行者对象
    stmt = conn.createStatement();
    // 执行 SQL 语句
    rs = stmt.executeQuery("SELECT * FROM employee");
    // 获取表结构
    ResultSetMetaData metaData = rs.getMetaData();
    // 获取列数
    int columnCount = metaData.getColumnCount();
    // 遍历结果集
    while (rs.next()){
        // 遍历 employee 表的内容
        for (int i = 1; i <= columnCount; i++) {
            System.out.print(rs.getString(i) + "\t");
        }
        System.out.println();
    }
} catch (Exception e) {
    e.printStackTrace();
} finally {
    //关闭资源
    UtilsC3P0.close(rs, stmt, conn);
}
```

![1735-110-117.png](https://doc.shiyanlou.com/courses/1735/1207281/85dc747df7bd975a695d7cafe077d01a-0)

4）执行代码，右键 EnvBuildingMainTest.java，选择 Edit 'EnvBuildingMai....main()'

![1735-110-118.png](https://doc.shiyanlou.com/courses/1735/1207281/d2a34472c4e021038d5f7443bcd368e3-0)

5）修改参数为 testUtilsC3P0

![1735-110-113.png](https://doc.shiyanlou.com/courses/1735/1207281/1c9c7d8d7a4987681bd78aa903700e12-0)

6）右键 EnvBuildingMainTest.java，选择 Run 'EnvBuildingMai....main()'，运行代码

![1735-110-119.png](https://doc.shiyanlou.com/courses/1735/1207281/ce564b103119e82ba8f6f1673a33c10b-0)

7）查看结果

![1735-110-8.png](https://doc.shiyanlou.com/courses/1735/1207281/f4509b033025bf54cfb6f85831e89999-0)

## 使用常用函数

MySQL 有很多实用的内置函数，这里简单举例 NOW 函数讲解，更多的MySQL函数请前往第七章《常用函数》学习。

#### NOW()

返回当前的日期和时间。

1）打开 FuncTest.java

![1735-110-123.png](https://doc.shiyanlou.com/courses/1735/1207281/21827351cc36fce5d34e1f7fc4ad5c4e-0)

2）在 run 方法中找到 TODO code 1

![1735-110-115.png](https://doc.shiyanlou.com/courses/1735/1207281/29565eea603d7d55dac7bc8c952a11da-0)

3）将下方代码粘贴到 TODO code 1 区域内，使用 now 函数获取当前日期时间

```java
// Write a SQL statement to get the current date and time
String sql = "SELECT NOW();";
// Create a Statement object to send SQL statements to the database
stmt = conn.createStatement();
// Get the result set
rs = stmt.executeQuery(sql);
// Traverse the result set
while (rs.next()) {
    // Traverse to get the current date and time
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

代码粘贴结果如图所示：

![1735-110-1004.png](https://doc.shiyanlou.com/courses/1735/1207281/715005f94c57110715b64787c17ec474-0)

4）修改参数，右键 EnvBuildingMainTest.java，选择Edit 'EnvBuildingMain....main()'

![1735-110-118.png](https://doc.shiyanlou.com/courses/1735/1207281/d2a34472c4e021038d5f7443bcd368e3-0)

5）修改参数为 function

![1735-110-10.png](https://doc.shiyanlou.com/courses/1735/1207281/b84401ee488c773a4baa449b67b17977-0)

6）执行代码，右键 EnvBuildingMainTest.java，选择 Run 'EnvBuildingMai....main()'，运行代码

![1735-110-119.png](https://doc.shiyanlou.com/courses/1735/1207281/ce564b103119e82ba8f6f1673a33c10b-0)

7）查看结果

![1735-110-12.png](https://doc.shiyanlou.com/courses/1735/1207281/3d09511576c5cc29ad873cd970f3210f-0)

## 总结

本课程讲述了 MySQL 开发环境的搭建：配置 JDBC 连接、配置连接池，以及间要概述了MySQL的常用函数。

