---
title: JSON语法
time: 2019-10-28
categories: JSON
tags: JSON
---
# JSON语法
---
## JSON 语法规则
JSON 语法是 JavaScript 对象表示法语法的子集。

* 数据在名称/值对中
* 数据由逗号分隔
* 花括号保存对象
* 方括号保存数组

## JSON 名称/值对
JSON 数据的书写格式是：名称/值对。

名称/值对包括字段名称（在双引号中），后面写一个冒号，然后是值：
```
"firstName" : "John"
```
这很容易理解，等价于这条 JavaScript 语句：
```
firstName = "John"
```

## JSON 值
JSON 值可以是：

* 数字（整数或浮点数）
* 字符串（在双引号中）
* 逻辑值（true 或 false）
* 数组（在方括号中）
* 对象（在花括号中）
* null

## JSON 对象
JSON 对象在花括号中书写：

对象可以包含多个名称/值对：
```
{ "firstName":"John" , "lastName":"Doe" }
```
这一点也容易理解，与这条 JavaScript 语句等价：
```
firstName = "John"
lastName = "Doe"
```

## JSON 数组
JSON 数组在方括号中书写：

数组可包含多个对象：
```
{
"employees": [
{ "firstName":"John" , "lastName":"Doe" },
{ "firstName":"Anna" , "lastName":"Smith" },
{ "firstName":"Peter" , "lastName":"Jones" }
]
}
```
在上面的例子中，对象 "employees" 是包含三个对象的数组。每个对象代表一条关于某人（有姓和名）的记录。