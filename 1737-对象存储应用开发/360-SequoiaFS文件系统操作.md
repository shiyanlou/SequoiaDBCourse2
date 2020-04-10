
## 课程介绍



## 环境查看



## 在SequoiaFS上写入文件

#### 代码编写

双击打开SequoiaFSWrite类，找到引导行 **TODO 1 通过java api 写入数据**

引导行图示



复制下方代码到引导行下方

```
        FileInputStream fis = new FileInputStream("/home/sdbadmin/tmp.txt");
        FileOutputStream fos = new FileOutputStream("/home/sdbadmin/tmp2.txt");
        int len=0;
        //一次读取多少字节的文件,这里可以选择tmp.txt的所有字节长度
        byte[] b = new byte[fis.available()];
        while((len=fis.read(b))!=-1){
        //对字节进行排序
        Arrays.sort(b);
        fos.write(b,0,len);
        fos.flush();
        }
```

#### 打包



#### 运行

复制jar包



运行

## 在SequoiaFS上读取文件

#### 代码编写

双击打开SequoiaFSRead类，找到引导行 **TODO 1 通过java api 读取数据**

引导行图示



复制下方代码到引导行下方

```
       InputStreamReader put = new InputStreamReader(new FileInputStream("/opt/sequoiadb/sequoiafs/mountpoint/u_8.txt"), "utf-8");

        char[] cbuf = new char[1024];

        int len = 0;

        while((len = put.read(cbuf))!= -1){
            System.out.println(new String(cbuf, 0, len));
        }

        put.close();
```

#### 打包



#### 运行

复制jar包



运行
