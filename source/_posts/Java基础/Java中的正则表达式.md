---
title: Java中的正则表达式
time: 2019-11-09
categories: Java
tags: Java
---

# Java中的正则表达式
---
正则表达式(regular expression)描述了一种字符串匹配的模式（pattern），可以用来检查一个串是否含有某种子串、将匹配的子串替换或者从某个串中取出符合某个条件的子串等。

例如：
* runoo+b，可以匹配 runoob、runooob、runoooooob 等，+ 号代表前面的字符必须至少出现一次（1次或多次）
* runoo\*b，可以匹配 runob、runoob、runoooooob 等，* 号代表字符可以不出现，也可以出现一次或者多次（0次、或1次、或多次）。
* colou?r 可以匹配 color 或者 colour，? 问号代表前面的字符最多只可以出现一次（0次、或1次）。

## 正则表达式中的元字符
见(https://www.runoob.com/regexp/regexp-metachar.html)

String类还自带了一个非常有用的正则表达式工具---split()方法，其功能是“将字符串从正则表达式匹配的地方切开”。

## Pattern和Matcher类
&emsp;&emsp;一般来说，比起功能有限的String类，我们更愿意构造功能更强大的正则表达式对象。只需导入 java.util.regax包，然后使用static Pattern.compile()方法来编译你的正则表达式即可。它会根据你的String类型的正则表达式生成一个Pattern对象。接下来，把你想要检索的字符串传入Pattern对象的matcher()方法。matcher()方法会生成一个Matcher对象，它有很多功能可用。

```
Pattern compile = Pattern.compile("\\d+");
        Matcher matcher = compile.matcher("345");
        System.out.println(matcher.find());
```

