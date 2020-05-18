---
title: 创建Vue实例传入的options
time: 2020/4/20
categories: Vue
tags: Vue
---

## el:
	类型：string | HTMLElement
	作用: 决定之后vue实例会管理哪一个DOM
	el: "#app"
	vue.js内部会根据el的值是否为字符串来调用相应的函数，如果el的值为字符串，则调用document.querySelector(el),
	如果el不为字符串，则直接使用el的值来绑定对应的实例。

## data:
	类型：Object| Function(组件当中data必须是一个函数)
	作用：Vue实例对应的数据对象

## Methods:
	类型：Function
	作用：定义属于Vue的一些方法，可以在其他地方调用，也可以在指令中调用 