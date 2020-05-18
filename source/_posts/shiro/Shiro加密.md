---
title: shiro加密
time: 2019-11-12
categories: shiro
tags: shiro
---

# shiro加密
---
在这里主要讲如何利用md5散列算法对密码进行加密与校验，md5是一种不可逆的散列算法，因此广泛应用与密码的加密，以及文件的md5校验等。

## SimpleHash生成密码散列值：
```
package cc.mrbird.febs.common.utils;

import org.apache.commons.lang3.StringUtils;
import org.apache.shiro.crypto.hash.SimpleHash;
import org.apache.shiro.util.ByteSource;

/**
 * @author MrBird
 */
public class MD5Util {

    protected MD5Util() {

    }

    //表明使用散列算法
    private static final String ALGORITH_NAME = "md5";

    //hash的次数
    private static final int HASH_ITERATIONS = 5;



    public static String encrypt(String username, String password) {
        String source = StringUtils.lowerCase(username);
        password = StringUtils.lowerCase(password);
        return new SimpleHash(ALGORITH_NAME, password, ByteSource.Util.bytes(source), HASH_ITERATIONS).toHex();
    }
}
```

SimpleHash类
```
  public SimpleHash(String algorithmName, Object source, Object salt, int hashIterations) throws CodecException, UnknownAlgorithmException {
        this.hexEncoded = null;
        this.base64Encoded = null;
        if (!StringUtils.hasText(algorithmName)) {
            throw new NullPointerException("algorithmName argument cannot be null or empty.");
        } else {
            this.algorithmName = algorithmName;
            this.iterations = Math.max(1, hashIterations);
            ByteSource saltBytes = null;
            if (salt != null) {
                saltBytes = this.convertSaltToBytes(salt);
                this.salt = saltBytes;
            }

            ByteSource sourceBytes = this.convertSourceToBytes(source);
            this.hash(sourceBytes, saltBytes, hashIterations);
        }
    }
```
自定义类设置了散列算法与散列次数，还对加密算法随机加盐处理，这样即使用户设置了相同的密码，在数据库生成的密码散列值也不一样，增加了用户的安全级别。