---
title: Shiro简介
time: 2019-11-4
categories: shiro
tags: shiro
---

# Shiro简介
---

## 什么是Apache Shiro
Apache Shiro是一个功能强大且灵活的开源安全框架，可以干净地处理身份验证，授权，企业会话管理和加密。

Apache Shiro的首要目标是易于使用和理解。安全有时可能非常复杂，甚至会很痛苦，但这不是必须的。框架应尽可能掩盖复杂性，并公开简洁直观的API，以简化开发人员确保其应用程序安全的工作。
您可以使用Apache Shiro进行以下操作：
* 验证用户身份以验证其身份
* 对用户执行访问控制，例如：
 	+	确定是否为用户分配了特定的安全角色
 	+	确定是否允许用户做某事
* 即使在没有Web或EJB容器的情况下，也可以在任何环境中使用Session API。
* 在身份验证，访问控制或会话的生存期内对事件做出反应。
* 汇总1个或更多用户安全数据的数据源，并将其全部显示为单个复合用户“视图”。
* 启用单点登录（SSO）功能
* 启用“记住我”服务以进行用户关联，而无需登录
等等-所有这些都集成到一个易于使用的统一API中。
Shiro尝试在所有应用程序环境中实现这些目标-从最简单的命令行应用程序到最大的企业应用程序，而不必强加对其他第三方框架，容器或应用程序服务器的依赖。当然，该项目旨在尽可能地集成到这些环境中，但是可以在任何环境中直接使用它。

## Apache Shiro功能
Apache Shiro是具有许多功能的全面的应用程序安全框架。下图显示了Shiro集中精力的地方，本参考手册的组织方式也类似：
![ShiroFeatures.png](https://i.loli.net/2019/11/04/7TkNwl2PFvadBRI.png)
Shiro以Shiro开发团队所谓的“应用程序安全性的四个基石”为目标-身份验证，授权，会话管理和密码学：
* 身份验证：有时称为“登录”，这是证明用户是他们所说的身份的行为。

* 授权：访问控制的过程，即确定“谁”有权访问“什么”。

* 会话管理：即使在非Web或EJB应用程序中，也可以管理用户特定的会话。

* 密码术：使用密码算法保持数据安全，同时仍然易于使用。

在不同的应用程序环境中，还具有其他功能来支持和加强这些问题，尤其是：
* Web支持：Shiro的Web支持API可帮助轻松保护Web应用程序的安全。
* 缓存：缓存是Apache Shiro API的第一层公民，可确保安全操作保持快速有效。
* 并发性：Apache Shiro的并发功能支持多线程应用程序。
* 测试：测试支持可以帮助您编写单元测试和集成测试，并确保您的代码将按预期进行保护。
* “运行方式”：一种功能，允许用户采用其他用户的身份（如果允许），有时在管理方案中很有用。
* “记住我”：在整个会话中记住用户的身份，因此他们只需要在必要时登录。

shiro不会去维护用户，维护权限；这些需要我们自己去设计/提供；然后通过相应的接口注入给shiro
![shiro1.PNG](https://i.loli.net/2019/11/04/CjdnexzlNaWQByI.png)
直接与代码交互的对象是Subject，也就是说Shiro对外API的核心是Subject
Subject: 主体，代表当前“用户”，所有的Subject都绑定到SecurityManager，都委托给SecurityManager, SUbject相当于门面，而SecurityManager才是实际的执行者；
SecurityManager：我是安全管理器，所有有安全相关的操作，都会交给我来处理，我管理着所有的Subject，我是核心，我负责与其他组件进行交互，你也可以你把我比作springmvc里面的前端控制器，
Realm: 我叫域，Shiro要从我这里获取安全数据（用户，角色，权限），也就是说SecurityManager要验证身份，需要从我这里获取相应用户的身份，也需要从我这里获取权限，可以把我 看作安全数据源；


从上面也可以看出，Shiro不提供维护用户和权限，而是用过Realm让开发人员自己注入；
![shiro2.PNG](https://i.loli.net/2019/11/04/KOv4hQXotmpJ9DY.png)
Subject: 主体
SecurityManager:心脏
Authentication:认证器
Authorizer: 授权器
Realm:可以有一个或者多个，安全的实体数据源
SessionManager: Shiro并不仅仅可以用在Web环境，也可以用在如普通的JavaSE环境、EJB等环境
SessionDao:
CacheManager:缓存控制器 ，放到 缓存中可以提高访问性能；
Cryptography: 密码模块；
