---
title: Javascript的基本概念
categories: JavaScript
---
# JavaScript的基本概念
---

## 区分大小写
&ensp;&ensp;&ensp;&ensp;Javascript的一切变量，函数等都区分大小写。

## 注释
&emsp;&emsp;Javascript的注释风格和Java类似

## 语句
虽然可以不以分号结尾，但是建议任何时候都不要省略。在写函数时，代码块的大括号也不要省。

## 变量
ECMAScript的变量是松散类型的，所谓松散类型就是可以用来保存任何类型的数据。定义变量时要使用var操作符，后跟一个标识符，如下所示
```
var message;
```
这行代码定义了一个名为message的变量，该变量可以用来保存任何类型的数据(像这样未经过初始化的变量，会保存一个特殊的值---undefined)。可以一条语句定义多个变量，用逗号分割即可。

## 数据类型
ECMAScript中有五种简单的数据类型(也称为基本数据类型):Undefined,Null,Boolean,Number和String。还有一种复杂的数据类型---Object，Object本质上是由一组无序的名值对组成的。

### typeof 操作符
鉴于ECMAScript是松散类型的，因此需要有一种手段来检测给定变量的数据类型 --- typeof就是负责提供这方面的操作符。对一个值使用typeof操作符可能放回下列某个字符串：
+ undefined：如果这个值未定义
+ boolean：如果这个值是布尔值
+ string： 如果这个值是字符串
+ number： 如果这个值是数值
+ object： 如果这个值是对象或null
+ function： 如果这个值是函数

### Undefined类型
Undefined类型只有一个值，即特殊的undefined。
```
注意：即使未初始化的变量会自动被赋予undefined值，但显示地初始化变量依旧是明智的选择，因为这样可以判断变量是未被声明，而不是尚未被初始化。(因为 typeof 一个未声明的变量也会返回undefined)
```
## NULL类型
NUll类型是第二个只有一个值的数据类型，这个特殊的值是null。从逻辑上看，null值表示的是一个空对象指针，而这也是使用typeof操作符检测null值时会返回"object"的原因。如果定义的变量准备在将来用来保存变量，那么最好将该变量初始化未null而不是其他值。

### Boolea类型
该类型只有两个值，true和false。要将一个值转换为其对应的Boolean值，可以调用转型函数Boolean().关于这个类型转换为布尔类型有一个规则，后面再写。

### Number类型

## 函数
通过函数可以封装任意多条语句，而且可以在任何地方，任何时候调用执行。ECMAScript中的函数使用function关键字来声明，后跟一组参数以及函数体。基本的语句如下
```
function functionname(arg0,arg1....argn){
	statement;
}
```
ECMAScript中的函数在定义时不必指定是否返回值，实际上，任何函数在任何时候都可以通过return语句后跟要返回的值来实现返回值。

### 理解参数
ECMAScript函数的参数与大多数其他语言中的函数有所不同。ECMAScript函数不介意传递进来多少个参数，也不在乎进来参数是什么数据类型。也就是说，即使你定义的函数只接收两个参数，在调用这个函数时也未必一定要传递两个参数。可以传递一个，三个甚至不传递参数，而不会出现问题。因为ECMAScript中的参数在内部是用一个数组来表示的。函数接收的始终是这个数组，而不关心数组中包含哪些参数。*实际上，在函数体内可以通过arguments对象来访问这个参数数组，从而获取传递给函数的每一个参数*
其实，arguments对象只是与数组类似(它并不是Array的实例)，因为可以使用方括号语法访问它的每一个元素，使用length属性来确定传递进来多少个参数。

### 没有重载
如果在script中定义了两个名字相同的函数，则该名字只属于后定义的函数。