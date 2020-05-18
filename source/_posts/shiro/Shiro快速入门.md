---
title: Shiro快速入门
time: 2019-11-04
categories: shiro
tags: shiro
---

# Shiro快速入门
---

## 搭建Shiro环境
引入Shiro的依赖
```
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.1</version>
</dependency>
```

如果要和springboot项目整合，则需要引入shiro的starter
```
  <dependency>
	    <groupId>org.apache.shiro</groupId>
	    <artifactId>shiro-spring-boot-starter</artifactId>
	    <version>1.4.0</version>
  </dependency>
```

## 快速入门

### Enable Shiro
在应用程序中启用Shiro时要了解的第一件事是，Shiro中的几乎所有内容都与称为的中央/核心组件有关SecurityManager。对于那些熟悉Java安全性的人来说，这是Shiro的SecurityManager的概念-它与并不相同java.lang.SecurityManager。
虽然我们将在“ 体系结构”一章中详细介绍Shiro的设计，但现在足以知道Shiro SecurityManager是应用程序Shiro环境的核心，SecurityManager每个应用程序必须存在一个。因此，我们在Tutorial应用程序中必须做的第一件事就是设置SecurityManager实例。

#### 配置
尽管我们可以SecurityManager直接实例化一个类，但Shiro的SecurityManager实现具有足够的配置选项和内部组件，这使得在Java源代码中很难做到这一点- SecurityManager使用基于文本的灵活配置格式进行配置会容易得多。

为此，Shiro通过基于文本的INI配置提供了默认的“共母”解决方案。如今，人们已经厌倦了使用庞大的XML文件，并且INI易于阅读，易于使用并且几乎不需要依赖。稍后您还将看到，通过简单地了解对象图导航，可以有效地使用INI来配置简单的对象图，例如SecurityManager。

```
Shiro许多配置选项

Shiro的SecurityManager实现和所有支持组件都与JavaBeans兼容。这使得Shiro几乎可以使用任何配置格式进行配置，例如XML（Spring，JBoss，Guice等），YAML，JSON，Groovy Builder标记等。INI只是Shiro的“通用分母”格式，允许在没有其他选项的情况下在任何环境中进行配置。
```

shiro.ini
因此，我们将使用INI文件SecurityManager为该简单应用程序配置Shiro 。首先，创建一个src/main/resources目录，从所在的目录开始pom.xml。然后shiro.ini在该新目录中创建一个文件，其内容如下：

src / main / resources / shiro.ini

```
# =============================================================================
# Tutorial INI configuration
#
# Usernames/passwords are based on the classic Mel Brooks' film "Spaceballs" :)
# =============================================================================

# -----------------------------------------------------------------------------
# Users and their (optional) assigned roles
# username = password, role1, role2, ..., roleN
# -----------------------------------------------------------------------------
[users]
root = secret, admin
guest = guest, guest
presidentskroob = 12345, president
darkhelmet = ludicrousspeed, darklord, schwartz
lonestarr = vespa, goodguy, schwartz

# -----------------------------------------------------------------------------
# Roles with assigned permissions
# roleName = perm1, perm2, ..., permN
# -----------------------------------------------------------------------------
[roles]
admin = *
schwartz = lightsaber:*
goodguy = winnebago:drive:eagle5
```

#### 引用配置
现在已经定义了一个INI文件，我们可以SecurityManager在Tutorial应用程序类中创建实例。更改main方法以反映以下更新：
```
public static void main(String[] args) {

    log.info("My First Apache Shiro Application");

    //1.
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");

    //2.
    SecurityManager securityManager = factory.getInstance();

    //3.
    SecurityUtils.setSecurityManager(securityManager);

    System.exit(0);
}
```

### 使用Shiro
现在我们的SecurityManager已经设置好并且可以使用了，现在我们可以开始做我们真正关心的事情了-执行安全操作。

在保护我们的应用程序安全时，我们可能问自己最相关的问题是“当前用户是谁？”或“是否允许当前用户执行X”？在编写代码或设计用户界面时，通常会问这些问题：应用程序通常是基于用户案例构建的，并且您希望基于每个用户来表示（并保护）功能。因此，我们考虑应用程序安全性的最自然方法是基于当前用户。Shiro的API从根本上代表了“当前用户”的Subject概念。

在几乎所有环境中，您都可以通过以下调用获取当前执行的用户：
```
Subject currentUser = SecurityUtils.getSubject();
```

使用SecurityUtils.getSubject()，我们可以获得当前正在执行的Subject。一个主题仅仅是一个应用程序用户的特定安全“视图”。我们实际上想将其称为“用户”，因为那“很合理”，但我们决定反对：太多的应用程序具有已经具有自己的用户类/框架的现有API，并且我们不想与它们发生冲突。同样，在安全领域，该术语Subject实际上是公认的术语。好吧，继续...

getSubject()独立应用程序中的调用可能会Subject在特定于应用程序的位置中返回基于用户数据的，而在服务器环境（例如Web应用程序）中，它会获取Subject与当前线程或传入请求相关联的基于用户数据的。

现在您有了Subject，您可以使用它做什么？
如果要在用户与应用程序的当前会话期间使事情可用，则可以获取他们的会话：
```
Session session = currentUser.getSession();
session.setAttribute( "someKey", "aValue" );
```
这Session是Shiro特有的实例，它提供了常规HttpSession所使用的大部分功能，但具有一些额外的优点和一个很大的不同：它不需要HTTP环境！如果部署在Web应用程序内部，则默认情况下Session将HttpSession基于该应用程序。但是，在非Web环境中，例如简单的快速入门，Shiro将默认自动使用其企业会话管理。这意味着无论部署环境如何，您都可以在任何层的应用程序中使用相同的API。这就打开了一个全新的应用程序世界，因为不需要强制要求使用会话的任何应用程序使用HttpSession或EJB状态会话Bean。而且，任何客户端技术现在都可以共享会话数据。

因此，现在您可以获取Subject和Session。怎么样真的像检查，如果他们被允许做的事情，比如对角色和权限检查有用的东西？

好吧，我们只能对已知用户进行检查。我们Subject上面的实例表示当前用户，但谁是当前用户？好吧，他们是匿名的-也就是说，直到他们至少登录一次。因此，让我们这样做：
```
if ( !currentUser.isAuthenticated() ) {
    //collect user principals and credentials in a gui specific manner
    //such as username/password html form, X509 certificate, OpenID, etc.
    //We'll use the username/password example here since it is the most common.
    //(do you know what movie this is from? ;)
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
    //this is all you have to do to support 'remember me' (no config - built in!):
    token.setRememberMe(true);
    currentUser.login(token);
}
```
而已！再简单不过了。

但是，如果他们的登录尝试失败了怎么办？您可以捕获各种特定的异常，这些异常可以准确地告诉您发生了什么，并允许您进行相应的处理和响应：

```
try {
    currentUser.login( token );
    //if no exception, that's it, we're done!
} catch ( UnknownAccountException uae ) {
    //username wasn't in the system, show them an error message?
} catch ( IncorrectCredentialsException ice ) {
    //password didn't match, try again?
} catch ( LockedAccountException lae ) {
    //account for that username is locked - can't login.  Show them a message?
}
    ... more types exceptions to check if you want ...
} catch ( AuthenticationException ae ) {
    //unexpected condition - error?
}
```
您可以检查许多不同类型的异常，也可以针对自定义条件抛出自己的异常，而Shiro可能无法解决。有关更多信息，请参见AuthenticationException JavaDoc(https://shiro.apache.org/static/1.4.1/apidocs/org/apache/shiro/authc/AuthenticationException.html)。

*安全最佳实践是将通用的登录失败消息发送给用户，因为您不想帮助攻击者尝试闯入您的系统。*

好的，现在，我们已经有一个登录用户。我们还能做什么？

假设他们是谁：
```
//print their identifying principal (in this case, a username): 
log.info( "User [" + currentUser.getPrincipal() + "] logged in successfully." );
```

我们还可以测试一下它们是否具有特定作用：
```
if ( currentUser.hasRole( "schwartz" ) ) {
    log.info("May the Schwartz be with you!" );
} else {
    log.info( "Hello, mere mortal." );
}
```
我们还可以查看他们是否有权对某种类型的实体采取行动：
```
if ( currentUser.isPermitted( "lightsaber:weild" ) ) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}
```

此外，我们可以执行功能非常强大的实例级权限检查-能够查看用户是否具有访问类型的特定实例的能力：
```
if ( currentUser.isPermitted( "winnebago:drive:eagle5" ) ) {
    log.info("You are permitted to 'drive' the 'winnebago' with license plate (id) 'eagle5'.  " +
                "Here are the keys - have fun!");
} else {
    log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
}
```
小菜一碟吧？

最后，当用户使用完应用程序后，他们可以注销：
```
currentUser.logout(); //removes all identifying information and invalidates their session too.
```

