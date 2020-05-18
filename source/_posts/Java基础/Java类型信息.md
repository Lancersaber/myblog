---
title: Java类型信息
time: 2019-11-04
categories: Java
tags: Java
---

# Java类型信息
---
运行时类型信息使得你可以在程序运行时发现和使用类型信息。
现在我们来讨论如何让我们在运行时识别对象和类的信息的。主要有两种方式：
* 传统的RTTI,它假定我们在编译时就已经知道了所有的类型
* 反射机制，它允许我们在运行时发现和使用类的信息。

## 为什么需要RTTI(Run-Time Type Identification)
RTTI:在运行时，识别一个对象的类型。

面向对象编程中的基本目的是：让代码只操作对基类的引用。这样，如果要添加一个新类来扩展程序，就不会影响到原来的代码。下面举个例子：
```
class Shape{
	//draw函数
	draw();
}

class Circle extends Shape;
class Square extends Shape;
calss Triangle extends Shape;

test(Shape shape){
	shape.draw();
}
```
三个类都继承自Shape，复写各自的drwa()函数，在test函数中，传入不同的子类可以触发各自的draw函数，实现这个功能就需要RTTI的支持。

## Class对象
&emsp;&emsp;要理解RTTI在Java中的工作原理，首先必须知道类型信息在运行时是如何表示的。这项工作是由称为Class对象的特殊对象完成的，它包含了与类相关的信息。事实上，Class对象就是用来创建类的所有的“常规”对象的。Java使用Class对象来执行其RTTI。
&emsp;&emsp;类是程序的一部分，每个类都有一个Class对象。换言之，每当编写并且编译了一个新类，就会产生一个Class对象(更恰当的说，是被保存在一个同名的.class文件中)。为了生成这个类的对象，运行这个程序的Java虚拟机将使用被称为“类加载器”的子系统。所有的类都是在对其第一次使用时，动态加载到JVM中的。
&emsp;&emsp;无论何时，只要你想在运行时使用类型信息，就必须首先获得对恰当的Class对象的引用。Class.forName()就是实现此功能的便捷途径，它是用一个包含目标类的文本名的String作输入参数，返回的一个Class对象的引用。你不需要为了获得Class引用而持有该类型的对象。*但是，如果你已经拥有了一个感兴趣的类型的对象，那就可以通过调用getClass()方法来获取Class引用了，这个方法属于Object，表示将返回该对象的实际类型的Class引用*。

使用Class对象可以获得一些关于这个Class定义的基本信息，包括包名、注释、父类、接口等等，使用Class的newInstance()方法可以创建一个实例，注意，使用newInstance()创建的类，必须带有默认的构造器。

## 类字面常量
Java还提供了另一种方法来生成对Class对象的引用，即使用类字面常量。即类名.class,这样做不仅更简单，而且更安全，因为它在编译时就会受到检查(因此不需要置于try语句块中)。

## 泛化的Class引用
&emsp;&emsp;Class引用总是指向某个Class对象，Class表示的就是它所指向的对象的确切类型，而该对象便是Class类的一个对象。
```
class GenericClassReferences{
    public static void main(String[] args){
        Class intClass=int.class;
        Class<Integer> genericIntClass=int.class;
        genericIntClass=intClass;// Same thing
        intClass=double.class;//No problem
//        genericIntClass=double.class;//Illegal
    }
}
```
普通的类引用不会产生警告信息，你可以看到，尽管泛型类引用只能赋值为指向其声明的类型，但是普通的类引用可以重新赋值为指向任何其他的Class对象。
&emsp;&emsp;为了在使用泛型化的Class引用时放松限制，可以使用通配符，通配符就是"?",表示任何事物。因此，我们可以在上例的普通Class引用中添加通配符，并产生相同的结果。
```
public class WildcardClassReferences{
	  public static void main(String[] args){
        Class<?> intClass=int.class;
        intClass=double.class;
    }
}
```
在Java中，Class<?>优于Class，即使它们等价，Class<?>的好处是它表示你并非是碰巧或者是由于疏忽，而使用了一个非具体的类引用，你就是选择了非具体的版本。


## InstanceOf
关键字instanceof,它返回一个布尔值，告诉我们对象是不是某个特定类型的实例。
对instanceof有比较严格的限制，只可以将其与命名类型进行比较，而不能与Class对象做比较。

## 反射：运行时的类信息
&emsp;&emsp;Class类与java.lang.reflect类库一起对反射的概念进行了支持，该类库包含了Field,Method以及Constructor类(每个都实现了Member接口)。这些类型的对象是由JVM在运行时创建的，用以表示未知类里对应的成员。*这样你就可以使用Constructor创建新的对象，用get()和set()方法读取和修改与Field对象关联的字段，用invoke()方法调用与Method对象关联的方法。另外，还可以调用getFields(),getMethods()和getConstructors()等便利的方法，以返回表示字段，方法以及构造器的对象的数组。*这样，匿名对象的类信息就能在运行时被完全确定下来，而在编译时不需要知道任何信息。
&emsp;&emsp;在使用反射时，必须获得目标类的.class文件，要么在本地文件，要么通过网络获取，所以RTTI和反射之间真正的区别在于，对RTTI来说，编译器在编译时打开和检查.class文件。而对于反射机制来说，.class文件在编译时是不可获取的，所以是在运行时打开和检查.class文件。