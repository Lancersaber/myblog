---
title: Springboot aop记录用户操作日志
categories: spring
tags: springboot&AOP
---

#Springboot aop记录用户操作日志
---
在Spring框架中，使用AOP配合自定义注解可以方便的实现用户操作的监控。首先搭建一个基本的Spring Boot Web环境开启Spring Boot，然后引入必要依赖：

## 自定义注解
定义一个方法级别的@Log注解，用于标注需要监控的方法：
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {
    String value() default "";
}
```

