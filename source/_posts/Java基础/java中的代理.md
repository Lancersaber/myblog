---
title: Java中的代理
categories: Java
---

# Java中的代理
---
## 代理是什么?
代理是基本的设计模式之一，它是你为了提供额外的或不同的操作，而插入的用来替代"实际"对象的对象。这些操作通常涉及与"实际"对象的通信，因此代理通常充当着中间人的角色。

## 静态代理
由程序员创建或工具生成代理类的源码，再编译代理类。**所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。**
	下面来看一个静态代理的例子
```
public interface Interface {
    void doSomething();
    void somethingElse(String arg);
}

public class RealObject implements Interface {
    @Override
    public void doSomething() {
        System.out.println("doSomething");
    }

    @Override
    public void somethingElse(String arg) {
        System.out.println("somethingElse "+arg);
    }
}

public class SimpleProxy implements Interface {

    private Interface proxied;

    public SimpleProxy(Interface proxied){
        this.proxied=proxied;
    }

    @Override
    public void doSomething() {
    	//注意这里在调用 doSomething()前做了一些动作
        System.out.println("SimpleProxy doSomething");
        proxied.doSomething();
    }

    @Override
    public void somethingElse(String arg) {
        System.out.println("SimpleProxy somethingElse "+arg);
        proxied.somethingElse(arg);
    }
}


```

上面的代码就是一个最简单的静态代理示例。从中可以看到：
+ 实际进行操作的类和代理类都实现了目标接口。
+ 实际操作的类封装在了代理类中。
+ 在调用实际的逻辑代码之前可以进行一些额外的操作

在任何时刻，只要你想要将额外的操作从"实际"对象分离到不同的地方，特别是当你希望能够很容易地做出修改，从没有使用额外操作转为使用这些操作，代理就显得很有用。

## 动态代理
Java的动态代理比代理的思想更向前迈进了一步，因为它可以动态地创建代理并动态地处理对所代理方法的调用。在动态代理上所做的所有调用都会被重定向到单一的调用处理器上，它的工作是揭示调用的类型并确定相应的对策。下面来看个小例子。
```

public class DynamicProxyHandler implements InvocationHandler {

    private Object proxied;

    public DynamicProxyHandler(Object proxied){
        this.proxied=proxied;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("***** proxy: "+proxy.getClass()+" , method: "+method+" , args: "+args);
        if (args!=null){
            for (Object arg: args){
                System.out.println(" "+arg);
            }
        }
        return method.invoke(proxied,args);
    }
}

public class SimpleDynamicProxy {

    public static void main(String[] args){
        RealObject real=new RealObject();
        consumer(real);
        //Insert a proxy and call again
        Interface proxy=(Interface) Proxy.newProxyInstance(Interface.class.getClassLoader(),new Class[]{Interface.class},new DynamicProxyHandler(real));
        consumer(proxy);
    }

    public static void consumer(Interface iface){
        iface.doSomething();
        iface.somethingElse("bonobo");
    }

}
```
---
通过调用静态方法Proxy.newProxyInstance()可以创建动态代理，这个方法需要得到一个类加载器(你通常可以从已经加载的对象中获取该其类加载器，然后传递给它)，一个你希望该类实现的接口列表，已经InvocationHandler接口的一个实现。动态代理可以将所有重定向到调用处理器，因此通常会向调用处理器的构造器传递给一个"实际"对象的引用，从而使得调用处理器在执行其中介任务时，可以将请求转发。

invoke()方法中传递进来了代理对象，以防你需要区分请求的来源，但是在许多情况下，你并不关心这一点。然而，在invoke()内部，在invoke
()内部，在代理上调用方法时需要格外小心，因为对接口的调用将被重定向为对代理的调用。通常，你会执行被代理的操作，然后使用Method.invoke()将请求转发给代理对象，并传入必需的参数。

## 动态代理的原理简要
