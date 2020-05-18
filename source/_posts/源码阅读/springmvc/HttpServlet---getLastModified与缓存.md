---
title: HttpServlet---getLastModified与缓存
time: 2019-12-1
categories: springmvc
tags: springmvc
---

# HttpServlet---getLastModified与缓存
---
今天看HttpServlet源码看到一个函数getLastModified，上网查阅与浏览器缓存有关系，根据源码，画了个流程图理解下：
源码：
![2019-12-01 14_09_28-springmvcsourceread _D__垃圾_springmvcsourceread_ - D__编程辅助文档_apache-maven-3.6.1-b.png](https://i.loli.net/2019/12/01/qvCSluYkAeTipOg.png)

在service函数中，它先获取上一次修改的时间，如果为-1(负数)，就直接使用get方法，否则获取头部的If-Modified-Since字段的值，如果时间过期了就重新调用get方法，否则返回状态码304，告诉浏览器继续使用缓存。

下面这张图展示了这一整个过程。
![2019-12-01 14_16_02-_2条消息_HttpServlet---getLastModified与缓存 - 奔跑的蜗牛 - CSDN博客.png](https://i.loli.net/2019/12/01/2DnmJAptjVRZxQY.png)

