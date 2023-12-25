### appdemo 开发过程记录



#### 基于注解实现登陆拦截器

* 实现一个自定义注解

  ```java
  @Target({ElementType.METHOD, ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  public @interface LoginRequired {
  }
  ```

* 基于反射获取类或方法上的注解信息

  ##### 基于反射获取方法中的注解信息

  ```java
  /**
   * ReflexUtil
   * 通过处理器方法获取类或方法上是否存在指定的注解类型
   */
  public static <T extends Annotation> T getClazzOrMethodAnnotation(HandlerMethod  handlerMethod, Class<T> annotationClass){
  
      Class<?> clazz = handlerMethod.getBeanType();			// 获取 handlerMethod 的类型信息（calss 对象）
      T annotation = clazz.getAnnotation(annotationClass);	// 获取指定类型的注解（类）
      if (annotation != null){
          return annotation;
      }
  	// 获取方法上的注解
      return handlerMethod.getMethod().getAnnotation(annotationClass);
  }
  ```

​		HandlerMethod：HandlerMethod 是 Spring MVC 中的一个接口，它表示一个处理请求的方法。当一个控制器类中的方法被调用时，Spring MVC 会将这个方法封装成一个 HandlerMethod 对象，然后通过该对象来执行相应的业务逻辑和视图渲染等操作。HandlerMethod 是 Controller 中定义的处理特定请求的方法。	

* 在拦截器实现中获取类或方法上的注解信息

  ```java
  // 获取注解信息
  HandlerMethod handlerMethod = (HandlerMethod) handler;
  LoginRequired loginRequired = ReflexUtil.getClazzOrMethodAnnotation(handlerMethod, LoginRequired.class);
  
  if (loginRequired == null) {     // 没有特定注解，不拦截请求
      return true;
  }
  ```

* 基于注解的拦截器的使用

  在 Controller 类中方法上添加 `@LoginRequired` 注解

  ```java
  /**
   * 获取用户信息	
   * @param token
   * @return
  */
  @LoginRequired
  @GetMapping(value = "/getUserInfo")
  public UserInfoVo getUserInfo(@RequestHeader(name = "token") String token){
      UserInfoVo userInfo = userService.getUserInfo(token);
      return userInfo;
  }
  ```



#### RequestContextHolder

​		RequestContextHolder 是 Spring 框架中的类，用于存放应用程序和当前请求的上下文信息。提供了一种方便的方式来访问与当前请求相关的数据，例如用户身份验证信息、主题、本地化等。

​		RequestContextHolder 通过一个静态的 ThreadLocal 变量来存储当前请求的上下文信息。当一个请求开始时，可以通过调用 `RequestContextHolder.currentRequestAttributes()` 方法来获取当前请求的上下文对象。这个上下文对象可以用于访问各种与请求相关的属性和方法。

* 在拦截器中的应用

```java
// 将完成登陆的用户放入 RequestContextHolder 中
RequestContextHolder
    .currentRequestAttributes()
    .setAttribute(AppConstants.RESULT_USER, userInfo, RequestAttributes.SCOPE_REQUEST);

// RequestAttributes 
int SCOPE_REQUEST = 0;		// 表示在当前请求中可用

```

​		RequestAttributes.SCOPE_REQUEST 是 Spring 框架中的一个常量，用于指定请求范围的属性。它表示该属性将在整个 HTTP 请求的生命周期内可用，直到请求被处理完毕。



#### 应用程序的国际化

​		国际化：DispatcherSerlvet 使用 LocaleResolver 根据用户的语言环境自动解析信息，需要 3 种 bean：

* LocaleResolver：解析用户当前语言环境
* MessageSources：解析消息源的信息
* LocaleChangeInterceptor：允许根据请求信息跟换语言环境
* [国际化](F:\Typora-note\国际化.md)
* 常见的异常信息

1. `org.springframework.context.NoSuchMessageException: No message found under code 'error.code.1007' for locale 'zh_CN'.`

   可能的原因：LocaleResolver 无法正确读取 yml 文件中的 message 配置

2. ##### 语言环境拦截器实现

   1. 自定义语言环境注解

   ```java
   @Target({ElementType.METHOD, ElementType.TYPE})
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   public @interface LocaleRequired {
   }
   ```

   2. 拦截器实现，基于 RequestContextHolder 获取 Locale 信息

   ```java
   // LocaleInterceptor
   @Slf4j
   @Component
   public class LocaleInterceptor implements HandlerInterceptor {
   
       @Autowired
       private RequestComponent requestComponent;
   
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
   
           if (!(handler instanceof HandlerMethod)){
               return true;
           }
   
           HandlerMethod method = (HandlerMethod) handler;
           LocaleRequired localeRequired = ReflexUtil.getClazzOrMethodAnnotation(method, LocaleRequired.class);
   
           if (localeRequired == null){
               return true;
           }
   
           Locale locale = null;
   
           String langHeader = requestComponent.getHeader(AppConstants.ACCEPT_LANGUAGE);
   
           log.info("langHeader:" + langHeader+", requestUrl: "+requestComponent.getRequest().getRequestURL());
   
           if (StringUtils.isNullOrEmpty(langHeader)){
               locale = Locale.US;
           }
   
           if ("en_US".equals(langHeader)){
               locale = Locale.US;
           }
   
           if ("zh_CN".equals(langHeader)){
               locale = Locale.CHINA;
           }
   
           RequestContextHolder.getRequestAttributes().setAttribute(AppConstants.LOCALE, locale, RequestAttributes.SCOPE_REQUEST);
   
           return true;
       }
   }
   ```

   根据 i18nKey 获取信息

   ```java
   // AppExceptionHandler
   @ResponseBody
   @ExceptionHandler(LoginException.class)
   public Result loginException(LoginException e){
       log.error("AppExceptionHandler, loginException", e);
   
       Locale locale = (Locale)RequestContextHolder.getRequestAttributes().getAttribute(AppConstants.LOCALE, RequestAttributes.SCOPE_REQUEST);
   
       String message = messageUtils.getMessage(
               AppConstants.ERROR_CODE_PREFIX + e.getErrorCode().getAppCode(),
               locale,
               null);
   
       log.info("异常信息：" + message);
   
       Integer code = e.getErrorCode().getAppCode();
   
       return Result.builder(e, message, code);
   }
   ```

   

#### 整合 log4j2 日志框架

* 关闭部分依赖传递

  ```xml
  // 排除 spring-boot-starter-web 依赖的 spring-boot-starter-logging 发生依赖传递
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <exclusions>
          <exclusion>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-logging</artifactId>
          </exclusion>
      </exclusions>
  </dependency>
  ```


* `spring-boot-starter-logging` 是  `Spring-boot-starter` 的依赖包
* [日志](F:\Typora-note\Spring Boot 3\日志.md)
* 常见异常信息：

<img src="\Git-repository\Typora-note\img\image-20231208171548672.png" alt="image-20231208171548672"  />

​		表示 log4j2 中的依赖包 `log4j-slf4j-impl` 和 `log4j-to-slf4j` 冲突，`log4j-to-slf4j` 属于 `spring-boot-starter-logging`

​		异常解决方案：使用 `<exclusion></exclusion>` 排除所有依赖 `Spring-boot-starter` 的依赖包中的 `spring-boot-starter-logging`



#### 整合第三方的 Mybatis 逆向工程插件

* 将第三方的 jar 包放在 repository 模块中的 lib 文件夹中

* 在 repository 模块中的 POM 文件中引入插件

  ```xml
  <plugin>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-maven-plugin</artifactId>
      <version>1.3.7</version>
      <dependencies>
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>8.0.12</version>
          </dependency>
          <dependency>
              <!-- 第三方依赖 -->
              <groupId>com.github.greatwqs.mybatisplugin</groupId>
              <artifactId>mybatis-generator-plugin</artifactId>
              <version>1.0</version>
              <scope>system</scope>
              <!-- jar 包的相对路径 -->
              <systemPath>${basedir}\libs\mybatis-generator-plugin-1.0.jar</systemPath>
          </dependency>
      </dependencies>
      <configuration>
          <verbose>true</verbose>
          <overwrite>false</overwrite
   		<configurationFile>
              <!-- 逆向工程 XML 的相对路径 -->
              ${basedir}\src\main\resources\generatorConfig.xml
          </configurationFile>
      </configuration>
  </plugin>
  ```

* 使用逆向插件创建 po、mapper、mapper.xml

  ```shell
  mvn -DgTable={tablename} mybatis-generator:generate
  // 执行时除去 {}，将 tablename 更换成目标表名
  ```

* 逆向工程的执行

  在控制台 Terminal 中 `cd` 到 repository 模块，执行上述命令

* 常见异常

![image-20231208180344699](\Git-repository\Typora-note\img\image-20231208180344699.png)

​		异常信息：当前项目和插件组中无法发现前缀为 mybatis-generator 的插件，即在 F:\maven\repository 中无法获取到指定的插件。

![image-20231208180624039](\Git-repository\Typora-note\img\image-20231208180624039.png)

​		异常信息：



#### 整合 Swagger 

* [Swagger UI](http://127.0.0.1:8501/swagger-ui/index.html#/user-controller/login) 接口测试

* 依赖引入

​		由于使用的 SpringBoot 的版本为 SpringBoot 3.0.5，因此直接引入以下的 jar 包无法正常加载 bean，出现的异常信息：

​	Application run failed（应用程序启动失败）

​	UnsatisfiedDependencyException（依赖不足）由于 SpringBoot 升至 3.0 需要的 Swagger 依赖包需要更新。

```xml
<!-- Swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

​		SpringBoot 3.0 整合 Swagger 需要的依赖包

```xml
<!-- Swagger2 - SpringBoot3 -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-api</artifactId>
    <version>2.0.4</version>
</dependency>
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.0.4</version>
</dependency>
```

* 编写配置类

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public OpenAPI springShopOpenApi(){
        return new OpenAPI().info(new Info()
                .title("appdemo 接口文档")
                .description("appdemo 接口文档")
                .version("1.0.0"));
    }
}
```

* 配置文件

```yml
springdoc:
  swagger-ui:
    path: /swagger-ui.html
```



#### 定时任务

​		无需引入额外的依赖，Spring Boot 自带定时任务，需要在启动类上添加注解 `@EbableScheduling` 就能开启定时任务。

​		如果存在定时方法，在方法上添加注解 `@Scheduled` ，将该方法定义为定时方法。 



#### 整合 hibernate-validator

​		在前后端分离的开发模式中，后端必须对前端传入的参数进行校验。但是在 controller 层加上参数验证，会导致代码出现冗余。

​		引用 Hibernate Validator 在实体类进行参数校验，验证失败直接返回错误信息给前端，减少 controller 层的代码量。

- 依赖引入

```xml
<dependency>
    <groupId>org.hibernate.validator</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>8.0.0.Final</version>
</dependency>
```

- 直接在实体类上使用注解规定参数的格式

```java
@Data
public class User extends BaseEntity {

    @NotNull(message = "账号不能为空")
    private String userName;

    @NotNull(message = "密码不能为空")
    private String password;

    @NotNull(message = "用户名不能为空")
    private String name;

    @Length(min = 11, max = 11, message = "输入正确的手机号！")
    private String phone;

    @URL
    private String avatar;

    private String description;

    /**
     * 1：表示可用，0：表示禁用
     */
    @Range(min = 0, max = 1)
    private Integer status;

}
```

* [hibernate-validator 中的注解](F:\Typora-note\hibernate-validator.md)

