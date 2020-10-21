---
title: spring中将配置文件的属性注入到静态属性里
date: 2019-11-02 09:53:54
category: code
tags:
- spring
toc: true
---
![](https://i.loli.net/2019/11/26/IOCUvxrVcesMJab.jpg)
<!-- more -->
# 场景
如在工具类中，不想将该工具类被spring管理，通过引用其他类的的静态属性获取值。



# 步骤
## application.properties
```properties
#局域网地址
file.lanServer=\\\\DESKTOP-3FE3ORR\\app\\
#app下载地址
file.remoteServer=http://127.0.0.1:8080/app/
file.fileUploadDir=H:/vadx
file.relatePath=/vadx/app
```

## 新建类, 如文件路径配置
```java
@Component
@Getter
@Setter
public class FileProp {

    public static String remoteServer;
    public static String lanServer;
    public static String fileUploadDir;
    public static String relatePath;

    @Value("${file.remoteServer}")
    public void setRemoteServer(String remoteServer) {
        FileProp.remoteServer = remoteServer;
    }

    @Value("${file.lanServer}")
    public void setLanServer(String lanServer) {
        FileProp.lanServer = lanServer;
    }

    @Value("${file.fileUploadDir}")
    public void setFileUploadDir(String fileUploadDir) {
        FileProp.fileUploadDir = fileUploadDir;
    }

    @Value("${file.relatePath}")
    public void setRelatePath(String relatePath) {
        FileProp.relatePath = relatePath;
    }
}
```

## 使用，在工具类中使用
```java
@Slf4j
public class UploadFileUtil {

    //项目前缀路径
    public static String destFile = FileProp.fileUploadDir;
}
```
