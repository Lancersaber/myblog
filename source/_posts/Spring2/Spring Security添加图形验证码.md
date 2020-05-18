---
title: Spring Security添加图形验证码
time: 2019-11-11
categories: springsecurity
tags: springsecurity
---

# Spring Security添加图形验证码
---
添加验证码大致可以分为三个步骤：根据随机数生成验证码图片；将验证码图片显示到登录页面；认证流程中加入验证码校验。Spring Security的认证校验是由UsernamePasswordAuthenticationFilter过滤器完成的，所以我们的验证码校验逻辑应该在这个过滤器之前。下面一起学习下如何在上一节Spring Security自定义用户认证的基础上加入验证码校验功能。

## 生成图形验证码
验证码功能需要用到spring-social-config依赖：
```
<dependency>
    <groupId>org.springframework.social</groupId>
    <artifactId>spring-social-config</artifactId>
</dependency>
```

首先定义一个验证码对象ImageCode：
```
public class ImageCode {

    private BufferedImage image;

    private String code;

    private LocalDateTime expireTime;

    public ImageCode(BufferedImage image, String code, int expireIn) {
        this.image = image;
        this.code = code;
        this.expireTime = LocalDateTime.now().plusSeconds(expireIn);
    }

    public ImageCode(BufferedImage image, String code, LocalDateTime expireTime) {
        this.image = image;
        this.code = code;
        this.expireTime = expireTime;
    }

    boolean isExpire() {
        return LocalDateTime.now().isAfter(expireTime);
    }
    // get,set 略
}
```

ImageCode对象包含了三个属性：image图片，code验证码和expireTime过期时间。isExpire方法用于判断验证码是否已过期。

接着定义一个ValidateCodeController，用于处理生成验证码请求：
