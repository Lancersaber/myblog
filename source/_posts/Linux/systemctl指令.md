---
title: 每天一个Linux指令-systemctl
date: 2019-10-24 23:15:15
categories: Linux
tags: Linux
---

# 每天一个Linux指令-systemctl
---
一般来说，服务的启动有两个阶段，一个是*开机的时候设定要不要启动这个服务*，以及*你现在要不要启动这个服务*。

## 语法
```
systemctl [command] [unit]
```

## command
```
start:立刻启动后面接的unit
stop：立刻关闭后面接的unit
restart：立刻关闭后重启后面接的unit，亦即执行stop再start的意思
reload:不关闭后面接的unit的情况下，重载配置文件，让设定生效
enable：设定下次开机时，后面接的unit会被启动
disable：设定下次开机时，后面接的unit不会被启动
status：目前后面接的这个unit的状态，会列出有没有正在执行、开机预设执行否、登录等信息
is-active:目前有没有正在运行中
is-enable:开机有没有预设要启用这个unit
```

## 透过systemctl观察系统上所有的服务
---
### 语法
```
systemctl [command] [--type=TYPE] [--all]

command:
	list-units	:依据unit列出目前有启动的unit，若加上--all才会列出没启动的
	list-unit-files:
--type=TYPE:就是unit type，主要有service，socket，target
```

## systemctl指令的几个应用

### 设置Target
```
systemctl [command] [unit.target]
选项与参数
command:
	get-default:取得目前的target
	set-default:设定后面接的target成为默认的target
	isolate	   :切换到后面接的模式
```

### 设置模式
```
systemctl poweroff:系统关机
systemctl reboot  ：重新启动
systemctl suspend ：进入暂停模式
systemctl hibernate：进入休眠模式
systemctl rescue	：强制进入救援模式
systemctl emergency：强制进入紧急救援模式
```