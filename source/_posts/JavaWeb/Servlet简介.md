---
title: Servlet简介
time: 2019-11-07
categories: Servlet
tags: Servlet
---

# Servlet简介
---

## Servlet是什么？
Java Servlet 是运行在 Web 服务器或应用服务器上的程序，它是作为来自 Web 浏览器或其他 HTTP 客户端的请求和 HTTP 服务器上的数据库或应用程序之间的中间层。

使用 Servlet，您可以收集来自网页表单的用户输入，呈现来自数据库或者其他源的记录，还可以动态创建网页。
Java Servlet 通常情况下与使用 CGI（Common Gateway Interface，公共网关接口）实现的程序可以达到异曲同工的效果。但是相比于 CGI，Servlet 有以下几点优势：
* 性能明显更好。
* Servlet 在 Web 服务器的地址空间内执行。这样它就没有必要再创建一个单独的进程来处理每个客户端请求。
* Servlet 是独立于平台的，因为它们是用 Java 编写的。
* 服务器上的 Java 安全管理器执行了一系列限制，以保护服务器计算机上的资源。因此，Servlet 是可信的。
* Java 类库的全部功能对 Servlet 来说都是可用的。它可以通过 sockets 和 RMI 机制与 applets、数据库或其他软件进行交互。

## Servlet架构
下图显示了 Servlet 在 Web 应用程序中的位置。
![servlet-arch.jpg](https://i.loli.net/2019/11/07/y4XLSgaeMVrslY7.jpg)

## Servlet 任务
Servlet 执行以下主要任务：
* 读取客户端（浏览器）发送的显式的数据。这包括网页上的 HTML 表单，或者也可以是来自 applet 或自定义的 HTTP 客户端程序的表单。
* 读取客户端（浏览器）发送的隐式的 HTTP 请求数据。这包括 cookies、媒体类型和浏览器能理解的压缩格式等等
* 处理数据并生成结果。这个过程可能需要访问数据库，执行 RMI 或 CORBA 调用，调用 Web 服务，或者直接计算得出对应的响应。
* 发送显式的数据（即文档）到客户端（浏览器）。该文档的格式可以是多种多样的，包括文本文件（HTML 或 XML）、二进制文件（GIF 图像）、Excel 等。
* 发送隐式的 HTTP 响应到客户端（浏览器）。这包括告诉浏览器或其他客户端被返回的文档类型（例如 HTML），设置 cookies 和缓存参数，以及其他类似的任务。

## Servlet接口
```
public interface Servlet {
	public void init(ServletConfig config) throws ServletException;
	public ServletConfig getServletConfig();
	public void service(ServletRequest req, ServletResponse res)
            throws ServletException, IOException;
    public String getServletInfo();
    public void destroy();
}
```
init方法在容器启动时被容器调用，只会调用一次；getServletConfig方法用于获取ServletConfig；service方法用于具体处理一个请求；getServletInfo方法可以获取一些Servlet相关的信息，这个方法需要自己实现，默认返回空字符串，destroy主要用于在Servlet销毁(一般指关闭服务器)时释放一些资源，只会调用一次。