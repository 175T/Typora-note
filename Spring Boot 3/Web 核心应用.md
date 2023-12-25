# Spring Boot Web 核心应用

> Spring Boot 支持两种 Web 类型：
>
> SERVLET（Spring MVC）
>
> REACTIVE（响应式的 WebFlux）



## 嵌入式容器

> 默认的 Servlet 容器：Tomcat、Jetty、Undertow，都有对应的一站式启动器

* *spring-boot-starter-web* 默认导入 Tomcat 容器

* 容器配置资源绑定类 *ServerProperties* ，通过 *server* 前缀配置参数

![image-20231221103018914](F:\Typora-note\img\image-20231221103018914.png)

* 通过 Java 类自定义 Servlet 容器

  > 实现 *WebServerFactoryCustomizer<ConfigurableServletWebServerFactory>* 接口

  ```java
  @Component
  public class CustomizerWebServerFactory implements WebServerFactoryCustomizer<ConfigurableServletWebServerFactory> {
  
      @Override
      public void customize(ConfigurableServletWebServerFactory factory) {
          server.setPort(8001);
      }
  }
  ```

* *ConfigurableServletWebServerFactory* 类有三个子类，分别代表 Tomcat、Jetty、Undertow 三种容器，分别为自定义不同类型的容器提供支持

  > *TomcatServletWebServerFactory* 
  >
  > *UndertowServletWebServerFactory* 
  >
  > *JettyServletWebServerFactory* 

* 切换容器：排除默认的 Tomcat 依赖，再导入其他容器依赖即可

* 随机空闲端口

  > server.prot = 0，Spring Boot 会自动扫描空闲端口，随机挑选一个使用
  >
  > 

### SSL

> 通过配置 SSL 使用 Spring Boot 应用支持 HTTPS 访问，可以通过 server.ssl 前缀配置 SSL

* 使 Spring Boot 应用同时支持两个协议

  > Spring Boot 不能以配置同时支持 HTTP 和 HTTPS
  >
  > 需要将另一个协议用 Java 配置类的方式配置，通常配置 HTTP

* 使用 Java 配置类配置 HTTP

```java
@Configuration
public class HTTPConfig {
    @Bean
    public ServletWebServerFactory servletWebServerFactory(){
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
        tomcat.addAdditionalTomcatConnectors(createStandardConnector());
        return tomcat;
    }

    private Connector createStandardConnector(){
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setPort(8002);
        return connector;
    }
}
```



### 持久化

> 会话持久化配置：server.servlet.session.*
>
> 具体配置参数：*org.springframework.boot.web.servlet.server.Session* 

* 持久化需要使用 kill -15 pid 触发优雅关闭，不能使用 kill -9 pid 强制关闭



### 优雅关闭

> 优雅关闭：应用正常退出时完成前有请求的处理和关闭响应的资源

* 实现优雅关闭配置

  ```properties
  server:
    shutdown: graceful
  ```

* 缓冲时间配置

  ```properties
  spring:
    lifecycle:
      timeout-per-shutdown-phase: 50s
  ```



## Web 自动配置流程

* 自动配置

  * 引入 Web 开发场景，快速启动一个基于 Servlet 的 HTTP 服务

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

  * Web 开发环境中包含 `spring-boot-starter` 依赖，其中有 `spring-boot-autocinfigure` 包，实现自动配置
  * `@EnableAutoConfiguration` 使用注解 `@Import(AutoConfigurationImportSelector.class)` 批量引入组件，由 `AutoConfigurationImportSelector` 类从 `"META-INF/spring/%s.imports"` 中获取需要加载的组件，`%s` 代表注解 `@AutoConfiguration` 的全类名，即从指定的文件中获取需要加载的组件。

* 与 Web 有关的自动配置类

```
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration

// reactive --- 响应式编程
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveMultipartAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.WebSessionIdResolverAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration
// reactive --- 响应式编程

org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration				// 文件上传
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration					// Spring MVC
```

* Spring Boot MVC  自动配置的应用

​		1、不使用 `@EnableWebMvc` 注解

​		如果要保留 Spring Boot MVC 自动配置并进行 MVC 自定义（拦截器、格式化程序、视图控制器和其他功能），可以添加自己的 WebMvcConfigurer 类型的 `@Configuration` 类（实现 `WebMvcConfigurer `接口），不使用 `@EnableWebMvc` 注解

​		2、使用 `@EnableWebMvc` 注解

​		禁用 Spring Boot MVC 自动配置，使用自定义配置，`@EnableWebMvc` + `@Configuration` 

​			

### WebMvcAutoConfiguration

> Spring Boot 提供的关于 Spring MVC 的自动配置类

* 生效条件

```java
// 在某些配置完成之后，才进行本类的配置，
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
		ValidationAutoConfiguration.class })
		
		
@ConditionalOnWebApplication(type = Type.SERVLET)		// 只有 Web 应用才生效

// 配置的条件，必须要有以下的类
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })

// 不能有的类
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)

// 优先级
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)

@ImportRuntimeHints(WebResourcesRuntimeHints.class)		// 导入
public class WebMvcAutoConfiguration {
    
}
```

* 产生的作用

  * 给 spring 容器中添加了两个 Filter

    * `OrderedHiddenHttpMethodFilter` 作用：滤掉HTTP请求中不需要的隐藏方法，例如PUT、DELETE等。这样可以防止恶意用户通过这些方法对服务器进行攻击。

    * `OrderedFormContentFilter` 作用：解析表单数据，包括 PUT、DELETE 请求·
* `WebMvcAutoConfigurationAdapter` 内部类，实现 `WebMvcConfigurer` 接口，添加 Spring MVC 功能
* 默认的静态资源映射规则

```java
// 定义静态资源映射规则源码，ResourceHandlerRegistry：静态资源注册表
@Override
public void addResourceHandlers(ResourceHandlerRegistry registry) {
    if (!this.resourceProperties.isAddMappings()) {
        logger.debug("Default resource handling disabled");
        return;
    }
    addResourceHandler(registry, this.mvcProperties.getWebjarsPathPattern(),
            "classpath:/META-INF/resources/webjars/");
    addResourceHandler(registry, this.mvcProperties.getStaticPathPattern(), (registration) -> {
        registration.addResourceLocations(this.resourceProperties.getStaticLocations());
        if (this.servletContext != null) {
            ServletContextResource resource = new ServletContextResource(this.servletContext, SERVLET_LOCATION);
            registration.addResourceLocations(resource);
        }
    });
}
```



### WebMvcConfigurer

* 暴露的接口

![image-20231213182242320](\Git-repository\Typora-note\img\image-20231213182242320.png)

* 接口的功能
  * `addArgumentResolvers` 参数解析器，解析 Controller 方法的参数
  * `addCorsMappings` 处理跨域请求
  * `addFormatters` 格式化器
  * `addInterceptors` 添加拦截器
  * `addResourceHandlers` 添加资源处理器，定义静态资源映射规则
  
  * `addReturnValueHandlers` 返回值处理器，处理 Controller 方法的返回值
  * `addViewControllers` 视图控制器，控制页面跳转
  * `configureAsyncSupport` 异步支持
  * `configureContentNegotiation` 内容协商
  * `configureDefaultServletHandling` 默认请求处理
  * `configureHandlerExceptionResolvers` 配置异常解析器
  * `configureMessageConverters` 消息转化
  * `configurePathMatch` 路径匹配
  * `configureViewResolvers` 视图解析
  * `extendHandlerExceptionResolvers` 扩展异常解析器 
  * `extendMessageConverters` 扩展消息转换器
* WebMvcConfigurer  的功能
  * 提供了配置 Spring MVC 所有组件的接口
  * WebMvcConfigurer 和两个资源类绑定，可以使用对应的配置项定制功能
    * `WebMvcProperties` ，对应的配置前缀：`spring.mvc`
    * `WebProperties` ，对应的配置前缀：`spring.web`



## 注册消息转换器

> Spring Boot 使用 *HttpMessageConverter* 作为消息转换器接口，用于转换 HTTP 请求和响应
>
> 自动将 JSON 和 XML 数据类型的请求格式转换为 Java Bean 对象

* Spring Boot 支持三种 JSON 库的自动配置
  * Jackson  
  * JSON - B
  * Gson



## 国际化

> 本地化信息

* Spring Boot 默认会在类的根路径下搜索消息资源包

* 国际化的自动配置类 *org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration* 

* 核心源码

  ```java
  @AutoConfiguration
  @ConditionalOnMissingBean(name = AbstractApplicationContext.MESSAGE_SOURCE_BEAN_NAME, search = SearchStrategy.CURRENT)
  @AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
  @Conditional(ResourceBundleCondition.class)
  @EnableConfigurationProperties
  public class MessageSourceAutoConfiguration {
  
      private static final Resource[] NO_RESOURCES = {};
  
      @Bean
      @ConfigurationProperties(prefix = "spring.messages")		// 配置参数前缀
      public MessageSourceProperties messageSourceProperties() {
         return new MessageSourceProperties();
      }
  
      // MessageSourceProperties 绑定的资源类
      @Bean
      public MessageSource messageSource(MessageSourceProperties properties) {
         ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
         if (StringUtils.hasText(properties.getBasename())) {
            messageSource.setBasenames(StringUtils
               .commaDelimitedListToStringArray(StringUtils.trimAllWhitespace(properties.getBasename())));
         }
         if (properties.getEncoding() != null) {
            messageSource.setDefaultEncoding(properties.getEncoding().name());
         }
         messageSource.setFallbackToSystemLocale(properties.isFallbackToSystemLocale());
         Duration cacheDuration = properties.getCacheDuration();
         if (cacheDuration != null) {
            messageSource.setCacheMillis(cacheDuration.toMillis());
         }
         messageSource.setAlwaysUseMessageFormat(properties.isAlwaysUseMessageFormat());
         messageSource.setUseCodeAsDefaultMessage(properties.isUseCodeAsDefaultMessage());
         return messageSource;
      }
      ···
  }
  ```

* 重要的配置

  ```java
  String basename = "messages";		// 需要扫描的国际化文件名
  
  Charset encoding = StandardCharsets.UTF_8;		// 编码方式
  
  @DurationUnit(ChronoUnit.SECONDS)		// 国际化文件加载后的缓存时间
  private Duration cacheDuration;
  
  private boolean fallbackToSystemLocale = true;		// 找不到对应资源文件是否使用当前操作系统的语言环境
  ```

* 切换国际化

  * 判断用户的请求信息中是否包含语言环境信息，如果没有携带，那么使用默认的语言环境
  * 注册一个拦截器，设置切换语言环境的参数名