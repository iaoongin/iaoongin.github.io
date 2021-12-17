---
title: IDEA打印彩色日志
date: 2019-08-20 14:23:24
category: code
tags:
- idea
- log
toc: true
---
<!-- ![](https://i.loli.net/2020/01/08/Di8CxeQKAbTNvBI.png) -->
<!-- more -->

# 完成以下步骤即可,非必须完成所有步骤

## 修改logback.xml

```xml
<!-- logback.xml -->
<!-- 彩色日志 -->
<!-- 彩色日志依赖的渲染类 -->
<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />
<conversionRule conversionWord="wex" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter" />
<conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter" />

<!-- 彩色日志格式 -->
<property name="FILE_LOG_PATTERN" value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}" />

```



## 打开设置 `File–Settings–Maven–Runner` 

在 `VM option` 添加

```
-Dspring.output.ansi.enabled=ALWAYS
```

## 打开设置 `Edit-Cofiguration` 

在 `VM option` 添加

```
-Dspring.output.ansi.enabled=ALWAYS
```

# 效果图

![彩色日志](/img/idea/彩色日志.png)