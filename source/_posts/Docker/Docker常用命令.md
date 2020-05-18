---
title: Docker常用命令
date: 2019-10-24 23:15:15
categories: docker
tags: docker
---

# Docker常用命令
---
##Docker帮助命令
```
docker version
docker info
docker --help：docker帮助手册，列出docker的一些常用指令
```

##镜像命令
```
docker images:列出本地镜像
docker search 镜像名:列出仓库中所有可用的镜像
docker pull 镜像名[:tag]：下载镜像,如果不带版本号，默认下载的是latest
docker rmi [-f 强制删除] 唯一镜像名or镜像id:删除目标镜像
```

## 容器命令
启动容器
```
docker run [options] image [command] [arg..]
--name:"容器的新名字"，为容器指定一个名字
-d: 后台运行容器，并返回容器ID，也即启动守护式容器
-i: 以交互模式运行容器，通常与-t同时使用
-t: 为容器重新分配一个伪终端，
-P(大P): 随机端口映射
-p:	指定端口映射，有以下四种格式
	ip：hostPort:containerPort
	ip::containerPort
	hostPort:containerPort(常用)
	containerPort
```

列出当前所有正在运行的容器
```
docker ps [options]
-a:列出当前正在运行的容器+历史上运行过的
-l:显示最近创建的容器
-n：显示最近n个创建的容器
-q：静默模式，只显示容器编号
```

启动容器
```
docker start 容器ID或者容器名
```

停止容器
```
docker stop 容器ID或者容器名
```

强制关闭
```
docker kill 容器ID或者容器名
```

删除已停止的容器
```
docker rm 容器ID
```

查看容器日志
```
docker logs -f -t --tail 容器ID
-t: 加入时间戳
-f：跟随最新的日志打印
--tail：数字 显示最后多少条
```

查看容器内运行的进程
```
docker top 容器ID
```

查看容器内部细节
```
docker inspect 容器ID
```

进入正在运行的容器并以命令行交互
```
docker exec -it 容器ID bashShell
重新进入docker attach 容器ID
```
这两种方式的区别
* attach: 直接进入容器启动命令的终端，不会启动新的进程
* exec：是在容器中打开新的终端，并且可以启动新的进程

从容器中拷贝文件到主机上
```
docker cp 容器ID:容器内路径 目的主机路径
```

提交容器副本使之成为一个新的镜像
```
docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]
```