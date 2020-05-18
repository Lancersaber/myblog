---
title: Shiro中的Exception
time: 2019-11-06
categories: shiro
tags: shiro
---

# Shiro中的Exception
---

## 1、身份认证异常  身份令牌异常，不支持的身份令牌  
org.apache.shiro.authc.pam.UnsupportedTokenException  
  
## 2、未知账户/没找到帐号,登录失败  
org.apache.shiro.authc.UnknownAccountException  

## 3、帐号锁定  
org.apache.shiro.authc.LockedAccountException  

## 4、用户禁用  
org.apache.shiro.authc.DisabledAccountException  

## 5、登录重试次数，超限。只允许在一段时间内允许有一定数量的认证尝试  
org.apache.shiro.authc.ExcessiveAttemptsException  

## 6、 一个用户多次登录异常：不允许多次登录，只能登录一次 。即不允许多处登录  
org.apache.shiro.authc.ConcurrentAccessException  

## 7、账户异常  
org.apache.shiro.authc.AccountException  
  
## 8、过期的凭据异常  
org.apache.shiro.authc.ExpiredCredentialsException  

## 9、错误的凭据异常  
org.apache.shiro.authc.IncorrectCredentialsException

## 10、凭据异常  
org.apache.shiro.authc.CredentialsException  
org.apache.shiro.authc.AuthenticationException  
  
## 11、权限异常  
没有访问权限，访问异常  
org.apache.shiro.authz.HostUnauthorizedException  
org.apache.shiro.authz.UnauthorizedException  

## 12、授权异常  
org.apache.shiro.authz.UnauthenticatedException  
org.apache.shiro.authz.AuthorizationException  
  
## 13、shiro全局异常  
org.apache.shiro.ShiroException