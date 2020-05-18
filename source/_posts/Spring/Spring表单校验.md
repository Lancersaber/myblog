---
title: Spring表单校验
time: 2019-11-08
categories: spring
tags: spring
---

# Spring表单校验
---
从Spring 3.0开始，Spring对Java校验API（Java Validation API，又称JSR-303）提供了支持。在Spring MVC中要使用Java校验API的话，并不需要什么额外的配置。只要保证在类路径下包含这个Java API的实现即可，比如Hibernate Validator。

引入hibernate-validator：
```
<!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-validator -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

## 校验注解
所有的注解都位于javax.validation.constraints和org.hibernate.validator.constraints包中。下表列出了这些校验注解。

注解	|描述
---|----
@Null|	限制只能为null
@NotNull|	限制必须不为null
@AssertFalse|	限制必须为false
@AssertTrue	|限制必须为true
@DecimalMax(value)|	限制必须为一个不大于指定值的数字
@DecimalMin(value)|	限制必须为一个不小于指定值的数字
@Digits(integer,fraction)|	限制必须为一个小数，且整数部分的位数不能超过integer， 小数部分的位数不能超过fraction
@Future	|限制必须是一个将来的日期
@Past	|限制必须是一个过去的日期
@Max(value)|	限制必须为一个不大于指定值的数字
@Min(value)	|限制必须为一个不小于指定值的数字
@Past|	限制必须是一个过去的日期
@Pattern(value)|	限制必须符合指定的正则表达式
@Size(max,min)	|限制字符长度必须在min到max之间
@SafeHtml	|字符串是安全的html
@URL	|字符串是合法的URL
@NotBlank|	字符串必须有字符
@NotEmpty|	字符串不为NULL，集合有字符
@AssertFalse|	必须是false
@AssertTrue	|必须是true

## 使用校验注解配置实体类
新建一个表单实体类，并加上注解：
```
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
import org.hibernate.validator.constraints.Email;
 
public class Form {
    @NotNull
    @Size(min = 5, max = 16, message = "{name.msg}")
    private String name;

    @NotNull()
    @Email(message = "{email.msg}")
    private String email;

    @NotNull
    @Size(min = 5, max = 16, message = "{password.msg}")
    private String password;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

## 自定义校验规则
上面的注解都是较为简单的注解，实际编程中校验的规则可能五花八门。当自带的这些注解无法满足我们的需求时，我们也可以自定义校验注解。下面是一个自定义校验注解的基本格式：
```
import javax.validation.Constraint;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target

@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = MyConstraintValidator.class)
public @interface MyConstraint {

    String message();

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};
}
```

其中@Constraint注解表明这个注解是用于规则校验的，validatedBy属性表明用什么去校验，这里我们指定的类为MyConstraintValidator。注解还包含了三个书属性，属性message指定当校验不通过的时候提示什么信息。

接下来编写MyConstraintValidator，代码如下所示：
```
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;

public class MyConstraintValidator implements ConstraintValidator<MyConstraint, Object> {

    @Override
    public void initialize(MyConstraint myConstraint) {
        System.out.println("my validator init");
    }

    @Override
    public boolean isValid(Object o, ConstraintValidatorContext constraintValidatorContext) {
        System.out.println(o);
        return false;
    }
}
```

MyConstraintValidator实现了ConstraintValidator接口，该接口必须指定两个泛型，第一个泛型指的是上面定义的注解类型，第二个泛型表示校验对象的类型。MyConstraintValidator实现了ConstraintValidator接口的initialize方法和isValid方法。

initialize方法用于该校验初始化的时候进行一些操作；isValid方法用于编写校验逻辑，第一个参数为需要校验的值，第二个参数为校验上下文。

## 编写controller
Regester控制器如下所示：
```
import javax.validation.Valid;
import mrbird.mvc.entity.Form;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class LeanoteController {
    @RequestMapping(value = "/registerindex", method = RequestMethod.GET)
    public String register(Model model) {
        //对应<sf:form/>标签的commandName属性值
        model.addAttribute(new Form());
        return "register";
    }

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    //form参数添加了@Valid注解，这会告知Spring，需要确保这个对象满足校验限制。
    //Errors参数要紧跟在带有@Valid注解的参数后面
    public String submit(@Valid Form form, BindingResult result) {
        if (result.hasErrors()) {
            // 输出校验失败信息
            result.getAllErrors().stream().forEach(e -> {
                        FieldError fieldError = (FieldError) e;
                        System.out.println(((FieldError) e).getField() + " " + e.getDefaultMessage());
                    }
            );
            //出错时回到注册页面，这里不能够用重定向，不然看不到错误提示信息
            return "register";
        }
        //校验通过，重定向到/success/{name}
        return "redirect:/success/" + form.getName();
    }

    @RequestMapping(value = "/success/{name}", method = RequestMethod.GET)
    public String success(@PathVariable String name, Model model) {
        model.addAttribute(name);
        return "success";
    }
}
```