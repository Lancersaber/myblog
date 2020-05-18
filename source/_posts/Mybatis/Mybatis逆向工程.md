---
title: Mybatis逆向工程
categories: mybatis
tags: mybatis逆向工程
---

# Mybatis逆向工程
---
MyBatis生成器（MBG）是MyBatis MyBatis 和iBATIS的代码生成器。它将为MyBatis的所有版本以及版本2.2.0之后的iBATIS生成代码。它将内省一个数据库表（或多个表），并将生成可用于访问表的工件。这减轻了设置对象和配置文件以与数据库表进行交互的麻烦。MBG试图对简单CRUD（创建，检索，更新，删除）的大部分数据库操作产生重大影响。您仍将需要手工编写SQL和对象代码以进行联接查询或存储过程。

## 导入依赖
```
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.2</version>
</dependency>
```

## 创建配置文件
创建一个xml文件，名字可以任取
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <!--<classPathEntry location="/Program Files/IBM/SQLLIB/java/db2java.zip" />-->

    <!--
    id 此上下文的唯一标识符。该值将在某些错误消息中使用。
    targetRuntime:此属性用于为生成的代码指定运行时目标。该属性支持以下特殊值：MyBatis3,MyBatis3Simple
    -->
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!--
            配置连接数据库需要的属性值
        -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://49.234.9.177:3306/mybatis"
                        userId="root"
                        password="3468989340">
        </jdbcConnection>

        <!--<javaTypeResolver >-->
            <!--<property name="forceBigDecimals" value="false" />-->
        <!--</javaTypeResolver>-->

        <!--
            <javaModelGenerator>元素用于定义Java模型生成器的属性。Java模型生成器将构建与自省表匹配的主键类，记录类和“按示例查询”类。此元素是<context>元素的必需子元素。
            targetPackage:This is the package where the generated classes will be placed.
            targetProject:This is used to specify a target project for the generated objects.
        -->
        <javaModelGenerator targetPackage="com.lancer.bean" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
            <property name="trimStrings" value="true" />
        </javaModelGenerator>


        <!--
            配置sqlMapper存放的位置等一些配置
        -->
        <sqlMapGenerator targetPackage="com.lancer.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>

        <!--
            配置sql的接口文件
        -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.lancer.Interface"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>

        <!--<table tableName="test" domainObjectName="Test" >-->
            <!--<property name="useActualColumnNames" value="true"/>-->
            <!--<generatedKey column="ID" sqlStatement="DB2" identity="true" />-->
            <!--<columnOverride column="DATE_FIELD" property="startDate" />-->
            <!--<ignoreColumn column="FRED" />-->
            <!--<columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />-->
        <!--</table>-->

        <!--
            需要生成的表名，以及映射到哪个类的类名
            中间还可以进行一些其他的配置
        -->
        <table tableName="tb_dept1" domainObjectName="Dept" >
            <property name="useActualColumnNames" value="true"/>
            <generatedKey column="ID" sqlStatement="DB2" identity="true" />
            <columnOverride column="DATE_FIELD" property="startDate" />
            <ignoreColumn column="FRED" />
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
        </table>

        <table tableName="tb_emp5" domainObjectName="Emp" >
            <property name="useActualColumnNames" value="true"/>
            <generatedKey column="ID" sqlStatement="DB2" identity="true" />
            <columnOverride column="DATE_FIELD" property="startDate" />
            <ignoreColumn column="FRED" />
            <columnOverride column="LONG_VARCHAR_FIELD" jdbcType="VARCHAR" />
        </table>

    </context>
</generatorConfiguration>
```
对各个标签的解释(为了避免显示问题,两边不加尖括号了)
* context元素用于指定生成一组对象的环境。子元素用于指定要连接的数据库，要生成的对象的类型以及要进行内省的表。可以在generatorConfiguration 元素内列出多个context元素， 以允许在MyBatis Generator（MBG）的同一运行中从不同的数据库或具有不同的生成参数生成对象。


## 让配置文件生效
使用Java代码的方式
```
    @Test
    public void test08() throws IOException, XMLParserException, InvalidConfigurationException, SQLException, InterruptedException {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        //这里将配置文件的地址传入即可，其余部分不变
        File configFile = new File("mbg.xml");
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = cp.parseConfiguration(configFile);
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
    }
```

## 使用逆向生成后的代码
```
        //指定要进行去重查询
        userBankCardExample.setDistinct(true);
        //指定查询条件
        userBankCardExample.createCriteria().andUserIdEqualTo(iptUserBankCard.getUserId());
        //调用查询方法
        List<UserBankCard> userBankCadList = userBankCardDao.selectByExample(userBankCardExample);
```


## 常出现的错误
```
org.apache.ibatis.binding.BindingException: Invalid bound statement (not found):
即在mybatis中dao接口与mapper配置文件在做映射绑定的时候出现问题，简单说，就是接口与xml要么是找不到，要么是找到了却匹配不到。
```

## 必不可少的配置
```
spring:
  datasource:
    username: chen
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://116.62.189.4:3306/StudentManager
mybatis:
  mapper-locations: classpath:sqlXml/*.xml
```


# 使用插件实施逆向工程

## 引入插件
```
<plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.5</version>
    <configuration>
        <verbose>true</verbose>
        <overwrite>true</overwrite>
    </configuration>
</plugin>
```


## 注意点
1、注意配置文件名必须为generatorConfig.xml且必须在resource目录下
2、注意配置
<classPathEntry location="D:\编程辅助文档\apache-maven-3.6.1-bin\apache-maven-3.6.1\repository\mysql\mysql-connector-java\8.0.19\mysql-connector-java-8.0.19.jar" />
这是链接mysql所需要的jar包位置。如果采用java代码方式启动则可以不配置这一行。

3、配置完成后点开maven的mybatis generator插件运行即可。