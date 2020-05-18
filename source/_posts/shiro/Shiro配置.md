---
title: Shiro配置
time: 2019-11-04
categories: shiro
tags: shiro
---

# Shiro配置
---

## 程序配置
创建SecurityManager并将其提供给应用程序的最简单的绝对方法是创建一个org.apache.shiro.mgt.DefaultSecurityManager并将其连接到代码中。例如：
```
Realm realm = //instantiate or acquire a Realm instance.  We'll discuss Realms later.
SecurityManager securityManager = new DefaultSecurityManager(realm);

//Make the SecurityManager instance available to the entire application via static memory: 
SecurityUtils.setSecurityManager(securityManager);
```

出乎意料的是，仅需三行代码，您便拥有了适用于许多应用程序的功能齐全的Shiro环境。那有多容易！！

## INI配置
相反，大多数应用程序受益于可以独立于源代码进行修改的基于文本的配置，对于不熟悉Shiro API的人来说，甚至可以使事情更容易理解。

为了确保基于共同点的基于文本的配置机制可以在所有环境中以最少的第三方依赖性运行，Shiro支持INI格式来构建SecurityManager对象图及其支持组件。INI易于阅读，易于配置，设置简单，非常适合大多数应用。

### 从INI创建SecurityManager
这是两个如何基于INI配置构建SecurityManager的示例。

### INI资源中的SecurityManager
我们可以从INI资源路径创建SecurityManager实例。资源可以从文件系统，类路径，或者网址时前缀来获取file:，classpath:或url:分别。本示例使用a 从类路径的根Factory提取shiro.ini文件并返回SecurityManager实例：
```
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.util.Factory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.config.IniSecurityManagerFactory;

...

Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

### INI的各个节点
INI基本上是一种文本配置，由按唯一命名的部分组织的键/值对组成。键仅在每个部分中是唯一的，而不是在整个配置中都是唯一的（不同于JDK Properties）。但是，每个部分都可以视为一个单独的Properties定义。

带注释的行可以以Octothorpe（＃-又名“哈希”，“英镑”或“数字”符号）或分号（';'）开头。

这是Shiro理解的部分示例：
```
# =======================
# Shiro INI configuration
# =======================

[main]
# Objects and their properties are defined here,
# Such as the securityManager, Realms and anything
# else needed to build the SecurityManager

[users]
# The 'users' section is for simple deployments
# when you only need a small number of statically-defined
# set of User accounts.

[roles]
# The 'roles' section is for simple deployments
# when you only need a small number of statically-defined
# roles.

[urls]
# The 'urls' section is used for url-based security
# in web applications.  We'll discuss this section in the
# Web documentation
```
#### main
在此[main]部分中，您可以配置应用程序的SecurityManager实例及其任何依赖项，例如Realm。

配置对象实例（如SecurityManager或其任何依赖项）听起来很难使用INI，因为在INI中，我们只能使用名称/值对。但是通过一些约定和对对象图的了解，您会发现您可以做很多事情。Shiro使用这些假设来启用简单但相当简洁的配置机制。

我们经常喜欢将这种方法称为“穷人”依赖注入，尽管它不如成熟的Spring / Guice / JBoss XML文件强大，但是您会发现它可以完成很多工作而没有太多复杂性。当然，这些其他配置机制也可用，但是使用Shiro并不是必需的。

只是为了激发您的胃口，这里有一个有效的[main]配置示例。我们将在下面详细介绍它，但是您可能会发现，仅凭直觉就已经了解了很多事情：
```
[main]
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher

myRealm = com.company.security.shiro.DatabaseRealm
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
myRealm.password = secret
myRealm.credentialsMatcher = $sha256Matcher

securityManager.sessionManager.globalSessionTimeout = 1800000
```

##### 定义对象
请考虑以下[main]部分代码片段：
```
[main]
myRealm = com.company.shiro.realm.MyRealm
...
```

该行实例化一个类型为的新对象实例，com.company.shiro.realm.MyRealm并使该对象在myRealm名称下可用，以供进一步参考和配置。

如果实例化的对象实现了该org.apache.shiro.util.Nameable接口，则将Nameable.setName使用名称值在该对象上调用方法（myRealm在此示例中）。

#### 设置对象属性
原始值：
只需使用等号即可分配简单的原始属性：
```
...
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
...
```

这些配置行转换为方法调用：
```
...
myRealm.setConnectionTimeout(30000);
myRealm.setUsername("jsmith");
...
```

在幕后，默认情况下，Shiro 在设置这些属性时使用Apache Commons BeanUtils进行所有繁重的工作。因此，尽管INI值是文本，但是BeanUtils知道如何将字符串值转换为适当的原始类型，然后调用相应的JavaBeans setter方法。

#### 引用类型的值
如果您需要设置的值不是原始类型，而是另一个对象，该怎么办？好了，您可以使用美元符号（$）来引用先前定义的实例。例如：
```
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
myRealm.credentialsMatcher = $sha256Matcher
...
```

这只是找到名称为sha256Matcher定义的对象，然后使用BeanUtils在myRealm实例上设置该对象（通过调用myRealm.setCredentialsMatcher(sha256Matcher)方法）。

嵌套属性
使用INI线的等号左侧的虚线表示法，可以遍历对象图以到达要设置的最终对象/属性。例如，此配置行：
```
...
securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

集合属性
可以像设置其他任何属性一样直接设置列表，集合和地图，也可以将其设置为嵌套属性。对于集和列表，只需指定一个以逗号分隔的值或对象引用集。

例如，一些SessionListeners：
```
sessionListener1 = com.company.my.SessionListenerImplementation
...
sessionListener2 = com.company.my.other.SessionListenerImplementation
...
securityManager.sessionManager.sessionListeners = $sessionListener1, $sessionListener2
```

对于map，您可以指定键值对的逗号分隔列表，其中每个键值对均以冒号'：'分隔。
```
object1 = com.company.some.Class
object2 = com.company.another.Class
...
anObject = some.class.with.a.Map.property

anObject.mapProperty = key1:$object1, key2:$object2
```

覆盖实例
任何对象都可以由稍后在配置中定义的新实例覆盖。因此，例如，第二个myRealm定义将覆盖第一个：
```
...
myRealm = com.company.security.MyRealm
...
myRealm = com.company.security.DatabaseRealm
...
```

这将导致myRealm成为一个com.company.security.DatabaseRealm实例，并且以前的实例将永远不会被使用（并收集垃圾）。

#### 默认的SecurityManager
您可能已经在上面的完整示例中注意到，尚未定义SecurityManager实例的类，而我们直接跳到设置了一个嵌套属性：
```
myRealm = ...

securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

这是因为securityManager实例是一个特殊的实例-它已经为您实例化并且可以使用了，因此您无需知道SecurityManager要实例化的特定实现类。

当然，如果您实际上要指定自己的实现，则可以按照上面“覆盖实例”部分中的定义定义实现：

```
...
securityManager = com.company.security.shiro.MyCustomSecurityManager
...
```

当然，这几乎是不需要的-Shiro的SecurityManager实现是非常可定制的，通常可以配置任何必要的东西。您可能想问问自己（或用户列表）是否真的需要这样做。

### User
本[users]部分允许您定义一组静态的用户帐户。这在用户帐户数量很少或不需要在运行时动态创建用户帐户的环境中最有用。这是一个例子：
```
[users]
admin = secret
lonestarr = vespa, goodguy, schwartz
darkhelmet = ludicrousspeed, badguy, schwartz
```

#### 行格式
[users]部分中的每一行都必须符合以下格式：
username= password，roleName1，roleName2，...，roleNameN
* 等号左侧的值是用户名
* 等号右边的第一个值是用户的密码。必须输入密码。
* 密码后的任何逗号分隔值都是分配给该用户的角色的名称。角色名称是可选的。

#### 加密密码
如果您不希望[users]部分的密码为纯文本格式，则可以根据自己的喜好使用自己喜欢的哈希算法（MD5，Sha1，Sha256等）对它们进行加密，并将结果字符串用作密码值。默认情况下，密码字符串应为十六进制编码，但可以将其配置为Base64编码（请参见下文）。

```
简易安全密码
为了节省时间并使用最佳实践，您可能需要使用Shiro的Command Line Hasher，它将对密码以及任何其他类型的资源进行哈希处理。这对于加密INI [users]密码特别方便。
```

指定哈希文本密码值后，您必须告诉Shiro这些密码已加密。您可以通过配置iniRealm[main]部分中隐式创建的配置，以使用CredentialsMatcher与您指定的哈希算法相对应的适当实现来做到这一点：
```
[main]
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
iniRealm.credentialsMatcher = $sha256Matcher
...

[users]
# user1 = sha256-hashed-hex-encoded password, role1, role2, ...
user1 = 2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b, role1, role2, ...
```


您可以配置在任何属性CredentialsMatcher像任何其他对象，以反映你的散列策略，例如，指定是否使用或有多少哈希迭代执行腌制。请参阅org.apache.shiro.authc.credential.HashedCredentialsMatcherJavaDoc以更好地了解哈希策略以及它们是否对您有用。

例如，如果用户的密码字符串是Base64编码的，而不是默认的十六进制，则可以指定：
```
[main]
...
# true = hex, false = base64:
sha256Matcher.storedCredentialsHexEncoded = false
```

### roles
本[roles]部分允许您将权限与[用户]部分中定义的角色相关联。同样，这在角色数量少的环境中或不需要在运行时动态创建角色的环境中很有用。这是一个例子：
```
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = *
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
```

#### 行格式
[角色]部分中的每一行都必须使用以下格式定义角色到权限的键/值映射：
rolename= PermissionDefinition1，PermissionDefinition2，...，permissionDefinitionN
这里的PermissionDefinition是一个任意字符串，但是大多数人会希望使用符合格式的字符串，
以org.apache.shiro.authz.permission.WildcardPermission易于使用和灵活。有关权限以及如何从中受益的更多信息，请参见权限(https://shiro.apache.org/permissions.html)文档。
```
 内部逗号
请注意，如果单个的PermissionDefinition需要在内部用逗号分隔（例如printer:5thFloor:print,info），则需要在该定义周围加上双引号（“），以避免解析错误："printer:5thFloor:print,info"
```

### [urls]
[urls]项允许你做一些在我们已经见过的任何 Web 框架都不存在的东西：在你的应用程序中定义自适应过滤器链来匹配URL 路径！

这将更为灵活，功能更为强大，比你通常在 web.xml 中定义的过滤器链更为简洁：即使你从未使用任何 Shiro 提供的其他功能并仅仅使用了这个，但它即使是单独使用也是值得的。

[urls]
在urls 项的每一行格式如下：
URL_Ant_Path_Expression = Path_Specific_Filter_Chain
举例：
```
...
[urls]

/index.html = anon
/user/create = anon
/user/** = authc
/admin/** = authc, roles[administrator]
/rest/** = authc, rest
/remoting/rpc/** = authc, perms["remote:invoke"]
```

接下来我们将讨论这些行的具体含义

#### URL Path Expressions
等号左边是一个与Web 应用程序上下文根目录相关的 Ant 风格的路径表达式。
例如，假设你有如下的[urls]行：
```
/account/** = ssl, authc
```

此行表明，“任何对我应用程序的/accout 或任何它的子路径（/account/foo, account/bar/baz，等等）的请求都将触发'ssl, authc'过滤器链”。我们将在下面讨论过滤器链。

请注意，所有的路径表达式都是相对于你的应用程序的上下文根目录而言的。这意味着如果某一天你在某个位置部署了你的应用程序，如 www.somehost.com/myapp ，然后又将它部署到了 www.anotherhost.com （没有'myapp'子目录），这样的匹配模式仍将继续工作。所有的路径都是相对于 HttpServletRequest.getContextPath() 的值来的。

Order Matters! 秩序
URL 路径表达式按事先定义好的顺序判断传入的请求，并遵循 FIRST MATCH WINS 这一原则。例如，让我们假设有如下链的定义：
```
/account/** = ssl, authc
/account/signup = anon
```

如果传入的请求旨在访问 /account/signup/index.html（所有 'anon'ymous 用户都能访问），那么它将永不会被处理！原因是因为/account/* 的模式第一个匹配了传入的请求，“短路”了其余的定义。 始终记住基于FIRST MATCH WINS 的原则定义你的过滤器链！*

#### Filter Chain Definitions 定义
等号右边是逗号隔开的过滤器列表，用来执行匹配该路径的请求。它必须符合以下格式：
```
filter1[optional_config1], filter2[optional_config2], ..., filterN[optional_configN]
```

并且：
* filterN 是一个定义在[main]项中的 filter bean 的名字
* [optional_configN]是一个可选的括号内的对特定的路径，特定的过滤器有特定含义的字符串（每个过滤器，每个路径的具体配置！）。若果该过滤器对该 URL 路径并不需要特定的配置，你可以忽略括号，于是 filteNr[] 就变成了 filterN.

因为过滤器标志符定义了链（又名列表），所以请记住顺序问题！请按顺序定义好你的逗号分隔的列表，这样请求就能够流通这个链。
最后，每个过滤器按照它期望的方式自由的处理请求，即使不具备必要的条件（例如，执行一个重定向，响应一个 HTTP 错误代码，直接渲染等）。否则，它有可能允许该请求继续通过这个过滤器链到达最终的视图。

确保能够对路径的具体配置作出反应，即[optional_configN]一个过滤器标志符的一部分，是一个对所有Shiro 过滤器 适用的独有的功能。

你也可以这样做，如果你想创建你自己的 javax.servlet.Filter 的实现的话，确保你的过滤器子类
(http://shiro.apache.org/static/1.4.1/apidocs/org/apache/shiro/web/filter/PathMatchingFilter.html)

#### Available Filters 可用的过滤器
在过滤器链中能够使用的过滤器“池”被定义在[main]项。在[main]项中指派给它们的名字就是在过滤器链定义中使 用的名字。例如：
```
[main]
...
myFilter = com.company.web.some.FilterImplementation
myFilter.property1 = value1
...

[urls]
...
/some/path/** = myFilter
```

#### Default Filters
当运行一个 Web 应用程序时，Shiro 将会创建一些有用的默认 Filter 实例，并自动地在[main]项中将它们置为可用。

你可以在 main 中配置它们，当作在你的链的定义中你是否有任何其他的bean 和 reference。例如：
```
[main]
...
# Notice how we didn't define the class for the FormAuthenticationFilter ('authc') - it is instantiated and available already:
authc.loginUrl = /login.jsp
...

[urls]
...
# make sure the end-user is authenticated.  If not, redirect to the 'authc.loginUrl' above,
# and after successful authentication, redirect them back to the original account page they
# were trying to view:
/account/** = authc
...
```

自动地可用的默认的Filter 实例是被 DefaultFilter 枚举定义的，枚举的名称字段是可供配置的名称。它们是：

iframe id="tmp_downloadhelper_iframe" style="display: none;" /iframe

![shiro13.PNG](https://i.loli.net/2019/11/08/RPKzTFsi6vDruGy.png)

Filter Name	|Class|	Description
---|--|----------------------
anon|	org.apache.shiro.web.filter.authc.AnonymousFilter	|匿名拦截器，即不需要登录即可访问；一般用于静态资源过滤；示例/static/**=anon
authc|	org.apache.shiro.web.filter.authc.FormAuthenticationFilter	|基于表单的拦截器；如/**=authc，如果没有登录会跳到相应的登录页面登录
authcBasic|	org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter	|Basic HTTP身份验证拦截器
logout	|org.apache.shiro.web.filter.authc.LogoutFilter	|退出拦截器，主要属性：redirectUrl：退出成功后重定向的地址（/），示例/logout=logout
noSessionCreation|	org.apache.shiro.web.filter.session.NoSessionCreationFilter	|不创建会话拦截器，调用subject.getSession(false)不会有什么问题，但是如果subject.getSession(true)将抛出DisabledSessionException异常
perms|	org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter	|权限授权拦截器，验证用户是否拥有所有权限；属性和roles一样；示例/user/**=perms["user:create"]
port	|org.apache.shiro.web.filter.authz.PortFilter|	端口拦截器，主要属性port(80)：可以通过的端口；示例/test= port[80]，如果用户访问该页面是非80，将自动将请求端口改为80并重定向到该80端口，其他路径/参数等都一样
rest	|org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter|	rest风格拦截器，自动根据请求方法构建权限字符串；示例/users=rest[user]，会自动拼出user:read,user:create,user:update,user:delete权限字符串进行权限匹配（所有都得匹配，isPermittedAll）
roles|	org.apache.shiro.web.filter.authz.RolesAuthorizationFilter|	角色授权拦截器，验证用户是否拥有所有角色；示例/admin/**=roles[admin]
ssl	|org.apache.shiro.web.filter.authz.SslFilter	|SSL拦截器，只有请求协议是https才能通过；否则自动跳转会https端口443；其他和port拦截器一样；
user|	org.apache.shiro.web.filter.authc.UserFilter	|用户拦截器，用户已经身份验证/记住我登录的都可；示例/**=user

#### 启用和禁用过滤器启动，补充过滤器
