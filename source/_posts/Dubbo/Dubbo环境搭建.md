---
title: Dubbo环境搭建
categories: Dubbo
tags: Dubbo
---
# Dubbo环境搭建
---
## 安装注册中心
注册中心有多种，官网推荐使用zookeeper。我们在windows安装，供学习使用。下载zookeeper的zip包后，启动bin目录下的zkServer.cmd即可。

## 安装监控中心
去dubbo的github地址下载incubator-dubbo-ops-master这个项目，将这个项目使用maven打成jar包并运行。注意需要在配置文件中将zookeeper的地址更改为正确的地址。然后运行打包好的jar文件即可。最后可以访问localhost:7001,默认账号和密码都是root。（zookeeper服务器必须处于启动状态）