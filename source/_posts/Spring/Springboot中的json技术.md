---
title: Springboot中的json技术
time: 2019-10-31
categories: spring
---
# Springboot中的json技术
---
平日里在项目中处理JSON一般用的都是阿里巴巴的Fastjson，后来发现使用Spring Boot内置的Jackson来完成JSON的序列化和反序列化操作也挺方便。Jackson不但可以完成简单的序列化和反序列化操作，也能实现复杂的个性化的序列化和反序列化操作。

## 自定义ObjectMapper
我们都知道，在Spring中使用@ResponseBody注解可以将方法返回的对象序列化成JSON，比如：
```
@RequestMapping("getuser")
@ResponseBody
public User getUser() {
    User user = new User();
    user.setUserName("mrbird");
    user.setBirthday(new Date());
    return user;
}
```

User类：
```
public class User implements Serializable {
    private static final long serialVersionUID = 6222176558369919436L;
    
    private String userName;
    private int age;
    private String password;
    private Date birthday;
    ...
}
```
访问getuser页面输出：
```
{"userName":"mrbird","age":0,"password":null,"birthday":1522634892365}
```

可看到时间默认以时间戳的形式输出，如果想要改变这个默认行为，我们可以自定义一个ObjectMapper来替代：
```
import java.text.SimpleDateFormat;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.fasterxml.jackson.databind.ObjectMapper;

@Configuration
public class JacksonConfig {
    @Bean
    public ObjectMapper getObjectMapper(){
        ObjectMapper mapper = new ObjectMapper();
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        return mapper;
    }
}
```
上面配置获取了ObjectMapper对象，并且设置了时间格式。再次访问getuser，页面输出：
```
{"userName":"mrbird","age":0,"password":null,"birthday":"2018-04-02 10:14:24"}
```

## 序列化
Jackson通过使用mapper的writeValueAsString方法将Java对象序列化为JSON格式字符串：
```
@Autowired
ObjectMapper mapper;

@RequestMapping("serialization")
@ResponseBody
public String serialization() {
    try {
        User user = new User();
        user.setUserName("mrbird");
        user.setBirthday(new Date());
        String str = mapper.writeValueAsString(user);
        return str;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

## 反序列化
使用@ResponseBody注解可以使对象序列化为JSON格式字符串，除此之外，Jackson也提供了反序列化方法。

### 树遍历
当采用树遍历的方式时，JSON被读入到JsonNode对象中，可以像操作XML DOM那样读取JSON。比如：
```
@Autowired
ObjectMapper mapper;

@RequestMapping("readjsonstring")
@ResponseBody
public String readJsonString() {
    try {
        String json = "{\"name\":\"mrbird\",\"age\":26}";
        JsonNode node = this.mapper.readTree(json);
        String name = node.get("name").asText();
        int age = node.get("age").asInt();
        return name + " " + age;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```
readTree方法可以接受一个字符串或者字节数组、文件、InputStream等， 返回JsonNode作为根节点，你可以像操作XML DOM那样操作遍历JsonNode以获取数据。

解析多级JSON例子：
```
String json = "{\"name\":\"mrbird\",\"hobby\":{\"first\":\"sleep\",\"second\":\"eat\"}}";;
JsonNode node = this.mapper.readTree(json);
JsonNode hobby = node.get("hobby");
String first = hobby.get("first").asText();
```

### 绑定对象
我们也可以将Java对象和JSON数据进行绑定，如下所示：
```
@Autowired
ObjectMapper mapper;

@RequestMapping("readjsonasobject")
@ResponseBody
public String readJsonAsObject() {
    try {
        String json = "{\"name\":\"mrbird\",\"age\":26}";
        User user = mapper.readValue(json, User.class);
        String name = user.getUserName();
        int age = user.getAge();
        return name + " " + age;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

## Jackson注解
Jackson包含了一些实用的注解：

### @JsonProperty
@JsonProperty，作用在属性上，用来为JSON Key指定一个别名。
```
@JsonProperty("bth")
private Date birthday;
```
再次访问getuser页面输出：
```
{"userName":"mrbird","age":0,"password":null,"bth":"2018-04-02 10:38:37"}
```
key birthday已经被替换为了bth。

### @Jsonlgnore
@Jsonlgnore，作用在属性上，用来忽略此属性。
```
@JsonIgnore
private String password;
```

再次访问getuser页面输出：
```
{"userName":"mrbird","age":0,"bth":"2018-04-02 10:40:45"}
```
password属性已被忽略。

### @JsonIgnoreProperties
@JsonIgnoreProperties，忽略一组属性，作用于类上，比如JsonIgnoreProperties({ "password", "age" })。
```
@JsonIgnoreProperties({ "password", "age" })
public class User implements Serializable {
    ...
}
```

再次访问getuser页面输出：
```
{"userName":"mrbird","bth":"2018-04-02 10:45:34"}
```

### @JsonFormat
@JsonFormat，用于日期格式化，如：
```
@JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
private Date birthday;
```

### @JsonNaming
@JsonNaming，用于指定一个命名策略，作用于类或者属性上。Jackson自带了多种命名策略，你可以实现自己的命名策略，比如输出的key 由Java命名方式转为下面线命名方法 —— userName转化为user-name。
```
@JsonNaming(PropertyNamingStrategy.LowerCaseWithUnderscoresStrategy.class)
public class User implements Serializable {
    ...
}
```
再次访问getuser页面输出：
```
{"user_name":"mrbird","bth":"2018-04-02 10:52:12"}
```

### @JsonSerialize
@JsonSerialize，指定一个实现类来自定义序列化。类必须实现JsonSerializer接口，代码如下：
```
import java.io.IOException;

import com.example.pojo.User;
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;

public class UserSerializer extends JsonSerializer<User> {

    @Override
    public void serialize(User user, JsonGenerator generator, SerializerProvider provider)
            throws IOException, JsonProcessingException {
        generator.writeStartObject();
        generator.writeStringField("user-name", user.getUserName());
        generator.writeEndObject();
    }
}
```

上面的代码中我们仅仅序列化userName属性，且输出的key是user-name。 使用注解@JsonSerialize来指定User对象的序列化方式：
```
@JsonSerialize(using = UserSerializer.class)
public class User implements Serializable {
    ...
}
```

再次访问getuser页面输出：
```
{"user-name":"mrbird"}
```

### @JsonDeserialize
@JsonDeserialize，用户自定义反序列化，同@JsonSerialize ，类需要实现JsonDeserializer接口。
```
import java.io.IOException;

import com.example.pojo.User;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonNode;

public class UserDeserializer extends JsonDeserializer<User> {

    @Override
    public User deserialize(JsonParser parser, DeserializationContext context)
            throws IOException, JsonProcessingException {
        JsonNode node = parser.getCodec().readTree(parser);
        String userName = node.get("user-name").asText();
        User user = new User();
        user.setUserName(userName);
        return user;
    }
}
```

使用注解@JsonDeserialize来指定User对象的反序列化方式：
```
@JsonDeserialize (using = UserDeserializer.class)
public class User implements Serializable {
    ...
}
```
测试
```
@Autowired
ObjectMapper mapper;

@RequestMapping("readjsonasobject")
@ResponseBody
public String readJsonAsObject() {
    try {
        String json = "{\"user-name\":\"mrbird\"}";
        User user = mapper.readValue(json, User.class);
        String name = user.getUserName();
        return name;
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

访问readjsonasobject，页面输出：
```
mrbird
```

### @JsonView
@JsonView，作用在类或者属性上，用来定义一个序列化组。 比如对于User对象，某些情况下只返回userName属性就行，而某些情况下需要返回全部属性。 因此User对象可以定义成如下：
```
public class User implements Serializable {
    private static final long serialVersionUID = 6222176558369919436L;
    
    public interface UserNameView {};
    public interface AllUserFieldView extends UserNameView {};
    
    @JsonView(UserNameView.class)
    private String userName;
    
    @JsonView(AllUserFieldView.class)
    private int age;
    
    @JsonView(AllUserFieldView.class)
    private String password;
    
    @JsonView(AllUserFieldView.class)
    private Date birthday;
    ...	
}
```
User定义了两个接口类，一个为userNameView，另外一个为AllUserFieldView继承了userNameView接口。这两个接口代表了两个序列化组的名称。属性userName使用了@JsonView(UserNameView.class)，而剩下属性使用了@JsonView(AllUserFieldView.class)。

Spring中Controller方法允许使用@JsonView指定一个组名，被序列化的对象只有在这个组的属性才会被序列化，代码如下：
```
@JsonView(User.UserNameView.class)
@RequestMapping("getuser")
@ResponseBody
public User getUser() {
    User user = new User();
    user.setUserName("mrbird");
    user.setAge(26);
    user.setPassword("123456");
    user.setBirthday(new Date());
    return user;
}
```
访问getuser页面输出：
```
{"userName":"mrbird"}
```

如果将@JsonView(User.UserNameView.class)替换为@JsonView(User.AllUserFieldView.class)，输出：
```
{"userName":"mrbird","age":26,"password":"123456","birthday":"2018-04-02 11:24:00"}
```

因为接口AllUserFieldView继承了接口UserNameView所以userName也会被输出。