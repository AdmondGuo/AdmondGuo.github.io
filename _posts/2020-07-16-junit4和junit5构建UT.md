---
layout: post category: unit test
---

## 1.依赖

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>${spring-boot.version}</version>
    <scope>test</scope>
</dependency>

<dependency>
<groupId>org.junit.jupiter</groupId>
<artifactId>junit-jupiter</artifactId>
<version>${junit.version}</version>
<exclusions>
    <exclusion>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-commons</artifactId>
    </exclusion>
    <exclusion>
        <groupId>org.junit.platform</groupId>
        <artifactId>junit-platform-engine</artifactId>
    </exclusion>
</exclusions>
<scope>test</scope>
</dependency>

<dependency>
<groupId>org.junit.vintage</groupId>
<artifactId>junit-vintage-engine</artifactId>
<version>${junit.version}</version>
<scope>test</scope>
</dependency>

<dependency>
<groupId>org.mockito</groupId>
<artifactId>mockito-core</artifactId>
<version>${mock.version}</version>
<scope>test</scope>
</dependency>
```

其中：

```xml

<spring-boot.version>2.0.8.RELEASE</spring-boot.version>
<java.version>1.8</java.version>
<junit.version>5.7.2</junit.version>
<mock.version>2.21.0</mock.version>
```

## 2.注入service

junit环境是4还是5，取决于@Test注解引自哪里

- 引入自import org.junit.jupiter.api.Test;就是junit5
- 引入自org.junit.Test;就是junit4

而junit4和junit5的spring上下文配置方法有所不同：

### junit4：

class注解：

@SpringBootTest

@ExtendWith(SpringExtension.class)

静态方法注解：

@BeforeAll

-------
### junit5：

class注解：

@SpringBootTest

@RunWith(SpringRunner.class)

静态方法注解：

@BeforeClass

------
- 其中都有@SpringBootTest，用来提供Spring-test环境
- @BeforeClass/@BeforeAll中可以配置System.properties, 也就是VM Options
- 需要配置spring.config.name/spring.config.location，否则spring无法加载配置文件（应该有其他加载方式，比如通过注解扫描）

### mock被注入的service内部使用的service
```java
@ExtendWith(SpringExtension.class)
@SpringBootTest
public class CServiceTest {
    
    @Mock
    EngineService engineService;

//    @Autowired
    @InjectMocks
    CatalogService catalogService;

    @Test
    public void testListDeltaDatabases(){
        
        when(engineService.runScript(isA(EngineService.RunScriptParams.class))).thenReturn(expectedResp);
        //  todo
    }
}
```
```java
public class CatalogService{
    @Autowired
    EngineService engineService;
    
    public String runScript(RunScriptParams params){
        // todo
    }
}
```
这种情况下，要用@InjectMocks替代使用的@Autowired

[reference](https://stackoverflow.com/questions/19003931/mock-services-inside-another-spring-service-with-mockito

---------

### spy与mock区别
- mock出来的对象如果没有初始化（mock某个方法或者注入Bean），默认其中的对象都是null，并且没有可用方法
- spy中有真实的方法，除了被mock的方法外，都是真实的


### when/thenReturn与doReturn/when区别
- when/thenReturn会调用真实的方法，只是会在最后的返回处做处理，将返回值变为期待值
- doReturn/when不会调用真实的方法，不会进入方法本身，直接返回期待值

-----------
### 不冗余的设置env
test的env设置希望与运行环境一致。
通过System.setProperties()方法设置。写在test的@BeforeAll()中，在进入测试方法之前进行设置。

但是每个测试类都进行一遍设置，过于冗余。

将设置的代码写在一个Base类中，其他所有test类继承Base类，可以解决这个问题。

-----------
### 将mock对象注入另一个mock对象
如图添加注解：
![111](/public/img/junit-1.png)

这样使用：
![2222](/public/img/junit-2.png)

---------
### mockMvc @PathVariable 匹配
｜ have to use string splicing instead of pathVariable like "/api/job/{jobId}", cause controller using @RestController

@RestController 和 @Controller 有所不同：
- Get请求，相差不大 
- Post请求，@RestController 中包含 @ResponseBody
- - headers中必须包含Content-Type："application/json;charset=UTF-8"
- - mockMvc有PathVariable时需要使用拼接字符串的方式，不能使用变量方式（原因未知）    

```java
        MockHttpServletResponse response = mockMvc.perform(MockMvcRequestBuilders.post("api/job/{jobId}/cancel", testId)
                .requestAttr("username", username)
                .header("Accept", "application/json")
                .header("Content-Type", "application/json;charset=UTF-8")
                )
```
结果
```text

MockHttpServletRequest:
      HTTP Method = POST
      Request URI = api/job/2147483647/cancel
       Parameters = {}
          Headers = {Accept=[application/json], Content-Type=[application/json;charset=UTF-8]}
             Body = null
    Session Attrs = {}

Handler:
             Type = null

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 404
    Error message = null
          Headers = {}
     Content type = null
             Body = 
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

java.lang.AssertionError: Status expected:<200> but was:<404>
Expected :200
Actual   :404
```
拼接方式：
```java
        MockHttpServletResponse response = mockMvc.perform(MockMvcRequestBuilders.post("/api/job/"+testId+"/cancel")
                .requestAttr("username", username)
                .header("Accept", "application/json")
                .header("Content-Type", "application/json;charset=UTF-8")
                )
```
结果：
![junit-3](/public/img/junit-3.png)

