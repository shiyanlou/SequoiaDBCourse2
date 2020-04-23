---
show: step
version: 1.0 
---

## 课程介绍

元数据是对象的组成部分。对象由对象数据和元数据组成，对象数据是要存储的具体内容，而元数据是包含了对该内容的描述。

#### SequoiaS3 开发简介

SequoiaDB 巨杉数据库兼容 AWS S3 接口。本节课将通过AWS SDK为进行S3操作。

#### 实验流程简述：

- 用户通过 IDEA 编辑器编写 Java 源码
- 实验相关核心代码，可从文档中的代码示例粘贴到项目指定文件的 TODO 标记处
- 通过编译、运行 Java 代码，操作 SequoiaS3实例

![](https://doc.shiyanlou.com/courses/1736/1207281/7b1731fc121e3b460dcd9841eb0218a6-0)

#### 实验环境

课程使用的实验环境为 Ubuntu Linux 16.04 64 位版本。SequoiaDB 数据库引擎为 3.4 版本。IDEA 编辑器为 16.0 版本。JDK 为 1.8 版本。

## 打开项目

#### 打开 IDEA

打开 IDEA 代码开发工具。

![](https://doc.shiyanlou.com/courses/1736/1207281/06650396616c742995bb63fcf933fac5-0)

#### 打开项目

打开object-java项目。

![image-20200414091915064](https://doc.shiyanlou.com/courses/1737/1207281/8fae6ec098d2e1f9a431636f6f919ad8-0)

#### 打开 Package

打开 lesson3_s3ObjectMetadata 包，在该 Package 完成后续课程。

![image-20200422175731680](https://doc.shiyanlou.com/courses/1737/1207281/d1f1d7c372b36c692de3884436792005-0)

## 设置元数据

在将文件上传为 S3 对象时，可以在上传的同时设定元数据参数，然后再上传。

1）双击打开 ObjectMetadataUtil 类，在 setMetadata() 函数内找到行**TODO code1 设置对象元数据**。

![image-20200422160132997](https://doc.shiyanlou.com/courses/1737/1207281/9922a17243650965c5eabd4017ff2ed6-0)

2）将下方代码粘贴到 TODO ~ TODO END区域内。

```java
    //Get the S3 connection
    AmazonS3 s3 = this.getS3();
    //Create a bucket to use
    s3.createBucket(bucketName);
    //Create file input stream
    File file = new File("/opt/sequoiadb/version.conf");
    InputStream inputStream = new FileInputStream(file);
    //Get metadata object
    ObjectMetadata objectMetadata = new ObjectMetadata();
    //Set metadata properties
    objectMetadata.setContentLength(file.length());
    objectMetadata.setContentLanguage("CH");
    objectMetadata.setContentEncoding("utf8");
    objectMetadata.setContentType("text/plain");
    //Save the uploaded file as an object and set the object metadata
    s3.putObject(bucketName,objectName,inputStream,objectMetadata);
```



## 查看元数据

在一个已有的 S3 实例中，可以通过 getObjectMetadata(String str,String str1) 函数获得指定对象的元数据对象。

1）双击打开 ObjectMetadataUtil 类，找到 queryMetadata() 函数内行**TODO code2 查询对象元数据**。

![image-20200422160225591](https://doc.shiyanlou.com/courses/1737/1207281/55a93138064f76b8f44a17711ea37c13-0)

2）将下方代码粘贴到 TODO ~ TODO END区域内。

```java
        //Get the S3 connection
        AmazonS3 s3 = this.getS3();
        //Get metadata object of the specified object
        ObjectMetadata objectMetadata =
        				s3.getObjectMetadata(bucketName,objectName);
        //Get metadata properties
        String contentLanguage = objectMetadata.getContentLanguage();
        String contentEncoding = objectMetadata.getContentEncoding();
        String contentType = objectMetadata.getContentType();
		//Print metadata properties
        System.out.println("contentLanguage:"+contentLanguage);
        System.out.println("contentEncoding:"+contentEncoding);
        System.out.println("contentType:"+contentType);
		//Clean up the environment
        s3.deleteObject(bucketName,objectName);
        s3.deleteBucket(bucketName);
```

## 执行代码

1）鼠标移动到屏幕左边 ObjectMetadataTest 类，右键点击，出现如图所示的选项条，左键单击**Edit 'ObjectMetadataTest'**选项。

![image-20200414005415883](https://doc.shiyanlou.com/courses/1737/1207281/5aa784d9a86d1bcfc46b2db1870d1712-0)

2）在出现下图所示界面后，将**SetAndQuery填入红框所选位置，然后点击**OK**按钮。

![image-20200414011543183](https://doc.shiyanlou.com/courses/1737/1207281/9b0e9a0733b4efd0cf05edaedfbc6a05-0)

3）鼠标移动到屏幕左边 ObjectMetadataTest 类上，右键点击，出现如图所示的选项条，左键单击**Run 'ObjectMetadataTest'**选项。

![image-20200414005608353](https://doc.shiyanlou.com/courses/1737/1207281/be73c5860b9238a4b2ac502165809309-0)

4）在屏幕下方查看结果输出，enter代表开始此项操作，exit代表结束这项操作，两者中间是执行中的输出信息，可以看到在queryMetadata区域出现了在设置元数据时设置的属性。

![image-20200414013041014](https://doc.shiyanlou.com/courses/1737/1207281/06c02c8569841998663502253b495c13-0)

## 总结

在该节课中，我们学到了如何在上传对象时设置对象的元数据，和查询已有对象的元数据。元数据是一种描述性信息，可以让我们更好的管理对象。
