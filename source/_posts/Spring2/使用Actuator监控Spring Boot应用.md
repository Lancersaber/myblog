---
title: 使用Actuator监控Spring Boot应用
time: 2019-11-10
categories: spring
tags: spring
---

# 使用Actuator监控Spring Boot应用
---
我们都知道Spring Boot是一个用于快速开发Java Web的框架，不需要太多的配置即可使用Spring的大量功能。Spring Boot遵循着“约定大于配置”的原则，许多功能使用默认的配置即可。这样的做法好处在于我们不需要像使用Spring那样编写一大堆的XML配置代码，但过于简单的配置过程会让我们在了解各种依赖，配置之间的关系过程上带来一些困难。不过没关系，在Spring Boot中，我们可以使用Actuator来监控应用，Actuator提供了一系列的RESTful API让我们可以更为细致的了解各种信息。

## 引入Actuator
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

