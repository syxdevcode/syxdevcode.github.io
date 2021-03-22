---
title: Spring Boot常用注解
date: 2021-03-19 15:23:57
tags:
- Java
- IDEA
- Spring Boot
- Java注解
categories: 
- Spring Boot
---

使用注解的优势：
1.采用纯java代码，不在需要配置繁杂的xml文件
2.在配置中也可享受面向对象带来的好处
3.类型安全对重构可以提供良好的支持
4.减少复杂配置文件的同时亦能享受到springIoC容器提供的功能

## 启动注解@SpringBootApplication

&emsp;&emsp;`@SpringBootApplication` 是一个复合注解，包含了 `@SpringBootConfiguration`，`@EnableAutoConfiguration`，`@ComponentScan` 这三个注解。

### @SpringBootConfiguration注解，继承@Configuration注解，主要用于加载配置文件

&emsp;&emsp;`@SpringBootConfiguration` 和 `@Configuration` 二者功能一致，标注当前类是配置类， 并会将当前类内声明的一个或多个以 `@Bean` 注解标记的方法的实例纳入到 `Spring` 容器中，并且实例名就是方法名。

`@Configuration`：等同于 `Spring` 的XML配置文件；使用Java代码可以检查类型安全。

### @EnableAutoConfiguration注解，开启自动配置功能

&emsp;&emsp;`@EnableAutoConfiguration`可以帮助 `SpringBoot` 应用将所有符合条件的 `@Configuration` 配置都加载到当前 `SpringBoot` 创建并使用的IoC容器。借助于 `Spring` 框架原有的一个工具类：`SpringFactoriesLoader` 的支持，`@EnableAutoConfiguration` 可以智能的自动配置功效才得以大功告成

### @ComponentScan注解，主要用于组件扫描和自动装配

&emsp;&emsp;`@ComponentScan` 的功能其实就是自动扫描并加载符合条件的组件或bean定义，最终将这些bean定义加载到容器中。我们可以通过 `basePackages` 等属性指定 `@ComponentScan` 自动扫描的范围，如果不指定，则默认 `Spring` 框架实现从声明 `@ComponentScan` 所在类的 `package` 进行扫描，默认情况下是不指定的，所以 `SpringBoot` 的启动类最好放在 `root package`下。

## Controller 相关注解

### @Controller

用于定义控制器类，在spring项目中由控制器负责将用户发来的URL请求转发到对应的服务接口（service层），一般这个注解在类中，通常方法需要配合注解 `@RequestMapping`。

### @ResponseBody

`@ResponseBody`：表示该方法的返回结果直接写入 `HTTP response body` 中，一般在异步获取数据时使用，用于构建 `RESTful` 的 `api`。在使用 `@RequestMapping` 后，返回值通常解析为跳转路径，加上 `@Responsebody` 后返回结果不会被解析为跳转路径，而是直接写入 `HTTP response body` 中。比如异步获取json数据，加上 `@Responsebody` 后，会直接返回json数据。该注解一般会配合`@RequestMapping`一起使用。

### @RestController 复合注解

用于标注控制层组件(如struts中的action)，`@ResponseBody` 和 `@Controller` 的合集，`@RestController` 效果是将方法返回的对象直接在浏览器上展示成json格式。

### @RequestBody

通过 `HttpMessageConverter` 读取 `Request Body` 并反序列化为 Object 对象。

使用 `@RequestBody` 接收数据时，前端不能使用GET方式提交数据，而是用POST方式进行提交。

### @RequestMapping

`@RequestMapping` 提供路由信息，负责URL到Controller中的具体函数的映射，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。`@RequestMapping` 用在类上可以没有，但是用在方法上必须有。

### @GetMapping

将 `HTTP Get` 请求映射到特定处理程序的方法注解。

```java
@RequestMapping(value = "/go",method = RequestMethod.GET) 等价于：@GetMapping(value = "/go")
```

### @PostMapping

将 `HTTP Post` 请求映射到特定处理程序的方法注解。

```java
@RequestMapping(value = "/go",method = RequestMethod.POST) 等价于：@PostMapping(value = "/go")
```

## 请求参数值

### @PathVariable

`@PathVariable`：接收请求路径中占位符的值。

注意：`@RequestMapping` 与 `@PathVariable` 中的值必须保持一致。

```java
/**
 * 占位符映射
 * 语法：@RequestMapping(value=”user/{userId}/{userName}”)
 * 请求路径：http://localhost:8080/springboot/show/1/wang
 * @param ids
 * @param names
 * @return
 */
@RequestMapping("show/{id}/{name}")
public ModelAndView test(@PathVariable("id") Long ids ,@PathVariable("name") String names){
    ModelAndView mv = new ModelAndView();
    mv.addObject("msg","占位符映射：id:"+ids+";name:"+names);
    mv.setViewName("hello");
    return mv;
}
```

### @RequestParam

`@RequestParam`：获取请求参数的值

常用来处理简单类型的绑定，通过 `Request.getParameter()` 获取的 `String` 可直接转换为简单类型的情况（String--> 简单类型的转换操作由 `ConversionService` 配置的转换器来完成）；因为使用 `request.getParameter()` 方式获取参数，所以可以处理 `get` 方式中`queryString`的值，也可以处理 `post`方式中 `body data` 的值；

```java
// 请求示例：/getUser?uid=123
@RequestMapping("/getUser")
public String getUser(@RequestParam("uid")Integer id, Model model) {
    System.out.println("id:"+id);
    return "user";
}
```

### @RequestHeader

把 `Request` 请求 `header` 部分的值绑定到方法的参数上

```java

@RequestMapping("/test1")  
public void test1(@RequestHeader("Accept-Encoding") String encoding, 
@RequestHeader("Keep-Alive") long keepAlive)  {  
}
```

### @CookieValue

把 `Request header` 中关于 `cookie` 的值绑定到方法的参数上

```java
@RequestMapping("/test1")  
public void test1(@CookieValue("UserInfo") String cookie)  {   
}
```

## 注入bean

### @Repository

`@Repository`：使用 `@Repository` 注解可以确保 `DAO` 或者 `repositories` 提供异常转译，这个注解修饰的 `DAO` 或者`repositories`类会被 `ComponetScan` 发现并配置，同时也不需要为它们提供XML配置项。

### @Service

* `@Service` 是 `@Component` 注解的一个特例，作用在类上
* `@Service` 注解作用域默认为单例
* 使用注解配置和类路径扫描时，被 `@Service` 注解标注的类会被 `Spring` 扫描并注册为 `Bean`
* `@Service` 用于标注服务层组件,表示定义一个 `bean`
* `@Service` 使用时没有传参数，`Bean` 名称默认为当前类的类名，首字母小写
* `@Service("serviceBeanId")` 或 `@Service(value="serviceBeanId")` 使用时传参数，使用 `value` 作为 `Bean` 名字

### @Scope作用域

`@Scope` 作用在类上和方法上，用来配置 `spring bean` 的作用域，它标识 `bean` 的作用域

```java
public  Scope {

    /**
     * Alias for {@link #scopeName}.
     * @see #scopeName
     */
    
    String value() default "";

    
    String scopeName() default "";

    ScopedProxyMode proxyMode() default ScopedProxyMode.DEFAULT;
}
```

属性介绍

**value**

* singleton   表示该bean是单例的。(默认)
* prototype   表示该bean是多例的，即每次使用该bean时都会新建一个对象。
* request     在一次http请求中，一个bean对应一个实例。
* session     在一个httpSession中，一个bean对应一个实例。

**proxyMode**
    
* DEFAULT         不使用代理。(默认)
* NO              不使用代理，等价于DEFAULT。
* INTERFACES      使用基于接口的代理(jdk dynamic proxy)。
* TARGET_CLASS    使用基于类的代理(cglib)。

### @Entity

`@Table(name="")`：表明这是一个实体类，必须与 `@Id` 注解 结合使用,否则  `No identifier specified for entity:`。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，`@Table`可以省略。
`@Table(name ="数据库表名")`，这个注解也注释在实体类上，对应数据库中相应的表。
`@Id`、`@Column` 注解用于标注实体类中的字段，pk字段标注为 `@Id`，其余 `@Column`。

### @Data

导入依赖：`lombok.Data` ,

```java
<dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.10</version>
 </dependency>
```

`@Data`: 注解在类上, 为类提供读写属性, 此外还提供了 `equals()`、`hashCode()`、`toString()` 方法。

### @Bean产生一个bean的方法

`@Bean` 明确告诉方法，产生一个 `Bean` 对象，并且交给 `Spring` 容器管理。支持别名 `@Bean("xx-name")`，产生这个 `Bean` 对象的方法 `Spring` 只会调用一次，随后这个 `Spring` 将会将这个 `Bean` 对象放在自己的IOC容器中。

### @Autowired 自动导入

* `@Autowired` 注解作用在构造函数、方法、方法参数、类字段以及注解上
* `@Autowired` 注解可以实现Bean的自动注入

### @Component

泛指组件，当组件不好归类的时候，我们可以使用这个注解进行标注。

把普通 `pojo` 实例化到 `spring` 容器中。

## 导入配置文件

### @PropertySource导入属性

```java
// 引入单个properties文件：
@PropertySource(value = {"classpath : xxxx/xxx.properties"})

// 引入多个properties文件：
@PropertySource(value = {"classpath : xxxx/xxx.properties"，"classpath : xxxx.properties"})
```

### @ImportResource导入xml配置文件

可以额外分为两种模式 `相对路径classpath`，`绝对路径file`
注意：单文件可以不写 `value` 或 `locations` ，`value`和 `locations`都可用

**相对路径（classpath）**

* 引入单个xml配置文件：`@ImportSource("classpath : xxx/xxxx.xml")`
* 引入多个xml配置文件：`@ImportSource(locations={"classpath : xxxx.xml" , "classpath : yyyy.xml"})`

**绝对路径（file）**

* 引入单个xml配置文件：`@ImportSource(locations= {"file : d:/hellxz/dubbo.xml"})`
* 引入多个xml配置文件：`@ImportSource(locations= {"file : d:/hellxz/application.xml" , "file : d:/hellxz/dubbo.xml"})`

取值：使用 `@Value` 注解取配置文件中的值

```java
@Value("${properties中的键}")
private String xxx;`
```

### @Import 导入额外的配置信息

功能类似XML配置的，用来导入配置类，可以导入带有 `@Configuration` 注解的配置类或实现了 `ImportSelector/ImportBeanDefinitionRegistrar`。

```java
@SpringBootApplication
@Import({SysConfig.class})
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

## 事务注解 @Transactional

在Spring中，事务有两种实现方式，分别是编程式事务管理和声明式事务管理两种方式

* 编程式事务管理： 编程式事务管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。

* 声明式事务管理： 建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务，通过@Transactional就可以进行事务操作，更快捷而且简单。推荐使用

## 全局异常处理

### @ControllerAdvice 统一处理异常

`@ControllerAdvice` 注解定义全局异常处理类

```java
@ControllerAdvice
public class GlobalExceptionHandler {
}
```

### @ExceptionHandler 注解声明异常处理方法

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    @ResponseBody
    String handleException(){
        return "Exception Deal!";
    }
}
```

参考：

[spring boot注解@RequestMapping、@RequestBody的详解](https://blog.csdn.net/qq_20957669/article/details/87686899)

[SpringBoot注解最全详解(整合超详细版本)](https://blog.csdn.net/weixin_40753536/article/details/81285046)

[Spring Boot 常用注解汇总](https://tqlin.cn/2019/11/22/issueGather/Spring%20Boot%20%E5%B8%B8%E7%94%A8%E6%B3%A8%E8%A7%A3/)
