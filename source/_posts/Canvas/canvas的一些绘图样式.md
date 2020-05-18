---
title: canvas的绘图样式
time: 2019-11-1
categories: canvas
tags: canvas
---
# Canvas的绘图样式
---

## 色彩
到目前为止，我们只看到了绘图上下文的方法。如果要对形状应用颜色，可以使用两个重要的属性：fillStyle和strokeStyle。
* fillStyle = color
设置填充形状时使用的样式。
* strokeStyle = color
设置形状轮廓的样式。

color是表示CSS <color>，渐变对象或图案对象的字符串。稍后我们将介绍渐变和图案对象。默认情况下，笔触和填充颜色设置为黑色（CSS颜色值#000000）。
tips：注意：设置strokeStyleand / or fillStyle属性时，新值将成为从此以后绘制的所有形状的默认值。对于您要使用不同颜色的每个形状，都需要重新分配fillStyle或strokeStyle属性。

根据规范，您可以输入的有效字符串应为CSS <color>值。下面的每个示例描述相同的颜色
```
// these all set the fillStyle to 'orange'

ctx.fillStyle = 'orange';
ctx.fillStyle = '#FFA500';
ctx.fillStyle = 'rgb(255, 165, 0)';
ctx.fillStyle = 'rgba(255, 165, 0, 1)';
```

## 透明度
除了在画布上绘制不透明形状外，我们还可以绘制半透明（或半透明）形状。这可以通过设置globalAlpha属性或为笔触和/或填充样式分配半透明颜色来完成。
* globalAlpha = transparencyValue
将指定的透明度值应用于画布上绘制的所有将来形状。该值必须介于0.0（完全透明）到1.0（完全不透明）之间。默认情况下，此值为1.0（完全不透明）。

globalAlpha如果要在画布上绘制许多具有相似透明度的形状，则此属性很有用，但是通常在设置单个形状的颜色时设置其透明度通常会更有用。
由于strokeStyleand fillStyle属性接受CSS rgba颜色值，因此我们可以使用以下符号为它们分配透明颜色。
```
// Assigning transparent colors to stroke and fill style

ctx.strokeStyle = 'rgba(255, 0, 0, 0.5)';
ctx.fillStyle = 'rgba(255, 0, 0, 0.5)';
```

该rgba()功能与该功能相似，rgb()但是有一个额外的参数。最后一个参数设置此特定颜色的透明度值。有效范围还是介于0.0（完全透明）和1.0（完全不透明）之间。

## 线型
有几个属性使我们可以设置线条样式。
* lineWidth = value
设置将来绘制的线条的宽度。
* lineCap = type
设置行尾的外观。
* lineJoin = type
设置线条相交处的“角”的外观。
* miterLimit = value
当两条直线以锐角连接时，对斜接进行限制，以使您可以控制接合处的厚度。
* getLineDash()
返回包含偶数个非负数的当前行虚线图案阵列。
* setLineDash(segments)
设置当前线的虚线图案。
* lineDashOffset = value
指定在一行上从何处开始破折号数组。

## 渐变色
就像任何普通的绘图程序一样，我们可以使用线性和径向渐变来填充和描边形状。我们CanvasGradient使用以下方法之一创建对象。然后，我们可以将此对象分配给fillStyle或strokeStyle属性。
* createLinearGradient(x1, y1, x2, y2)
创建一个线性渐变对象，其起点为（x1，y1），终点为（x2，y2）。
* createRadialGradient(x1, y1, r1, x2, y2, r2)
创建一个径向渐变。参数代表两个圆，一个圆的中心为（x1，y1），半径为r1，另一个圆的中心为（x2，y2），半径为r2。
例：
```
var lineargradient = ctx.createLinearGradient(0, 0, 150, 150);
var radialgradient = ctx.createRadialGradient(75, 75, 0, 75, 75, 100);
```
创建CanvasGradient对象后，可以使用addColorStop()方法为对象分配颜色
* gradient.addColorStop(position, color)
在gradient对象上创建一个新的色标。position是在0.0和1.0之间的数，并且限定在渐变中的颜色的相对位置，并且所述color参数必须是表示CSS一个字符串color，表示颜色的梯度应该偏移处的过渡达到。

您可以根据需要向渐变添加尽可能多的色标。下面是一个非常简单的从白色到黑色的线性渐变。
```
var lineargradient = ctx.createLinearGradient(0, 0, 150, 150);
lineargradient.addColorStop(0, 'white');
lineargradient.addColorStop(1, 'black');
```
