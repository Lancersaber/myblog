---
title: DOM
time: 2019-11-05
categories: Javascript
tags: Javascript
---

# DOM
---

## 节点关系
&emsp;&emsp;文档中所有的节点之间都存在这样或那样的关系。节点间的各种关系可以用传统的家族关系来描述，相当于把文档树比喻成家谱。在HTML中，可以将body元素看成是html的子元素，相应的，也就可以将html元素看成是body元素的父元素。而head元素，则可以看成是body元素的同胞元素，因为它们都是同一个父节点html的直接子元素。
&emsp;&emsp;每个节点都有一个ChildNodes属性，其中保存着一个NodeList对象。NodeList是一种类数组对象，用于保存一组有序的节点，可以通过位置来访问这些节点。*请注意，虽然可以通过方括号语法来访问NodeList的值，而且这个对象也有length属性，但它不是Array的实例*。下面展示了如何访问保存在NodeList中的节点。
```
var firstChild=someNode.childNodes[0];
var secondChild=someNode.childNodes.item(1);
var count=someNode.childNodes.length;
```