---
title: 基本知识
date: 2019-10-24 23:15:15
categories: canvas
tags:
---
# Canvas的基本知识
---
## canvas元素
定义：是HTML5提供的一种新标签, ie9才开始支持的，Canvas是一个矩形区域的画布，可以用JS控制每一个像素在上面绘画。canvas 标签使用 JavaScript 在网页上绘制图像，本身不具备绘图功能。canvas 拥有多种绘制路径、矩形、圆形、字符以及添加图像的方法。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

    <style>
        body{
            background: #dddddd;
        }
        #canvas{
            margin: 10px;
            padding: 10px;
            background: #ffffff;
            border: thin inset #aaaaaa;
        }
    </style>
</head>
<body>
<canvas id="canvas" width="600" height="300">

</canvas>
<script  src="example.js" type="text/javascript"></script>
</body>
</html>
```