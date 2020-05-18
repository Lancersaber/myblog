---
title: Springboot使用redis做缓存
categories: spring
tags: redis缓存
---
# 使用redis做Springboot的缓存
---
在程序中可以使用缓存的技术来节省对数据库的开销。Spring Boot对缓存提供了很好的支持，我们几乎不用做过多的配置即可使用各种缓存实现。这里主要介绍平日里个人接触较多的Ehcache和Redis缓存实现。

## 准备工作
可根据Spring-Boot中使用Mybatis.html搭建一个Spring Boot项目，然后yml中配置日志输出级别以观察SQL的执行情况：
```
logging:
  level:
    com:
      springboot:
        mapper: debug
```
其中com.spring.mapper为MyBatis的Mapper接口路径。
然后编写如下测试方法：
```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = Application.class)
public class ApplicationTest {

    @Autowired
    private StudentService studentService;
    
    @Test
    public void test() throws Exception {
        Student student1 = this.studentService.queryStudentBySno("001");
        System.out.println("学号" + student1.getSno() + "的学生姓名为：" + student1.getName());
        
        Student student2 = this.studentService.queryStudentBySno("001");
        System.out.println("学号" + student2.getSno() + "的学生姓名为：" + student2.getName());
    }
}
```
运行结果
```
2017-11-17 16:34:26.535 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : ==>  Preparing: select * from student where sno=? 
2017-11-17 16:34:26.688 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : ==> Parameters: 001(String)
2017-11-17 16:34:26.716 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : <==      Total: 1
学号001的学生姓名为：KangKang
2017-11-17 16:34:26.720 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : ==>  Preparing: select * from student where sno=? 
2017-11-17 16:34:26.720 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : ==> Parameters: 001(String)
2017-11-17 16:34:26.721 DEBUG 9932 --- [main] c.s.m.StudentMapper.queryStudentBySno    : <==      Total: 1
学号001的学生姓名为：KangKang
```
可发现第二个查询虽然和第一个查询完全一样，但其还是对数据库进行了查询。接下来引入缓存来改善这个结果。

## 使用缓存
要开启Spring Boot的缓存功能，需要在pom中引入spring-boot-starter-cache：
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

接着在Spring Boot入口类中加入@EnableCaching注解开启缓存功能：
```
@SpringBootApplication
@EnableCaching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class,args);
    }
}
```

在StudentService接口中加入缓存注解：
```
@CacheConfig(cacheNames = "student")
@Repository
public interface StudentService {
    @CachePut(key = "#p0.sno")
    Student update(Student student);
    
    @CacheEvict(key = "#p0", allEntries = true)
    void deleteStudentBySno(String sno);
    
    @Cacheable(key = "#p0")
    Student queryStudentBySno(String sno);
}
```
我们在StudentService接口中加入了@CacheConfig注解，queryStudentBySno方法使用了注解@Cacheable(key="#p0")，即将id作为redis中的key值。当我们更新数据的时候，应该使用@CachePut(key="#p0.sno")进行缓存数据的更新，否则将查询到脏数据，因为该注解保存的是方法的返回值，所以这里应该返回Student。

在Spring Boot中可使用的缓存注解有
### 缓存注解
* @CacheConfig：主要用于配置该类中会用到的一些共用的缓存配置。在这里@CacheConfig(cacheNames = "student")：配置了该数据访问对象中返回的内容将存储于名为student的缓存对象中，我们也可以不使用该注解，直接通过@Cacheable自己配置缓存集的名字来定义；

* @Cacheable：配置了queryStudentBySno函数的返回值将被加入缓存。同时在查询时，会先从缓存中获取，若不存在才再发起对数据库的访问。该注解主要有下面几个参数：
* value、cacheNames：两个等同的参数（cacheNames为Spring 4新增，作为value的别名），用于指定缓存存储的集合名。由于Spring 4中新增了@CacheConfig，因此在Spring 3中原本必须有的value属性，也成为非必需项了；
* key：缓存对象存储在Map集合中的key值，非必需，缺省按照函数的所有参数组合作为key值，若自己配置需使用SpEL表达式，比如：@Cacheable(key = "#p0")：使用函数第一个参数作为缓存的key值，
* condition：缓存对象的条件，非必需，也需使用SpEL表达式，只有满足表达式条件的内容才会被缓存，比如：@Cacheable(key = "#p0", condition = "#p0.length() < 3")，表示只有当第一个参数的长度小于3的时候才会被缓存；
* unless：另外一个缓存条件参数，非必需，需使用SpEL表达式。它不同于condition参数的地方在于它的判断时机，该条件是在函数被调用之后才做判断的，所以它可以通过对result进行判断；
* cacheManager：用于指定使用哪个缓存管理器，非必需。只有当有多个时才需要使用；
* cacheResolver：用于指定使用那个缓存解析器，非必需。需通过org.springframework.cache.interceptor.CacheResolver接口来实现自己的缓存解析器，并用该参数指定；
* @CachePut：配置于函数上，能够根据参数定义条件来进行缓存，其缓存的是方法的返回值，它与@Cacheable不同的是，它每次都会真实调用函数，所以主要用于数据新增和修改操作上。它的参数与@Cacheable类似，具体功能可参考上面对@Cacheable参数的解析；
* @CacheEvict：配置于函数上，通常用在删除方法上，用来从缓存中移除相应数据。除了同@Cacheable一样的参数之外，它还有下面两个参数：
	+ allEntries：非必需，默认为false。当为true时，会移除所有数据；
	+ beforeInvocation：非必需，默认为false，会在调用方法之后移除数据。当为true时，会在调用方法之前移除数据。

## 缓存实现
要使用上Spring Boot的缓存功能，还需要提供一个缓存的具体实现。Spring Boot根据下面的顺序去侦测缓存实现：
* Generic

* JCache (JSR-107)

* EhCache 2.x

* Hazelcast
* Infinispan

* Redis

* Guava

* Simple
除了按顺序侦测外，我们也可以通过配置属性spring.cache.type来强制指定。
接下来主要介绍基于Redis和Ehcache的缓存实现。

## Redis
1、安装redis
2、准备工作做完后，接下来开始在Spring Boot项目里引入Redis：
```
<!-- spring-boot redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

在application.yml中配置Redis：
```
spring:
  redis:
    # Redis数据库索引（默认为0）
    database: 0
    # Redis服务器地址
    host: localhost
    # Redis服务器连接端口
    port: 6379
    pool:
      # 连接池最大连接数（使用负值表示没有限制）
      max-active: 8
      # 连接池最大阻塞等待时间（使用负值表示没有限制）
      max-wait: -1
      # 连接池中的最大空闲连接
      max-idle: 8
      # 连接池中的最小空闲连接
      min-idle: 0
    # 连接超时时间（毫秒）
    timeout: 0
```
更多关于Spring Boot Redis配置可参考：https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html# REDIS

### 缓存的原理
* CacheManager===>(创建）Cache缓存组件 ---用于实际地存储数据
1）引入Redis starter以后，容器中保存的是RedisCacheManager
2）RedisCacheManager 帮我们创建RedisCache来作为缓存组件，RedisCache通过操作Redis来缓存数据

