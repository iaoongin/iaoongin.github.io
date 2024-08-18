---
title: mysql导入大文件失败解决方法
date: 2019-08-20 14:15:19
category: code
tags:
- mysql
toc: true
---
<!-- ![](https://i.loli.net/2020/01/08/RYBJqgFQIXW7hAH.png) -->
<!-- more -->
# 解决方式

修改`/etc/my.conf`,在以指定区域新增以下行

```
[mysqld]

#### 避免导入失败
max_allowed_packet = 200M


[mysqlhotcopy]

interactive-timeout=28800000
wait_timeout=28800000
#### 避免导入失败
```



# 场景

导入的sql文件较大时，或者网络状况不好。

# 原因

mysql 默认配置超时时间及支持文件大小限制太小