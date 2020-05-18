---
title: 每天一个Linux指令-ps
date: 2019-10-24 23:15:15
categories: Linux
tags: Linux
---

# 每天一个Linux指令-ps
---
ps指令是用于查阅系统上面正在运行的进程(ps是截取一个时间点的进程状态，而后面介绍的top可以持续侦测进程运作的状态)。用法
```
ps [options]
```
选项与参数
* -A：所有的process均显示出来，与-e具有同样的效用
* -a：不与terminal有关的所有process
* -u：有效使用者和相关的process
* x ：通常与a这个参数一起使用，可列出教完整的信息
输出格式规划
* l ：较长、较详细地将该PID的信息列出
* j ：工作的格式(jobs format)
* -f :做一个更为完整的输出

## 仅观察自己的bash相关进程
```
ps -l
```
![](https://i.loli.net/2019/10/26/cC4X1lVZorFujxB.png)
系统整体的进程运作是非常多的，但如果使用ps -l则仅列出你的操作环境(bash)有关的进程而已，亦即最上层的父进程会是你自己的bash而没有沿伸到systemd这支进程。ps -l显示出来的资料包含以下内容：
* F:代表这个进程旗标(process flags),说明这个进程的总结权限，常见号码有
	+ 若为4表示此进程的权限为root
	+ 若为1则表示此进程仅复制(fork)而没有实际执行(exec)

* S:代表这个进程的状态(STAT),主要的状态有
	+ R(Running): 该程序正在运作中
	+ S(Sleep)  : 该程序目前正在睡眠状态(idle)，但可以被唤醒(signal)
	+ D:		: 不可被唤醒的睡眠状态，通常这只程序可能正在等待I/O的情况
	+ T:		：停止状态(stop),可能是在工作控制(背景暂停)或除错状态
	+ Z:		：僵尸状态，进程已经终止但却无法被移除到内存外
* UID/PID/PPID：代表【此进程被UID所拥有/进程的PID号码/此进程的父进程PID号码】
* C：代表CPU的使用率，单位为百分比
* PRI/NI：Priority/Nice的缩写，代表此进程被CPU所执行的优先级，数值越小代表该进程越快被CPU执行。
* ADDR/SZ/WCHAN：都与内存有关
* TTY：登入者的终端机位置，若为远程登录则使用动态终端接口(pts/n)
* Time:使用掉的CPU时间
* CMD：就是command的缩写，造成此进程的触发程序指令为何

## 观察系统所有进程 ps aux
![捕获13.PNG](https://i.loli.net/2019/10/26/4oWKpXvA63rGwRV.png)
在ps aux显示的项目中，各字段的意义为：
+ 	User:该process属于哪个使用者账号的
+	PID：该process的进程标识符
+	%CPU：该process使用掉的CPU资源百分比
+	%MEM：该process所占用的物理内存百分比
+	VSZ：该process使用掉的虚拟内存量(Kbytes)
+	RSS:该process占用的固定的内存量(Kbytes)
+	TTY:该process是在哪个终端机上面运作，若与终端机无关则显示？，另外，tty1-tty6是本机上面的登入者进程，若为pts/0等等的，则表示为由网络连接进主机的进程
+	STAT：该进程目前的状态，状态显示与ps -l 的S 旗标相同
+	START：该process被触发启动的时间
+	TIME：该process 实际使用CPU运作的时间
+	COMMAND：该进程的实际指令为何？
