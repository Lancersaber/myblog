---
title: Springboot通用Mapper和PageHelper
time: 2019-11-2
categories: spring
tags: spring
---

# Springboot通用Mapper和PageHelper
---
如果项目中使用到了MyBatis框架，那么使用通用Mapper和PageHelper分页插件将极大的简化我们的操作。通用Mapper可以简化对单表的CRUD操作，PageHelper分页插件可以帮我们自动拼接分页SQL，并且可以使用MyBatis Geneator来自动生成实体类，Mapper接口和Mapper xml代码，非常的方便。插件地址及作者链接https://gitee.com/free。

## 引入依赖
这里使用Spring Boot来构建，可参考Spring-Boot中使用Mybatis.html搭建一个Spring boot + MyBatis的框架，然后在pom中引入：
```
<!-- mybatis -->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
<!-- 通用mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-spring-boot-starter</artifactId>
    <version>1.1.5</version>
</dependency>
<!-- pagehelper 分页插件 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.3</version>
</dependency>
```
接着在pom中配置MyBatis Geneator：
```
<build>
    <plugins>
        <plugin>
            <groupId>org.mybatis.generator</groupId>
            <artifactId>mybatis-generator-maven-plugin</artifactId>
            <version>1.3.5</version>
            <dependencies>
                <dependency>
                    <!-- 数据库连接驱动 -->
                    <groupId>com.oracle</groupId>
                    <artifactId>ojdbc6</artifactId>
                    <version>6.0</version>
                </dependency>
                <dependency>
                    <groupId>tk.mybatis</groupId>
                    <artifactId>mapper</artifactId>
                    <version>3.4.0</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <id>Generate MyBatis Artifacts</id>
                    <phase>package</phase>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <!--允许移动生成的文件 -->
                <verbose>true</verbose>
                <!-- 是否覆盖 -->
                <overwrite>true</overwrite>
                <!-- 自动生成的配置 -->
                <configurationFile>src/main/resources/mybatis-generator.xml</configurationFile>
            </configuration>
        </plugin>
    </plugins>
</build>
```
src/main/resources/mybatis-generator.xml为生成器的配置，下文会介绍到。

## 配置插件
在Spring Boot配置文件application.yml中配置MyBatis：
```
mybatis:
  # type-aliases扫描路径
  type-aliases-package: com.springboot.bean
  # mapper xml实现扫描路径
  mapper-locations: classpath:mapper/*.xml
  property:
    order: BEFORE
```

接下来开始配置插件。

### 配置通用Mapper
在Spring Boot配置文件application.yml中配置通用Mapper：
```
#mappers 多个接口时逗号隔开
mapper:
  mappers: com.springboot.config.MyMapper
  not-empty: false
  identity: oracle
```

关于参数的说明，参考https://gitee.com/free/Mapper/blob/master/wiki/mapper3/2.Integration.md中的可配参数介绍。

除此之外，我们需要定义一个MyMapper接口：

```
import tk.mybatis.mapper.common.Mapper;
import tk.mybatis.mapper.common.MySqlMapper;

public interface MyMapper<T> extends Mapper<T>, MySqlMapper<T> {
	
}
```

值得注意的是，该接口不能被扫描到，应该和自己定义的Mapper分开。自己定义的Mapper都需要继承这个接口。

### 配置PageHelper
在Spring Boot配置文件application.yml中配置通用配置PageHelper：
```
#pagehelper
pagehelper: 
  helperDialect: oracle
  reasonable: true
  supportMethodsArguments: true
  params: count=countSql
```

参数相关说明参考https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md中的分页插件参数介绍

### 配置Geneator*
在路径src/main/resources/下新建mybatis-generator.xml：
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
    PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
    "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <context id="oracle" targetRuntime="MyBatis3Simple" defaultModelType="flat">

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <!-- 该配置会使生产的Mapper自动继承MyMapper -->
            <property name="mappers" value="com.springboot.config.MyMapper" />
            <!-- caseSensitive默认false，当数据库表名区分大小写时，可以将该属性设置为true -->
            <property name="caseSensitive" value="false"/>
        </plugin>

        <!-- 阻止生成自动注释 -->
        <commentGenerator>
            <property name="javaFileEncoding" value="UTF-8"/>
            <property name="suppressDate" value="true"/>
            <property name="suppressAllComments" value="true"/>
        </commentGenerator>

        <!-- 数据库链接地址账号密码 -->
        <jdbcConnection 
            driverClass="oracle.jdbc.driver.OracleDriver"
            connectionURL="jdbc:oracle:thin:@localhost:1521:ORCL"
            userId="scott"
            password="6742530">
        </jdbcConnection>

        <javaTypeResolver>
            <property name="forceBigDecimals" value="false"/>
        </javaTypeResolver>

        <!-- 生成Model类存放位置 -->
        <javaModelGenerator targetPackage="com.springboot.bean" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
            <property name="trimStrings" value="true"/>
        </javaModelGenerator>

        <!-- 生成映射文件存放位置 -->
        <sqlMapGenerator targetPackage="mapper" targetProject="src/main/resources">
            <property name="enableSubPackages" value="true"/>
        </sqlMapGenerator>

        <!-- 生成Dao类存放位置 -->
        <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件的代码
            type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
            type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.springboot.mapper" targetProject="src/main/java">
            <property name="enableSubPackages" value="true"/>
        </javaClientGenerator>

        <!-- 配置需要生成的表 -->
        <table tableName="T_USER" domainObjectName="User" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
            <generatedKey column="id" sqlStatement="oralce" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

更详细的说明可参考链接：http://blog.csdn.net/isea533/article/details/42102297。

## 代码生成
配置好MyBatis Geneator后，在eclipse中运行命令mybatis-generator:generate：
