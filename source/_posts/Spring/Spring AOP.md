---
title: Spring aop
categories: spring
tags: aop
---
# Spring AOP
---

## AOP概念

### Aspect
A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes (the schema-based approach) or regular classes annotated with the @Aspect annotation (the @AspectJ style).
这段话讲解了切面的意义。在某个类上使用@Aspect注解表示这个类是一个切面类。

### 连接点(Join point)
程序执行过程中某个特定的点，比如，调用方法或者处理异常。作用是指定在执行哪些方法时进行额外操作。

### 通知(Advice)
Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.
重点在第一句话--*Action taken by an aspect at a particular join point.*，指在切面的某个特定的连接点上执行的动作。Different types of advice include “around”, “before” and “after” advice.这句话表示Advice包含几种类型，环绕通知，前置通知和后置通知。

### 切入点(Pointcut)
A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.
*Advice is associated with a pointcut expression and runs at any join point matched by the pointcut*这句话表示Advice和一个切入点表达式是关联的，它会对所有符合这个切入点表达式的函数生效。Spring使用AspectJ 切入点表达式作为默认方式。

### Introduction

### Target object

### AOP proxy
 An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy.
 由aop框架为我们创建的代理对象。在spring框架中，这个aop代理对象是一个jdk的动态代理对象或者一个CGLIB代理对象。

### Weaving

## SpringAop涉及到的注解
|注解名|功能|
|:--:|:--:|
|@Aspect|声明该类是一个切面类|
|@After|通知方法会在目标方法返回或抛出异常后调用|
|@AfterRetruening|通常方法会在目标方法返回后调用|
|@AfterThrowing|通知方法会在目标方法抛出异常后调用|
|@Around|通知方法将目标方法封装起来|
|@Before|通知方法会在目标方法执行之前执行|
|@Pointcut|注解声明一个通用的切点|

### @PointCut注解的表达式
标准的Aspectj Aop的pointcut的表达式类型是很丰富的，但是Spring Aop只支持其中的9种，外加Spring Aop自己扩充的一种一共是10种类型的表达式，分别如下。

```
execution：一般用于指定方法的执行，用的最多。
within：指定某些类型的全部方法执行，也可用来指定一个包。
this：Spring Aop是基于代理的，生成的bean也是一个代理对象，this就是这个代理对象，当这个对象可以转换为指定的类型时，对应的切入点就是它了，Spring Aop将生效。
target：当被代理的对象可以转换为指定的类型时，对应的切入点就是它了，Spring Aop将生效。
args：当执行的方法的参数是指定类型时生效。
@target：当代理的目标对象上拥有指定的注解时生效。
@args：当执行的方法参数类型上拥有指定的注解时生效。
@within：与@target类似，看官方文档和网上的说法都是@within只需要目标对象的类或者父类上有指定的注解，则@within会生效，而@target则是必须是目标对象的类上有指定的注解。而根据笔者的测试这两者都是只要目标类或父类上有指定的注解即可。
@annotation：当执行的方法上拥有指定的注解时生效。
bean：当调用的方法是指定的bean的方法时生效。
```

#### execution表达式
```

```