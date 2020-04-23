---
show: step
version: 1.0 
---

## 课程介绍

SequoiaFS文件系统是基于FUSE在Linux系统下实现的一套文件系统，支持通用的文件操作API。SequoiaFS利用SequoiaDB的元数据集合存储文件和目录的属性信息，lob对象存储文件的数据内容，从而实现了类似NFS分布式网络文件系统。用户可以将远程SequoiaDB的某个目标集合通过映射的方式挂载到本地FS节点，在FS节点的挂载目录下实现通过通用文件系统API对文件和目录进行操作。

下面为其基本逻辑结构图：



![img](http://doc.sequoiadb.com/cn/index/Public/Home/images/302/sequoiafs/model.png)

#### SequoiaFS 开发简介

使用Java语言，通过通用文件系统API对文件和目录进行操作。

#### 实验流程简述：

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

打开 object-java 项目。

![image-20200414091915064](https://doc.shiyanlou.com/courses/1737/1207281/8fae6ec098d2e1f9a431636f6f919ad8-0)

#### 打开 Package

打开 lesson6_seuqoiaFS 包，在该Package完成后续课程。

![image-20200422180054863](https://doc.shiyanlou.com/courses/1737/1207281/75dc18d192031b8ed0a1b44c95c79a69-0)

## 在SequoiaFS上写入文件

SequoiaFS 支持通用文件系统API，使用 Java IO 类对 SequoiaFS 的挂载目录进行操作。

#### 代码编写

1）双击打开 SequoiaFSWrite 类，找到 main() 方法内行 **TODO  通过java api 写入数据**。

![image-20200415013416784](https://doc.shiyanlou.com/courses/1737/1207281/8bf47c3fce31ae205234af2281eecbfd-0)



2）将下方代码粘贴到 TODO ~ TODO END区域。

```java
    InputStream put = new FileInputStream("/home/sdbadmin/sequoiadb.txt");
    OutputStream out  = new FileOutputStream("/opt/sequoiafs/mountpoint/sequoiadb.txt");
    byte[] cbuf = new byte[1024];
    int len = 1024;
    //How many bytes of file are read at a time
    while((len = put.read(cbuf))!= -1){
        out.write(cbuf,0,len);
        out.flush();
    }
    put.close();
```

#### 执行代码

1）鼠标移动到屏幕左边 SequoiaFSWrite 类，右键点击，出现如图所示的选项条，左键单击**Edit 'SequoiaFSWrite'**选项。

![image-20200415013625601](https://doc.shiyanlou.com/courses/1737/1207281/17be31c6f7fcbd90a079e7a0465a9e24-0)

2）在屏幕下方查看结果输出。无输出，无报错。

![image-20200415193806468](https://doc.shiyanlou.com/courses/1737/1207281/94ffd505eff6f66ed8b396965c4c0eda-0)

3）通过可视化界面进入 /opt/sequoiafs/mountpoint/ 目录，查看文件已被写入到sequoiafs中，同时查看文件内容。

![image-20200415200958485](https://doc.shiyanlou.com/courses/1737/1207281/790fb7b0e8332734f3ae4e11c030c385-0)

## 在SequoiaFS上读取文件

SequoiaFS 支持通用文件系统API，使用 Java IO 类对 SequoiaFS 的挂载目录进行操作。

#### 代码编写

1）双击打开 SequoiaFSWrite 类，找到main()方法内行 **TODO  通过java api 读取数据。**

![image-20200415013904075](https://doc.shiyanlou.com/courses/1737/1207281/bb01ba093d96700045ec6a27d6449262-0)

2）将下方代码粘贴到 TODO ~ TODO END区域。

```java
//Get the file input stream
InputStreamReader put = new InputStreamReader(new FileInputStream("/opt/sequoiafs/mountpoint/version.conf"), "utf-8");

char[] cbuf = new char[1024];

int len = 1024;
//Read the file content and output to console
while((len = put.read(cbuf))!= -1){
	System.out.println(new String(cbuf, 0, len));
}

put.close();
```

#### 执行代码

1）鼠标移动到屏幕左边 SequoiaFSWrite 类，右键点击，出现如图所示的选项条，左键单击**Edit 'SequoiaFSWrite'**选项。

![image-20200415014035554](https://doc.shiyanlou.com/courses/1737/1207281/04777d9aa3321edbd0005ec52535d519-0)

2）在屏幕下方查看结果输出。

![image-20200415201106328](https://doc.shiyanlou.com/courses/1737/1207281/08e91c5ef3cdc80a1b18d93eb3f37bf2-0)

