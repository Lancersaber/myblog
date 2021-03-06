---
title: lambda表达式
time: 2019-11-09
categories: Java
tags: Java
---

# lambda表达式
可以把lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但它有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表。这个定义够大的，让我们慢慢道来。

* 匿名---我们说匿名，是因为它不像普通的方法那样有一个明确的名称
* 函数---我们说它是函数，是因为Lambda函数不像方法那样属于某个特定的类，但和方法一样，Lambda有参数列表、函数主体、返回类型
* 传递---Lambda表达式可以作为参数传递给方法或存储在变量中
* 简洁---无需像匿名类那样写很多模板代码

Lambda表达式有三个部分
* 参数列表
* 箭头
* Lambda主体
几个有效的Lambda表达式
```
(String s)->s.length
(Apple a) -> a.getWeight()>150
(int x,int y) ->{
	System.out.println(x+y);
}

() -> 42
```

## 在哪里使用Lambda表达式
可以在函数式接口上使用Lambda表达式。什么是函数式接口？简单来说，就是只含有一个方法的接口。