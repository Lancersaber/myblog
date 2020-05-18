---
title: Linux切换界面
date: 2019-10-24 23:15:15
categories: Linux
tags: Linux
---
# Linux切换工作模式
---
进入/etc目录下，打开inittab文件，这个文件包含设置用户界面和控制台界面的设置方式。
```
# inittab is no longer used when using systemd.
#
# ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
#
# systemd uses 'targets' instead of runlevels. By default, there are two main targets:
#
# multi-user.target: analogous to runlevel 3
# graphical.target: analogous to runlevel 5
#
# To view current default target, run:
# systemctl get-default
#
# To set a default target, run:
# systemctl set-default TARGET.target
#

systemctl set-default graphical.target //默认启用图形化界面
init 5;切换图形界面
init 3;切换控制台
```