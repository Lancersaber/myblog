---
title: Springboot Devtools热部署
平日里开发项目中，修改了Java代码或者配置文件的时候，必须手动重启项目才能生效。所谓的热部署就是在你修改了后端代码后不需要手动重启，工具会帮你快速的自动重启是修改生效。其深层原理是使用了两个ClassLoader，一个Classloader加载那些不会改变的类（第三方Jar包），另一个ClassLoader加载会更改的类，称为restart ClassLoader，这样在有代码更改的时候，原来的restart ClassLoader 被丢弃，重新创建一个restart ClassLoader，由于需要加载的类相比较少，所以实现了较快的重启时间。

本文将介绍如何通过使用Spring-Boot-devtools来实现Spring Boot项目的热部署。IDE使用的是Eclipse Oxygen，并且使用Maven构建。
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

## 测试热部署
在入口类中添加一个方法，用于热部署测试：
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@SpringBootApplication
public class DemoApplication {
    @RequestMapping("/")
    String index() {
        return "hello spring boot";
    }
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

启动项目访问http://localhost:8080/，页面输出hello spring boot。

将方法的返回值修改为hello world并在保存的瞬间，应用便重启好了，刷新页面，内容也将得到更改。

## 所有配置
下面是所有Devtools在Spring Boot中的可选配置:
```
# Whether to enable a livereload.com-compatible server.
spring.devtools.livereload.enabled=true 

# Server port.
spring.devtools.livereload.port=35729 

# Additional patterns that should be excluded from triggering a full restart.
spring.devtools.restart.additional-exclude= 

# Additional paths to watch for changes.
spring.devtools.restart.additional-paths= 

# Whether to enable automatic restart.
spring.devtools.restart.enabled=true

# Patterns that should be excluded from triggering a full restart.
spring.devtools.restart.exclude=META-INF/maven/**,META-INF/resources/**,resources/**,static/**,public/**,templates/**,**/*Test.class,**/*Tests.class,git.properties,META-INF/build-info.properties

# Whether to log the condition evaluation delta upon restart.
spring.devtools.restart.log-condition-evaluation-delta=true 

# Amount of time to wait between polling for classpath changes.
spring.devtools.restart.poll-interval=1s 

# Amount of quiet time required without any classpath changes before a restart is triggered.
spring.devtools.restart.quiet-period=400ms 

# Name of a specific file that, when changed, triggers the restart check. If not specified, any classpath file change triggers the restart.
spring.devtools.restart.trigger-file=
```