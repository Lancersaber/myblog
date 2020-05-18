---
title: 在springboot中引入mybatis-plus
time: 2020/1/14
categories: mybatis-plus
tags: mybatis-plus
---

## 添加依赖
```
 <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.3.0</version>
    </dependency>
```

## 配置
注意这里需要配置MapperScan注解
```
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

```
# DataSource Config
spring:
  datasource:
    driver-class-name: org.h2.Driver
    schema: classpath:db/schema-h2.sql
    data: classpath:db/data-h2.sql
    url: jdbc:h2:mem:test
    username: root
    password: test
```

## 编码
```
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```
编写Mapper类 UserMapper.java
```
public interface UserMapper extends BaseMapper<User> {

}
```

