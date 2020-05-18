---
title: canvas中的转换函数
time: 2019-11-2
categories: canvas
tags: canvas
---
# canvas中的转换函数
---

## 保存和还原状态
在介绍转换方法之前，让我们看一下另外两种方法，一旦开始生成更复杂的图形，这些方法必不可少。
* save()
保存画布的整个状态。
* restore()
恢复最近保存的画布状态

画布状态存储在堆栈中。每次save()调用该方法时，当前的绘图状态都会被压入堆栈。绘图状态包括
* 已应用转换（即translate，rotate和scale-见下文）。
* 以下属性的当前值：strokeStyle，fillStyle，globalAlpha，lineWidth，lineCap，lineJoin，miterLimit，lineDashOffset，shadowOffsetX，shadowOffsetY，shadowBlur，shadowColor，globalCompositeOperation，font，textAlign，textBaseline，direction，imageSmoothingEnabled。
* 当前的剪切路径，我们将在下一部分中看到。

您可以save()根据需要多次调用该方法。每次restore()调用该方法时，都会从堆栈中弹出最后一个保存的状态，并还原所有保存的设置。

## 平移
我们将介绍的第一种转换方法是translate()。此方法用于将画布及其原点移动到网格中的其他点。
* translate(x, y)
在画布上移动画布及其原点。x指示要移动的水平距离，并y指示垂直移动网格的距离。


## 旋转
第二种转换方法是rotate()。我们使用它来围绕当前原点旋转画布。
* rotate(angle)
按angle弧度数围绕当前原点*顺时针*旋转画布。

旋转中心点始终是画布原点。要更改中心点，我们需要使用translate()方法移动画布。
```
提醒：角度以弧度而不是度为单位。要进行转换，我们使用：radians = (Math.PI/180)*degrees。
```

## 缩放
下一个转换方法是缩放。我们使用它来增加或减少画布网格中的单位。这可用于绘制按比例缩小或放大的形状和位图。
* scale(x, y)
水平按x缩放画布单位，垂直按y缩放画布单位。这两个参数都是实数。小于1.0的值将减小单位大小，而大于1.0的值将增大单位大小。值1.0会使单位保持相同大小。
使用负数可以进行轴镜像（例如，使用translate(0,canvas.height); scale(1,-1);会具有众所周知的笛卡尔坐标系，其原点位于左下角）。

默认情况下，画布上的一个单位正好是一个像素。例如，如果我们应用比例因子0.5，则结果单位将变为0.5像素，因此将以一半大小绘制形状。以类似的方式将缩放因子设置为2.0将增加单位大小，并且一个单位现在变为两个像素。这将导致绘制两倍大的形状。