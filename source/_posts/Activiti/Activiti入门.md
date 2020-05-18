---
title: Activiti入门
time: 2019-11-02
categories: Activiti
tags: Activiti
---

# Activiti入门
---

## 简介
Activiti是领先的轻量级，以Java为中心的开源BPMN引擎，可满足现实世界中的流程自动化需求。Activiti Cloud现在是新一代的业务自动化平台，提供了一组旨在在分布式基础架构上运行的云原生模块。

## 引入Activiti的依赖
```
<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-engine</artifactId>
    <version>7.1.0.M4</version>
</dependency>

<dependency>
    <groupId>org.activiti</groupId>
    <artifactId>activiti-spring</artifactId>
    <version>7.1.0.M4</version>
</dependency>
```

## 配置

### 配置一个ProcessEngine
Activiti流程引擎通过名为的XML文件进行配置activiti.cfg.xml。请注意，如果您使用的是Spring风格的流程引擎构建，则此方法不适用。

获得的最简单方法ProcessEngine是使用org.activiti.engine.ProcessEngines类：
```
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
```

这将activiti.cfg.xml在类路径上查找文件，并根据该文件中的配置构造一个引擎。以下代码段显示了示例配置。以下各节将详细介绍配置属性。
```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">

    <property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
    <property name="jdbcDriver" value="org.h2.Driver" />
    <property name="jdbcUsername" value="sa" />
    <property name="jdbcPassword" value="" />

    <property name="databaseSchemaUpdate" value="true" />

    <property name="asyncExecutorActivate" value="false" />

    <property name="mailServerHost" value="mail.my-corp.com" />
    <property name="mailServerPort" value="5025" />
  </bean>

</beans>
```
请注意，配置XML实际上是Spring配置。*这并不意味着Activiti只能在Spring环境中使用！*我们只是在内部利用Spring的解析和依赖注入功能来构建引擎。

也可以使用配置文件以编程方式创建ProcessEngineConfiguration对象。也可以使用其他bean id（例如，参见第3行）。
```
ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault();
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource);
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);
```

也可以不使用配置文件，并创建一个基于默认配置（见所支持的不同类的更多信息）。
```
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```

所有这些ProcessEngineConfiguration.createXXX()方法都返回一个a ProcessEngineConfiguration，可以根据需要对其进行进一步调整。调用buildProcessEngine()操作后，将创建一个ProcessEngine：
```
ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
  .setAsyncExecutorActivate(false)
  .buildProcessEngine();
```

### ProcessEngineConfiguration bean
在activiti.cfg.xml必须包含有ID的Bean 'processEngineConfiguration'。
```
<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
```

然后，此bean用于构造ProcessEngine。有多个可用的类可用于定义processEngineConfiguration。这些类表示不同的环境，并相应地设置默认值。最佳做法是选择与您的环境最匹配的类，以最大程度地减少配置引擎所需的属性数量。当前提供以下类（将来的版本中将有更多类）：

* org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration：以独立方式使用流程引擎。Activiti将负责交易。默认情况下，仅在引擎启动时检查数据库（如果没有Activiti架构或架构版本不正确，则会引发异常）。

* org.activiti.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration：这是用于单元测试的便捷类。Activiti将负责交易。默认情况下使用H2内存数据库。引擎启动和关闭时，将创建并删除数据库。使用此功能时，可能不需要其他配置（例如，使用作业执行程序或邮件功能时除外）。

* org.activiti.spring.SpringProcessEngineConfiguration：在Spring环境中使用流程引擎时使用。有关更多信息，请参见Spring集成部分。

* org.activiti.engine.impl.cfg.JtaProcessEngineConfiguration：当引擎以JTA事务在独立模式下运行时使用。

### 数据库配置
有两种方法可以配置Activiti引擎将使用的数据库。第一种选择是定义数据库的JDBC属性：
* jdbcUrl：数据库的JDBC URL。

* jdbcDriver：针对特定数据库类型的驱动程序的实现。

* jdbcUsername：用于连接数据库的用户名。

* jdbcPassword：连接数据库的密码。

基于提供的JDBC属性构造的数据源将具有默认的MyBatis连接池设置。可以选择以下属性来调整该连接池（取自MyBatis文档）：

* jdbcMaxActiveConnections：连接池在任何时候最多可以包含的活动连接数。默认值是10。

* jdbcMaxIdleConnections：连接池在任何时候最多可以包含的空闲连接数。

* jdbcMaxCheckoutTime：在强制返回连接之前，可以从连接池中检出连接的时间（以毫秒为单位）。默认值为20000（20秒）。

* jdbcMaxWaitTime：这是一个较低级别的设置，它使池有机会打印日志状态，并在连接花费异常长的情况下重新尝试获取连接（以避免在池配置错误时永久性地失败）。是20000（20秒）。

示例数据库配置：
```
<property name="jdbcUrl" value="jdbc:h2:mem:activiti;DB_CLOSE_DELAY=1000" />
<property name="jdbcDriver" value="org.h2.Driver" />
<property name="jdbcUsername" value="sa" />
<property name="jdbcPassword" value="" />
```

我们的基准测试表明，在处理大量并发请求时，MyBatis连接池不是最有效或弹性最大的连接池。因此，建议给我们一个javax.sql.DataSource实现并将其注入到流程引擎配置中（例如DBCP，C3P0，Hikari，Tomcat连接池等）：

```
<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" >
  <property name="driverClassName" value="com.mysql.jdbc.Driver" />
  <property name="url" value="jdbc:mysql://localhost:3306/activiti" />
  <property name="username" value="activiti" />
  <property name="password" value="activiti" />
  <property name="defaultAutoCommit" value="false" />
</bean>

<bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">

    <property name="dataSource" ref="dataSource" />
    ...
```
请注意，Activiti并未附带允许定义此类数据源的库。因此，您必须确保这些库位于类路径中。
不管您使用的是JDBC还是数据源方法，都可以设置以下属性：

databaseType：通常无需指定此属性，因为将从数据库连接元数据中自动分析该属性。仅在自动检测失败的情况下才应指定。可能的值：{h2，mysql，oracle，postgres，mssql，db2}。此设置将确定将使用哪些创建/删除脚本和查询。有关支持哪些类型的概述，请参见受支持的数据库部分。

databaseSchemaUpdate：允许设置策略以在流程引擎启动和关闭时处理数据库架构。

false （默认值）：在创建流程引擎时，对照库检查数据库模式的版本，如果版本不匹配，则引发异常。

true：在构建流程引擎时，将执行检查并在必要时执行模式的更新。如果该模式不存在，则会创建它。

create-drop：在创建流程引擎时创建架构，并在关闭流程引擎时删除架构。


### 数据库表的说明
* ACT_RE_ \*：RE代表repository。具有此前缀的表包含静态信息，例如流程定义和流程资源（图像，规则等）。

* ACT_RU_ \*：RU代表runtime。这些是运行时表，其中包含流程实例，用户任务，变量，作业等的运行时数据。Activiti仅在流程实例执行期间存储运行时数据，并在流程实例结束时删除记录。这样可以使运行时表较小而又快速。

* ACT_ID_ \*：ID代表identity。这些表包含身份信息，例如用户，组等。

* ACT_HI_ \*：HI代表history。这些表包含历史数据，例如过去的流程实例，变量，任务等。

* ACT_GE_ \*：general数据，用于各种用例中。

### 数据库升级
在运行升级之前，请确保已使用数据库备份功能对数据库进行了备份。

默认情况下，每次创建流程引擎时都会执行版本检查。这通常在应用程序或Activiti Web应用程序启动时发生一次。如果Activiti库注意到库版本与Activiti数据库表的版本之间存在差异，则将引发异常。

要进行升级，必须首先将以下配置属性放入activiti.cfg.xml配置文件中：
```
<beans >

  <bean id="processEngineConfiguration" class="org.activiti.engine.impl.cfg.StandaloneProcessEngineConfiguration">
    <!-- ... -->
    <property name="databaseSchemaUpdate" value="true" />
    <!-- ... -->
  </bean>

</beans>
```

另外，在类路径中包括适合您的数据库的数据库驱动程序。在应用程序中升级Activiti库。或启动新版本的Activiti并将其指向包含旧版本的数据库。随着databaseSchemaUpdate设置为true，Activiti的会自动将DB模式，当它注意到图书馆和数据库架构不同步的第一次升级到新版本。

或者，您也可以运行升级DDL语句。也可以运行Activiti下载页面上的升级数据库脚本

