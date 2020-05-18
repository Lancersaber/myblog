---
title: BOM(浏览器对象模型)
date: 2019-10-24 22:59:25
categories: JavaScript
tags:
---
# BOM
---
## window对象
&emsp;&emsp;BOM的核心对象是window，它表示浏览器的一个实例。在浏览器中，window对象有双重角色，既是通过Js访问浏览器窗口的一个接口，又是ECMAScript规定的Global对象。这意味着在网页定义的任何一个对象、变量和函数，都以window作为其Global对象，因此有权访问。

### 全局作用域
&emsp;&emsp;由于window对象是Global对象，因此所有在全局作用域中声明的变量、函数都会变成window对象的属性和方法。
```
var age=29;
function sayAge() {
    alert(this.age);
}

alert(window.age);
sayAge();
window.sayAge();
```
我们在全局作用域中定义了一个变量age和一个函数sayAge()，它们被自动归在了window对象名下。于是，可以通过window.age访问变量age，可以通过window.sayAge()访问函数sayAge()。由于sayAge()存在于全局作用域中，因此this.age被映射到window.age,最终显示的仍然是正确的结果。

### 窗口位置
