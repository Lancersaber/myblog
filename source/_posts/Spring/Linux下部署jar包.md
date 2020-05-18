---
title: Linux下部署springboot jar包
time: 2019-10-31
categories: spring
tags: spring
---
# Linux下部署springboot jar包
---
部署Spring Boot项目可以使用Maven命令mvn:clean package将项目打包成只执行的jar文件，然后使用命令java -jar XX.jar来执行。但这样做无法将shell命令行释放，关闭terminal后项目也随之关闭了。这里介绍在Linux系统中后台运行Spring Boot jar文件的方法。

实现这个功能主要依赖于Linux的nohup命令。nohup命令可以将程序以忽略挂起信号的方式运行起来，被运行的程序的输出信息将不会显示到终端。
nohup语法：
```
nohup 命令
用途：不挂断地运行命令。
语法：nohup Command [ Arg … ][ & ]
```

所以只需要在启动命令前加上nohup命令，末尾加上&即可：nohup java -jar XX.jar &。
为了方便，我们可以编写启动脚本start.sh：
```
nohup java -jar XX.jar &
```

关停脚本stop.sh：
```
PID=`ps -ef | grep sms-2.0.jar | grep -v grep | awk '{print $2}'`
if [ -z "$PID" ]
then
    echo Application is already stopped
else
    echo kill $PID
    kill -9 $PID
fi
```
使用了nohup命令后，会在jar文件目录下生成一个nohup.out文件，可通过其观察当前项目的运行情况：
```

$ ll
总用量 76612
drwxrwxr-x 2 zjrun zjrun     4096 2月   8 08:49 log
-rw------- 1 zjrun zjrun 58695723 2月   8 10:15 nohup.out
-rwxrw-r-- 1 zjrun zjrun       88 2月   7 15:17 run.sh
-rw-rw-r-- 1 zjrun zjrun 19730199 2月   8 10:11 sms-1.0.jar
-rwxrw-r-- 1 zjrun zjrun       60 2月   7 15:22 start.sh
-rwxrw-r-- 1 zjrun zjrun      184 2月   7 15:19 stop.sh

$ tail -10f nohup.out 
10:14:31.309 logback [main] INFO  o.s.j.e.a.AnnotationMBeanExporter - Registering beans for JMX exposure on startup
10:14:31.478 logback [main] INFO  o.a.coyote.http11.Http11NioProtocol - Initializing ProtocolHandler ["http-nio-8963"]
10:14:31.498 logback [main] INFO  o.a.coyote.http11.Http11NioProtocol - Starting ProtocolHandler ["http-nio-8963"]
10:14:31.506 logback [main] INFO  o.a.tomcat.util.net.NioSelectorPool - Using a shared selector for servlet write/read
10:14:31.634 logback [main] INFO  o.s.b.c.e.t.TomcatEmbeddedServletContainer - Tomcat started on port(s): 8963 (http)
10:14:31.644 logback [main] INFO  com.xingyi.sms.SmsApplication - Started SmsApplication in 7.213 seconds (JVM running for 8.03)
complete!
10:15:26.978 logback [http-nio-8963-exec-1] INFO  o.a.c.c.C.[.[localhost].[/mobilePre] - Initializing Spring FrameworkServlet 'dispatcherServlet'
10:15:26.979 logback [http-nio-8963-exec-1] INFO  o.s.web.servlet.DispatcherServlet - FrameworkServlet 'dispatcherServlet': initialization started
10:15:27.004 logback [http-nio-8963-exec-1] INFO  o.s.web.servlet.DispatcherServlet - FrameworkServlet 'dispatcherServlet': initialization completed in 25 ms
```