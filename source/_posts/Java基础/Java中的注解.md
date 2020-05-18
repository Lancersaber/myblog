---
title: Java中的注解
date: 2019-10-24 23:15:15
categories: Java
tags: Java
---

# Java中的基本注解
---
&emsp;&emsp;注解(也称为元数据)为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些数据。

## Rentention注解类
注解的生命周期：Java源文件—》class文件—》内存中的字节码。编译或者运行时，都有可能会取消注解。Rentention的3种取值意味让注解保留到哪个阶段，RententionPolicy.SOURCE、RententionPolicy.CLASS(默认值)、RententionPolicy.RUNTIME。Rentention的值是枚举RententionPolicy的值，只有3个：SOURCE、CLASS、RUNTIME。
* source：源代码中，注解将被编译器丢弃
* class：类文件中，注解在class文件中可用
* runtime:运行时，VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息

## Target注解类
性质和Rentention一样，都是注解类的属性，表示注解类应该在什么位置，对那一块的数据有效。例如，@Target(ElementType.METHOD)
Target内部的值使用枚举ElementType表示，表示的主要位置有：注解、构造方法、属性、局部变量、函数、包、参数和类(默认值)。多个位置使用数组，例如，@Target({ElementType.METHOD,ElementType.TYPE})。

## 使用注解的例子
下面是一个简单的注解，我们可以用它来跟踪一个项目中的用例。如果一个方法或一组方法实现了某个用例的需求，那么程序员可以为此方法加上该注解。
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    public int id();
    public String description() default "no description";
}
```
注意：id和description类似方法定义。编译器会对id和description进行类型检查。description元素有一个default值，如果在注解某个方法时没有给出description的值，则此注解的处理器就会使用元素的默认值。

使用注解的类
```
public class PasswordUtils {
    
    @UseCase(id=47,description = "Passwords must contain ar least one numeric")
    public boolean validatePassword(String password){
        return (password.matches("\\w*\\d\\w*"));
    }
    
    @UseCase(id=48)
    public String encryptPassword(String password){
        return new StringBuilder(password).reverse().toString();
    }
    
    @UseCase(id=49,description = "New passwords can't equal previously used ones")
    public boolean checkForNewPassword(List<String> prevPasswords,String password){
        return !prevPasswords.contains(password);
    }
}
```


### 编写注解处理器
```
public class UseCaseTracker {

    public static void trackUseCases(List<Integer> useCases,Class<?> cl){
        for (Method m: cl.getDeclaredMethods()){
            UseCase annotation = m.getAnnotation(UseCase.class);

            if (annotation!=null){
                System.out.println("Found Use Case:"+annotation.id()+" "+annotation.description());

                useCases.remove(new Integer(annotation.id()));
            }
        }

        for (int i:useCases){
            System.out.println("Warning: Missing use case-"+i);
        }
    }

    public static void main(String[] args){
        List<Integer> useCases=new ArrayList<>();
        Collections.addAll(useCases,47,48,49,50);
        trackUseCases(useCases,PasswordUtils.class);
    }
}
```
这里用到了两个反射的方法：getDeclaredMethods()和getAnnotation(),它们都属于AnnotatedElement接口(class,Method与Field等类都实现了该接口)。getAnnoation()方法返回指定类型的注解对象，在这里就是UseCase。如果被注解的方法上没有该类型的注解，则返回null值。

### 默认值限制
&emsp;&emsp;编译器对元素的默认值有些过分挑剔。首先，元素不能有不确定的值。也就是说，元素必须具有默认值，要么在使用注解时提供元素的值。
&emsp;&emsp;其次，对于非基本类型的元素，无论是在源代码中声明时，或是在注解接口定义默认值时，都不能以null为其值。为了绕开这个约束，我们可以定义一些特殊的值，例如空字符串或负数，以此表示某个元素不存在。
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SimulatingNull {
    public int id() default -1;
    public String descption() default "";
}
```