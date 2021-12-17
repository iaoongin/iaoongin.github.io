---
title: Spring Cloud Gateway 的使用
date: 2020-06-09 11:35:13
category: code
tags: 
    - gateway
    - spring cloud
toc: true
---

## 框架说明

用于统一服务的出入口。

<!-- more -->

## 注意

由于该项目使用的是Spring WebFlux，区别与Spring WebMvc。

## 带来的优势

- 做统一权限（目前）
- 统一日志
- ...



## 词汇

路由 Route：一个组合，包含下面两个部分。

断言 Predicate：相当于转发条件。

过滤器 Filter：相当于转发规则。



### 有关类

org.springframework.cloud.gateway.route.RouteDefinition

| 属性       | 类型                      | 说明                     |
| ---------- | ------------------------- | ------------------------ |
| id         | String                    | 路由id，唯一标识。       |
| predicates | List<PredicateDefinition> | 断言集合。               |
| filters    | List<FilterDefinition>    | 过滤集合。               |
| uri        | URI                       | 目标地址。支持http和lb。 |
| metadata   | Map<String, Object>       | 元数据。                 |
| order      | int                       | 顺序。                   |



org.springframework.cloud.gateway.handler.predicate.PredicateDefinition

| 属性 | 类型                | 说明                                                |
| ---- | ------------------- | --------------------------------------------------- |
| name | String              | RoutePredicateFactory名称。如：Path                 |
| args | Map<String, String> | 参数。不同的RoutePredicateFactory，有不一样的参数。 |



org.springframework.cloud.gateway.filter.FilterDefinition

| 属性 | 类型                | 说明                                                     |
| ---- | ------------------- | -------------------------------------------------------- |
| name | String              | GatewayFilterFactory名称。如：RewritePath，StripPrefix。 |
| args | Map<String, String> | 参数。不同的GatewayFilterFactory，有不一样的参数。       |



## 使用

### 动态路由

#### 说明

​		通过实现 `org.springframework.cloud.gateway.route.RouteDefinitionRepository` 接口，实现路由获取方式。

​		来自`org.springframework.cloud.gateway.route.CachingRouteDefinitionLocator#fetch` 的调用。

#### 代码

```
package com.ystmob.mg.api.config.gatewayroute;

import cn.hutool.json.JSONUtil;
import com.ystmob.mg.api.service.GatewayRouteService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionRepository;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import javax.annotation.Resource;
import java.util.stream.Collectors;

@Slf4j
@Component
public class DynamicRouteDefinitionRepository implements RouteDefinitionRepository {

    @Resource
    private GatewayRouteService gatewayRouteService;

    @Override
    public Flux<RouteDefinition> getRouteDefinitions() {

        return gatewayRouteService.listStatusOn()
                .flatMapIterable(gatewayRoutes ->
                        gatewayRoutes.stream().map(gr -> {
                            String jsonString = JSONUtil.toJsonStr(gr);
                            RouteDefinition routeDefinition = JSONUtil.toBean(jsonString, RouteDefinition.class);

                            // id 对 routeId
                            routeDefinition.setId(gr.getRouteId());
                            return routeDefinition;
                        }).collect(Collectors.toList()));

    }

    @Override
    public Mono<Void> save(Mono<RouteDefinition> route) {
        return Mono.empty();
    }

    @Override
    public Mono<Void> delete(Mono<String> routeId) {
        return Mono.empty();
    }


}
```



#### 路由刷新

通过发布事件，通知监听器刷新。

```
@Resource
ApplicationEventPublisher publisher;

...
	publisher.publishEvent(new RefreshRoutesEvent(this));
...
```



监听器

`org.springframework.cloud.gateway.route.CachingRouteDefinitionLocator#onApplicationEvent`

`org.springframework.cloud.gateway.route.CachingRouteLocator#onApplicationEvent`



### 静态路由



通过配置yml文件，即提前声明的形式。



比如

```
spring:
  application:
    name: stock-gateway
  cloud:
    gateway:
      routes:
          - id: stock-finance
            uri: lb://stock-finance
            predicates:
              - Path=/stock-finance/**
            filters:
              - RewritePath=/stock-finance, /
          - id: stock-ii
            uri: http://localhost:8510
            predicates:
              - Path=/stock-ii/**
            filters:
              - RewritePath=/stock-ii, /
```

