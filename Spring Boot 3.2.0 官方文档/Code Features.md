# Code Features

> Spring Boot 的核心特性
>
> [Code Features 官网连接](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features)





## SpringApplication

* SpringApplication 提供了静态的 *run* 方法，用于在 *main()* 方法中引导 Spring 应用程序启动。

```java
// 使用示例
@SpringApplication
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

* 启动成功后，控制台默认显示 INFO 级别的日志信息



### Startup Failure

> Spring 应用启动失败

* Web 应用端口 8080 被占用，导致应用启动失败，注册的 *FailureAnalyzers* （故障分析）会提供错误信息和解决方案

```xml
***************************
APPLICATION FAILED TO START
***************************

Description:

Embedded servlet container failed to start. Port 8080 was already in use.

Action:

Identify and stop the process that is listening on port 8080 or configure this application to listen on another port.
```

* 如果没有 *FailureAnalyzers* 能提供错误信息和解决方案，可以使用以下方法完整地展示情况报告
  * 启动 *debug* 属性
  * 启动 *debug* 日志
* Java 程序运行过程中启动调试的命令：`java -jar myproject-0.0.1-SNAPSHOT.jar --debug` 



### Lazy Initialization

> 延时初始化

* SpringApplication 允许应用程序延时初始化。当启动延时初始化时，bean 只有在需要时创建，而不是在程序启动时创建。
  * 延时初始化可以减少程序启动的时间
  * 在 Web 环境中启动延时初始化后，与 HTTP 请求有关的 bean 只有在接收到对应的 HTTP 请求后创建。
* 延时初始化的缺点
  * 不会立即发现与 bean 的创建有关的错误，只有在具体创建时才会发现问题
  * 可能会导致 *JVM* 内存溢出（*OOM*）需要确保 *JVM* 有足够的内存空间存放延时创建的 bean
  * 因此默认关闭延时初始化，需要使用时可以调整 *JVM* 内存大小，满足 bean 的使用
* 启动延时初始化
  * 调用 *SpringApplicaiton* 类中的 `setLazyInitialization(boolean)` 方法
  * 调用 *SpringApplicationBuilder* 类中的 `lazyInitialization(boolean)` 方法
  * 在 properties 或 yml 文件中使用以下配置开启延时初始化

```properties
spring.main.lazy-initialization=true
```

* 如果想要禁用某些 bean 的延时初始化，可以使用 `@Lazy(false)` 显式声明禁用该 bean 的延时初始化



### Customizing the Banner

> [Customizing the Banner 定制标志官网链接](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.banner)

* Spring Boot 官方的 Banner

![image-20231214162132207](F:\Typora-note\img\image-20231214162132207.png)



### Customizing SpringApplication

> 定制 Application

* 创建一个 SpringApplication 实例，手动传入各种参数定制 SpringApplication

```java
@SpringBootApplication
public class TaskApplication{
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(TaskApplication.class);
        application.setBannerMode(Banner.Mode.OFF);		// 关闭 Banner
        application.run(args);
    }
}
```



### Fluent Builder API

> 流式调用的构建器API

* Spring Boot 提供了流式 API 的实现： *SpringApplicationBuilder* ，可以用来构建具有层次结构的 *ApplicationContext* 
* *SpringApplicationBuilder* 使用示例

```java
new SpringApplicationBuilder()
                .sources()
                .child()
                .run();
```



### Application Availability

> 应用程序可用性

* 当应用程序部署到平台运行时，应用程序可以使用 *K8S probes* 向平台提供应用程序可用性信息
* Spring Boot 提供了对于常用 *liveness* （活跃）和 *readiness* （就绪）状态的开箱即用支持，如果需要使用 Spring Boot 关于 *actuator* 的支持，需要公开这些状态。

#### Liveness state

> 活跃状态

#### Readiness state

> 就绪状态

#### 管理 Application Availability State

* 



### Application Events and Listeners

> 应用程序事件和侦听器

* 除了常见的 Spring 框架事件，SpringApplication 还会发送一些额外事件
* 部分额外事件在创建 *ApplicationContext* 之前触发的，因此无法将这些事件的 *Listeners* 注册为容器中的 bean，可以使用以下方法注册 *Listeners* 

```java
SpringApplication.addListeners(···);
SpringApplicationBuilder.listeners(···);
```

* Spring Boot 使用事件来处理各种任务，开发人员无需直接使用，Spring Boot 中的 Event 主要用于实现组件之间的解耦和通信，允许开发人员在应用程序中创建和使用自定义事件。

* Spring Boot 中 Event 的实现和自定义，Event 实现基于观察者设计模式

  * 在Spring Boot中，事件主要通过 `ApplicationEvent` 和 `ApplicationListener` 这两个接口进行实现。
  * 开发人员自定义 Event 只需要继承 `ApplicationEvent` 并实现业务逻辑。
  * 实现 `ApplicationListener` 接口或者使用 `@EventListener ` 注解来创建监听器，当事件发生时，对应的处理方法就会被自动调用

* 程序运行时的应用程序事件发送顺序

  > [Events 触发顺序官网链接](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.spring-application.application-events-and-listeners)

  1. `ApplicationStartingEvent` 

     * > 应用程序启动事件。在程序开始运行之后，任何初始化操作执行之前触发，需要排除 *Listeners* 和初始化器的注册

  2. `ApplicationEnvironmentPreparedEvent` 

     * > 应用程序环境准备事件。已知上下文环境 *Environment* 之后，在创建上下文 *context* 之前

  3. `ApplicationContextInitializedEvent` 

     * > 程序上下文初始化事件。在准备好 *Application* 之后，在调用 *ApplicationContextInitializers* 加载任意一个 bean 之前触发

  4. `ApplicationPreparedEvent ` 

     * > 程序准备事件。

  5. `ApplicationStartedEvent` 

     * > 程序启动事件。

  6. `AvailabilityChangeEvent` 

     * > 程序可用性变化事件。

  7. `ApplicationReadyEvent` 

     * > 程序就绪事件。

  8. `AvailabilityChangeEvent` 

     * > 程序可用性变化事件。

  * `ApplicationFailedEvent` 

    * > 程序失败事件。

* 以上的事件仅包括绑定到 *SpringApplication* 的 *SpringApplicationEvent* 



* 事件的 Listeners 默认在同一线程中执行，因此 Listeners 的任务执行时间不能太长



* 应用程序的发布是通过 Spring 框架的事件发布机制发布的，此机制保证给子 *Context* 中对应 *Listeners* 发送的事件时，也会给父 *Context* 中的 *Listeners* 发送相同事件，因此如果应用程序使用 *SpringApplication* 的层次结构，*Listeners* 或收到多个来自相同事件的多个实例。

* 为了使 *Listeners* 可以分辨该事件是来自当前层次的 *context* 还是来自子 *context* 的事件，可以将 *context* 注入 *Listeners* 中，和接收的事件的 *context* 进行比较。

  * 实现的注入的方式：

    * 实现 *ApplicationContextAware*，获取 *ApplicationContext* 

      * > 作用是让实现它的 bean 能够方便地获取 *ApplicationContext* ，当 spring 容器实例化这个 bean 时，会自动调用 *setApplicationContext()* 方法，为 bean 注入 *ApplicationContext* 

    + 如果这个 *Listeners* 是一个 bean ，可以使用 `@Autowired` 注解注入 *ApplicationContext* 

      + > 问题：如何应用？



### Web Environment

> Web 环境

* SpringApplication 会尝试替代开发人员创建正确的 *ApplicationContext* ，确定 *ApplicationContext* 类型的策略如下：

  * 如果存在 Spring MVC 使用 *AnnotationConfigServletWebServerApplicationContext* 

    * > 基于注解配置的 Servlet Web 服务应用程序上下文

  * 如果 Spring MVC 不存在，但是存在 Spring WebFlux，使用 *AnnotationConfigReactiveWebServerApplicationContext* 

    * > 基于注解配置的响应式 Web 服务应用程序上下文

  * 如果两者都不存在使用 *AnnotationConfigApplicationContext* 

    * > 基于注解配置的应用程序上下文

  * 如果在应用程序中同时使用 Spring MVC 和  Spring WebFlux 中的 *new WebClient*，将默认使用 Spring MVC 可以通过

    * > *new WebClient* 是用于创建 WebClient 实例的方法，WebClient 是一个非阻塞式的 HTTP 客户端，允许开发人员以声明式方式发送 HTTP 请求和处理响应

* 覆盖原有规则的方式

  * 调用  `setWebApplicationType(WebApplicationType)` 设置 *ApplicationContext* 的类型
  * 调用 `setApplicationContextFactory(···)` 方法完全控制 Application 的类型和创建过程
  * 在 Junit 测试中通常调用 `setWebApplicationType(WebApplicationType.NONE)` 方法



### Accessing Application Arguments

> 访问 Application 的参数

* 如果想要访问注入给 *SpringApplication.run()* 方法的应用程序参数，可以注入 *org.springframework.boot.ApplicationArguments* 类型的 bean

```java
// 应用示例
@Component
public class MyConfig {

    @Autowired
    public MyConfig(ApplicationArguments arguments){
        // 检查传入的参数中是否包含 debug 选项
        boolean debug = arguments.containsOption("debug");		
        
        List<String> nonOptionArgs = arguments.getNonOptionArgs();	// 获取非选项参数列表
        if (debug){		// 遍历非选项参数列表
            for (String nonOptionArg : nonOptionArgs) {
                System.out.println(nonOptionArg);
            }
        }
    }
}
```

* *ApplicationArguments* 提供对原始的 String[] 参数、选项参数、非选项参数的访问方法
* Spring Boot 会将 *CommandLinePropertySource* 注册到 Spring Environment 中，可以使用 `@Value` 注入应用程序参数



### Using the ApplicationRunner or CommandLineRunner

> 应用运行器和命令行运行程序

* 如果需要在 SpringApplication 启动之后执行某些特定代码，可以实现 *ApplicationRunner* 和 *CommandLineRunner*  接口

  * > ​		Spring Boot 容器在启动后，会自动调用实现了 CommandLineRunner 或 ApplicationRunner 接口的类的 run 方法，且在整个应用生命周期内只会执行一次。因此，这类接口非常适合用于执行那些需要在应用启动后立即执行且仅需要执行一次的操作。
    >
    > ​		如果有多个接口的实现类，可以使用 `@Order ` 注解来指定任务的执行顺序，或者实现 *org.springframework.core.Ordered* 接口。

* 这两个接口使用相同的方式工作，都提供了一个 *run* 方法，该方法在 *SpringApplication.run(···)* 完成之前调用

* *CommandLineRunner* 提供了对字符串形式的应用程序参数的方法方式，而 *ApplicationRunner* 使用 *ApplicationArguments* 访问参数

```java
// CommandLineRunner 的 run() 方法使用
@Component
public class MyConfig implements CommandLineRunner {
    
    @Override
    public void run(String... args) throws Exception {
        ···
    }
}

// 实现了 Ordered 接口，定义操作的执行顺序
// org.springframework.core.Ordered
@Component
public class MyConfig implements CommandLineRunner, Ordered {

    @Override
    public void run(String... args) throws Exception {
        for (String arg : args) {
            System.out.println("Application Argument:" + arg);
        }
    }

    @Override
    public int getOrder() {
        return 0;
    }
}
```



### Application Exit

> 应用程序出口

* 每个 *SpringApplication* 在 JVM 中注册了一个 shutdown hook（关闭钩子）以保证在 *ApplicationContext* 退出之后正常关闭 *SpringApplication* ，可以使用所有 Spring 标准生命周期回调函数，例如 *DisposableBean* 接口中的 *@PreDestory* 



## Admin Features

> 特征管理

* 通过 *spring.application.admin.enabled* 属性启动应用程序管理功能
* 可以通过该功能实现 Spring Boot 应用程序的远程管理



## Application Startup tracking

> 应用程序启动跟踪



## Externalized Configuration

> Externalized Configuration：外部化配置
>
> 可以使用 properties、YML、环境变量、命令行参数作为 Spring Boot 应用程序的外部配置

* 使用 `@Value` 注解将属性注入到 *bean* 中，可以通过 Spring 环境抽象访问，或者使用 `@ConfigurationProperties` 注解将配置绑定到对象上（*JavaBean*）
* *PropertySources* 是一个接口，用于表示应用程序中的属性源，*Spring Boot*  提供了很多的 *PropertySources* ，可以合理地覆盖之前的配置，如果同一个属性在多个 *PropertySources* 中都有配置，使用优先级最高的 *PropertySources*。



### *PropertySources* 的作用顺序

> 优先级逐渐从低到高

1. 默认配置，使用 *SpringApplication* 类中的 *setDefaultProperties*  方法设置的默认配置

2. `@Configuration` 类上使用 `@PropertySources` 注解绑定的配置

   * 使用 `@PropertySource` 注解，开发人员可以将属性文件加载到 *Spring* 的环境中，然后在应用程序中使用这些属性。这些属性可以在 *Spring* 的 *Environment* 中以 *key-value* 键值对的形式存在，使得在应用程序中可以通过 `@Value` 或者占位符 `${key}` 的形式来使用这些属性。

3. 配置数据，例如：*application.properties* 

   * > 应用程序中配置文件的参数

4. 只包含随机属性（random.*）的 *RandomValuePropertySource* 

   * > 配置了 random.* 的参数

5. 操作系统环境变量

6. `System.getProperties()`，用于获取当前系统的所有属性包括操作系统、硬件、*JVM* 等信息。返回值是一个 Properties 对象

   * > Java System properties

7. 来自 `java:comp/env` 的 JNDI 属性

8. `ServletContext` 初始化参数

9. `ServletConfig` 初始化参数

10. 来自 *SPRING_APPLICATION_JSON* 的参数

11. 命令行参数

12. 单元测试上的参数

13. 使用 *@TestPropertySource* 绑定的参数

14. Devtools 全局设置参数

    + > 来自 $HOME/.config/spring-boot

15. 应用配置文件

* 配置文件的加载顺序，同一层次同时具有 .properties 文件和 .yml 文件，.properties 文件优先使用

  * *Jar* 包中的 application.properties 或 *.yml
  * *Jar* 包中的 application-{profile}.properties 或 *.yml 
  * *Jar* 包之外的  application.properties 或 *.yml
  * *Jar* 包之外的 application-{profile}.properties 或 *.yml





