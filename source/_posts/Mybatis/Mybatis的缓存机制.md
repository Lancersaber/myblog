---
title: Mybatis缓存机制
time: 2019-11-06
categories: mybatis
tags: mybatis
---

# Mybatis的缓存机制
---

## 1、查询缓存

### 什么是查询缓存？
mybatis提供查询缓存，用于减轻数据压力，提高数据库性能。

mybaits提供一级缓存，和二级缓存。
![20150726164148424.png](https://i.loli.net/2019/11/06/kDpf5uaFsi2XIJT.png)
一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个(内存区域)数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。
一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存也就不存在了。Mybatis默认开启一级缓存。

二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession去操作数据库得到数据会存在二级缓存区域，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。

二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同namespace下的sql语句且向sql中传递参数也相同即最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。Mybatis默认没有开启二级缓存需要在setting全局参数中配置开启二级缓存。

如果缓存中有数据就不用从数据库中获取，大大提高系统性能。

### 一级缓存

#### 一级缓存工作原理
下图是根据id查询用户的一级缓存图解
![20150726164134583.png](https://i.loli.net/2019/11/06/xtQ1Okhenw74ArV.png)

第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。

得到用户信息，将用户信息存储到一级缓存中。

如果sqlSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。

第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。

#### 一级缓存测试
mybatis默认支持一级缓存，不需要在配置文件去配置。

按照上边一级缓存原理步骤去测试。

```
@Test

   public void testCache1() throws Exception{
      SqlSessionsqlSession = sqlSessionFactory.openSession();//创建代理对象
      UserMapperuserMapper = sqlSession.getMapper(UserMapper.class);
      //下边查询使用一个SqlSession
      //第一次发起请求，查询id为1的用户
      Useruser1 = userMapper.findUserById(1);
      System.out.println(user1);
//    如果sqlSession去执行commit操作（执行插入、更新、删除），清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读。    
      //更新user1的信息
      user1.setUsername("测试用户22");
      userMapper.updateUser(user1);
      //执行commit操作去清空缓存
      sqlSession.commit(); 
      //第二次发起请求，查询id为1的用户

      Useruser2 = userMapper.findUserById(1);
      System.out.println(user2);
      sqlSession.close();
   }
```

####  一级缓存应用
正式开发，是将mybatis和spring进行整合开发，事务控制在service中。

一个service方法中包括很多mapper方法调用。
```
service{

         //开始执行时，开启事务，创建SqlSession对象

         //第一次调用mapper的方法findUserById(1)

        

         //第二次调用mapper的方法findUserById(1)，从一级缓存中取数据

        

//aop控制 只要方法结束，sqlSession关闭 sqlsession关闭后就销毁数据结构，清空缓存

         Service结束sqlsession关闭

}
```
如果是执行两次service调用查询相同的用户信息，不走一级缓存，因为Service方法结束，sqlSession就关闭，一级缓存就清空。

## 二级缓存

### 原理

![20150726164234783.png](https://i.loli.net/2019/11/06/7ETSbwqtsj4JGFR.png)
首先开启mybatis的二级缓存。
sqlSession1去查询用户id为1的用户信息，查询到用户信息会将查询数据存储到二级缓存中。
如果SqlSession3去执行相同 mapper下sql，执行commit提交，清空该 mapper下的二级缓存区域的数据。
sqlSession2去查询用户id为1的用户信息，去缓存中找是否存在数据，如果存在直接从缓存中取出数据。
二级缓存与一级缓存区别，二级缓存的范围更大，多个sqlSession可以共享一个UserMapper的二级缓存区域。数据类型仍然为HashMap
UserMapper有一个二级缓存区域（按namespace分，如果namespace相同则使用同一个相同的二级缓存区），其它mapper也有自己的二级缓存区域（按namespace分）。
每一个namespace的mapper都有一个二缓存区域，两个mapper的namespace如果相同，这两个mapper执行sql查询到数据将存在相同的二级缓存区域中。

### 开启二级缓存
mybaits的二级缓存是mapper范围级别，除了在SqlMapConfig.xml设置二级缓存的总开关，还要在具体的mapper.xml中开启二级缓存。
在核心配置文件SqlMapConfig.xml中加入
```
<setting name="cacheEnabled"value="true"/>

<!-- 全局配置参数，需要时再设置 -->

    <settings>
       <!-- 开启二级缓存  默认值为true -->
    <setting name="cacheEnabled" value="true"/>
</settings>

 
cacheEnabled:对在此配置文件下的所有cache 进行全局性开/关设置。默认为false
```

在UserMapper.xml中开启二缓存，UserMapper.xml下的sql执行完成会存储到它的缓存区域（HashMap）。
```
<mapper namespace="cn.hpu.mybatis.mapper.UserMapper">

<!-- 开启本mapper namespace下的二级缓存 -->

<cache></cache>
```

## 调用pojo类实现序列化接口
二级缓存需要查询结果映射的pojo对象实现java.io.Serializable接口实现序列化和反序列化操作，注意如果存在父类、成员pojo都需要实现序列化接口。

pojo类实现序列化接口是为了将缓存数据取出执行反序列化操作，因为二级缓存数据存储介质多种多样，不一定在内存有可能是硬盘或者远程服务器。

