---
title: Docker安装
date: 2019-10-24 23:15:15
categories: docker
tags: docker
---
# Docker安装
---

##Docker的组成

### 镜像
Docker镜像(Image)就是一个只读的模板，镜像可以用来创建Docker容器，一个镜像可以创建多个容器。可以把Docker中的镜像类比为Java中的类。

### 容器
Docker 利用容器(Container)独立运行的一个或一组应用。容器是用镜像创建的运行实例。它可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。可以把容器看作一个简易版的Linux环境和运行在其中的应用程序。容器的定义和镜像几乎一模一样，也是一堆层的统一视角，唯一的区别在于容器的最上面的那一层是可读可写的。

### 仓库
仓库(Repository)是集中存放镜像的地方。
仓库和仓库注册服务器(Registry)。仓库注册服务器上往往存放着多个仓库，每个仓库又包含了多个镜像，每个镜像有不同的标签(tag)。
仓库分为公开仓库和私有仓库。

### Docker的运行流程
Docker本身是一个容器运行载体或称之为管理引擎。我们把应用程序和配置依赖打包形成的一个可交付的运行环境，这个打包好的运行环境就是image镜像文件。只有通过这个镜像文件才能生成Docker容器。image文件可以看做是容器的模板。Docker根据image文件生成容器的实例。

### 安装指令
```
yum update
yum install docker
```

### 阿里云镜像加速
1、登陆https://cr.console.aliyun.com/cn-hangzhou/instances/repositories
2、注册一个账号后进入控制台，然后搜索镜像容器服务，找到镜像加速器
3、根据自己的系统找到对应的文件(/etc/docker/daemon.json)，将文件中的内容修改即可。
```
{
  "registry-mirrors": ["https://lg0skm2p.mirror.aliyuncs.com"]
}
```
配置完成后，还需重启应用
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 利用docker run hello-world 指令进行测试
docker run 镜像名：这个指令先从远程仓库中拉取镜像，然后运行这个镜像的一个实例。