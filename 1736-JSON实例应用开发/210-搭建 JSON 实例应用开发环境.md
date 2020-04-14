---
show: step
version: 1.0

---

## 课程介绍

本课程将介绍在 Java 开发环境下如何连接 SequoiaDB 数据库的 JSON 实例。

对于所有数据库操作，都要先创建数据库链接。关于数据库链接的作用，可以形象地理解为一根网线，用来连接到远程服务器，有了这个链接，然后才能访问服务器的资源。创建完数据库链接，意味着建立了一个通道，然后就可以开始数据库操作了。

创建数据库链接需要鉴权，鉴权时需要写明数据库访问地址、用户名和密码。如果不存在鉴权，使得任何用户都可以访问数据库资源，就会存在安全隐患。

#### JSON 开发简介

SequoiaDB 巨杉数据库为应用提供通过 SDK 驱动进行数据库操作和集群操作的接口。

#### 实验流程简述

- 用户通过 IDEA 编辑器编写 Java 源码
- 实验相关核心代码，可从文档中的代码示例粘贴到项目指定文件的 TODO 标记处
- 通过编译、运行 Java 代码，操作 SequoiaDB 数据库 JSON 实例

![](https://doc.shiyanlou.com/courses/1736/1207281/7b1731fc121e3b460dcd9841eb0218a6-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本。IDEA 编辑器为 16.0 版本。JDK 为 1.8 版本。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![](https://doc.shiyanlou.com/courses/1736/1207281/06650396616c742995bb63fcf933fac5-0)

#### 打开项目

打开指定项目，在该项目中完成所有实验步骤。

![](https://doc.shiyanlou.com/courses/1736/1207281/9f17386c8098e8f4e46634f208fcd36b-0)

#### 检查 POM 依赖

关于 JSON 实例开发的实验，会用到 fastjson-1.2.67.jar 和 sequoiadb-driver-3.4.jar 两个 jar 包依赖。其中 sequoiadb-driver 为 Java 驱动包，属于必要依赖，fastjson 为市面常见 JSON 字符串格式化工具类，仅为了实验中调试方便，添加的非必要依赖。实验中依赖的形式采用 Maven 本地依赖的方式，将依赖包添加到 POM 文件中去。实验初始环境已经修改 POM 文件完毕，用户核实即可。

POM 文件位置：

![](https://doc.shiyanlou.com/courses/1736/1207281/30b1f7ba3794815a0498e9a4c9f74760-0)

具体 POM 内容如下：

![](https://doc.shiyanlou.com/courses/1736/1207281/d90499be09e608cfdf801e7d9e9522a5-0)

#### 打开 Package

打开指定的 Package，在该 Package 中完成后续课程。

![](https://doc.shiyanlou.com/courses/1736/1207281/fe6a1ec2f1298f67b6eda21bcebf8bad-0)

## 通过 Java SDK 进行数据库连接

#### 概述

使用 Java SDK API 对 SequoiaDB 数据库进行操作，需要先创建数据库链接。

用户可以通过创建 com.sequoiadb.base.Sequoiadb 对象，创建一个数据库链接。创建链接的过程中，需要指定连接目标的 IP、端口号、用户名和密码。

为了便于用户理解，接下来演示如何创建数据库链接，并在创建完链接后，通过链接获取数据库的数据组信息，测试链接的连通性。

#### 操作步骤

打开指定的 Java 源文件。

![](https://doc.shiyanlou.com/courses/1736/1207281/812e1a2eddc2096ba4df5fc491986425-0)

复制以下代码样例。

```java
// 创建数据库链接
Sequoiadb db = new Sequoiadb("sdbserver1", 11810,
                             "sdbadmin", "sdbadmin");
try {
    // 获取数据组信息
    DBCursor rgs = db.listReplicaGroups();
    System.out.println("连接数据库后查询到的数据组信息为：");
    // 格式化打印数据组信息
    JsonUtil.formatPrint(rgs);
    // 关闭cursor结果集
    rgs.close();
} finally {
    // 关闭数据库链接
    db.close();
}
```

将代码样例粘贴到第 17 行的 TODO ~ TODO END 位置区间内。

![// TODO 贴图](https://doc.shiyanlou.com/courses/1736/1207281/210c37dfeecbdcc1f58bf4f3858aaf8b-0)

右键点击 BaseMainTest 类，创建/编辑主类运行环境。

![](https://doc.shiyanlou.com/courses/1736/1207281/83df5b68653c29e9b5ad072f3d796319-0)

配置 Configuration 页中的 Program arguments 选项，写入指定的参数，点击”OK“。

![// TODO 贴图](https://doc.shiyanlou.com/courses/1736/1207281/92654200ea5f6c60ba2675e471281325-0) 

右键点击 BaseMainTest 类，运行主程序。

![](https://doc.shiyanlou.com/courses/1736/1207281/3379a6374114e3d5d99f54681797e281-0)

在 IDEA 输出窗口查看运行效果。

![// TODO 贴图](https://doc.shiyanlou.com/courses/1736/1207281/4f5f062cc4adaf1d4fc970936a6ca054-0)

双击放大控制台输出窗口，查看详细输出信息。

![](https://doc.shiyanlou.com/courses/1736/1207281/9cb655d55310e713fcbb089d4763b8bc-0)

通过输出信息，可以看到成功获取了数据组的详细信息，证明了数据库的连通性。

## 总结

本课程介绍了在 Java 开发环境下如何连接 SequoiaDB 数据库的 JSON 实例。需要注意的是，无论通过何种语言操作数据库，创建数据库链接都是必要前提，Java 语言也不例外。成功创建数据库链接后，用户就可以进行数据库的详细操作了。有了数据库链接，用户可以操作集合空间或者集合，也可以对数据进行查询、插入、删除，或者是对数据库进行系统信息查询，配置。总之，数据库链接是所有操作的前提。
