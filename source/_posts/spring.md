---
title: spring
date: 2025-02-24
categories:
  - 记录
tags:
  - Spring
  - Spring Boot
  - Java
---

# Spring Framework

## IOC

IoC 是 Inversion of Control 的简写，译为“控制反转”，它不是一门技术，而是一种设计思想，是一个重要的面向对象编程法则，能够指导我们如何设计出松耦合、更优良的程序。

Spring 通过 IoC 容器来管理所有 Java 对象的实例化和初始化，控制对象与对象之间的依赖关系。我们将由 IoC 容器管理的 Java 对象称为 Spring Bean，它与使用关键字 new 创建的 Java 对象没有任何区别。Bean管理说的是：Bean对象的创建，以及Bean对象中属性的赋值（或者叫做Bean对象之间关系的维护）。

IoC 容器是 Spring 框架中最重要的核心组件之一，它贯穿了 Spring 从诞生到成长的整个过程。

- 控制反转是一种思想。

- 控制反转是为了降低程序耦合度，提高程序扩展力。

- 控制反转，反转的是什么？

  - 将对象的创建权利交出去，交给第三方容器负责。
  - 将对象和对象之间关系的维护权交出去，交给第三方容器负责。

- 控制反转这种思想如何实现呢？
  - DI（Dependency Injection）：依赖注入

### 依赖注入

依赖注入常见的实现方式包括两种：

- 构造器注入
- Setter 注入

### Spring IoC 容器

Spring 的 IoC 容器就是 IoC思想的一个落地的产品实现。IoC容器中管理的组件也叫做 bean。在创建 bean 之前，首先需要创建IoC 容器。Spring 提供了IoC 容器的两种实现方式：

- BeanFactory
- ApplicationContext

BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身，是 Spring 框架的基础接口，不提供完整的 IoC 功能，但是 Spring 框架中的其他组件都是以 BeanFactory 为基础构建的。

ApplicationContext 是 BeanFactory 的子接口，提供了更多的高级特性，它是 Spring 框架的一个更高级的容器，它除了提供 IoC 容器的基本功能外，还提供了事件传播、国际化信息绑定等功能。

ApplicationContext的实现类有：

- ClassPathXmlApplicationContext：从类路径下加载配置文件
- FileSystemXmlApplicationContext：从文件系统中加载配置文件
- XmlWebApplicationContext：在 Web 应用中加载配置文件
- AnnotationConfigApplicationContext：基于注解的配置类加载容器

### Bean 的生命周期

Spring 容器管理 Bean 的生命周期，Spring 容器负责创建 Bean 实例，Spring 容器负责 Bean 的初始化和销毁。

Bean 的生命周期包括以下阶段：

- 实例化 Bean：容器根据配置文件中的信息创建 Bean 实例。
- 设置对象属性：容器在创建 Bean 的时候，会通过 set 方法设置 Bean 的属性值。
- Bean 的后置处理器：容器会调用 Bean 的后置处理器方法。（postProcessBeforeInitialization）
- Bean 的初始化：容器会调用 Bean 的初始化方法。
- Bean 的后置处理器：容器会再次调用 Bean 的后置处理器方法。（postProcessAfterInitialization）
- Bean 的销毁：容器会调用 Bean 的销毁方法。

### Bean 的作用域

Bean 的作用域是指 Bean 实例的作用范围，Spring 容器支持以下几种作用域：

- singleton：单例模式，一个 Bean 容器只有一个实例。
- prototype：原型模式，每次从容器中获取 Bean 时，都会创建一个新的实例。
- request：每次 HTTP 请求都会创建一个新的 Bean 实例。
- session：每次 HTTP Session 都会创建一个新的 Bean 实例。
- global-session：全局 Session 作用域，仅在基于 Servlet 的 Web 应用中有效。
- application：全局作用域，整个应用中只有一个实例。
- websocket：WebSocket 作用域，仅在基于 WebSocket 的 Web 应用中有效。
- custom：自定义作用域。

## AOP

AOP（Aspect Oriented Programming）是一种设计思想，是软件设计领域中的面向切面编程，它是面向对象编程的一种补充和完善，它以通过预编译方式和运行期动态代理方式实现，在不修改源代码的情况下，给程序动态统一添加额外功能的一种技术。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### AOP 的术语

- Aspect：切面，是一个类，它包含了一些方法，这些方法可以在切入点之前或之后执行。
- Joinpoint：连接点，是程序执行过程中的一个点，比如方法的调用或异常的处理。
- Pointcut：切入点，是一组连接点的集合，可以通过表达式来描述。
- Advice：通知，是切面在连接点执行的动作。
- Introduction：引介，是一种特殊的通知，它向现有的类添加新方法和属性。
- Target：目标对象，是被一个或多个切面所通知的对象。
- Weaving：织入，是把切面连接到目标对象并创建新的代理对象的过程。
- Proxy：代理，是一个对象，它通过连接到目标对象的方式间接控制对目标对象的访问。

### AOP 的实现方式

- 代理对象有接口：JDK 动态代理
- 代理对象没有接口：CGLIB 动态代理

## Resources

Spring用于访问资源的接口是 `org.springframework.core.io.Resource`，它是一个接口，Spring 提供了多种实现类，用于访问不同的资源。

### 实现类

- `UrlResource`：用于访问 URL 资源。
- `ClassPathResource`：用于访问类路径下的资源。
- `FileSystemResource`：用于访问文件系统资源。
- `ServletContextResource`：用于访问 ServletContext 资源。
- `InputStreamResource`：用于访问输入流资源。
- `ByteArrayResource`：用于访问字节数组资源。

### 加载资源

Spring 提供了 `ResourceLoader` 接口，用于加载资源。该接口实现类的实例可以获得一个Resource实例。`ResourceLoaderAware`接口是一个回调接口，用于设置ResourceLoader实例。如果 Bean 实例需要访问资源，有如下两种解决方案：

- 获取 `ResourceLoader` 实例，然后调用 `getResource()` 方法。
- 使用依赖注入

对于第一种方式，当程序获取 Resource 实例时，总需要提供 Resource 所在的位置，不管通过 FileSystemResource 创建实例，还是通过 ClassPathResource 创建实例，或者通过 ApplicationContext 的 getResource() 方法获取实例，都需要提供资源位置。这意味着：资源所在的物理位置将被耦合到代码中，如果资源位置发生改变，则必须改写程序。因此，通常建议采用第二种方法，让 Spring 为 Bean 实例依赖注入资源。

## Validation

在开发中，我们经常遇到参数校验的需求，比如用户注册的时候，要校验用户名不能为空、用户名长度不超过20个字符、手机号是合法的手机号格式等等。如果使用普通方式，我们会把校验的代码和真正的业务处理逻辑耦合在一起，而且如果未来要新增一种校验逻辑也需要在修改多个地方。而spring validation允许通过注解的方式来定义对象校验规则，把校验和业务逻辑分离开，让代码编写更加方便。Spring Validation其实就是对Hibernate Validator进一步的封装，方便在Spring中使用。一般有多种校验方式：

- 基于Validator接口的校验
- 基于注解的校验
- 基于自定义注解的校验

# Spring Data

## Transactions

Spring框架中的事务是通过AOP来实现的，Spring事务管理的底层实现是通过对DataSource和JDBC的封装来实现的。

### 基于注解的事务管理

Spring 通过 `@Transactional` 注解来实现事务管理，`@Transactional` 注解可以加在类上，也可以加在方法上。

### 事务属性

`@Transactional` 注解有以下几个属性：

- `propagation`：事务的传播行为
  - `Propagation.REQUIRED`：如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。
  - `Propagation.SUPPORTS`：支持当前事务，如果当前没有事务，就以非事务方式执行。
  - `Propagation.MANDATORY`：使用当前的事务，如果当前没有事务，就抛出异常。
  - `Propagation.REQUIRES_NEW`：新建事务，如果当前存在事务，把当前事务挂起。
  - `Propagation.NOT_SUPPORTED`：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
  - `Propagation.NEVER`：以非事务方式执行，如果当前存在事务，则抛出异常。
  - `Propagation.NESTED`：如果当前存在事务，则在嵌套事务内执行，如果当前没有事务，则新建一个事务。
- `isolation`：事务的隔离级别
- `timeout`：事务的超时时间
- `readOnly`：事务是否只读
- `rollbackFor`：发生哪些异常回滚
- `noRollbackFor`：发生哪些异常不回滚

# Spring Boot

Spring Boot 是 Spring 的一个项目，它简化了 Spring 应用的初始搭建，提供了一些默认配置，可以快速地开发 Spring 应用。约定大于配置，Spring Boot 通过自动配置和起步依赖简化了项目的配置。

## Spring Boot Starter

Spring Boot Starter 是 Spring Boot 的一个重要特性，它是一个依赖描述符，用于简化 Maven 或 Gradle 的配置。Spring Boot Starter 可以让你不需要关心 Spring Boot 的版本，只需要引入 Starter，Spring Boot 会自动配置项目。将功能场景提取出来，打包成一个 Starter，然后在项目中引入 Starter，就可以使用这个功能场景。

## Spring Boot AutoConfiguration

### 版本

spring boot父项目spring-boot-starter-parent的父项目spring-boot-dependencies中定义了大量的依赖版本，这些版本是经过测试的，可以放心使用。如果我们在项目中引入了spring-boot-dependencies，就可以不用再指定版本号了。

### 注解

- `@SpringBootApplication`：Spring Boot 应用的入口，它是一个组合注解，包含了 `@Configuration`、`@EnableAutoConfiguration` 和 `@ComponentScan` 注解。
- `@EnableAutoConfiguration`：启用自动配置，Spring Boot 会根据项目中的依赖自动配置项目。
- `@Configuration`：配置类，相当于 Spring 中的 XML 配置文件。
- `@AutoConfigurationPackage`：自动配置包，用于自动配置类的扫描。将主配置类（`@SpringBootApplication` 注解的类）所在包及其子包下的所有类都注册到 Spring 容器中。
- `@Import`：导入其他配置类。
- `@SpringBootConfiguration`：Spring Boot 的配置类，它是 `@Configuration` 的派生注解。
- `@ConfigurationProperties`：绑定配置文件中的属性值。
- `@Value`：获取配置文件中的属性值。
- `@PropertySource`：加载指定的配置文件。
- `@ImportResource`：导入 Spring 的 XML 配置文件。

### 自动配置原理

spring boot启动时找到`@SpringBootApplication`注解的类，然后开启`@EnableAutoConfiguration`。其中在`@EnableAutoConfiguration`注解中，有一个`@Import`注解，这个注解导入了`AutoConfigurationImportSelector`类，这个类中有一个`selectImports`方法，这个方法会根据`META-INF/spring.factories`文件中的配置，获取`EnableAutoConfiguration`注解的值，然后根据这个值，导入对应的自动配置类。自动配置类中有`@EnableConfigurationProperties`注解，这个注解会导入`ConfigurationPropertiesBindingPostProcessor`类，这个类会将`@ConfigurationProperties`注解的类绑定到配置文件中的属性。自动配置类中还有很多`@ConditionalOn`注解，这些注解会根据条件来判断是否需要导入这个自动配置类。自动配置类中有Properties类，这个类中有很多属性，这些属性会从配置文件中读取，然后根据这些属性来配置项目。

### 配置加载位置

Spring Boot 会按照以下顺序加载配置文件，优先级从高到低：

- 当前目录下的 `config` 目录
- 当前目录
- classpath 下的 `config` 目录
- classpath 根目录
- `@PropertySource` 注解指定的位置
- `spring.config.location` 环境变量指定的位置

### 外部配置加载顺序

Spring Boot 会按照以下顺序加载外部配置，优先级从高到低：

- 命令行参数
- 来自 `java:comp/env` 的 JNDI 属性
- Java 系统属性（System.getProperties()）
- 操作系统环境变量
- `RandomValuePropertySource` 中的属性
- `application.properties` 或 `application.yml` 文件中的属性
- `@PropertySource` 注解指定的属性
- 默认属性
- `SpringApplication.setDefaultProperties` 指定的属性

### bootstrap和application

Spring Boot 有两个配置文件，一个是 `bootstrap.properties` 或 `bootstrap.yml`，另一个是 `application.properties` 或 `application.yml`。`bootstrap` 配置文件用于 Spring Boot 应用的引导阶段，它是 Spring Cloud 的配置文件，用于配置应用的上下文信息。`application` 配置文件用于 Spring Boot 应用的运行阶段，它是 Spring Boot 的配置文件，用于配置应用的运行信息。

加载顺序：

- `bootstrap.properties` 或 `bootstrap.yml`
- `bootstrap-{profile}.properties` 或 `bootstrap-{profile}.yml`
- `application.properties` 或 `application.yml`
- `application-{profile}.properties` 或 `application-{profile}.yml`

## 运行流程

Spring Boot 应用的运行流程如下：

1. 创建 Spring Boot 应用。
   - 保存主配置类
   - 判断是否为 Web 应用
   - 获取所有的 `ApplicationContextInitializer`
   - 获取所有的 `ApplicationListener`
   - 从多个配置类中找到主配置类
2. 调用 `SpringApplication.run()` 方法
   - 获取`SpringApplicationRunListeners`，并调用`starting`方法
   - 准备环境，创建环境后调用监听器`environmentPrepared`方法
   - 创建应用上下文（IOC容器）
   - 准备上下文，调用Initializer的初始化方法，调用监听器`contextPrepared`方法，最后调用监听器的`contextLoaded`方法
3. 刷新应用上下文
   - 刷新应用上下文，IOC容器初始化
   - 回调事件，调用监听器的`started`方法
4. 运行应用
   - callRunners方法，调用所有的`CommandLineRunner`和`ApplicationRunner`的run方法
   - 回调事件ready，调用监听器的`ready`方法

## 日志

日志由日志门面（抽象层）和日志实现构成，Spring Boot 默认使用 SLF4J 作为日志门面，Logback 作为日志实现，并且引入了其他日志框架的适配器，可以使用其他日志框架。在导入依赖时，需排除默认的非 SLF4J 日志框架。