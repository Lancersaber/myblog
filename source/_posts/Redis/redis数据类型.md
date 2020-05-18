---
title: redis数据类型
time: 2019-11-12
categories: redis
tags: redis
---

# redis数据类型
---
Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

## String（字符串）
string 是 redis 最基本的类型，你可以理解成与 Memcached 一模一样的类型，一个 key 对应一个 value。
string 类型是二进制安全的。意思是 redis 的 string 可以包含任何数据。比如jpg图片或者序列化的对象。
string 类型是 Redis 最基本的数据类型，string 类型的值最大能存储 512MB。

实例：
```
redis 127.0.0.1:6379> SET runoob "菜鸟教程"
OK
redis 127.0.0.1:6379> GET runoob
"菜鸟教程"
```
在以上实例中我们使用了 Redis 的 SET 和 GET 命令。键为 runoob，对应的值为 菜鸟教程。

注意：一个键最大能存储 512MB。

