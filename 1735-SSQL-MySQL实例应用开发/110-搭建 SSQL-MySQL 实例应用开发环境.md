---
show: step
version: 1.0 

---

## 课程介绍

本课程将带领您在已经部署 SequoiaDB 巨杉数据库引擎及创建了 MySQL 实例的环境中，熟悉并搭建SSQL-MySQL开发环境

#### 请点击右侧选择使用的实验环境

#### 部署架构：

本课程中 SequoiaDB 巨杉数据库的集群拓扑结构为三分区单副本，其中包括：1个 SequoiaSQL-MySQL 数据库实例节点、1个引擎协调节点，1个编目节点与3个数据节点。

![图片描述](https://doc.shiyanlou.com/courses/1469/1207281/8d88e6faed223a26fcdc66fa2ef8d3c5)

详细了解 SequoiaDB 巨杉数据库系统架构：

- [SequoiaDB 系统架构](http://doc.sequoiadb.com/cn/sequoiadb-cat_id-1519649201-edition_id-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎以及 SequoiaSQL-MySQL 实例均为 3.4 版本。

## 打开项目

#### 打开idea

打开idea代码开发工具

![1735-110-1.png](https://doc.shiyanlou.com/courses/1735/1207281/6f87a8c93937c3c51f6d4839559de710-0)

#### 打开SSQL-MySQL项目

打开SSQL-MySQL项目，在该课程中完成后续试验

![1735-110-13.png](https://doc.shiyanlou.com/courses/1735/1207281/40a9e7b6fbd5c3853dc09f69d0a06c86-0)

#### 打开lesson1_environmentBuilding包

打开lesson1_environmentBuilding packge，在该packge中完成后续课程。

![1735-110-2.png](https://doc.shiyanlou.com/courses/1735/1207281/f5ec2ca3949feed5c2a1c22262fa7619-0)

## 配置连接属性，执行SQL

#### 配置连接数据库的相关属性，访问MySQL数据库。

打开JdbcDEV.java类

![1735-110-3.png](https://doc.shiyanlou.com/courses/1735/1207281/1b614ee23c8c3d4d02a218eaf34a81ae-0)

修改TODO中的内容，配置连接信息，查询employee表

```java
String user = "root";
String password = "root";
String url = "jdbc:mysql://sdbserver1:3306/mysqlTest";
Connection conn = DriverManager.getConnection(url, user, password);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM employee");
boolean isHeaderPrint = false;
//遍历结果集
while (rs.next()) {
    //获得表结构
    ResultSetMetaData md = rs.getMetaData();
    //取得列数
    int col_num = md.getColumnCount();
    if (!isHeaderPrint){
        //遍历数据库字段名
        for (int i = 1; i  <= col_num; i++) {
            System.out.print(md.getColumnName(i) + "\t");
        }
        isHeaderPrint = true;
    }
    System.out.println();
    //遍历每一行查到得信息
    for (int i = 1; i <= col_num; i++) {
        System.out.print(rs.getString(i) + "\t");
    }
}
stmt.close();
conn.close();
```

![1735-110-14.png](https://doc.shiyanlou.com/courses/1735/1207281/6d6cc3c5a8e1111b4c7c6962e1c57de8-0)

#### 执行JdbcDEV

右键JdbcDEV.java，选择Run，执行JdbcDEV.java

![1735-110-5.png](https://doc.shiyanlou.com/courses/1735/1207281/1807a51b5815cc33e6a5c6c05f53bf85-0)

查看结果：

![1735-110-6.png](https://doc.shiyanlou.com/courses/1735/1207281/6bf4e7b063c2f2e01bda3bf5d938da79-0)

## 配置连接池

#### 什么是连接池？ 

数据库连接池（Database Connection Pooling）在程序初始化时创建一定数量的数据库连接对象并将其保存在一块内存区中，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个；释放空闲时间超过最大空闲时间的数据库连接以避免因为没有释放数据库连接而引起的数据库连接遗漏。

​     即在程序初始化的时候创建一定数量的数据库连接，用完可以放回去，下一个在接着用，通过配置连接池的参数来控制连接池中的初始连接数、最小连接、最大连接、最大空闲时间这些参数保证访问数据库的数量在一定可控制的范围类，防止系统崩溃，使用户的体验好。

#### 为什么要用连接池？

​    数据库连接是一种**关键、有限且昂贵的**资源，创建和释放数据库连接是一个很耗时的操作，频繁地进行这样的操作将占用大量的性能开销，进而导致网站的响应速度下降，严重的时候可能导致服务器崩溃；数据库连接池可以节省系统许多开销。

#### C3P0连接池

C3P0是一个开源的JDBC连接池，它实现了数据源与JNDI绑定，支持JDBC3规范和实现了JDBC2的标准扩展说明的Connection和Statement池的DataSources对象。

打开c3p0工具类UtilsC3P0，使用c3p0获得连接对象

创建一个静态ComboPooledDataSource对象，配置数据库连接信息

```java
private static ComboPooledDataSource dataSource=new ComboPooledDataSource();
```

在静态代码块中设置数据库连接信息

```java
//设置注册驱动
dataSource.setDriverClass("com.mysql.jdbc.Driver");
//url
dataSource.setJdbcUrl("jdbc:mysql://sdbserver1:3306/mysqlTest");
//数据库用户名
dataSource.setUser("root");
//数据库密码
dataSource.setPassword("root");
```

![1735-110-15.png](https://doc.shiyanlou.com/courses/1735/1207281/2f6f06795cf0e7a8201b3fc43facb2a1-0)

在方法getConnection()的TODO中，编写代码获取连接

```java
try {
    conn = dataSource.getConnection();
} catch (SQLException e) {
    throw new RuntimeException("数据库连接失败"+e);
}
```

![1735-110-19.png](https://doc.shiyanlou.com/courses/1735/1207281/f3f7fe2a595124566c42febd1f7231e4-0)

在close方法的TODO中，编写代码释放资源

```java
if (rs!=null){
    try {
        rs.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (state!=null){
    try {
        state.close();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
if (conn!=null){
    try {
        conn.close();//归还
    } catch (SQLException e) {
        e.printStackTrace();
    }
}
```

![1735-110-17.png](https://doc.shiyanlou.com/courses/1735/1207281/6a8b2ec00da48cdf43b28b895ea64ab6-0)

## 验证连接池

打开验证连接池的TestUtilsC3P0类

修改test01方法TODO中的内容

```java
// 使用c3p0工具类获得getConnection
Connection conn = UtilsC3P0.getConnection();
System.out.println(conn);
// 获得执行者对象
Statement state = conn.createStatement();
// 执行SQL语句
ResultSet rs = state.executeQuery("SELECT * FROM employee");
ResultSetMetaData metaData = rs.getMetaData();
int columnCount = metaData.getColumnCount();
while (rs.next()){
    for (int i = 1; i <= columnCount; i++) {
        System.out.print(rs.getString(i) + "\t");
    }
    System.out.println();
}
// 关闭资源
UtilsC3P0.close(rs,state,conn);
```

![1735-110-18.png](https://doc.shiyanlou.com/courses/1735/1207281/e4a37a121a343aa585c89cbacd89978f-0)

右键TestUtilsC3P0.java，选择Run，执行TestUtilsC3P0.java

![1735-110-7.png](https://doc.shiyanlou.com/courses/1735/1207281/2eecbffdfb70a5c0af788a86c26a629e-0)

查看结果

![1735-110-8.png](https://doc.shiyanlou.com/courses/1735/1207281/f4509b033025bf54cfb6f85831e89999-0)

## 使用常用函数

MySQL 有很多内置的函数,这里简单讲解三个函数（now、version、user）,更多的MySQL函数请前往第七章《常用函数》学习。

#### now() 返回当前的日期和时间。

打开FuncTest.java

修改TODO中内容为：

```java
String sql = "select now();";
stmt = conn.createStatement();
rs = stmt.executeQuery(sql);
while (rs.next()) {
    for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
        System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```



右键EnvBuildingMainTest.java，选择Edit 'EnvBuildingMain....main()'，修改参数为function

![1735-110-9.png](https://doc.shiyanlou.com/courses/1735/1207281/4f2e6e8dde86ee4694fc668ba569240d-0)

![1735-110-10.png](https://doc.shiyanlou.com/courses/1735/1207281/b84401ee488c773a4baa449b67b17977-0)

右键EnvBuildingMainTest.java，选择Run，运行代码

![1735-110-11.png](https://doc.shiyanlou.com/courses/1735/1207281/bca48948ed03e3e6abf5d55307ba2c1f-0)

查看结果

![1735-110-12.png](https://doc.shiyanlou.com/courses/1735/1207281/3d09511576c5cc29ad873cd970f3210f-0)

#### User() 返回用户信息

打开FuncTest.java

修改第8行内容为：

```java
String sql = "select user();";
```

#### Version() 返回当前数据库版本信息

打开FuncTest.java

修改第8行内容为：

```java
String sql = "select version();";
```

> 其余操作步骤同now（）一致

