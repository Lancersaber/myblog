---
title: Mysql中的一对多和多对多
time: 2019-11-2
categories: Mysql
tags: Mysql
---

# Mysql中的一对多和多对多
---

## 关联映射：一对多/多对一
存在最普遍的映射关系，简单来讲就如球员与球队的关系；
一对多：从球队角度来说一个球队拥有多个球员 即为一对多
多对一：从球员角度来说多个球员属于一个球队 即为多对一数据表间一对多关系如下图：
![15234704-7cacb7b37f794c0e91f92cae94f03753.jpg](https://i.loli.net/2019/11/02/7UbepvPs6gDkLtd.jpg)

sql语句:
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

## 关联映射：多对多
多对多关系也很常见，例如学生与选修课之间的关系，一个学生可以选择多门选修课，而每个选修课又可以被多名学生选择。
数据库中的多对多关联关系一般需采用中间表的方式处理，将多对多转化为两个一对多。
数据表间多对多关系如下图：
![15235342-153ef6a865fb47f391eb66f443e21370.jpg](https://i.loli.net/2019/11/02/cDXvu3BKI5FVonM.jpg)
