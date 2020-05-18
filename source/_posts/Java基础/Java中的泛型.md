---
title: Java中泛型
time: 2019-11-15
categories: Java
tags: Java
---

# Java中泛型
---
一般的类和方法，只能使用具体的类型，要么是基本类型，要么是自定义的类。如果要编写可以应用于多种类型的代码，这种刻板的限制对代码的束缚就会很大。

## 泛型类
```
package generics;//: generics/Holder3.java

public class Holder3<T> {
  private T a;
  public Holder3(T a) { this.a = a; }
  public void set(T a) { this.a = a; }
  public T get() { return a; }
  public static void main(String[] args) {
    Holder3<Automobile> h3 =
      new Holder3<Automobile>(new Automobile());
    Automobile a = h3.get(); // No cast needed
    // h3.set("Not an Automobile"); // Error
    // h3.set(1); // Error
  }
} ///:~

```
当你创建Holder3对象时，必须指明持有什么类型的对象，将其置于尖括号中。然后，你就只能在Holder3中存入该类型(或其子类，因为多态与泛型不冲突)。并且在你从Holder3中取出它持有的对象时，自动就是正确的类型。

这就是Java泛型的核心概念：告诉编译器想使用什么类型，然后编译器帮你处理一切细节。

## 泛型接口
泛型也可以应用于接口。
```
public interface Generator<T>{ T next();}
```

## 泛型方法
泛型同样可以使用于方法上。注意泛型方法所在的类可以是泛型类，也可以不是。也就是说，是否拥有泛型方法，与其所在的类是否是泛型没有关系。
对于static的方法，无法访问泛型类的类型参数，所以，如果static方法需要使用泛型能力，就必须使其成为泛型方法。

要成为泛型方法，只需将泛型参数列表置于返回值之前
```
public class GenericMethods{
	public <T> void f(T x){
		System.out.println(x.getClass().getName());
	} 
}
```

## 泛型的擦除
我们可以声明ArrayList.class，但是不能声明ArrayList<Integer>.class.看下面的代码
```
public class ErasedTypeEquivalence {
  public static void main(String[] args) {
    Class c1 = new ArrayList<String>().getClass();
    Class c2 = new ArrayList<Integer>().getClass();
    System.out.println(c1 == c2);
  }
} /* Output:
true
*///:~

```

在泛型代码内部，无法获得任何有关泛型参数类型的信息。
Java泛型是使用擦除来实现的，这意味着当你在使用泛型时，任何具体的类型信息都被擦除了，你唯一知道的就是你在使用一个对象。因为List<Integer>和List<String>在运行时事实上是相同的类型。这两种类型都i被擦除成它们的"原生类型"，即List

## 泛型的边界
因为擦除了类型信息，所以在无界泛型参数调用的方法只是那些可以用Object调用的方法。但是，如果能够将这个参数限制为某个类型子集，那么你就可以用这些子集来调用方法。为了执行这种限制，Java泛型重用了extends关键子。
```
interface HasColor { java.awt.Color getColor(); }

class Colored<T extends HasColor> {
  T item;
  Colored(T item) { this.item = item; }
  T getItem() { return item; }
  // The bound allows you to call a method:
  java.awt.Color color() { return item.getColor(); }
}
```
