---
show: step
version: 1.0 
---

## 课程介绍

SequoiaFS 文件系统是基于 FUSE 在 Linux 系统下实现的一套文件系统，支持通用的文件操作 API。SequoiaFS 利用 SequoiaDB 的元数据集合存储文件和目录的属性信息，LOB 对象存储文件的数据内容，从而实现了类似 NFS 分布式网络文件系统。用户可以将远程SequoiaDB 的某个目标集合通过映射的方式挂载到本地 FS 节点，在 FS 节点的挂载目录下实现通过通用文件系统 API 对文件和目录进行操作。

下面为其基本逻辑结构图：



![img](http://doc.sequoiadb.com/cn/index/Public/Home/images/302/sequoiafs/model.png)

#### SequoiaFS 开发简介

使用 Java 语言，通过通用文件系统 API 对文件和目录进行操作。

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

打开 scdd-object 项目。

![image-20200414091915064](https://doc.shiyanlou.com/courses/1737/1207281/8fae6ec098d2e1f9a431636f6f919ad8-0)

#### 打开 Package

打开 lesson6_seuqoiafs 包，在该 Package 完成后续课程。

![image-20200426171504828](https://doc.shiyanlou.com/courses/1737/1207281/17c743f6aa570c606bfe5c2f08995c9f-0)

## 在SequoiaFS上写入文件

SequoiaFS 支持通用文件系统 API，使用 Java IO 类对 SequoiaFS 的挂载目录进行操作。

#### 代码编写

1）双击打开 SequoiaFSWrite 类，找到 main() 方法内行 TODO  通过java api 写入数据。

![1587951782938](https://doc.shiyanlou.com/courses/1737/1207281/b3196e9fef92149ed866c2d7f840ca77-0)

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

3）代码粘贴后如图所示。

![1587951885069](https://doc.shiyanlou.com/courses/1737/1207281/bd9994cba491e42b01b5fb0ffa3fbef1-0)

#### 执行代码

1）鼠标移动到屏幕左边 SequoiaFSWrite 类，右键点击，出现如图所示的选项条，左键单击 Edit 'SequoiaFSWrite' 选项。

![image-20200426172014874](https://doc.shiyanlou.com/courses/1737/1207281/2a03c1bf11febcdc8b7e9b958ca20364-0)

2）在屏幕下方查看结果输出。

![1587952116925](https://doc.shiyanlou.com/courses/1737/1207281/b60b51645c9d4797bc5415497615a4e5-0)

3）回到桌面，双击打开Home文件夹，在地址行输入/opt/sequoiafs/mountpoint/，按 Enter 进入目录。

![image-20200426172444745](https://doc.shiyanlou.com/courses/1737/1207281/1e3b58b5870854e9993342b7f7a9ffcd-0)

双击打开文件，查看内容。

![image-20200426174722910](https://doc.shiyanlou.com/courses/1737/1207281/b0d317ab87725098bfddca138579326e-0)



## 在SequoiaFS上读取文件

SequoiaFS 支持通用文件系统 API，使用 Java IO 类对 SequoiaFS 的挂载目录进行操作。

#### 代码编写

1）双击打开 SequoiaFSRead 类，找到 main() 方法内行 TODO  通过java api 读取数据。

![1587952211352](https://doc.shiyanlou.com/courses/1737/1207281/a393ba2f5efc78e3e2dce6151a966ec7-0)

2）将下方代码粘贴到 TODO ~ TODO END区域。

```java
//Get the file input stream
InputStreamReader put = new InputStreamReader(new FileInputStream("/opt/sequoiafs/mountpoint/sequoiadb.txt"), "utf-8");
char[] cbuf = new char[1024];
int len = 1024;
//Read the file content and output to console
while((len = put.read(cbuf))!= -1){
    System.out.println(new String(cbuf, 0, len));
}
put.close();
```

3）代码粘贴后如图所示。

![1587952283588](https://doc.shiyanlou.com/courses/1737/1207281/a8a7b74d92d0789899df80494a240fd3-0)

#### 执行代码

1）鼠标移动到屏幕左边 SequoiaFSRead 类，右键点击，出现如图所示的选项条，左键单击 Edit 'SequoiaFSRead' 选项。

![image-20200426175459509](https://doc.shiyanlou.com/courses/1737/1207281/e2fb51e72406d6ea444e504ab7516bf1-0)

2）在屏幕下方查看结果输出。

![1587952347423](https://doc.shiyanlou.com/courses/1737/1207281/7306dd8c02c89415a60d71a190088bf6-0)

## 总结

在本节课中，我们通过通用的 Java IO 流，对挂载的 SequoiaFS 目录进行了文件写入和文件读取，可以体验到和操作普通目录并没有任何不同，可以通过各种文件输入输出流对该目录进行操作，证明了通过 SequoiaFS 挂载的目录的可用性。
