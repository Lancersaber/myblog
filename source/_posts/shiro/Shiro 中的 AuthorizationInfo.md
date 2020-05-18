---
title: Shiro 中的 AuthorizationInfo
time: 2019-11-08
categories: shiro
tags: shiro
---

# Shiro 中的 AuthorizationInfo
---
AuthorizationInfo 用于聚合授权信息的：
![shiro12.PNG](https://i.loli.net/2019/11/08/MOqDFE5wt98hesy.png)

AuthorizationInfo 源码：
```
public interface AuthorizationInfo extends Serializable {
    Collection<String> getRoles(); //获取角色字符串信息
    Collection<String> getStringPermissions(); //获取权限字符串信息
    Collection<Permission> getObjectPermissions(); //获取 Permission 对象信息
}
```
当我们使用 AuthorizingRealm 时，如果身份验证成功，在进行授权时就通过doGetAuthorizationInfo 方法获取角色/权限信息用于授权验证。Shiro 提供了一个实现 SimpleAuthorizationInfo，大多数时候使用这个即可。