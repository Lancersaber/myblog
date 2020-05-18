---
title: Java枚举类型
time: 2019-11-08
categories: Java
tags: Java
---

# Java枚举类型
---
&emsp;&emsp;关键字enum可以将一组具名的值的有限集合创建为一种新的类型，而这些具名的值可以作为常规的程序组件使用。这是一种非常有用的功能。

## 基本enum特性
&emsp;&emsp;创建enum时，编译器会为你生成一个相关的类，这个类继承自java.lang.Enum.下面是这个类的一些功能
* ordinal()：返回一个int值，这是每个enum实例在声明时的次序
* values():遍历enum实例的数组，而且该数组中的元素严格保持其在enum中声明时的顺序，
* Enum类实现了Comparable接口和Serializable接口

## 向enum中添加新方法
&emsp;&emsp;除了不能继承自一个enum之外，我们基本上可以将enum看作一个常规的类。也就是说，我们可以向enum中添加方法。enum甚至可以有main方法。
一般来说，我们希望每个枚举类型实例能够返回对自身的描述，而不仅仅只是默认的toString()实现，这只能返回枚举实例类型的名字。为此，你可以提供一个构造器，专门负责处理这个额外的信息，然后添加一个方法，返回这个描述信息。
```
public enum OzWitch {
  // Instances must be defined first, before methods:
  WEST("Miss Gulch, aka the Wicked Witch of the West"),
  NORTH("Glinda, the Good Witch of the North"),
  EAST("Wicked Witch of the East, wearer of the Ruby " +
    "Slippers, crushed by Dorothy's house"),
  SOUTH("Good by inference, but missing");

  private String description;

  // Constructor must be package or private access:
  private OzWitch(String description) {
    this.description = description;
  }

  public String getDescription() { return description; }

  public static void main(String[] args) {
    for(OzWitch witch : OzWitch.values())
      print(witch + ": " + witch.getDescription());
  }
} /* Output:
WEST: Miss Gulch, aka the Wicked Witch of the West
NORTH: Glinda, the Good Witch of the North
EAST: Wicked Witch of the East, wearer of the Ruby Slippers, crushed by Dorothy's house
SOUTH: Good by inference, but missing
*///:~

```

后面还有更加高级的知识，见Java编程思想