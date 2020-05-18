---
title: Linux中关于防火墙的指令
time: 2019-11-22
categories: Linux
tags: Linux
---

# Linux中关于防火墙的指令
---
CentOS 7.0默认使用的是firewall作为防火墙

## 一些操作

### 查看防火墙状态
```
firewall-cmd --state
```

### 停止firewall
```
systemctl stop firewalld.service
```

### 禁止firewall开机启动
```
systemctl disable firewalld.service 
```

