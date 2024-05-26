---
title: Spring Boot 复习笔记
toc: true
cover: https://source.unsplash.com/random
tags: ['Java', 'SpringBoot']
categories: ['后端']
date: 2024-05-26 23:44:42
---

Official Docs: [spring-boot](https://spring.io/projects/spring-boot)

Reference: [freeCodeCamp.org - Spring Boot & Spring Data JPA – Complete Course](https://youtu.be/5rNk7m_zlAg?si=WM7hFzuNvNxiC0YT)

# Springboot in action
主要的特性：
- IOC
- MVC
- AOP
- DAF(Data Access Framework)

<!-- more -->

# Spring beans in action
Spring bean object lifecycle is managed by the **spring container**.

## 复习Spring Bean的用法
### Spring注解分类（以IOC为中心）
广义上Spring注解可以分为两类，[注册和消费](https://zhuanlan.zhihu.com/p/99870991)Bean。

注册Bean的注解有`@Component`, `@Repository`, `@Controller`, `@Service`, `@Configration`, `@Bean`等。 把对应的类注册到Spring容器中。

消费Bean的注解有`@Autowired`，`@Resource`等，找到容器中对应的Bean对象使用。

### `@Bean`
`@Component`, `@Repository`, `@Controller`, `@Service`, `@Configration`都只能声明在类上，但是如果需要把已有的类放到IOC容器中，上述注解就无能为力了。

```Java
@Configuration
public class AppConfig {
    @Bean
    public PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    public DataSource dataSource() {
        return null;
    }
}
```
`@Bean`最好与`@Configration`配合使用。

### Bean naming
`@Component`, `@Repository`, `@Controller`, `@Service`, `@Configration`等会以类的名称（首字母小写）来命名Bean。

`@Bean`会以提供Bean的方法的方法名来命名。

所有的注册Bean的注解也可以提供名称，来标识Bean。

## Dependency Injection
1. Constructor Injection
2. Field Injection
3. Configuration Methods Injection
4. Setter Methods Injection

### Field Injection
属性注入最常见，也就是
```java
@RestController
public class PaymentController {
    @Autowired
    private PaymentService paymentService;

    @GetMapping("payment")
    public String getMethodName(@RequestParam String param) {
        return this.paymentService.toString();
    }

}
```
缺点是无法注入不可变的对象（final修饰的只能在初始化时赋值）。

### Constructor Injection

注意**Spring官方推荐构造方法注入。**
```java
@RestController
public class PaymentController {
    private PaymentService paymentService;

    @Autowired
    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    @Autowired
    public PaymentController(PaymentService paymentService, String name) {
        this.paymentService = paymentService;
    }

    @GetMapping("payment")
    public String getMethodName(@RequestParam String param) {
        return this.paymentService.toString();
    }
}

```

### Configuration Methods Injection
```java
@Service
public class PaymentService {
    @Autowired
    public void confidClass(PaymentRepository paymentRepository) {
        System.out.println(paymentRepository.toString());
    }
}
```

### Setter Methods Injection
Java中的getter和setter其实就是常规方法，但符合一定的命名约定，起到控制更新和访问的作用。

## `@Qualifier`

Qualifier 的意思是合格者，通过这个标示，表明了哪个实现类才是我们所需要的。`@Autowired`因为是按照类型选择Bean，所以如果有多个相同类型的Bean，就会报错。这个时候就需要`@Qualifier`的辅助。

### `@Primary`
另一个类似的注解是`@Primary`，如其名，也是和`@Bean`一起使用。有两个及以上相同类型的Bean注入时，在其中一个上声明`@Primary`，使用时会优先装配这一个。

## Bean Scope
1. singleton
2. prototype
3. request
4. session
5. application
6. websocket

> scope用来声明容器中的对象所应该处的限定场景或者说该对象的存活时间，即容器在对象进入其相应的scope之前生成并装配这些对象，在该对象不再处于这些scope的限定之后，容器通常会销毁这些对象.

```java
@Configuration
public class AppConfig {
    @Bean
    @Scope("prototype")
    public PaymentService paymentService() {
        return new PaymentService();
    }

    @Bean
    @SessionScope
    public DataSource dataSource() {
        return null;
    }
}

```

### singleton
单例。在Spring的IoC容器中只存在一个实例，所有对该对象的引用将共享这个实例。

### prototype
多例。每个请求方可以得到自己对应的一个对象实例。

### application
对象实例在应用程序的整个生命周期中存在。

### request，session，websocket
这几个Scope都是Web MVC相关的，对应周期是一个请求的周期，一个会话的周期和一个websocket的周期。

## `@Profile`
有点类似Angular中的`Environment`。在项目运行中，包括多种环境,QA,UAT,PROD等等。
`@Profile` 注解的作用是指定类或方法在特定的 Profile 环境生效。

```java
@Configuration
@Profile("prod")
public class JndiDataConfig {
    @Bean()
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```
然后在配置文件中设置需要使用哪一个环境。
```yaml
spring:
  profiles:
    active: dev
```
或者
```java
applicationContext.getEnvironment().setActiveProfiles("prod");
```

## `@Value`
`@Value`注解将配置文件中key对应的值赋值给它标注的属性。

## Best Practice
### Split Configuration
@Import主要用于导入类对象与另一个类对象的依赖.

```java
@Configuration
public class ServiceConfig {
    @Bean
    public PaymentService paymentService(){
        return new ...;
    }
}

@Configuration
public class RepositoryConfig {
    @Bean
    public Accountrepository accountrepository(){
        return new ...;
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class AppConfig {
    @Bean
    public DataSource dataSource(){
        return new ...;
    }
}
```

#### `@Import`
@Import注解是用来导入配置类或者一些需要前置加载的类.

## What is Spring Boot?
我的理解就是Spring框架的框架，提供一些开箱即用的配置。

### Spring Initialnizer 中的 snapshot 版本
snapshot是工程版本上的概念，表示快照版本，也就是相对稳定版本，但是会再进行改进，最好不要用于生产环境，在生产环境，应当选择release（发布版本）。 

### IDE
Both IDEA and VS Code is ok.

### Text Banner Generator
Spring的字符标语是怎么生成的？搜索Text Banner Generator，比如[ascii-banner](https://manytools.org/hacker-tools/ascii-banner/#google_vignette).


Spring Boot中可以自定义Banner。在`/src/main/resources`目录下新建一个`banner.txt`文件，放入上述网站生成的Banner字符即可。Spring Boot 框架在启动时会查找 banner 信息。

### 分析入口程序
```java
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```
鼠标放在`run`上，
```text
ConfigurableApplicationContext org.springframework.boot.SpringApplication.run(Class<?> primarySource, String... args)
Static helper that can be used to run a SpringApplication from the specified source using default settings.

Parameters:

primarySource the primary source to load

args the application arguments (usually passed from a Java main method)

Returns:

the running ApplicationContext
```
可以看到`SpringApplication.run(DemoApplication.class, args);`实际上返回了`ConfigurableApplicationContext`，也就是SpringBoot的运行上下文。


所以可以通过接口拿到Bean。
```java
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		ApplicationContext ctx = SpringApplication.run(DemoApplication.class, args);
		PaymentRepository paymentRepository = ctx.getBean(PaymentRepository.class);
		System.out.println(paymentRepository.toString());
	}

}
```

### 解析源码

`@Repository`, `@Controller`, `@Service`实际上都是`@Component`。

# DI
这部分是上一部分的总结和实践。不好记录。

# Spring Special Beans
## Envirnoment
Spring Boot启动后容器里会存在Environment对象实例，可以在 Environment 中获得系统参数，命令行参数，文件配置等信息。

`Environment = Property + Profile`

Property也就是配置，命令行参数、操作系统环境变量、配置文件等等都归为这一类。

Profile对应环境。

我们可以直接在应用中注入`Environment`对象，获取这些信息。

## `@PropertySource`
@PropertySource注解用于指定资源文件读取的位置。（默认只能读取`application.yaml`）

# Spring Profile
参考`Spring beans in action`部分的`@Profile`。

## SpringBoot多环境配置
在`application.yml`同一目录下新建以下三个文件：
1. `application-dev.yml`：本地开发环境
2. `application-test.yml`：测试环境
3. `application-prod.yml`：生产环境

那么只需要在配置文件中切换active的环境即可使用对应的配置文件。（`application.yml`中是通用的配置）

```yaml
spring:
  profiles:
    active: dev
```

# Spring REST
## 什么是REST
REST直译是表现层状态转移，很难理解。

目前看到最好的[解释](https://www.zhihu.com/question/28557115/answer/48094438)：

> URL定位资源，用HTTP动词（GET,POST,DELETE,DETC）描述操作。

非常简单高效地实现了接口的规范。

### 案例

举例说明，要对班级学生进行CRUD，没有REST的话，光Read就有很多种写法：

```
{{base_url}}/get_students
{{base_url}}/view_students
{{base_url}}/list_students
```
但是遵守REST规范呢？
```
GET {{base_url}}/students
```
删除就是
```
DELETE {{base_url}}/students
```

### 规范
参考： [RESTful API 设计指南](http://www.ruanyifeng.com/blog/2014/05/restful_api.html)

1. 命名必须全部小写
2. 资源（resource）的命名必须是名词，并且必须是复数形式，如果是取一个值，传递id：`GET /accounts/1`
3. 如果要使用连字符，建议使用‘-’
4. 嵌套的资源，规范类似：`GET /accounts/1/payments/56`

`POST`, `GET`, `PUT`, `DELETE` 方法分别对应`CRUD`。

### 响应码
- 1xx = Informatio（信息）
- 2xx = Success（成功）
- 3xx = Redirect（重定向）
- 4xx = User error（客户端错误）
- 5xx = Server error（服务器端错误）

# Spring REST in Action

基础的用法大概会：

```java
@Data
public class Student {
    private String name;
    private String gender;
    private Integer age;
}
```

```java
@RestController
@RequestMapping("/hello")
public class PaymentController {
    @GetMapping("payment")
    public String getMethodName(@RequestParam('hello') String param) {
        return this.paymentService.toString();
    }

    @PostMapping("student")
    public String postMethodName(@RequestBody String entity) {
        return entity;
    }

    @PostMapping("student2")
    public String studentMethod2(@RequestBody Student student) {
        return student.toString();
    }

    @GetMapping("/hello/{user-name}")
    public String getMethodName2(@PathVariable("user-name") String name) {
        return "name is" + name;
    }
}
```

1. `@RequestParam`
2. `@RequestBody`
3. `@PathVariable`
4. `@ResponseBody`
5. `@RequestMapping`

## `@ResponseStatus`
`@ResponseStatus` 是 Spring Framework 提供的一个注解，可以设置HTTP请求返回的状态码，使用场景主要用于指定控制器方法抛出异常时的 HTTP 状态码。

## Dev Tools & Postman
教了如何使用DevTools和Postman。基本都会。

## `@JsonProperty`
```java
@Data
public class Student {

    @JsonProperty("your_name")
    private String name;

    @JsonProperty("your_gender")
    private String gender;

    @JsonProperty("your_age")
    private Integer age;
}
```

`@JsonProperty`对实体类属性做映射，把属性名称转换为另一个名称。也就是前端得到的字段key是`@JsonProperty`中的字段，和后端的实际字段映射。

## Record
Java16 新特性。

Record关键字也是定义类的，主要是为了提供一种更为简洁、紧凑的final类的定义方式。避免了书写setter,getter,toString等等模板代码。

感觉和Lombok差不多，内置实现。

```java
public record StudentRecord(
        String name,
        String gender,
        Integer age) {
}
```

## `@ResponseBody`
`@ResponseBody`的作用是将`Java对象`转为`Json格式`的数据。

```java
@RequestMapping("/hello")
@ResponseBody
public Object hello(String name, String password) {
	return new DTO();
}
```

可以看到，return的实际是一个Java对象，需要`@ResponseBody`转化为Json后才能写入Response发送回前端。

## `@RestController`

`@RestController`是一个组合注解，它包含了`@Controller`和`@ResponseBody`两个注解的功能（Class Level）。

`@RestController`注解在处理请求时，会自动将方法的返回值转换为JSON格式的响应体，并返回给客户端（Class Level）。


## DispatcherServlet
![alt text](_media/7b8817122aa542e7b8e997ef5120688c.png)

#TODO
还不是很理解，有空需要再看一下[DispatcherServlet详解](https://blog.csdn.net/m0_45406092/article/details/115423861)

# Spring Data JPA

使用JPA来进行数据持久化。

## 引入依赖和连接
### 引入依赖
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'

    // 下面这两个就是需要的依赖
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'com.mysql:mysql-connector-j'
}

```

需要的依赖，一个是JPA，还有一个是给JDBC提供的具体数据库厂商的驱动（实现了JDBC规范）。

### 配置和连接JDBC
数据库的URL可以在类似DBeaver这样的数据库管理软件的连接界面复制，比如：

![alt text](<_media/CleanShot 2024-03-14 at 01.25.53.png>)

下面的配置文件是给JDBC使用的。
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/
    username: root
    password: 20010610
    driver-class-name: com.mysql.jdbc.Driver
```

配置完启动，阅读终端输出信息，发现`driver-class-name`更名了，所以改为如下格式：

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/helloworld
    username: root
    password: 20010610
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 配置JPA

```yml
spring:
  jpa:
    # jpa底层默认使用的ORM框架是hibernate
    hibernate:
    # DDL是数据定义语言, 这个字段设置怎么控制数据库里的表
      ddl-auto: create
    # 打印sql
    show-sql: true
    properties:
      hibernate:
        # 打印格式化的sql
        "[format_sql]": true
        # 数据库方言
        dialect: org.hibernate.dialect.MySQLDialect
    # 这个不是必须的，会自动检测
    database: mysql
```

一些常见的配置可以在[Spring Common Application Properties](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#appendix.application-properties.core)中查看。

设置一个 entity 类:

```java
@Entity
public class Teacher {
    @Id
    private Integer id;

    private String firstName;

    private String lastName;

    private String email;

    private Integer age;

    @Override
    public String toString() {
        return "Teacher [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email
                + ", age=" + age + "]";
    }
}
```

## `@Entity`

`@Entity` 说明这个 class 是实体类，并且使用默认的 orm 规则，即 class 名即数据库表中表名，class 字段名即表中的字段名。

查看`@Entity` 注解的源码，可以发现它有一个name属性，指定这个实体类的名称。

```java
@Documented
@Target(TYPE)
@Retention(RUNTIME)
public @interface Entity {

	/**
	 * (Optional) The entity name. Defaults to the unqualified
	 * name of the entity class. This name is used to refer to the
	 * entity in queries. The name must not be a reserved literal
	 * in the Jakarta Persistence query language.
	 */
	String name() default "";
}
```

## `@Table`

`@Table` 标注的常用选项是 `name`，用于指明数据库的表名。当实体类与其映射的数据库表名不同名时需要使用 `@Table` 标注说明。


## `@Id`

`@Id` 注解是一个非常重要的注解，它用于映射实体类中的主键字段（所以可以设置与主键相关的一些参数）。

除了 `@Id` 注解外，JPA 还提供了许多其他注解，例如 `@GeneratedValue`、`@SequenceGenerator`、`@TableGenerator` 等，用于控制主键生成策略和生成器的配置。

```java
@Entity
@Table(name = "teacher_table")
public class Teacher {
    @Id
    @GeneratedValue(strategy = GenerationType.UUID, generator = "")
    private UUID id;

    @Column(name = "first__name", nullable = false, updatable = false)
    private String firstName;
}
```

## `@Column`

用来标识实体类中属性与数据表中字段的对应关系。可以点击查看源码，里面有很多属性可以设置（DDL里设置在字段上的控制参数之类的）。其中有一些是前台控制，也就是说是Java 持久层API提供的特性而不是数据库原生的特性。

```java
@Entity
@Table(name = "teacher_table")
public class Teacher {
    @Id
    private Integer id;

    @Column(name = "first__name", nullable = false, updatable = false)
    private String firstName;

    private String lastName;

    private String email;

    private Integer age;

    @Override
    public String toString() {
        return "Teacher [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + ", email=" + email
                + ", age=" + age + "]";
    }

}
```

突然理解了正向工程是啥意思了。暂停课程，系统了解一下正向和逆向。

## 正向工程 和 逆向工程
- 正向工程：在Java中创建实体，执行后自动映射成数据库中的表
- 逆向工程：存在数据库的表，自动生成实体

JPA 支持 正向工程。

## Sequences
[序列](https://www.ibm.com/docs/en/db2/11.5?topic=objects-sequences)是一个数据库对象，它允许自动生成值，例如校验号。序列非常适合于生成唯一键值的任务。应用程序可以使用序列来避免用于跟踪数字的列值可能导致的并发性和性能问题。序列相对于在数据库外创建的数字的优势在于，数据库服务器可以跟踪生成的数字。崩溃和重新启动不会导致生成重复的数字。

这个东西我也是今天才知道。没见用过。

## Repository

### Interface Hierarchy
![alt text](_media/9c4e4c00c7a64047b819874635b4f7f9~tplv-k3u1fbpfcp-zoom-1.image.png)


### 使用
```java
@Repository
public interface PaymentRepository extends JpaRepository<Teacher, UUID> {

}
```
我们只需要声明这么一个接口，JPA会自动帮我们创建一个实现了这些接口的对象实例，并注入到Spring容器中。

使用的时候，使用依赖注入，拿到对象实例使用即可(封装，继承，多态)。
```java
@RestController
@RequestMapping("teacher")
public class TeacherController {
    private PaymentRepository paymentRepository;

    public TeacherController(PaymentRepository paymentRepository) {
        this.paymentRepository = paymentRepository;
    }

    @SuppressWarnings("null")
    @PostMapping("man")
    public String postMethodName(@RequestBody Teacher teacher) {
        // 调用jpa接口提供的方法
        this.paymentRepository.save(teacher);
        return teacher.toString();
    }

}
```

## 插播：HikariCP连接池

HikariCP是快速、简单、可靠且可用于生产的JDBC连接池。

### 依赖
在Spring Boot 2.0及以后的版本，不需要在pom.xml或build.gradle中加入HikariCP依赖，因为`spring-boot-starter-jdbc`和`spring-boot-starter-data-jpa`会依赖它并且是**默认开启**的。

我们可以通过在`application.yml`中的`spring.datasource.hikari`中自定义`HikariCP`连接池的参数。

```yml
spring:
  application:
    name: demo
  datasource:
    url: jdbc:mysql://localhost:3306/helloworld
    username: root
    password: 20010610
    driver-class-name: com.mysql.cj.jdbc.Driver
    # 下面就是hikari的配置。
    hikari:
      # 在连接池中维护的最小空闲连接数
      minimum-idle: 10
      # 允许一个连接在连接池中闲置的最大时间
      idle-timeout: 30000
      # 最大连接池数大小
      maximum-pool-size: 20
      # 连接关闭后的最长生存时间
      max-lifetime: 120000
      # 客户端从连接池等待连接的最大毫秒数
      connection-timeout: 30000
```

## 插播：Spring JDBC Template 和 Data Source 和 Spring Data JPA
Spring JDBC Template 和 Spring Data JPA 都是上层建筑，需要配置Data Source来操作数据库，Data Source为数据库的来源（为了显示层级结构，我理解为指向JDBC或者连接池）。

# `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@JoinColumn`

这部分没接触过，花了很久我才理解到。

参考：[Difference Between @JoinColumn and mappedBy](https://www.baeldung.com/jpa-joincolumn-vs-mappedby) 和 [@JoinColumn 详解](https://blog.csdn.net/u010588262/article/details/76667283)

## 一对一
主表：
```java
@Entity
@Data
public class StudentProfile {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Integer id;

  @Column
  private String bio;

  // mappedBy 表示放弃维护关系，是关系的被维护方
  @OneToOne(mappedBy = "studentProfile")
  private Student student;
}

```
副表：
```java
@Entity
@Data
@Table(name = "student")
public class Student {
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Integer id;

  @Column(nullable = false, length = 50)
  @JsonProperty("firstName")
  private String firstName;

  @Column(nullable = false, length = 50)
  @JsonProperty("lastName")
  private String lastName;

  @Column(unique = true, length = 50)
  @JsonProperty("email")
  private String email;

  @Column(nullable = true)
  @JsonProperty("age")
  private Integer age;

  // 在维护关系的一方，进行级联操作的声明
  @OneToOne(cascade = CascadeType.ALL)
  @JoinColumn(name = "student_profile_id", referencedColumnName = "id", nullable = true)
  private StudentProfile studentProfile;

  @ManyToOne(cascade = CascadeType.ALL)
  @JoinColumn(name = "school_id", referencedColumnName = "id", nullable = true)
  private School school;
}
```
## `@JoinColumn`
`@JoinColumn`写在控制关联关系的主控，会在该表中加一个外键。

## `@OneToOne`
一对一关系。

## 级联操作
当有了外键约束的时候，必须先修改或删除副表（有外键的表）中的所有关联数据，才能修改或删除主表（外键引用的表）。

级联操作可以实现在修改主表的时候，对副表执行相对应的操作（比如删除主主表数据就同时删除对应的所有副表数据）。

## `@JsonBackReference` 和 `@JsonManagedReference`
因为JPA的Entity声明中存在双向引用，所以当打印实体的时候，很容易一直循环调用。这时候就需要`@JsonBackReference` 和 `@JsonManagedReference`来避免循环。

`@JsonManagedReference`声明在`@OneToMany`的一方，`@JsonBackReference`声明在`@ManyToOne`的一方。

# DTO
DTO stands for data transfer object.

![alt text](<_media/CleanShot 2024-03-16 at 15.27.26.png>)

`@Entity`中声明的实体类对应的是数据库中的表结构，但是想象一下：

1. 数据库表结构中字段太多，全部返回给前端效率低下
2. 数据库中有敏感信息（比如地址等），不希望返回给前端

这就是DTO的使用场景。可以设置从前端接受到的数据的DTO和要返回给前端的属性的DTO。

# Service Layer

- Presentation Layer: Controler
- Logic Layer: Service
- Data Access Layer: Repository

Service层可以负责实现业务逻辑，是数据获取层和表现层的中间层，进行复杂的运算，读取数据，验证，转换（transformation）。

分层。测试的时候还可以专注测试服务层的逻辑方法。可复用，健壮性，好处多多。

## 文件夹结构

除了按照`Controller`,`Service`,`Repository`等层来分类，还可以按照`Entity`来分类(每个Entity里对应这个Entity的表现层，服务层，持久化层等等)。

# Spring Data Validation

好处：

- Data Integrity
- Preventing Attacks
- Error Prevention
- User Experience (UX)
- Performance
- Business Logic Compliance

## 引入依赖

```groovy
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-validation'
}
```

## 使用
参考： [Spring Validation参数效验各种使用姿势](https://juejin.cn/post/7087100869363122189)

### `@Valid` 和 `@NotEmpty（以这个注解为例）`

只是以`@NotEmpty`这个注解为例，实际上有非常多的校验注解。

使用对象接收参数，在需要校验对象的参数加上 `@NotBlank`注解，然后在需要校验的对象前面的`@RequestBody`注解的位置再加上`@Validated`或者`@Valid`注解。

### `ExceptionHandler`

Spring 提供统一处理方法抛出的异常的注解。当异常发生时，Spring会选择最接近抛出异常的处理方法。

参考： [Spring的@ExceptionHandler注解使用方法](https://blog.csdn.net/lkforce/article/details/98494922)

```java
  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<?> handleMethodArgumentNotValidException(
      MethodArgumentNotValidException methodArgumentNotValidException) {
    var errors = new HashMap<String, String>();
    methodArgumentNotValidException.getBindingResult().getAllErrors().forEach(error -> {
      var fieldName = ((FieldError) error).getField();
      var erroeMsg = error.getDefaultMessage();
      errors.put(fieldName, erroeMsg);
    });
    return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
  }
```

# Testing Overview

## 好处
- Quality Assurance：测试确保函数如期望一样工作，及早找到bug，在到达生产环境前
- Regression Testing：确保代码更改后存在的函数还是继续保持正确
- Documentation：提供函数该如何使用的样例
- Code Maintainability：好的代码实践的一环
- Refactoring Confidence：使你自信地重构代码，哪里出错了会马上知道
- Collaboration：确保不会影响其他人的数据和代码
- Continuous Integration/Continuous Deployment (CI/CD): 在CI/CD时自动执行测试
- Reduced Debugging Time：面向测试编程
- Scalability：代码越写越多，越来越复杂，测试会让你更加自信不会更改代码触发其它bug
- Security：排除潜在的安全漏洞

## 分类
- 单元测试
- 集成测试
- 端到端测试

## Spring Test In Action
```gradle
dependencies {
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

`spring-boot-starter-test`依赖了其他的很多库，提供了非常强大的测试能力。

### `@SpringBootTest`

SpringBoot程序提供的可以说就是一个`ApplicationContext`，如果要在测试程序中使用`SpringBoot`的特性，就需要使用`@SpringBootTest`注解，它会创建一个与在生产环境中启动的应用程序上下文非常相似的应用程序上下文。

### 使用

主要的注解和使用方法都可以阅读代码习得。

```java
@SpringBootTest
public class StudentServiceTest {

  @Autowired
  private StudentService studentService;

  @BeforeAll
  static void beforeAll() {
    System.out.println("beforeAll");
  }

  @AfterAll
  static void afterAll() {
    System.out.println("afterAll");
  }

  @BeforeEach
  void setUp() {
    System.out.println("Inside the before each method");
  }

  @AfterEach
  void tearDown() {
    System.out.println("Inside the after each method");
  }

  @Test
  void testAddStudent() {
    StudentDto studentDto = new StudentDto("Helo", "world", "213@qq.com", 1);
    Student student = this.studentService.toStudent(studentDto);
    Assertions.assertEquals(studentDto.firstName(), student.getFirstName());
    Assertions.assertNotNull(student.getEmail());
  }

  @Test
  void shouldMapStudentToDtoWhenNull() {
    Student student = this.studentService.toStudent(null);
    Assertions.assertEquals("", student.getFirstName());
  }

  @Test
  void shouldMapStudentToDtoWhenNullCustom() {
    Assertions.assertThrows(NullPointerException.class, () -> this.studentService.toStudent(null));
  }
}
```

## Mockito 模拟测试

有时候不想在实际的数据库测试，或者框架还没完全搭好，就需要自己模拟数据。

具体参考：[Java测试框架系列：Mockito使用手册](https://juejin.cn/post/6975525979418525709).

暂时跳过笔记。

# SpringDoc(Spring Fox的下一代)

Spring Fox 已经很久没更新了，Spring Boot3就推荐使用SpringDoc作为文档和接口管理工具。

## 引入
```gradle
dependencies {
  implementation group: 'org.springdoc', name: 'springdoc-openapi-starter-webmvc-ui', version: '2.4.0'
}
```

## 配置

```yaml
# 不再使用SpringFox，而是SpringDoc
springdoc:
  api-docs:
    # http://localhost:8080/api-docs
    path: /api-docs
    enabled: true
  swagger-ui:
    # 可以通过 http://localhost:8080/swagger-ui/index.html 和 http://localhost:8080/swagger-ui-custom.html 两个地址访问
    path: /swagger-ui-custom.html
    enabled: true
```

## 使用
```java
@RestController
@Tag(name = "StudentController", description = "学生管理")
@RequestMapping("/v1")
public class StudentController {
  @SuppressWarnings("unused")
  private StudentService studentService;

  StudentController(StudentService studentService) {
    this.studentService = studentService;
  }

  @GetMapping("/student")
  @Operation(summary = "获取学生列表")
  public List<Student> getAllStudents() {
    return this.studentService.getAllStudent();
  }

  @PostMapping("/student")
  public String addStudent(@RequestBody Student student) {
    this.studentService.addStudent(student);
    return "done";
  }
}
```

对应的注解需要使用时再查询即可。

# End
视频从7:34开始就是另外一个案例，看了一下，视频可以关闭了。

![alt text](<_media/CleanShot 2024-03-23 at 12.22.00.png>)

