# Spring Boot Starter

> Starter（一站式场景启动器）其中包括一系列可以集成到应用中的依赖项，可以快速启动 Spring 组件和框架，通过简单配置实现开箱即用

* 
* Spring Boot Starter 一般包含的组件
  * 自动配置模块，实现自动配置
  * 依赖项，为自动配置模块提供的所有依赖项



## Starter 分类

* application starter

  > 应用级，一共有 44 个

* production starter

  > 生产级，只有一个生产级 Starter

  *spring-boot-starter-actuator* 提供监控、管理应用的功能

* technical starter

  > 技术级 Starter

  * *spring-boot-starter-jetty* 集成 Jetty 作为内置 Servlet 容器
  * *spring-boot-starter-log4j2* 集成 Log4j2 作为日志框架
  * *spring-boot-starter-logging* 
  * *spring-boot-starter-tomcat* 
  * *spring-boot-starter-reactor-netty* 集成 Netty 作为内置响应式 HTTP 服务器
  * *spring-boot-starter-undertow* 集成 undertow 作为内置 Servlet 容器



## Spring Boot 自动配置

> Spring Boot 官方提供的自动配置类都是由 *spring-boot-autoconfigure* 模块提供，其中定义了和具体技术有关的自动配置类，底层实现需要导入对应的 Starter

* 默认的命名规则

  > Spring Boot 的自动配置类的命名约定：*xxxConfiguration* 

* 自动配置类需要注册到 Spring Boot 指定的自动配置文件中

  > Spring Boot 2.7+ 自动配置注册文件位置发生了变化，Spring Boot 2.7 之后的老注册文件不能用作应用级别的自动配置类的注册文件，仅保留系统组件的组件注册



### 自动配置加载原理

> 核心：*@EnableAutoConfiguration* 注解，开启自动配置

* *@EnableAutoConfiguration* 注解中的核心注解
  
  * *@AutoConfigurationPackage* 
  
    > 注册需要自动配置的包
  
  * *@Import(···)* 
  
    > 导入 *AutoConfigurationImportSelector* 类，其实现了 *ImportSelector* 接口
  
* *ImportSelector* 接口中的方法

  * `selectImports` 

    > 根据 *@Configuraion* 类的 *AnnotationMetadata* 导入配置类

  * `Predicate<String>` 

    > 返回需要排除的自动配置类

* Spring Boot 3.0 的获取自动配置列表原理

  > 获取自动配置列表的源码

```java
// ImportCandidates 类
public static ImportCandidates load(Class<?> annotation, ClassLoader classLoader) {
    // 断言，annotation 不能为空
    Assert.notNull(annotation, "'annotation' must not be null");		
    
    // 获取 classLoader 类加载器，decide：决定
    ClassLoader classLoaderToUse = decideClassloader(classLoader);		
    
    // 格式化字符串，LOCATION："META-INF/spring/%s.imports"，将 %s 使用 annotation 全类名代替
    String location = String.format(LOCATION, annotation.getName());
    
    // 使用 classLoader 在指定位置查找配置的 url，Enumeration：枚举
    Enumeration<URL> urls = findUrlsInClasspath(classLoaderToUse, location);
    
    // 存放候选的导入配置类
    List<String> importCandidates = new ArrayList<>();
    
    // 遍历 url 的枚举 
    while (urls.hasMoreElements()) {
       URL url = urls.nextElement();
        // 通过 url 读取候选的配置
       importCandidates.addAll(readCandidateConfigurations(url));	
    }
    return new ImportCandidates(importCandidates);
}
```

* Spring Boot 3.0 是加载 *META-INF/spring/%s.imports* 配置文件中的自动配置，*%s* 是 *@AutoConfiguration* 注解的全类名



### 自动配置原理

> 自动配置文件被加载之后，注册其中的配置类

```java
// WebMvcAutoConfiguration 部分源码
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@ImportRuntimeHints(WebResourcesRuntimeHints.class)
public class WebMvcAutoConfiguration {
    
}
```

+ *@AutoConfiguration* 注解是 Spring Boot 2.7 中新增的注解

  > 专门为自动配置定制的注解，组合了三个注解：*@Configuration*，*@AutoConfigureBefore*，*@AutoConfigureAfter* 

  * 实现了标识配置类、指定配置类需要在哪些配置完成之后或之前配置的功能

+ *@Conditionalxxx* 也是实现 Spring Boot 自动配置的核心注解

  > 负责定义加载配置的条件 



#### 自动配置报告

> 用于查看激活和未激活的自动配置

* 自动配置报告需要在 debug 模式中查看

  * Java 命令：`java -jar xx.jar --debug` 或者 `java -jar xx.jar --Ddebug` 
  * 配置文件：debug: true

* 报告格式

  * Positive matches:

    > 已启动的配置

  * Negative matches:

    > 未启动的配置

  * Exclusions:

    > 排除的配置

  * Unconditional classes:

    > 没有条件的配置类



#### 排除自动配置

* *@SpringBootApplication* 注解

  > `@SpringBootApplication(exclude = {xxxAutoConfiguration.class})`  
  >
  > `@SpringBootApplication(excludeName = {xxxAutoConfiguration的全类名})` 

* *@EnableAutoConfiguration* 

  > `@EnableAutoConfiguration(exclude = {xxxAutoConfiguration.class})` 
  >
  > `@EnableAutoConfiguration(excludeName = {xxxAutoConfiguration的全类名})` 

* 统一的排除方案

  * 在配置文件中使用以下参数排除

    ```properties
    spring: 
      autoconfigure:
        exclude: xxxAutoConfiguration 的全类名
    ```



#### 替换自动配置

> 使用自定义配置替换默认配置

* *@ConditionalOnMissBean(···)* 

  > 当应用中不存在指定的 Bean 才会加载配置，避免重复加载相同的 Bean，可以实现配置的替换



### 默认的包扫描规则

* 主程序，具有 `@SpringApplication` 注解的类，就是主程序

* 默认扫描主程序所在的包及其子包
* 自定义包扫描规则
  * `@SpringBootApplication(scanBasePackages = {"com.tang","com"})` 通过 `scanBsesPackages` 属性指定需要扫描的属性
  * `@ComponentScan(basePackages = {"com","com.tang"})` 指定扫描的包路径



### 自动配置流程

* 引入特定的场景启动器，例如 `spring-boot-starter-web`

  1. `spring-boot-starter-web` 中包含了特定场景中的必备依赖，例如：`spring-boot-starter` , `spring-boot-starter-tomcat` 等等
  2. `spring-boot-starter` 是每个场景启动器的 `starter` ，其中包含用于自动配置的 `spring-boot-autoconfigure` 依赖包
  3. `spring-boot-autoconfigure` 包含 Spring Boot 开发场景需要的全部配置


![image-20231212163340276](F:\Typora-note\img\image-20231212163340276.png)

* `@SpringBootApplication` 注解的组成

  1. `@SpringBootConfiguration` 	声明当前类为配置类
  2. `@EnableAutoConfiguration`     开启自动配置，是 Spring Boot 自动配置的核心注解
  3. `@ComponentScan`                          设置包扫描路径，定义默认的包扫描规则
* `@EnableAutoConfiguration` 注解的作用
  * `@Import(AutoConfigurationImportSelector.class)` 
    * 通过 `AutoConfigurationImportSelector` 类中的 `selectImports` 方法向容器批量注入 bean。
    * 在 Spring Boot 项目启动时，从 `spring-boot-autoconfigure` 包中的 `META-INF\spring\org.springframework.boot.autoconfigure.AutoConfiguration.imports` 文件中获取  `xxxAutoConfiguration`  自动配置类，将其导入 spring 容器


![自动配置](\Git-repository\Typora-note\img\image-20231212163631496.png)

* 导入的 `xxxAutoConfiguration` 自动配置类使用 `@ConditionalOnxxx` 注解实现按需加载
* `xxxAutoConfiguration` 自动配置类的作用
  1. 使用 `@Bean` 注解向 spring 容器中导入特定的 bean
  2. 若需要从配置文件中读取数据，加上 `@EnableConfigurationProperties` 注解将配置文件中的配置封装到 `xxxProperties` 特定的属性类中，通过修改配置文件修改 bean 的核心参数


```java
@AutoConfiguration
@ConditionalOnNotWarDeployment		// 当程序以非 war 启动时该配置生效
@ConditionalOnWebApplication
@EnableConfigurationProperties(ServerProperties.class)		// 属性绑定
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {
	···
}

// 属性类
@ConfigurationProperties(prefix = "server", ignoreUnknownFields = true)
public class ServerProperties {
    ···
}
```



#### 配置的默认值

* 配置文件中的配置项都和某一个配置属性类的对象绑定
* 所有配置按需加载





## 自定义 Starter

* 创建 Starter 工程

  > 按照 Starter 命名规范命名的 Spring Boot 工程

* 创建自动配置类

  > 生效条件：配置文件中存在 *mystarter.starter.name=true* 参数，作用创建一个 TestService 类型的 Bean

```java
@AutoConfiguration
// name：参数名，havingValue：参数值
@ConditionalOnProperty(prefix = "mystarter.starter", name = "enable", havingValue = "true")
public class TestServerAutoConfiguration {

    @Bean
    public TestService testService(){
        return new TestService();
    }
}
```

```java
// TestService 类
public class TestService {

    public String getServiceName(){
        return "自定义 Starter";
    }
}
```

* 注册自动配置类

  * Spring Boot 2.7 及以下

    * 按照规范创建自动配置文件

      > META-INF/spring/spring.factories

    * 在自动配置文件中输入自动配置类的全类名

      > org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
      > com.tang.mystarter.autoconfigure.TestServerAutoConfiguration

  * Spring Boot 3.0 以上

    * 创建新规范自动配置文件

      > META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports

    * 在自动配置文件中输入自动配置类的全类名

      > com.tang.mystarter.autoconfigure.TestServerAutoConfiguration

    * 重启 Spring Boot 应用，如果之后应用成功启动，说明配置生效

* 使用 Starter

  > 打包之后测试，与一般 Starter 的使用方法一致
