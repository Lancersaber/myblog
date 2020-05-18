---
title: 使用Spring Boot发送邮件
time: 2019-11-10
categories: spring
tags: mail
---

# 使用Spring Boot发送邮件
---
在项目的维护过程中，我们通常会在应用中加入短信或者邮件预警功能，比如当应用出现异常宕机时应该及时地将预警信息发送给运维或者开发人员，本文将介绍如何在Spring Boot中发送邮件。在Spring Boot中发送邮件使用的是Spring提供的org.springframework.mail.javamail.JavaMailSender，其提供了许多简单易用的方法，可发送简单的邮件、HTML格式的邮件、带附件的邮件，并且可以创建邮件模板。

## 引入依赖
在Spring Boot中发送邮件，需要用到spring-boot-starter-mail，引入spring-boot-starter-mail：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

## 邮件配置
在application.yml中进行简单的配置（以163邮件为例）：
```
server:
  port: 80

spring:
  mail:
    host: smtp.163.com
    username: 你的账号
    password: 你的密码
    properties:
      mail:
        smtp:
          auth: true
          starttls:
            enable: true
            required: true

```

## 发送简单的邮件
编写EmailController，注入JavaMailSender:
```
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
    
    @RequestMapping("sendSimpleEmail")
    public String sendSimpleEmail() {
        try {
            SimpleMailMessage message = new SimpleMailMessage();
            message.setFrom(from);
            message.setTo("888888@qq.com"); // 接收地址
            message.setSubject("一封简单的邮件"); // 标题
            message.setText("使用Spring Boot发送简单邮件。"); // 内容
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}
```
启动项目访问http://localhost/email/sendSimpleEmail，提示发送成功：

## 发送带附件的邮件
发送带附件的邮件和普通邮件相比，其实就只是多了个传入附件的过程。不过使用的仍是MimeMessage：
```
package com.springboot.demo.controller;

import java.io.File;

import javax.mail.internet.MimeMessage;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.io.FileSystemResource;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/email")
public class EmailController {

    @Autowired
    private JavaMailSender jms;
    
    @Value("${spring.mail.username}")
    private String from;
	
    @RequestMapping("sendAttachmentsMail")
    public String sendAttachmentsMail() {
        MimeMessage message = null;
        try {
            message = jms.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message, true);
            helper.setFrom(from); 
            helper.setTo("888888@qq.com"); // 接收地址
            helper.setSubject("一封带附件的邮件"); // 标题
            helper.setText("详情参见附件内容！"); // 内容
            // 传入附件
            FileSystemResource file = new FileSystemResource(new File("src/main/resources/static/file/项目文档.docx"));
            helper.addAttachment("项目文档.docx", file);
            jms.send(message);
            return "发送成功";
        } catch (Exception e) {
            e.printStackTrace();
            return e.getMessage();
        }
    }
}
```