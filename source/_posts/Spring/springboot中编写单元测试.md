---
title: Springboot中编写单元测试
time: 2019-10-31
categories: spring
tags: spring
---
# Springboot中编写单元测试
---
编写单元测试可以帮助开发人员编写高质量的代码，提升代码质量，减少Bug，便于重构。Spring Boot提供了一些实用程序和注解，用来帮助我们测试应用程序，在Spring Boot中开启单元测试只需引入spring-boot-starter-test即可，其包含了一些主流的测试库。本文主要介绍基于 Service和Controller的单元测试。

引入spring-boot-starter-test：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

## 知识准备

### Junit4注解
JUnit4中包含了几个比较重要的注解：@BeforeClass、@AfterClass、@Before、@After和@Test。其中， @BeforeClass和@AfterClass在每个类加载的开始和结束时运行，必须为静态方法；而@Before和@After则在每个测试方法开始之前和结束之后运行。见如下例子：
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestApplicationTests {

    @BeforeClass
    public static void beforeClassTest() {
        System.out.println("before class test");
    }
    
    @Before
    public void beforeTest() {
        System.out.println("before test");
    }
    
    @Test
    public void Test1() {
        System.out.println("test 1+1=2");
        Assert.assertEquals(2, 1 + 1);
    }
    
    @Test
    public void Test2() {
        System.out.println("test 2+2=4");
        Assert.assertEquals(4, 2 + 2);
    }
    
    @After
    public void afterTest() {
        System.out.println("after test");
    }
    
    @AfterClass
    public static void afterClassTest() {
        System.out.println("after class test");
    }
}
```

运行输出如下：
```
...
before class test
before test
test 1+1=2
after test
before test
test 2+2=4
after test
after class test
...
```

从上面的输出可以看出各个注解的运行时机。

### Assert
上面代码中，我们使用了Assert类提供的assert口方法，下面列出了一些常用的assert方法：
* assertEquals("message",A,B)，判断A对象和B对象是否相等，这个判断在比较两个对象时调用了equals()方法。

* assertSame("message",A,B)，判断A对象与B对象是否相同，使用的是==操作符。

* assertTrue("message",A)，判断A条件是否为真。

* assertFalse("message",A)，判断A条件是否不为真。

* assertNotNull("message",A)，判断A对象是否不为null。

* assertArrayEquals("message",A,B)，判断A数组与B数组是否相等。

### MockMvc
下文中，对Controller的测试需要用到MockMvc技术。MockMvc，从字面上来看指的是模拟的MVC，即其可以模拟一个MVC环境，向Controller发送请求然后得到响应。

在单元测试中，使用MockMvc前需要进行初始化，如下所示：
```
private MockMvc mockMvc;

@Autowired
private WebApplicationContext wac;

@Before
public void setupMockMvc(){
    mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
}
```

