
## 课程介绍



## 环境查看

展示包

## 更新元数据

双击打开 ObjectMetadataTest类，找到引导行**TODO 1 更新元数据**，在该行下方粘贴代码

引导行所在位置



将下方代码粘贴到引导行下方

```
        //设定对象元数据对象属性
        objectMetadata.setContentLanguage("CH");
        objectMetadata.setContentEncoding("utf8");
        objectMetadata.setContentType("text/plain");
        //更新对象元数据
        result.setObjectMetadata(objectMetadata);
```



## 查看元数据

双击打开 ObjectMetadataTest类，找到引导行**TODO 2 查看元数据**，在该行下方粘贴代码

引导行所在位置



将下方代码粘贴到引导行下方

```
        String contentLanguage = objectMetadata.getContentLanguage();
        String contentEncoding = objectMetadata.getContentEncoding();
        String contentType = objectMetadata.getContentType();

        System.out.println(contentLanguage);
        System.out.println(contentEncoding);
        System.out.println(contentType);
```

## 验证元数据更新查询操作

选择位于屏幕左边的小三角



单击后出现下图，单击箭头所指的第一项编译执行代码



在屏幕下方查看执行结果


