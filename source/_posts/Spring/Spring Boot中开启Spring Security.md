---
title: Spring Boot中开启Spring Security
time: 2019-11-06
categories: springsecurity
tags: springsecurity
---

# Spring Boot中开启Spring Security
---
Spring Security是一款基于Spring的安全框架，主要包含认证和授权两大安全模块，和另外一款流行的安全框架Apache Shiro相比，它拥有更为强大的功能。Spring Security也可以轻松的自定义扩展以满足各种需求，并且对常见的Web安全攻击提供了防护支持。如果你的Web框架选择的是Spring，那么在安全方面Spring Security会是一个不错的选择。

这里我们使用Spring Boot来集成Spring Security，Spring Boot版本为1.5.14.RELEASE，Spring Security版本为4.2.7RELEASE。

## 开启Spring Security
创建一个Spring Boot项目，然后引入spring-boot-starter-security：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

接下来我们创建一个TestController，对外提供一个/hello服务：
```
@RestController
public class TestController {
    @GetMapping("hello")
    public String hello() {
        return "hello spring security";
    }
}
```

这时候我们直接启动项目，访问http://localhost:8080/hello，可看到页面弹出了个HTTP Basic认证框：
![QQ截图20180707095933.png](https://i.loli.net/2019/11/06/fqK1C3RYeMzaXsj.png)
当Spring项目中引入了Spring Security依赖的时候，项目会默认开启如下配置：
```
security:
  basic:
    enabled: true
```

这个配置开启了一个HTTP basic类型的认证，所有服务的访问都必须先过这个认证，默认的用户名为user，密码由Sping Security自动生成，回到IDE的控制台，可以找到密码信息：
```
Using default security password: e9ed391c-93de-4611-ac87-d871d9e749ac
```

输入用户名user，密码e9ed391c-93de-4611-ac87-d871d9e749ac后，我们便可以成功访问/hello接口。

## 基于表单认证

## 基本原理
上面我们开启了一个最简单的Spring Security安全配置，下面我们来了解下Spring Security的基本原理。通过上面的的配置，代码的执行过程可以简化为下图表示：
![QQ截图20180707111356.png](https://i.loli.net/2019/11/06/l6niSf4jpP3yN8W.png)