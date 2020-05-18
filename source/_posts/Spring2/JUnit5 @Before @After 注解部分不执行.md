---
title: JUnit5 @Before @After 注解部分不执行
time: 2019-11-12
categories: junit
tags: junit
---

# JUnit5 @Before @After 注解部分不执行
---
单元测试使用@Before 初始化事务连接, @After 关闭事务连接.
```
	@Before
	public void init() {
		System.out.println("init");
	}
	
	@After
	public void destroy(){
		System.err.println("destroy");
	}
	
	
	@Test
	public void test() {
		System.out.println("test");
	} 
```

执行test() 方法时 输出只有test, 没有预想中的 init 以及 destory.

Google了一波, 并没有找到有效的解决方案, 于是转向JUnit官网.
在官网中发现了这样一段话
![20180524185539562.png](https://i.loli.net/2019/11/13/ihoH1WM4jGZup2S.png)

原来是 @Before 和@After 被 @BeforeEach 和@AfterEach给替代了. 还有一些其他的的注解也被替代了.