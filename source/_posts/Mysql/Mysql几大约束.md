---
title: Mysql几大约束
time: 2019-11-2
categories: Mysql
tags: Mysql
---

# Mysql几大约束
---

## 1、主键约束
&emsp;&emsp;Mysql单个字段设为主键很简单，怎么设置两个及以上字段为主键呢？
```
create table test 
( 
   name varchar(19), 
   id number, 
   value varchar(10), 
   primary key (name,id) 
) 
```
这样设置的主键称为复合主键，也就由一个以上的字段构成的主键。

## 2、外键约束
&emsp;&emsp;外键用来在两个表的数据之间建立链接，它可以是一列或者多列。一个表可以有一个或多个外键。外键对应的是参照完整性，一个表的外键可以是空值，若不为空值，则每个外键值必须等于另一个表中主键的某个值。
&emsp;&emsp;外键：外键的主要作用是保证数据引用的完整性，定义外键后，不允许删除在另一个表中的具有关联关系的行。
* 主表(父表)：相关联字段中主键所在的表
* 从表(子表)：相关联字段中外键所在的表
语法：
```
创建外键：
[constraint <外键名>] foreign key 字段名1[,字段名2,...] references <主表名> 主键列1【，主键列2,...]

删除外键
alter table tableName drop foreign key foreignKeyName;
```
示例
```
CREATE TABLE sockerteam(
id INT PRIMARY KEY AUTO_INCREMENT,
team_name VARCHAR(25)
);


CREATE TABLE sockerplayer
(
id INT PRIMARY KEY auto_increment,
`name` VARCHAR(25) NOT NULL, 
teamId INT,
CONSTRAINT team_player FOREIGN KEY (teamId) REFERENCES sockerteam(id)
)
```

## 3、非空约束

## 4、唯一性约束
&emsp;&emsp;唯一性约束(Unique Constraint)要求该列唯一，允许为空，但只能出现一个空值。
语法规则如下：
1、在定义完列之后直接指定唯一约束，语法如下
```
字段名 数据类型 UNIQUE
```

2、在定义完所有列之后指定唯一约束
```
[constraint <约束名>] UNIQUE (<字段名>)
```
## 5、默认约束
默认约束指定某列的默认值。
语法：
```
字段名数据类型 default 默认值
```

## 6、设置表的属性值自动增加
默认的，在Mysql中AUTO_INCREMENT的初始值是1，每新增一条记录，字段值自动加1。一个表只能有一个字段使用AUTO_INCREMENT约束，且该字段必须为主键的一部分。
语法规则如下：
```
字段名 数据类型 AUTO_INCREMENT
```