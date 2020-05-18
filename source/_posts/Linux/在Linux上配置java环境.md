---
title: 在Linux上配置java环境
date: 2019-10-24 23:15:15
categories: Linux
tags: Linux
---
# 在Linux上配置java环境
---
## 下载jdk1.8
```
# wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"  http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.tar.gz
```

## 解压
1.用tar命令解压后，会生成jdk1.8.0_131目录；

2.为了方便用mv把jdk1.8.0_131更名为jdk1.8；

3.把jdk1.8移动到/usr/local目录下(当然也可以移动到别的目录下，后面配置环境变量修改为自己的jdk路径即可)
```
# tar -zxvf jdk-8u131-linux-x64.tar.gz
# mv jdk1.8.0_131 jdk1.8
# mv jdk1.8 /usr/local
```

## 配置环境变量
1.修改/etc/profile文件
```
# vim /etc/profile
```
在profile文件中配置jdk环境
```
export JAVA_HOME=/usr/local/jdk1.8
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib:$CLASSPATH
export JAVA_PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin
export PATH=$PATH:${JAVA_PATH}
```

2.使profile生效
```
# source /etc/profie
```

## 查看配置环境是否成功
java -version命令查看jdk版本
```
# java -version
```