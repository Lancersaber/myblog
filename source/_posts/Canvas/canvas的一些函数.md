---
title: canvas的一些函数
date: 2019-10-25 16:22:10
categories: canvas
tags:
---
# canvas的一些函数
---

canvas元素并未提供很多api，实际上，它只提供了两个属性与三个方法。
* width: canvas元素绘图表面的宽度。在默认状况下，浏览器会将canvas元素的大小设定成与绘图表面大小一致。
* height: canvas元素绘图表面的高度
* getContext():返回与该canvas元素相关的绘图环境对象。每个canvas元素均有一个这样的环境对象，而且每个环境对象均与一个canvas元素相关联。

## canvas状态的保存和恢复
很多时候我们只是想临时性的改变某个canvas的绘图属性。
* save():保存当前canvas绘图环境的所有属性
* restore()：恢复当前canvas绘图环境的所有属性

## canvas 原点 x 水平方向、原点 y 垂直方向，添加平移变换的方法。
void ctx.translate(x, y);
x：水平方向的移动距离。
y：垂直方向的移动距离。

## 绘制矩形
与SVG不同，它canvas仅支持两种基本形状：矩形和路径（由线连接的点的列表）。所有其他形状必须通过组合一个或多个路径来创建。幸运的是，我们有各种各样的路径绘制功能，它们可以组成非常复杂的形状。

首先让我们看一下矩形。可以在画布上绘制矩形的三个函数：

fillRect(x, y, width, height)
绘制一个填充的矩形。
strokeRect(x, y, width, height)
绘制矩形轮廓。
clearRect(x, y, width, height)
清除指定的矩形区域，使其完全透明。
这三个函数中的每一个都具有相同的参数。x并y指定矩形左上角在画布上的位置（相对于原点）。width并height提供矩形的大小。

## 绘制路径
现在让我们看一下路径。路径是由线段连接的点的列表，这些线段可以具有不同的形状（弯曲或不弯曲），不同的宽度和不同的颜色。路径甚至子路径都可以关闭。为了使用路径制作形状，我们采取了一些额外的步骤：
1、 首先，创建路径。
2、 然后，您可以使用绘图命令来绘制路径。
3、 创建路径后，您可以描画或填充路径以进行渲染。

以下是用于执行这些步骤的功能：
* beginPath()
创建一个新路径。创建之后，将来的绘图命令将直接进入路径并用于构建路径。
* 路径方法
为对象设置不同路径的方法。
* closePath()
向路径添加一条直线，转到当前子路径的起点。
* stroke()
通过抚摸轮廓来绘制形状。
* fill()
通过填充路径的内容区域绘制实体形状。
创建路径的第一步是调用beginPath()。在内部，路径存储为一起形成形状的子路径（线，弧等）的列表。每次调用此方法时，列表都会重置，我们可以开始绘制新形状。
```
注意：当当前路径为空时，例如在调用之后beginPath()，或在新创建的画布上moveTo()，无论实际是什么，第一个路径构造命令始终被视为a 。因此，您几乎总是希望在重置路径后专门设置起始位置。
```
第二步是调用实际指定要绘制路径的方法。我们很快就会看到这些。

第三步（也是可选步骤）是closePath()。此方法尝试通过绘制从当前点到起点的直线来闭合形状。如果形状已经关闭或列表中只有一个点，则此功能不执行任何操作。
```
注意：调用时fill()，所有打开的形状都会自动关闭，因此您不必调用closePath()。这不，当你打电话的情况stroke()。
```

示例：画一个三角形
```
function draw() {
  var canvas = document.getElementById('canvas');
  if (canvas.getContext) {
    var ctx = canvas.getContext('2d');

    ctx.beginPath();
    ctx.moveTo(75, 50);
    ctx.lineTo(100, 75);
    ctx.lineTo(100, 25);
    ctx.fill();
  }
}
```

## 移动笔
该功能是一个非常有用的功能，它实际上不会绘制任何内容，但会成为上述路径列表的一部分moveTo()。您可能最能想到的是将笔或铅笔从一张纸上的一个地方提起，然后放在另一张纸上。
* moveTo(x, y)
移动笔由指定的坐标x和y。

初始化或beginPath()调用画布时，通常将需要使用该moveTo()函数将起点放置在其他位置。我们还可以moveTo()用来绘制未连接的路径。看看下面的笑脸。

要自己尝试，可以使用下面的代码片段。只需将其粘贴到draw()我们之前看到的函数中即可。
```
function draw() {
  var canvas = document.getElementById('canvas');
  if (canvas.getContext) {
     var ctx = canvas.getContext('2d');

    ctx.beginPath();
    ctx.arc(75, 75, 50, 0, Math.PI * 2, true); // Outer circle
    ctx.moveTo(110, 75);
    ctx.arc(75, 75, 35, 0, Math.PI, false);  // Mouth (clockwise)
    ctx.moveTo(65, 65);
    ctx.arc(60, 65, 5, 0, Math.PI * 2, true);  // Left eye
    ctx.moveTo(95, 65);
    ctx.arc(90, 65, 5, 0, Math.PI * 2, true);  // Right eye
    ctx.stroke();
  }
}
```

## 绘制直线
要绘制直线，请使用lineTo()方法。
* lineTo(x, y)
绘制一条从当前绘图位置到所指定的位置的线x和y。

此方法有两个参数x和y，它们是直线终点的坐标。起点取决于先前绘制的路径，其中先前路径的终点是后续路径的起点，依此类推。也可以使用该moveTo()方法更改起点。

下面的示例绘制两个三角形，一个三角形填充，一个轮廓绘制。
```
function draw() {
  var canvas = document.getElementById('canvas');
  if (canvas.getContext) {
    var ctx = canvas.getContext('2d');

    // Filled triangle
    ctx.beginPath();
    ctx.moveTo(25, 25);
    ctx.lineTo(105, 25);
    ctx.lineTo(25, 105);
    ctx.fill();

    // Stroked triangle
    ctx.beginPath();
    ctx.moveTo(125, 125);
    ctx.lineTo(125, 45);
    ctx.lineTo(45, 125);
    ctx.closePath();
    ctx.stroke();
  }
}
```

## 绘制弧线
要绘制圆弧或圆，我们使用arc()或arcTo()方法。
* arc(x, y, radius, startAngle, endAngle, anticlockwise)
绘制一个以（x，y）位置为中心的圆弧，其半径r从startAngle开始，到endAngle为止，沿逆时针指示的给定方向（默认为顺时针方向）移动。
* arcTo(x1, y1, x2, y2, radius)
用给定的控制点和半径绘制一条弧，并通过一条直线将其连接到上一个点。
让我们更详细地看一下该arc方法，该方法有六个参数：x和y是应在其上绘制圆弧的圆心的坐标。radius是不言自明的。的startAngle和endAngle以弧度为单位的参数定义弧的起点和终点，沿该圆的曲线。这些是从x轴测量的。所述anticlockwise参数是一个布尔值，当true，得出弧逆时针; 否则，将顺时针绘制圆弧。
```
注意：arc功能中的角度以弧度而不是度为单位。要将度数转换为弧度，可以使用以下JavaScript表达式：radians = (Math.PI/180)*degrees。
```
