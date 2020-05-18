---
title: Springboot从前台获取数据的方式汇总
time: 2019-11-05
categories: spring
tags: spring,springmvc
---

# Springboot从前台获取数据的方式汇总
---

## 1、自动注入
直接将参数写在controller方法中的形参中，注意前后端名称需要保持一致，示例：
```
后台函数
@RequestMapping("/submit")
@ResponseBody
public String getAccount(String name,String password){
    if (name!=null){
        System.out.println(name);
    }
    if (password!=null){
        System.out.println(password);
    }
    return "get";
}

请求的URL
http://localhost:8080/submit?name=1&password=haha

可以得到前台的数据
```

## 2、@PathVariable获取路径中的参数
可以通过@PathVariable获取路径上的参数，示例
```
后台代码
@RequestMapping("/submit/{id}")
    @ResponseBody
    public String getAccount(@PathVariable("id")Integer id){
        System.out.println(id);
        return "get";
    }

http://localhost:8080/submit/1
```

## @RequestParam绑定请求参数
前端页面,采用thymeleaf来传递参数
```
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>account</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <link rel="stylesheet" th:href="@{/css/style.css}" type="text/css">
</head>
<body>
<table>
    <tr>
        <th>no</th>
        <th>account</th>
        <th>name</th>
        <th>password</th>
        <th>accountType</th>
        <th>tel</th>
    </tr>
    <tr th:each="list,stat:${accountList}">
        <td th:text="${stat.count}"></td>
        <td th:text="${list.name}"></td>
        <td th:text="${list.address}"></td>
        <td th:text="${list.password}"></td>
        <td th:text="${list.role}"></td>
        <td th:text="${list.id}"></td>
    </tr>
    <form action="/submit" method="post">
        请输入姓名:<input type="text" id="name" name="name"><br/>
        请输入密码:<input type="text" id="password" name="password"><br/>
        请输入地址:<input type="text" id="address" name="address"><br/>
        请输入角色:<input type="text" id="role" name="role"><br/>
        请输入id:<input type="text" id="id" name="id"><br/>
        <input type="submit" value="提交"><br/>
    </form>
</table>
</body>
</html>
```

注意，@RequestParam("name")是根据标签里的name属性来取值的，不是根据id来取值
```
@RequestMapping("/submit")
@ResponseBody
public String getAccount(@RequestParam("name")String name){
    if (name!=null){
        System.out.println(name);
    }
    return "get";
}
```
如果前端没有传递属性名为name的值，那么将会报400错误，表示请求出错。


## 通过HttpServletRequest对象手动get , 获取的值均为String型
```
@RequestMapping("/submit")
@ResponseBody
public String getAccount(HttpServletRequest request){
    String name=request.getParameter("name");
    if (name!=null){
        System.out.println(name);
    }
    return "get";
}
```
![提交数据.PNG](https://i.loli.net/2019/11/05/nJ1HQN2kItaMZhv.png)

## 通过@ModelAttribute获取form中的参数
```
 @RequestMapping("/submit")
    @ResponseBody
    public String getAccount(@ModelAttribute Account account){
        if (account!=null){
            System.out.println(account);
        }
        return "get";
    }
```
account中的成员变量名称需要和前台form中的name值保持一致 , 且必须要有get/set 方法 , 否则无法注入
![获取数据2.PNG](https://i.loli.net/2019/11/05/8ngjPAfbWqVueaS.png)