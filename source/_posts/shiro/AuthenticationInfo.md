---
title: AuthenticationInfo
time: 2019-11-06
categories: shiro
tags: shiro
---

# AuthenticationInfo
---
AuthenticationInfo对象中存储的是主体（Subject）的身份认证信息。Shiro会调用CredentialsMatcher对象的doCredentialsMatch方法对AuthenticationInfo对象和AuthenticationToken进行匹配。匹配成功则表示主体（Subject）认证成功，否则表示认证失败。

![shiro3.PNG](https://i.loli.net/2019/11/06/ECzpZuIYTc3iBwv.png)

```
public interface AuthenticationInfo extends Serializable {
    // 获取用户身份信息
    //每个Realm反回一个AuthenticationInfo ，
    //所有AuthenticationInfo 中的PrincipalCollection 会根据指定规则进行合并
    PrincipalCollection getPrincipals();

    // 获取用户的 凭证信息（credentials），当有多个Realm返回多个AuthenticationInfo 时
    //  凭证信息 也会根据指定的规则进行合并
    // Shiro会使用 凭证信息（credentials） 验证用户身份的合法性。
    Object getCredentials();
}

public interface MergableAuthenticationInfo extends AuthenticationInfo {
    // 用于合并多个Realm返回的AuthenticationInfo 
    void merge(AuthenticationInfo info);
}

public interface Account extends AuthenticationInfo, AuthorizationInfo {

}

public interface SaltedAuthenticationInfo extends AuthenticationInfo {
    ByteSource getCredentialsSalt();// 获取凭证 信息的 盐
}
```

1、MergableAuthenticationInfo 用于提供在多 Realm 时合并 AuthenticationInfo 的功能，主要合并 Principal、如果是其他的如 credentialsSalt，会用后边的信息覆盖前边的。

2、SaltedAuthenticationInfo 用于对 凭证 信息加盐。比如 HashedCredentialsMatcher，在验证时会判断AuthenticationInfo 是否是SaltedAuthenticationInfo 子类，来获取盐信息。

3、Account 不仅继承了AuthenticationInfo，继承了AuthorizationInfo，也就是说它不仅包含主体的身份认证信息，还包含了主体的授权信息（角色、权限）。SimpleAccount 是Account的一个实现。在 IniRealm、PropertiesRealm这种静态创建帐号信息的场景中使用，这些 Realm 直接继承了 SimpleAccountRealm，而SimpleAccountRealm 提供了相关的 API 来动态维护 SimpleAccount；即可以通过这些 API来动态增删改查 SimpleAccount；动态增删改查角色/权限信息。如果您的帐号不是特别多，可以使用这种方式。

4、其他情况一般返回 SimpleAuthenticationInfo 即可。

因为我们可以在 Shiro 中同时配置多个 Realm，所以呢身份信息可能就有多个；因此其提供了 PrincipalCollection 用于聚合这些身份信息。
![shiro5.PNG](https://i.loli.net/2019/11/06/zwbWCY4a7kUyHjP.png)

```
public interface PrincipalCollection extends Iterable, Serializable {
    Object getPrimaryPrincipal(); //得到主要的身份
    <T> T oneByType(Class<T> type); //根据身份类型获取第一个
    <T> Collection<T> byType(Class<T> type); //根据身份类型获取一组
    List asList(); //转换为 List
    Set asSet(); //转换为 Set
    Collection fromRealm(String realmName); //根据 Realm 名字获取
    Set<String> getRealmNames(); //获取所有身份验证通过的 Realm 名字
    boolean isEmpty(); //判断是否为空
}
```

MutablePrincipalCollection 是一个可变的 PrincipalCollection 接口，即提供了如下可变方法：
```
public interface MutablePrincipalCollection extends PrincipalCollection {
    void add(Object principal, String realmName); //添加 Realm-Principal 的关联
    void addAll(Collection principals, String realmName); //添加一组 Realm-Principal 的关联
    void addAll(PrincipalCollection principals);//添加 PrincipalCollection
    void clear();//清空
}
```

目前 Shiro只提供了MutablePrincipalCollection的一个实现 SimplePrincipalCollection，在多 Realm 时 SimpleAuthenticationInfo 会合并多个 Principal为一个 PrincipalCollection