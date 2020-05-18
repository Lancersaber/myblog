---
title: DispatcherServlet工作流程
time: 2019-12-01
categories: springmvc
tags: springmvc
---

# DispatcherServlet工作流程
---

![dqtheoa4.2ai.jpg](https://i.loli.net/2019/12/01/zdQGOuWAxaHUp9M.jpg)

1. 当DispatcherServlet接到请求时，他先回查找适当的处理程序来处理请求。DispatcherServlet通过一个或者多个处理程序映射，将每个请求映射到处理程序中。处理程序映射配置在web应用程序的上下文中，是实现了HandlerMapping接口的Bean。它负责为请求返回一个适当的处理程序（也就是Controller）。处理程序映射通常根据请求的URL将请求映射到处理程序（Controller）。 
2. 一旦DispatcherServlet选择了适当的控制器，它就会调用这个控制器来处理请求。 
3. 控制器处理完请求后，会将模型和视图名（有时候是视图对象）返回给DispatcherServlet。模型包含了控制器要传递给视图进行显示的属性。如果返回的是视图名称，它会被解析成视图对象再进行呈现。绑定模型和视图的基本类是ModelAndView
4. 当DispatcherServlet接收到模型和视图名称时，它会将逻辑视图名称解析成视图对象再进行呈现。DispatcherServlet从一个或者多个视图解析器中解析视图。视图解析器配置在Web应用程序上下文中，是实现了ViewResolver接口的Bean。它的任务是根据逻辑视图名称返回试图对象。
5. 一旦DispatcherServlet将视图名称解析称为试图对象，它就会呈现视图对象，并传递控制器返回的模型。视图的任务是将模型属性展示给用户。