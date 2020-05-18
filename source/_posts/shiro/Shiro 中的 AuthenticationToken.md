---
title: Shiro 中的 AuthenticationToken
time: 2019-11-06
categories: shiro
tags: shiro
---

# Shiro 中的 AuthenticationToken
---
AuthenticationToken 用于收集用户提交的身份（如用户名）及凭据（如密码）。Shiro会调用CredentialsMatcher对象的doCredentialsMatch方法对AuthenticationInfo对象和AuthenticationToken进行匹配。匹配成功则表示主体（Subject）认证成功，否则表示认证失败。
![shiro10.PNG](https://i.loli.net/2019/11/06/dxCujFRki79Uw6N.png)

```
public interface AuthenticationToken extends Serializable {
    Object getPrincipal(); //身份
    Object getCredentials(); //凭据
}

public interface HostAuthenticationToken extends AuthenticationToken {
    String getHost();// 获取用户“主机”
}

public interface RememberMeAuthenticationToken extends AuthenticationToken {
    boolean isRememberMe();// 记住我
}
```

Shiro 仅提供了一个可以直接使用的 UsernamePasswordToken，用于实现基于用户名/密码主体（Subject）身份认证。UsernamePasswordToken实现了 RememberMeAuthenticationToken 和 HostAuthenticationToken，可以实现“记住我”及“主机验证”的支持。

## 总结
一般情况下UsernamePasswordToken已经可以满足我们的大我数需求。当我们遇到需要声明自己的Token类时，可以根据需求来实现AuthenticationToken，HostAuthenticationToken或RememberMeAuthenticationToken。

1、如果不需要“记住我”，也不需要“主机验证”，则可以实现AuthenticationToken；
2、如果需要“记住我”，则可以实现RememberMeAuthenticationToken；
3、如果需要“主机验证”功能，则可以实现HostAuthenticationToken；
4、如果需要“记住我”，且需要“主机验证”，则可以像UsernamePasswordToken一样，同时实现RememberMeAuthenticationToken和HostAuthenticationToken。
5、如果需要其他自定义功能，则需要自己实现。