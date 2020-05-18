---
title: 在Html中使用JavaScript
categories: JavaScript
---

# 在Html中使用JavaScript
---
## script元素
向html页面中插入Javascript的主要方法，就是使用script元素。这个元素包括以下5个属性：
+ charset: 可选
+ defer: 可选
+ language: 已废弃
+ scr: 可选。表示包含要执行代码的外部文件
+ type:必须。表示编写代码使用的脚本语言的内容类型。目前type属性的值依旧是text/JavaScript
使用script元素的方式有两种：直接在页面中嵌入Javascript代码和包含外部JavaScript文件
在使用 script 元素嵌入JavaScript代码时，只须为 script 指定type属性，然后，将JavaScript代码直接放入在元素内部即可。
如果要通过 script 元素来包含外部Javascript文件，那么src属性就是必须的。这个属性的值是一个指向外部Javascript文件的链接。

## 标签的位置
现在的web程序一般都把全部Javascript引用放在 body元素中，放在页面的内容后面。这样，在解析包含的Javascript代码之前，页面的内容将完全呈现在浏览器中。