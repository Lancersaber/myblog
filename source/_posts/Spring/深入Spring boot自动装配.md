---
title: 深入Spring boot自动装配
time: 2019-11-04
categories: spring
tags: spring
---

# 深入Spring boot自动装配
---

## 模式注解
Stereotype Annotation俗称为模式注解，Spring中常见的模式注解有@Service，@Repository，@Controller等，它们都“派生”自@Component注解。我们都知道，凡是被@Component标注的类都会被Spring扫描并纳入到IOC容器中，那么由@Component派生的注解所标注的类也会被扫描到IOC容器中。下面我们主要来通过自定义模式注解来了解@Component的“派生性”和“层次性”。

### @Component “派生性”
新建一个Spring Boot工程，Spring Boot版本为2.1.0.RELEASE，artifactId为autoconfig，并引入spring-boot-starter-web依赖
在com.example.demo下新建annotation包，然后创建一个FirstLevelService注解：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Service
public @interface FirstLevelService {
    String value() default "";
}
```
这个注解定义由@Service标注，查看@Service的源码会发现其被@Component注解标注，所以它们的层次关系为:
```
└─@Component
   └─@Service
      └─@FirstLevelService
```

即@FirstLevelService为@Component派生出来的模式注解，我们来测试一下被它标注的类是否能够被扫描到IOC容器中：

在com.example.demo下新建service包，然后创建一个TestService类：
```
@FirstLevelService
public class TestService {
}
```

在com.example.demo下新建bootstrap包，然后创建一个ServiceBootStrap类，用于测试注册TestService并从IOC容器中获取它：
```
@ComponentScan("com.example.demo.service")
public class ServiceBootstrap {

    public static void main(String[] args) {
        ConfigurableApplicationContext context = new SpringApplicationBuilder(ServiceBootstrap.class)
                .web(WebApplicationType.NONE)
                .run(args);
        TestService testService = context.getBean("testService", TestService.class);
        System.out.println("TestService Bean: " + testService);
        context.close();
    }
}
```

运行该类的main方法，控制台输出如下：
```
TestService Bean: com.example.ceshi.service.TestService@5dd1c9f2
```

### @Component “层次性”

我们在com.example.demo.annotation路径下再创建一个SecondLevelService注解定义，该注解由上面的@FirstLevelService标注：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@FirstLevelService
public @interface SecondLevelService {
    String value() default "";
}
```
这时层次关系为
```
└─@Component
   └─@Service
      └─@FirstLevelService
            └─@SecondLevelService
```

我们将TestService上的注解换成@SecondLevelService，然后再次运行ServiceBootStrap的main方法，输出如下：

```
TestService Bean: com.example.ceshi.service.TestService@134d26af
```
可见结果也是成功的。

	这里有一点需要注意的是：@Component注解只包含一个value属性定义，所以其“派生”的注解也只能包含一个vlaue属性定义。

## @Enable模块驱动
@Enable模块驱动在Spring Framework 3.1后开始支持。这里的模块通俗的来说就是一些为了实现某个功能的组件的集合。通过@Enable模块驱动，我们可以开启相应的模块功能。

@Enable模块驱动可以分为“注解驱动”和“接口编程”两种实现方式，下面逐一进行演示：

### 注解驱动
Spring中，基于注解驱动的示例可以查看@EnableWebMvc源码：
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({DelegatingWebMvcConfiguration.class})
public @interface EnableWebMvc {
}
```

该注解通过@Import导入一个配置类DelegatingWebMvcConfiguration：
![QQ截图20190220170120.png](https://i.loli.net/2019/11/04/upAFjtLqYCJhbeX.png)
该配置类又继承自WebMvcConfigurationSupport，里面定义了一些Bean的声明。
```
所以，基于注解驱动的@Enable模块驱动其实就是通过@Import来导入一个配置类，以此实现相应模块的组件注册，当这些组件注册到IOC容器中，这个模块对应的功能也就可以使用了。
```

我们来定义一个基于注解驱动的@Enable模块驱动。
在com.example.demo下新建configuration包，然后创建一个HelloWorldConfiguration配置类：
```
//@Configuration
public class HelloWorldConfiguration {

    @Bean
    public String hello() {
        return "hello world";
    }
}
```

这个配置类里定义了一个名为hello的Bean，内容为hello world。
在com.example.demo.annotation下创建一个EnableHelloWorld注解定义：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(HelloWorldConfiguration.class)
public @interface EnableHelloWorld {
}
```

运行该类的main方法，控制台输出如下：
```
hello Bean: hello world
```
说明我们自定义的基于注解驱动的@EnableHelloWorld是可行的。

注意：在HelloWorldConfiguration类不要加上@Configuration注解，否则@EnableHelloWorld注解不起作用

### 接口编程
除了使用上面这个方式外，我们还可以通过接口编程的方式来实现@Enable模块驱动。Spring中，基于接口编程方式的有@EnableCaching注解，查看其源码：
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({CachingConfigurationSelector.class})
public @interface EnableCaching {
    boolean proxyTargetClass() default false;

    AdviceMode mode() default AdviceMode.PROXY;

    int order() default 2147483647;
}
```

