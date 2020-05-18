---
title: Shiro中Subject类的一些函数
time: 2019-11-08
categories: shiro
tags: shiro
---

# Shiro中Subject类的一些函数
---


* boolean hasAllRoles(Collection<String> roleIdentifiers)
Returns true if this Subject has all of the specified roles, false otherwise.

* boolean hasRole(String roleIdentifier)
Returns true if this Subject has the specified role, false otherwise.

* boolean isAuthenticated()
Returns true if this Subject/user proved their identity during their current session by providing valid credentials matching those known to the system, false otherwise.

* void	login(AuthenticationToken token)
Performs a login attempt for this Subject/user.

* void 	logout()
Logs out this Subject and invalidates and/or removes any associated entities, such as a Session and authorization data.

* boolean[] isPermitted(List<Permission> permissions)
Checks if this Subject implies the given Permissions and returns a boolean array indicating which permissions are implied.

* boolean 	isPermitted(Permission permission)
Returns true if this Subject is permitted to perform an action or access a resource summarized by the specified permission.

* boolean isPermitted(String permission)
Returns true if this Subject is permitted to perform an action or access a resource summarized by the specified permission string.

* boolean 	isRemembered()
Returns true if this Subject has an identity (it is not anonymous) and the identity (aka principals) is remembered from a successful authentication during a previous session.

## 总结一下
Subject接口中的函数主要是关于三个方面：
* 用户认证，及登陆，注销
* 用户的角色是否匹配
* 用户的权限是否匹配