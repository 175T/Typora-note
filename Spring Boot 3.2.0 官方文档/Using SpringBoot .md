# Using Spring Boot

> [Spring Using Spring Boot 官网连接](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using)



## Developer Tools

> [Developer Tools 官网连接](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.devtools)

* Developer Tools 是 Spring Boot 提供的开发者工具包，包含多个程序开发工具，可以简化应用程序开发。`spring-boot-devetools` 依赖的 Maven 坐标

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        
        <!-- 希望 Maven 在构建项目时，不需要强制解析和下载该依赖项，表示该依赖为可选的 -->
        <!-- 构建项目时，如果没有显式地声明该依赖项，则不会将该依赖项包含在项目的构建过程中 -->
        <optional>true</optional>		
        
        <!-- 将依赖的范围设置成只在开发时有效 -->
        <scope>developmentOnly</scope>			
    </dependency>
</dependencies>
```

* 可能出现的问题
  * 类加载（classloading issues）问题，尤其是在 mulit-module-project（多模块项目）中
* 禁用 devTools 
  * 排除依赖
  * 使用系统属性：`-Dspring.devtools.restart.enabled=false` ，该属性用于配置 Spring Boot 应用程序的自动重启功能，属于 Java 命令行参数，可以在 Java 程序运行时使用 `java` 命令给应用程序传递参数。
* 启用 devTools ，开启自动重启功能：`-Dspring.devtools.restart.enabled=true` ，运行 devTools 且存在安全风险的环境中禁用此操作。

### Diagnosing Classloading Issues

> Diagnosing Classloading Issues（诊断类加载问题）

* 

### Automatic Restart

> Automatic Restart（自动重启）

* Spring Boot Application 的 restart（重启）功能由两个 classLoader（类加载器）共同完成
  * *base classLoader* （基本类加载器）
    * 负责加载不发生改变的类，例如第三方的依赖的类
  * *restart classLoader* （重启类加载器）
    * 负责开发人员正在编写的类
* 当应用程序重新启动时，*restart classLoader* 将会被丢弃，重新创建一个 *restart classLoader* ，使用该方法的优势是部分类（第三方提供的）不会在程序重启时被加载，只会加载发生变化的类（正在开发的类）。
* 当 *restart* 消耗的时间不满足开发人员的需要，可以使用 *reloding* 技术，比如 *JRebel* 
  * *JRebel* 是一个用于 Java 应用程序的实时代码热替换（Hot Swapping）工具，可以在不重启应用的情况下完成代码的更新。



## Packaging Your Application for Production

> 打包生产应用环境

* 可执行的 *jar* 包可用于生产环境，由于 *jar* 包是独立存在的，因此非常适合部署在云端
* 想要添加 *production ready* （生产就绪）功能的工具，使用 `spring-boot-actuator` 启动器
  * `Spring-Boot-Actuator` 是一个用于监控和管理 Spring Boot 应用程序的开源工具
  * `Spring-Boot-Actuator` 提供了许多内置端点，可以用于收集有关应用程序的各种信息，例如健康检查、度量、环境变量等。
  * 提供的功能：health,、auditing、metric REST、JMX end-points