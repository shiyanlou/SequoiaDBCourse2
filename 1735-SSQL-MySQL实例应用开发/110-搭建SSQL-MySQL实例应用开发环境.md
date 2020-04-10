# 1.1搭建SSQL-MySQL开发环境

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

## 搭建开发环境

用idea搭建基础开发环境，打开项目 SSQL-MySQL项目下的**com.sequoiadb.lesson.mysql.lesson1_environmentBuilding**包

导入项目需要的jar包 （mysql驱动包，sequoiadb驱动包）

> 环境中已经导入

在包下创建JdbcDEV.java类

## 配置连接属性

#### 打开JdbcDEV.java类

#### 在JdbcDEV类下编写连接属性。

在第9行，加载mysql驱动

```java
Class.forName("com.mysql.jdbc.Driver");
```

在第17~19行，配置mysql的url，username，password

```java
String user = "root";
String password = "root";
String url = "jdbc:mysql://sdb:3306/mysqlTest";
```

> sdb是主机名，mysqlTest是数据库名

在第23~25行，建立JDBC和数据库之间的Connection连接,创建Statement接口，执行SQL语句，查看表employee的数据

```java
Connection conn = DriverManager.getConnection(url, user, password);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM employee");
```

## 执行SQL，查看结果

#### 编写代码，遍历查询到的结果

在29~44行，写入如下代码,遍历rs的结果

```java
boolean isHeaderPrint = false;
while (rs.next()) {
	ResultSetMetaData md = rs.getMetaData();
	int col_num = md.getColumnCount();
	if (!isHeaderPrint){
		for (int i = 1; i  <= col_num; i++) {
			System.out.print(md.getColumnName(i) + "\t");
		}
		isHeaderPrint = true;
	}
	System.out.println();

	for (int i = 1; i <= col_num; i++) {
		System.out.print(rs.getString(i) + "\t");
	}
}
```

#### 执行jdbcDEV

单击第5行，左侧的三角，选择Run 'JdbcDEV.main()'，运行

​	![1586397234851](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586397234851.png)

查看结果：

![1586397286857](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586397286857.png)

## 配置连接池

在项目中导入c3p0的jar包和mchange-commons-java-0.2.15 jar包

> 项目中已导入

打开c3p0工具类UtilsC3P0，使用c3p0获得连接对象

在第9行，创建一个静态ComboPooledDataSource对象

```java
private static ComboPooledDataSource dataSource=new ComboPooledDataSource();
```

在第14~21行，在静态代码块中设置数据库连接信息

```java
//设置注册驱动
dataSource.setDriverClass("com.mysql.jdbc.Driver");
//url
dataSource.setJdbcUrl("jdbc:mysql://sdb:3306/mysqlTest");
//数据库用户名
dataSource.setUser("root");
//数据库密码
dataSource.setPassword("root");
```

在第28~34行，定义一个静态方法从ComboPooledDataSource对象中获得数据库连接Connection

```java
 public static Connection getConnection(){
        try {
            return dataSource.getConnection();
        } catch (SQLException e) {
            throw new RuntimeException("数据库连接失败"+e);
        }
 }
```

在37~59行，编写代码释放资源

```java
public static void close(ResultSet rs, Statement state, Connection conn){
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
}
```

## 验证连接池

打开验证连接池的TestUtilsC3P0类

在第11行，使用c3p0工具类获取connection连接

```java
Connection conn = UtilsC3P0.getConnection();
```

在第14行，获得执行者对象

```java
Statement state = conn.createStatement();
```

在第16行，执行SQL语句

```java
ResultSet rs = state.executeQuery("SELECT * FROM employee");
```

在第20~25行，遍历输出结果

```java
while (rs.next()){
	for (int i = 1; i <= columnCount; i++) {
		System.out.print(rs.getString(i) + "\t");
	}
	System.out.println();
}
```

在第27行，关闭资源

```java
UtilsC3P0.close(rs,state,conn);
```

单击第5行左侧的三角，执行

查看结果

![1586398064706](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586398064706.png)



## 使用常用函数

#### now()

打开FuncTest.java

修改第8行内容为：

```java
String sql = "select now();";
```

修改第10~17行，创建statement接口，执行sql，遍历结果

```java
stmt = conn.createStatement();
rs = stmt.executeQuery(sql);
while (rs.next()) {
	for (int i = 1; i <= rs.getMetaData().getColumnCount() ; i++) {
		System.out.print(rs.getString(i)+"\t");
    }
    System.out.println();
}
```

打开EnvBuildingMainTest.java，单击第6行的三角，选择Edit 'EnvBuildingMain....main()',修改参数为function

![1586398731067](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586398731067.png)

## ![1586398747050](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586398747050.png)

单击第6行的三角，选择Run 'EnvBuildingMain....main()

![1586398731067](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586398731067.png)

查看结果

![1586398835420](C:\Users\ChengYueyi\AppData\Roaming\Typora\typora-user-images\1586398835420.png)

#### User()

打开FuncTest.java

修改第8行内容为：

```java
String sql = "select user();";
```

#### Version

打开FuncTest.java

修改第8行内容为：

```java
String sql = "select version();";
```

