---
title: Spring Boot –如何知道使用哪个连接池
time: 2019-11-09
categories: spring
tags: spring
---

# Spring Boot –如何知道使用哪个连接池？
---
在Spring Boot中，@Autowired一个javax.sql.DataSource，您将知道当前正在运行的应用程序正在使用哪个数据库连接池。

## 1.测试默认
Spring Boot示例打印一个 javax.sql.DataSource

```
package com.mkyong;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import javax.sql.DataSource;

@SpringBootApplication
public class SpringBootConsoleApplication implements CommandLineRunner {

    @Autowired
    DataSource dataSource;

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SpringBootConsoleApplication.class, args);
    }

    @Override
    public void run(String... args) throws Exception {

        System.out.println("DATASOURCE = " + dataSource);

    }

}
```

输出，Spring Boot默认使用Tomcat池。
```
DATASOURCE = org.apache.tomcat.jdbc.pool.DataSource@7c541c15...
```

## 2.测试HikariCP
要切换到另一个连接池，例如HikariCP，只需排除默认值，并将HikariCP包含在类路径中即可。
```
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>org.apache.tomcat</groupId>
                    <artifactId>tomcat-jdbc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <!-- connection pools -->
        <dependency>
            <groupId>com.zaxxer</groupId>
            <artifactId>HikariCP</artifactId>
            <version>2.6.0</version>
        </dependency>
```

输出值
```
DATASOURCE = HikariDataSource (HikariPool-1)
```