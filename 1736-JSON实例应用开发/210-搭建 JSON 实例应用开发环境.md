---
show: step
version: 1.0
---

## 课程介绍

本课程将介绍在 Java 开发环境下如何连接 Sequoiadb 数据库的 JSON 实例。

#### JSON 开发简介

SequoiaDB 巨杉数据库为应用提供通过 SDK 驱动进行数据库操作和集群操作的接口。

#### 实验流程简述：

- 用户通过 IDEA 编辑器编写 Java 源码
- 实验相关核心代码，可从文档中的代码示例粘贴到项目指定文件的 TODO 标记处
- 通过编译、运行 Java 代码，操作 SequoiaDB 数据库 JSON 实例

![](C:\Users\SequoiaDB\Desktop\开发者课程\drawing\net.png)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本。IDE 编辑器为 idea。JDK 为 1.8 版本。

## 打开环境

#### 进入操作系统图形界面

输入密码，进入桌面

>Note:
>
>用户 sdbadmin 的密码为 sdbadmin

![// TODO 贴图](C:\Users\SequoiaDB\Desktop\开发者课程\img\login.png)

#### 打开 IDE 编辑器

双击打开桌面左侧 IntelliJ IDEA 程序

![](C:\Users\SequoiaDB\Desktop\开发者课程\img\open_idea.png)

## 通过 Java SDK 进行数据库连接

#### 代码示例

连接 SequoiaDB 需要指定连接 IP、端口号、用户名和密码。具体代码如下：

```
        // TODO
        // 创建数据库链接
        Sequoiadb db = new Sequoiadb(Constants.SDB_HOST, Constants.SDB_PORT, Constants.SDB_USERNAME, Constants.SDB_PASSWORD);
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
        // TODO END
```

#### 打开 Java 源文件

在 idea 窗口的左侧 Project 栏中双击打开指定的 Java 源码文件。

![//  TODO 贴图](C:\Users\SequoiaDB\Desktop\开发者课程\img\connection_dir.png)

#### 添加代码块

将”代码示例”中的代码块，粘贴到 Java 源文件中的 TODO ~ TODO END 位置区间内

![// TODO 贴图](C:\Users\SequoiaDB\Desktop\开发者课程\img\todo.png)

#### 为程序主类运行配置参数

1. 点击 Run 下的 Edit Configurations

   ![](C:\Users\SequoiaDB\Desktop\开发者课程\img\run_conf.png)

2. 配置 Configuration 页中的 Main class 选项，点击参数框右侧的 ”....“ 选择按钮，选择指定的主类，点击 ”OK“

   ![// TODO](C:\Users\SequoiaDB\Desktop\开发者课程\img\connection_main.png)

3. 配置 Configuration 页中的 Program arguments 选项，写入指定的参数，点击”OK“

   ![// TODO 贴图](C:\Users\SequoiaDB\Desktop\开发者课程\img\connection_param.png) 

#### 运行程序

点击 Run ，运行主程序

![// TODO 贴图](C:\Users\SequoiaDB\Desktop\开发者课程\img\connection_run.png)

#### 查看运行效果

在 idea 输出窗口查看运行效果

![// TODO 贴图呀](C:\Users\SequoiaDB\Desktop\开发者课程\img\connection_result.png)

## 总结

本课程介绍了在 Java 开发环境下如何连接 SequoiaDB 数据库的 JSON 实例。
 
