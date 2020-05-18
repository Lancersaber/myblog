---
title: 连接远端的redis服务
time: 2019-11-22
categories: redis
tags: redis
---

# 连接远端的redis服务

根据我的经验，直接连接运行在虚拟机上的redis不会成功，需要修改一个配置。

1、关闭防火墙
systemctl stop firewalld.service

2、修改redis.conf中的两处数据
* bind 127.0.0.1 ===> #bind 127.0.0.1
* protect mode yes===> no