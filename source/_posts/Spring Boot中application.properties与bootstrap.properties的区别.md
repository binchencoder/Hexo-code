---
title: Spring Boot中application.properties与bootstrap.properties的区别
tags:
  - Spring
  - SpringBoot
  - Spring Cloud
categories:
  - 框架
abbrlink: 584
date: 2019-08-01 00:00:18
---

# application.propertes (application.yml)

现在使用SpringBoot是Java世界里的主流选择，它大大简化了应用初始搭建以及开发过程，该框架使用了特定的方式来进行配置，从而是开发人员不在需要定义样板化的配置。今天就带大家来了解下application.properties

使用application.properties，一般情况下主要用来配置服务中一些最基本的属性，如 数据库连接、日志相关配置。其他的用法还有 一般属性使用、自定义属性使用、属性间的引用(占位符)、随机数的使用、数据类型自动转换、嵌套属性注入

如：
```
spring.profiles.active=dev,vexillary-service,skylb,file_server,kafka_monitor_0.8,metrics

#gRPC
grpc.port=6567

#metrics
metrics.scrapePort=10000

# logging level
logging.level.root=error
logging.level.com.jingoal.grpc=debug
logging.level.com.jingoal.approval=debug
```

## application.properties与bootstrap.properties的区别

两者主要区别是加载顺序不同，bootstrap.properties在application.properties 之前加载，bootstrap.properties用于应用程序上下文的引导阶段

### 典型场景

1. 当时用Spring Cloud Config Server的时候，你应该在bootstraop.properties里面指定spring.application.name 和 spring.cloud.config.server.git.uri
2. 一些加密/解密的信息

技术上, bootstrap.properties由父Spring ApplicationContext加载。父ApplicationContext 被加载到使用application.properties的之前。

当使用SpringCloud的时候，配置信息一般是从config server加载的，为了取得配置信息（比如密码等），你需要一些提早的或引导配置。因此，把config server信息放在bootstrap.properties，用来加载真正需要的配置信息。

### 属性覆盖问题

启动上下文时，**Spring Cloud**会创建一个```BootStrap Context```，作为Spring应用的```Application Context```的父上下文。初始化的时候，```Bootstrap Context```负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的```Environment```。

**Bootstrap属性有高优先级，默认情况下，他们不会被本地配置覆盖**。
```Bootstrap  Context```和```Application Context```有着不同的约定，所以新增了一个```bootstrap.properties```，而不是使用```application.properties```。保证```Bootstrap Context```和```Application Context```配置的分离。
