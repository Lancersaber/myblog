---
title: Spring Boot Shiro用户认证
time: 2019-11-06
categories: spring
tags: spring,shiro
---

# Spring Boot Shiro用户认证
---
在Spring Boot中集成Shiro进行用户的认证过程主要可以归纳为以下三点
* 1、定义一个ShiroConfig，然后配置SecurityManager Bean，SecurityManager为Shiro的安全管理器，管理着所有Subject；

* 2、在ShiroConfig中配置ShiroFilterFactoryBean，其为Shiro过滤器工厂类，依赖于SecurityManager；

* 3、自定义Realm实现，Realm包含doGetAuthorizationInfo()和doGetAuthenticationInfo()方法，因为本文只涉及用户认证，所以只实现doGetAuthenticationInfo()方法。

## 引入依赖

```
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.4.0</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.shiro/shiro-cas -->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-cas</artifactId>
            <version>1.4.0</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

```

## ShiroConfig
定义一个Shiro配置类，名称为ShiroConfig：
```
@Configuration
public class ShiroConfig {
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        // 设置securityManager
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        // 登录的url
        shiroFilterFactoryBean.setLoginUrl("/login");
        // 登录成功后跳转的url
        shiroFilterFactoryBean.setSuccessUrl("/index");
        // 未授权url
        shiroFilterFactoryBean.setUnauthorizedUrl("/403");
        
        LinkedHashMap<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        
        // 定义filterChain，静态资源不拦截
        filterChainDefinitionMap.put("/css/**", "anon");
        filterChainDefinitionMap.put("/js/**", "anon");
        filterChainDefinitionMap.put("/fonts/**", "anon");
        filterChainDefinitionMap.put("/img/**", "anon");
        // druid数据源监控页面不拦截
        filterChainDefinitionMap.put("/druid/**", "anon");
        // 配置退出过滤器，其中具体的退出代码Shiro已经替我们实现了 
        filterChainDefinitionMap.put("/logout", "logout");
        filterChainDefinitionMap.put("/", "anon");
        // 除上以外所有url都必须认证通过才可以访问，未通过认证自动访问LoginUrl
        filterChainDefinitionMap.put("/**", "authc");
        
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
        return shiroFilterFactoryBean;
    }
	
    @Bean  
    public SecurityManager securityManager(){  
        // 配置SecurityManager，并注入shiroRealm
        DefaultWebSecurityManager securityManager =  new DefaultWebSecurityManager();
        securityManager.setRealm(shiroRealm());
        return securityManager;  
    } 
	
    @Bean  
    public ShiroRealm shiroRealm(){  
        // 配置Realm，需自己实现
        ShiroRealm shiroRealm = new ShiroRealm();  
        return shiroRealm;  
    }  
}
```

需要注意的是filterChain基于短路机制，即最先匹配原则，如：
```
/user/**=anon
/user/aa=authc 永远不会执行
```

其中anon、authc等为Shiro为我们实现的过滤器，具体如下表所示：

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


配置完ShiroConfig后，接下来对Realm进行实现，然后注入到SecurityManager中。

## Realm
自定义Realm实现只需继承AuthorizingRealm类，然后实现doGetAuthorizationInfo()和doGetAuthenticationInfo()方法即可。这两个方法名乍看有点像，authorization发音[ˌɔ:θəraɪˈzeɪʃn]，为授权，批准的意思，即获取用户的角色和权限等信息；authentication发音[ɔ:ˌθentɪ’keɪʃn]，认证，身份验证的意思，即登录时验证用户的合法性，比如验证用户名和密码。
```
public class ShiroRealm extends AuthorizingRealm {

    @Autowired
    private UserMapper userMapper;
    
    /**
    * 获取用户角色和权限
    */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principal) {
        return null;
    }

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

    	// 获取用户输入的用户名和密码
        String userName = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());
        
        System.out.println("用户" + userName + "认证-----ShiroRealm.doGetAuthenticationInfo");

        // 通过用户名到数据库查询用户信息
        User user = userMapper.findByUserName(userName);
        
        if (user == null) {
            throw new UnknownAccountException("用户名或密码错误！");
        }
        if (!password.equals(user.getPassword())) {
            throw new IncorrectCredentialsException("用户名或密码错误！");
        }
        if (user.getStatus().equals("0")) {
            throw new LockedAccountException("账号已被锁定,请联系管理员！");
        }
        SimpleAuthenticationInfo info = new SimpleAuthenticationInfo(user, password, getName());
        return info;
    }
}
```
因为本节只讲述用户认证，所以doGetAuthorizationInfo()方法先不进行实现。

其中UnknownAccountException等异常为Shiro自带异常，Shiro具有丰富的运行时AuthenticationException层次结构，可以准确指出尝试失败的原因。你可以包装在一个try/catch块，并捕捉任何你希望的异常，并作出相应的反应。例如：
```
try {
    currentUser.login(token);
} catch ( UnknownAccountException uae ) { ...
} catch ( IncorrectCredentialsException ice ) { ...
} catch ( LockedAccountException lae ) { ...
} catch ( ExcessiveAttemptsException eae ) { ...
} ... catch your own ...
} catch ( AuthenticationException ae ) {
    //unexpected error?
}
```

## 数据层
首先创建一张用户表，用于存储用户的基本信息（基于Mysql）：
```
CREATE TABLE user(
	ID INT PRIMARY KEY auto_increment,
   USERNAME VARCHAR(20) NOT NULL ,
   PASSWD VARCHAR(128) NOT NULL ,
   CREATE_TIME date ,
   STATUS CHAR(1) NOT NULL 
)

insert into user VALUES (2, 'test', '7a38c13ec5e9310aed731de58bbc4214','2017-11-19 17:20:21', '0');
insert into user VALUES (1, 'mrbird', '42ee25d1e43e9f57119a00d0a39e5250', '2017-11-19 10:52:48', '1');
```

## Controller
LoginController代码如下：
```
@Controller
public class LoginController {

    @GetMapping("/login")
    public String login() {
        return "login";
    }

    @PostMapping("/login")
    @ResponseBody
    public ResponseBo login(String username, String password) {
    	// 密码MD5加密
        password = MD5Utils.encrypt(username, password);
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        // 获取Subject对象
        Subject subject = SecurityUtils.getSubject();
        try {
            subject.login(token);
            return ResponseBo.ok();
        } catch (UnknownAccountException e) {
            return ResponseBo.error(e.getMessage());
        } catch (IncorrectCredentialsException e) {
            return ResponseBo.error(e.getMessage());
        } catch (LockedAccountException e) {
            return ResponseBo.error(e.getMessage());
        } catch (AuthenticationException e) {
            return ResponseBo.error("认证失败！");
        }
    }

    @RequestMapping("/")
    public String redirectIndex() {
        return "redirect:/index";
    }

    @RequestMapping("/index")
    public String index(Model model) {
    	// 登录成后，即可通过Subject获取登录的用户信息
        User user = (User) SecurityUtils.getSubject().getPrincipal();
        model.addAttribute("user", user);
        return "index";
    }
}
```

登录成功后，根据之前在ShiroConfig中的配置shiroFilterFactoryBean.setSuccessUrl("/index")，页面会自动访问/index路径。