---
title: Springboot整合Activiti
time: 2019-11-04
categories: Activiti
tags: Activiti
---

# Springboot整合Activiti
---

## 入门
Spring Boot完全是关于配置的约定。首先，只需将spring-boot-starters-basic依赖项添加到您的项目中。以Maven为例
```
<dependency>
	<groupId>org.activiti</groupId>
	<artifactId>activiti-spring-boot-starter-basic</artifactId>
	<version>${activiti.version}</version>
</dependency>
```

这就是所需要的。该依赖关系将向类路径中传递正确的Activiti和Spring依赖关系。您现在可以编写Spring Boot应用程序：
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan
@EnableAutoConfiguration
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
```

Activiti需要一个数据库来存储其数据。如果您运行上面的代码，它将为您提供一条提示性异常消息，您需要将数据库驱动程序依赖项添加到类路径中。现在，添加H2数据库依赖项：
```
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<version>1.4.183</version>
</dependency>
```
实际中换成自己使用的数据库即可。


## 更改数据库和连接池

如上所述，Spring Boot是关于配置的约定。默认情况下，通过仅在类路径上使用H2，它创建了一个内存数据源，并将其传递给Activiti流程引擎配置。

要更改数据源，只需提供Datasource bean即可覆盖默认值。我们在这里使用DataSourceBuilder类，它是Spring Boot的帮助器类。如果在类路径上有Tomcat，HikariCP或Commons DBCP，则将选择其中之一（首先按Tomcat的顺序）。例如，要切换到MySQL数据库：
```
@Bean
public DataSource database() {
    return DataSourceBuilder.create()
        .url("jdbc:mysql://127.0.0.1:3306/activiti-spring-boot?characterEncoding=UTF-8")
        .username("alfresco")
        .password("alfresco")
        .driverClassName("com.mysql.jdbc.Driver")
        .build();
}
```

对数据源的配置也可以写在Spring的配置文件中
```
spring:
  datasource:
    url: jdbc:mysql://49.234.9.177:3306/activiti_test?nullCatalogMeansCurrent=true
    username: root
    password: 3468989340
```
注意有的版本后面必须加上?nullCatalogMeansCurrent=true，否则会报错。数据源会自动读取配置文件的内容

## 添加配置类

```
package com.example.demo.config;

import org.activiti.engine.*;
import org.activiti.engine.impl.cfg.ProcessEngineConfigurationImpl;
import org.activiti.spring.ProcessEngineFactoryBean;
import org.activiti.spring.SpringProcessEngineConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.PlatformTransactionManager;

import javax.sql.DataSource;

/**
 * @author: Lancer
 * @date: 2019/11/10
 */
@Configuration
public class ActivitiConfig {

    @Bean
    public ProcessEngineConfiguration processEngineConfiguration(DataSource dataSource, PlatformTransactionManager transactionManager){
        SpringProcessEngineConfiguration processEngineConfiguration = new SpringProcessEngineConfiguration();
        processEngineConfiguration.setDataSource(dataSource);
        processEngineConfiguration.setDatabaseSchemaUpdate("true");
        processEngineConfiguration.setDatabaseType("mysql");

        processEngineConfiguration.setTransactionManager(transactionManager);

        //流程图字体
//        processEngineConfiguration.setActiviti("宋体");
//        processEngineConfiguration.setAnnotationFontName("宋体");
//        processEngineConfiguration.setLabelFontName("宋体");

        return processEngineConfiguration;
    }


    //流程引擎，与spring整合使用factoryBean
    @Bean
    public ProcessEngineFactoryBean processEngine(ProcessEngineConfiguration processEngineConfiguration){
        ProcessEngineFactoryBean processEngineFactoryBean = new ProcessEngineFactoryBean();
        processEngineFactoryBean.setProcessEngineConfiguration((ProcessEngineConfigurationImpl) processEngineConfiguration);
        return processEngineFactoryBean;
    }


    //八大接口
    @Bean
    public RepositoryService repositoryService(ProcessEngine processEngine){
        return processEngine.getRepositoryService();
    }

    @Bean
    public RuntimeService runtimeService(ProcessEngine processEngine){
        return processEngine.getRuntimeService();
    }

    @Bean
    public TaskService taskService(ProcessEngine processEngine){
        return processEngine.getTaskService();
    }

    @Bean
    public HistoryService historyService(ProcessEngine processEngine){
        return processEngine.getHistoryService();
    }

//    @Bean
//    public FormService formService(ProcessEngine processEngine){
//        return processEngine.getFormService();
//    }
//
//    @Bean
//    public IdentityService identityService(ProcessEngine processEngine){
//        return processEngine.getIdentityService();
//    }

    @Bean
    public ManagementService managementService(ProcessEngine processEngine){
        return processEngine.getManagementService();
    }

    @Bean
    public DynamicBpmnService dynamicBpmnService(ProcessEngine processEngine){
        return processEngine.getDynamicBpmnService();
    }

    //八大接口 end

}

```
在ProcessEngineConfiguration类中可以配置很多东西
![activiti1.PNG](https://i.loli.net/2019/11/10/dIQbE9ZsG4gxwBV.png)

注意：processEngine对象已经在ProcessEngineConfigurationImpl中被创建
![atciviti2.PNG](https://i.loli.net/2019/11/10/vbf7Ysr3LjN4TIB.png)