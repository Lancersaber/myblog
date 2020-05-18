---
title: 深入学习Springboot中的SpringApplication
time: 2019-11-04
categories: spring
tags: spring
---

# 深入学习Springboot中的SpringApplication
---
在Spring Boot的入口类中，我们通常是通过调用SpringApplication的run方法来启动Spring Boot项目。这节我们来深入学习下SpringApplication的一些细节。

## 自定义Springboot Application
默认的我们都是直接通过SpringApplication的run方法来直接启动Spring Boot，其实我们可以通过一些API来调整某些行为。

### 通过SpringApplication API调整
我们新建一个SpringBoot项目，Spring Boot版本为2.1.0.RELEASE，artifactId为SpringApplication，并引入spring-boot-starter-web依赖。
我们将入口类的代码改为：
```
SpringApplication application = new SpringApplication(DemoApplication.class);
application.setBannerMode(Banner.Mode.OFF);
application.setWebApplicationType(WebApplicationType.NONE);
application.setAdditionalProfiles("dev");
application.run(args);
```
通过调用SpringApplication的方法，我们关闭了Banner的打印，设置应用环境为非WEB应用，profiles指定为dev。除此之外，SpringApplication还包含了许多别的方法，具体可以查看源码或者官方文档：
![QQ截图20190223101959.png](https://i.loli.net/2019/11/04/UXtg3GvHKd9a8Ar.png)

### 通过SpringApplicationBuilder API调整
SpringApplicationBuilder提供了Fluent API，可以实现链式调用，下面的代码和上面的效果一致，但在编写上较为方便：
```
new SpringApplicationBuilder(DemoApplication.class)
        .bannerMode(Banner.Mode.OFF)
        .web(WebApplicationType.NONE)
        .profiles("dev")
        .run(args);
```

## SpringApplication准备阶段
SpringApplication的生命周期阶段大致可以分为准备阶段和运行阶段。

我们通过源码来查看SpringApplication的有参构造器：
![QQ截图20190223102806.png](https://i.loli.net/2019/11/04/lnBQ2OJhU3Am5KN.png)
通过有参构造器里的代码我们可以将SpringApplication的准备阶段分为以下几个步骤：

### 配置源
构造器中this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));这行代码用于加载我们配置的Spring Boot Bean源。。通常我们使用SpringApplication或者SpringApplicationBuilder的构造器来直接指定源。

所谓的Spring Boot Bean源指的是某个被@SpringBootApplication注解标注的类，比如入口类：

![QQ截图20190223104742.png](https://i.loli.net/2019/11/04/alymOoqibSn1Dkj.png)

我们也可以将上面的代码改为下面这种方式：
```
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(ApplicationResource.class);
        application.run(args);
    }

    @SpringBootApplication
    public static class ApplicationResource {

    }
}
```

这样也是可行的。查看SpringApplication的单个参数构造器：
![QQ截图20190223105200.png](https://i.loli.net/2019/11/04/XkeFJrHvYgV19DW.png)
说明我们除了配置单个源外，还可以配置多个源。

### 推断应用类型
构造器中这行this.webApplicationType = WebApplicationType.deduceFromClasspath();代码用于推断当前Spring Boot应用类型。
Spring Boot 2.0后，应用可以分为下面三种类型：
1、WebApplicationType.NONE：非WEB类型；

2、WebApplicationType.REACTIVE：Web Reactive类型；

3、WebApplicationType.SERVLET：Web Servlet类型。