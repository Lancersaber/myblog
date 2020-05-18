---
title: Spring Boot异常处理详解
time: 2019-11-09
categories: spring
tags: spring
---

# Spring Boot异常处理详解
---
在Spring MVC异常处理详解中，介绍了Spring MVC的异常处理体系，本文将讲解在此基础上Spring Boot为我们做了哪些工作。下图列出了Spring Boot中跟MVC异常处理相关的类。

![SpringBootWebExceptionResolver.png](https://i.loli.net/2019/11/09/NBiAebnwZcoRQuO.png)

Spring Boot在启动过程中会根据当前环境进行AutoConfiguration，其中跟MVC错误处理相关的配置内容，在ErrorMvcAutoConfiguration这个类中。以下会分块介绍这个类里面的配置。

## 在Servlet容器中添加了一个默认的错误页面
因为ErrorMvcAutoConfiguration类实现了EmbeddedServletContainerCustomizer接口，所以可以通过override customize方法来定制Servlet容器。以下代码摘自ErrorMvcAutoConfiguration：
```
@Value("${error.path:/error}")
private String errorPath = "/error";

@Override
public void customize(ConfigurableEmbeddedServletContainer container) {
    container.addErrorPages(new ErrorPage(this.properties.getServletPrefix()
        + this.errorPath));
}
```

可以看到ErrorMvcAutoConfiguration在容器中，添加了一个错误页面/error。因为这项配置的存在，如果Spring MVC在处理过程抛出异常到Servlet容器，容器会定向到这个错误页面/error。

那么我们有什么可以配置的呢？
* 我们可以配置错误页面的url，/error是默认值，我们可以再application.properties中通过设置error.path的值来配置该页面的url；
* 我们可以提供一个自定义的EmbeddedServletContainerCustomizer，添加更多的错误页面，比如对不同的http status code，使用不同的错误处理页面。就像下面这段代码一样：

```
@Bean
public EmbeddedServletContainerCustomizer containerCustomizer() {
    return new EmbeddedServletContainerCustomizer() {
        @Override
        public void customize(ConfigurableEmbeddedServletContainer container) {
            container.addErrorPages(new ErrorPage(HttpStatus.NOT_FOUND, "/404"));
            container.addErrorPages(new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/500"));
        }
    };
}
```