---
title: windows-运行关闭jar包cmd脚本
date: 2019-08-26 14:41:09
category: code
tags:
- jar
- windows
- cmd
toc: true
---
![](https://i.loli.net/2020/01/08/m9aWsZgrMkCXnxG.jpg)
<!-- more -->

# 项目结构
```
|--web
    |-- appname.txt     // 存放jar包名字
    |-- xxx.jar         //  jar包
    |-- run.bat         // 前台启动脚本
    |-- run-demo.bat    // 后台启动脚本
    |-- stop.bat        // 停止脚本
```


# 前台启动脚本

```cmd
chcp 65001
for /f "tokens=1" %%j in ('findstr .* appname.txt') do (
	title %%j
	java -jar -Dfile.encoding=UTF-8 -Dlogging.config=config/logback.xml -Dspring.profiles.active=pro -Dmybatis-plus.mapper-locations=file:config/mapper/*.xml "%%j" -Xms50m -Xmx1024m
)
```

# 后台启动脚本
```cmd
chcp 65001
for /f "tokens=1" %%j in ('findstr .* appname.txt') do (
	title %%j
	start javaw -jar -Dlogging.config=config/logback.xml -Dspring.profiles.active=pro -Dmybatis-plus.mapper-locations=file:config/mapper/*.xml "%%j" -Xms256m -Xmx1g -Xss256k
)
```

# 停止脚本
注：`"C:\Program Files\Java\jdk1.8.0_131\bin\jps"` 为jdk工具包
```cmd
@echo off
:: 找到jar的pid进程，并杀死

for /f "tokens=1" %%j in ('findstr .* appname.txt') do (
	for /f "tokens=1" %%i in ('"C:\Program Files\Java\jdk1.8.0_131\bin\jps" -l ^| findstr %%j') do (
		set n=%%i
	)
)

taskkill /f /pid %n%
set n=

exit
```

# 参考

[win bat脚本 后台运行jar包](https://www.jianshu.com/p/bcffbc009a79)	

[获取当前用户所有java进程及jps命令的实现](https://www.iteye.com/blog/chainhou-1872938)	