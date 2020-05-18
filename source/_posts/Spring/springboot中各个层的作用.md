---
title: springboot中各个层的作用
categories: spring
---
# springboot中各个层的作用
---
## Model层
+ model层即数据库实体层，也被称为entity层，pojo层。
+ 一般数据库一张表对应一个实体类，类属性同表字段一一对应。

## dao层
+ dao层即数据持久层，也被称为mapper层。
+ dao层的作用为访问数据库，向数据库发送sql语句，完成数据的增删改查任务。

## service层
+ service层即业务逻辑层
+ service层的作用为完成功能涉及
+ service层调用dao层提供的方法完成特定的功能

## controller层
+ controller层即控制层
+ controller的功能为响应用户的请求
+ controller层负责前后端交互，接受前端请求，调用service层得到返回的数据，再将数据返回给页面。