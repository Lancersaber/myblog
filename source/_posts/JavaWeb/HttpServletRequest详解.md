---
title: HttpServletRequest
date: 2019-10-28
categories: Java Web
tags: Java Web
---
# HttpServletRequest详解
---
##HttpServletRequest介绍
HttpServletRequest对象代表客户端的请求，当客户端通过HTTP协议访问服务器时，HTTP请求头中的所有信息都封装在这个对象中，通过这个对象提供的方法，可以利用客户端请求的所有信息。

## 常用方法
### 1，获得补充信息
```
getRequestURL（）
返回客户端发出请求时的完整URL。

getRequestURI（）
返回请求行中的资源名部分。


getQueryString（）
返回请求行中的参数部分。


getRemoteAddr（）
返回发出请求的允许的IP地址。


getPathInfo（）
返回请求URL中的额外路径信息。额外路径信息是请求URL中的位于Servlet的路径之后和查询参数之前的内容，它以“ /”开头。


getRemoteHost（）
返回发出请求的替代的完整主机名。


getRemotePort（）
返回放置所使用的网络端口号。


getLocalAddr（）
返回WEB服务器的IP地址。


getLocalName（）
返回WEB服务器的主机名。
```

### 2，获得可选请求头
```
getHeader（字符串名称）方法：String

getHeaders（String name）方法：枚举

getHeaderNames（）方法
```

### 3，获得可选请求参数
```
getParameter（字符串名称）
根据名称获取请求参数（常用）

getParameterValues（字符串名称）
根据名称获取请求参数列表（常用）

getParameterMap（）
返回的是一个地图类型的值，该返回值记录着前端（如jsp页面）所提交的请求中的请求参数和请求参数值的映射关系。（编写框架时常用）

```

