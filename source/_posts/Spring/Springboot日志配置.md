---
title: springboot日志配置
time: 2019-11-1
categories: spring
tags: spring日志
---
# springboot日志配置
---
Spring Boot在所有内部日志中使用Commons Logging，但是默认配置也提供了对常用日志的支持，如：Java Util Logging，Log4J, Log4J2和Logback。每种Logger都可以通过配置使用控制台或者文件输出日志内容。

SLF4J —— Simple Logging Facade For Java，它是一个针对于各类Java日志框架的统一Facade抽象。Java日志框架众多——常用的有java.util.logging, log4j, logback，commons-logging, Spring框架使用的是Jakarta Commons Logging API（JCL）。而SLF4J定义了统一的日志抽象接口，而真正的日志实现则是在运行时决定的——它提供了各类日志框架的绑定。

Logback是log4j框架的作者开发的新一代日志框架，它效率更高、能够适应诸多的运行环境，同时天然支持SLF4J。

默认情况下，Spring Boot会用Logback来记录日志，并用INFO级别输出到控制台。在运行应用程序和其他例子时，你应该已经看到很多INFO级别的日志了。
```
  _   _   _   _   _   _  
 / \ / \ / \ / \ / \ / \ 
( m | r | b | i | r | d )
 \_/ \_/ \_/ \_/ \_/ \_/ 
2018-02-08 15:05:03.368  INFO 14404 --- [ main] cc.mrbird.Application                    : Starting Application on SC-201802012049 with PID 14404 (D:\neonWorkspace\mrbird\target\classes started by Administrator in D:\neonWorkspace\mrbird)
2018-02-08 15:05:03.375  INFO 14404 --- [ main] cc.mrbird.Application                    : No active profile set, falling back to default profiles: default
2018-02-08 15:05:03.777  INFO 14404 --- [ main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@31321838: startup date [Thu Feb 08 15:05:03 CST 2018]; root of context hierarchy
2018-02-08 15:05:05.083  INFO 14404 --- [ main] o.s.b.f.s.DefaultListableBeanFactory     : Overriding bean definition for bean 'advisorAutoProxyCreator' with a different definition: replacing [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=shiroConfig; factoryMethodName=advisorAutoProxyCreator; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [cc/mrbird/config/ShiroConfig.class]] with [Root bean: class [null]; scope=; abstract=false; lazyInit=false; autowireMode=3; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=com.alibaba.druid.spring.boot.autoconfigure.stat.DruidSpringAopConfiguration; factoryMethodName=advisorAutoProxyCreator; initMethodName=null; destroyMethodName=(inferred); defined in class path resource [com/alibaba/druid/spring/boot/autoconfigure/stat/DruidSpringAopConfiguration.class]]
2018-02-08 15:05:05.554  INFO 14404 --- [ main] trationDelegate$BeanPostProcessorChecker : Bean 'shiroConfig' of type [cc.mrbird.config.ShiroConfig$$EnhancerBySpringCGLIB$$b7e43ac8] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-02-08 15:05:05.600  INFO 14404 --- [ main] trationDelegate$BeanPostProcessorChecker : Bean 'com.alibaba.druid.spring.boot.autoconfigure.stat.DruidSpringAopConfiguration' of type [com.alibaba.druid.spring.boot.autoconfigure.stat.DruidSpringAopConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2018-02-08 15:05:06.231  INFO 14404 --- [ main] trationDelegate$BeanPostProcessorChecker : Bean 'mybatis-org.mybatis.spring.boot.autoconfigure.MybatisProperties' of type [org.mybatis.spring.boot.autoconfigure.MybatisProperties] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
...
```
从上面可以看到，日志输出内容元素具体如下：
1、时间日期：精确到毫秒；

2、日志级别：ERROR, WARN, INFO, DEBUG or TRACE；

3、进程ID；

4、分隔符：---标识实际日志的开始；

5、线程名：方括号括起来（可能会截断控制台输出）；

6、Logger名：通常使用源代码的类名；

7、日志内容。

## 添加日志依赖
假如maven依赖中添加了spring-boot-starter-logging：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```
那么，我们的Spring Boot应用将自动使用logback作为应用日志框架，Spring Boot启动的时候，由org.springframework.boot.logging.Logging-Application-Listener根据情况初始化并使用。

## 默认配置属性支持
Spring Boot为我们提供了很多默认的日志配置，所以，只要将spring-boot-starter-logging作为依赖加入到当前应用的classpath，则“开箱即用”。 下面介绍几种在application.properties就可以配置的日志相关属性。

## 控制台输出
日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出。 Spring Boot中默认配置ERROR、WARN和INFO级别的日志输出到控制台。您还可以通过启动您的应用程序–debug标志来启用“调试”模式（开发的时候推荐开启）,以下两种方式皆可：
* 在运行命令后加入–debug标志，如：$ java -jar springTest.jar --debug。

* 在application.properties中配置debug=true，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

## 文件输出
默认情况下，Spring Boot将日志输出到控制台，不会写到日志文件。如果要编写除控制台输出之外的日志文件，则需在application.properties中设置logging.file或logging.path属性。
* logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log。

* logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log。
如果只配置 logging.file，会在项目的当前路径下生成一个 xxx.log 日志文件。

如果只配置 logging.path，在 /var/log文件夹生成一个日志文件为 spring.log。

## 级别控制
所有支持的日志记录系统都可以在Spring环境中设置记录级别（例如在application.properties中） 格式为：’logging.level.* = LEVEL’
* logging.level：日志级别控制前缀，\*为包名或Logger名

* LEVEL：选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF
举例：
* logging.level.com.mrbird=DEBUG：com.mrbird包下所有class以DEBUG级别输出。

* logging.level.root=WARN：root日志以WARN级别输出。

## 自定义日志配置
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：
* Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy

* Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml

* Log4j2：log4j2-spring.xml, log4j2.xml

* JDK (Java Util Logging)：logging.properties

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml），命名为logback-spring.xml的日志配置文件，spring boot可以为它添加一些spring boot特有的配置项（下面会提到）。

上面是默认的命名规则，并且放在src/main/resources下面即可。
如果你即想完全掌控日志配置，但又不想用logback.xml作为Logback配置的名字，可以在application.properties配置文件里面通过logging.config属性指定自定义的名字：

```
logging.config=classpath:logging-config.xml
```

日志输出的格式有以下几个选项
* %d{HH: mm:ss.SSS}——日志输出时间。

* %thread——输出日志的进程名字，这在Web应用以及异步任务处理中很有用。

* %-5level——日志级别，并且使用5个字符靠左对齐。

* %logger{36}——日志输出者的名字。

* %msg——日志消息。

* %n——平台的换行符。

## logger在实际使用的时候有两种情况：
先来看一看代码中如何使用：
```
package com.mrbird.controller;

@Controller
public class LearnController {
    private Logger logger = LoggerFactory.getLogger(this.getClass());

    @RequestMapping(value = "/login",method = RequestMethod.POST)
    @ResponseBody
    public Map<String,Object> login(HttpServletRequest request, HttpServletResponse response){
        //日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出。
        logger.trace("日志输出 trace");
        logger.debug("日志输出 debug");
        logger.info("日志输出 info");
        logger.warn("日志输出 warn");
        logger.error("日志输出 error");
        Map<String,Object> map =new HashMap<String,Object>();
        String userName=request.getParameter("userName");
        String password=request.getParameter("password");
        if(!userName.equals("") && password!=""){
            User user =new User(userName,password);
            request.getSession().setAttribute("user",user);
            map.put("result","1");
        }else{
            map.put("result","0");
        }
        return map;
    }
}
```

