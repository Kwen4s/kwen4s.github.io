---
title: 'SpringBoot'
description: 'Spring Boot 学习笔记'
pubDate: 'Oct 4 2026'
tags: ['Java', 'Spring', 'Spring Boot']
---

## 一、Spring Boot 比直接使用 Spring 方便在哪里？

Spring Boot 基于 Spring，主要简化项目搭建、配置和运行，并不是替代 Spring。

- **自动配置**：根据依赖、已有 Bean 和配置属性等条件提供默认配置，减少手动配置；开发者仍可以自定义或覆盖配置。
- **Starter 依赖**：通过一组配套依赖快速集成 Web、数据访问等功能，并结合依赖管理减少版本搭配的工作。
- **内嵌服务器**：Web 应用可以使用内嵌服务器，打包为可执行 JAR 后直接运行，减少外部服务器部署步骤；通常通过相应 Starter 引入所需服务器，而不是默认同时启用多种服务器。

核心优势是约定优于配置，让开发者更快搭建可运行的 Spring 应用，将精力集中在业务逻辑上。

## 二、如何理解 Spring Boot 的约定优于配置？

约定优于配置，是指框架提供合理的默认行为，常见场景直接遵循约定即可，只在需求不同时显式配置，减少重复配置工作。

- **自动配置**：根据依赖、配置属性和已有 Bean 等条件配置组件，例如在满足条件时配置 Spring MVC 和内嵌服务器。
- **默认行为**：提供常用的日志、Web 等默认设置，开发者可以通过配置属性修改。数据库连接等应用特定信息仍可能需要自行提供。
- **包结构约定**：通常将标有 `@SpringBootApplication` 的启动类放在项目根包，让默认组件扫描覆盖其子包。`controller`、`service` 等包名只是组织习惯，并非框架强制要求。

约定不是固定限制：可以修改配置、提供自定义 Bean 或排除不需要的自动配置，在保留默认便利性的同时满足具体业务需求。

## 三、Spring Boot 自动配置的原理是什么？

### 1. `@SpringBootApplication` 的核心组成

- **`@SpringBootConfiguration`**：基于 `@Configuration`，标识 Spring Boot 配置类。
- **`@EnableAutoConfiguration`**：启用自动配置，通过导入选择器引入符合条件的自动配置类。
- **`@ComponentScan`**：扫描应用组件，默认从启动类所在包向下扫描；它与自动配置候选类的加载不是同一机制。

此外，`@Target(TYPE)` 表示可以标在类等类型声明上，`@Retention(RUNTIME)` 表示运行时保留，`@Documented` 用于生成文档，`@Inherited` 表示类级注解可沿父类继承。

### 2. 自动配置流程

1. `@EnableAutoConfiguration` 导入 `AutoConfigurationImportSelector`，读取自动配置候选类名单。
2. 处理排除项、排序等信息，并结合条件判断决定哪些配置生效。
3. 通过 `@ConditionalOnClass`、`@ConditionalOnMissingBean`、`@ConditionalOnProperty` 等条件，判断依赖类、已有 Bean 和配置属性是否满足要求。
4. 满足条件的自动配置注册相应 Bean；开发者提供自定义 Bean 时，使用缺失 Bean 条件的默认配置可以退让。

现代 Spring Boot 通过 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` 声明候选类；较早版本使用 `spring.factories` 中的 `EnableAutoConfiguration` 条目，理解源码时需要区分版本。

自动配置并不是把所有依赖中的类都扫描成 Bean，而是**导入候选配置，再按条件装配**。[自动配置官方说明](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html)。

## 四、常见的 Starter 有哪些？

Starter 是一组配套依赖的集合，用于快速集成某类功能，相关组件再由自动配置按条件装配。以下是 Spring Boot 2.x、3.x 中常见的名称，使用时应匹配项目版本。

| Starter | 主要用途 |
| --- | --- |
| `spring-boot-starter-web` | 构建 Servlet Web 应用，包含 Spring MVC，默认使用内嵌 Tomcat |
| `spring-boot-starter-security` | 集成 Spring Security，实现认证、授权等安全功能 |
| `mybatis-spring-boot-starter` | 由 MyBatis 团队提供，集成 MyBatis、SqlSessionFactory 和 Mapper 等组件 |
| `spring-boot-starter-data-jpa` | 集成 Spring Data JPA，默认使用 Hibernate，适合 ORM 数据访问 |
| `spring-boot-starter-jdbc` | 提供 Spring JDBC 和连接池支持，适合直接使用 JDBC、JdbcTemplate |
| `spring-boot-starter-data-redis` | 集成 Spring Data Redis，默认使用 Lettuce 客户端 |
| `spring-boot-starter-test` | 提供 Spring Test、JUnit Jupiter、AssertJ 等测试支持，通常用于测试范围 |

数据库 Starter 通常不包含目标数据库的驱动。例如连接 MySQL，还需引入 `mysql-connector-j` 并配置连接信息；Redis 也需要提供服务器连接配置。

引入 Starter 不代表所有功能立即可用，还需要满足自动配置条件，并完成必要的业务配置。

## 五、Spring Boot 中有哪些重要注解？配置相关的注解有哪些？

Spring Boot 应用中常见的注解包括 Boot 专有注解和 Spring Framework 提供的注解。

| 注解 | 作用 |
| --- | --- |
| `@SpringBootApplication` | 组合配置类声明、自动配置和组件扫描，通常标在启动类上 |
| `@Controller` | 标记 MVC 控制器 |
| `@RestController` | 组合 `@Controller` 与 `@ResponseBody`，将方法返回值写入响应，常用于返回 JSON |
| `@Component`、`@Service`、`@Repository` | 标记通用组件、业务层组件和数据访问层组件 |
| `@Autowired` | 声明依赖注入点 |
| `@Value` | 注入配置属性值，也支持 SpEL 表达式 |
| `@RequestMapping` | 根据路径、请求方法等条件映射请求 |
| `@GetMapping`、`@PostMapping`、`@PutMapping`、`@DeleteMapping` | 分别映射 GET、POST、PUT、DELETE 请求 |

配置相关的注解主要有：

- **`@Configuration`**：声明配置类，通常配合 `@Bean` 注册和配置对象。
- **`@ConfigurationProperties`**：将一组具有相同前缀的配置属性绑定到对象，适合结构化配置；需通过扫描、`@EnableConfigurationProperties` 等方式注册。
- **`@Bean`**：声明 Bean 工厂方法，将返回的对象交给容器管理。

如果问“标记配置类的注解”，回答 `@Configuration`；如果问“批量绑定配置属性的注解”，回答 `@ConfigurationProperties`。

## 六、Spring Boot 怎么开启事务？

配置好数据源和事务管理器后，在 Spring 管理的服务类方法上添加 `@Transactional`，即可使用声明式事务。满足自动配置条件时，Spring Boot 会自动启用事务管理，通常无需手动添加 `@EnableTransactionManagement`。

例如，定义保存用户的服务接口：

```java
public interface UserService {
    void saveUser(User user);
}
```

在实现类的方法上声明事务：

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    @Transactional(rollbackFor = Exception.class)
    public void saveUser(User user) {
        userRepository.save(user);
    }
}
```

事务使用时需要关注：

- **传播行为**：默认是 `REQUIRED`，有事务就加入，没有事务才新建；事务在所属边界结束时提交或回滚。
- **回滚规则**：默认对 `RuntimeException` 和 `Error` 回滚；示例中的 `rollbackFor = Exception.class` 将受检异常也纳入回滚范围。
- **代理调用**：默认代理模式下，应通过容器中的代理对象调用方法；同类内部的 `this` 调用不会触发被调用方法上的事务增强。
- **异常处理**：异常被捕获后没有继续抛出，也没有标记回滚时，事务可能正常提交。

以上代理调用与注解使用规则见 [Spring 官方事务说明](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)。

## 七、Spring Boot 的过滤器和拦截器有什么区别？执行顺序怎样？

在 Servlet Web 应用中，Filter 作用于 Servlet 过滤链，Interceptor 作用于 Spring MVC 的处理器执行链。

### 1. 过滤器与拦截器的区别

| 特性 | 过滤器（Filter） | 拦截器（HandlerInterceptor） |
| --- | --- | --- |
| 所属体系 | Servlet 规范；Boot 2.x 使用 `javax.servlet.Filter`，3.x 起使用 `jakarta.servlet.Filter` | Spring MVC 的 `HandlerInterceptor` |
| 作用范围 | 按 URL、Servlet 和分派类型匹配，可覆盖静态资源及非 MVC 请求 | 按 HandlerMapping 和路径配置匹配 MVC 处理器，也可作用于 MVC 静态资源处理器 |
| 执行位置 | 包裹 Servlet 调用，可在放行前后处理请求与响应 | DispatcherServlet 找到处理器后，在处理器执行前后回调 |
| 主要能力 | 放行或阻断请求，包装请求、响应对象 | 获取处理器信息，进行预处理和完成回调 |
| 依赖注入 | 由 Spring 创建的 Filter 支持注入 | 由 Spring 创建的拦截器支持注入 |
| 常见用途 | 编码、请求日志、跨域、认证等 | 处理器相关的校验、模型数据补充、耗时统计等 |

### 2. 执行顺序（洋葱模型）

假设两个 Filter 和两个 Interceptor 都匹配请求，且同步处理正常完成：

```text
Filter 1：chain.doFilter 前
  Filter 2：chain.doFilter 前
    DispatcherServlet
      Interceptor 1：preHandle
      Interceptor 2：preHandle
      Controller
      Interceptor 2：postHandle
      Interceptor 1：postHandle
      视图渲染（如有）
      Interceptor 2：afterCompletion
      Interceptor 1：afterCompletion
  Filter 2：chain.doFilter 后
Filter 1：chain.doFilter 后
```

Filter 通过 `chain.doFilter()` 继续执行链；Interceptor 通过 `preHandle()` 返回 `true` 放行。进入按顺序执行，返回阶段按逆序执行。

- Filter 可以通过 `FilterRegistrationBean#setOrder()`，或自动注册 Filter Bean 的 `@Order`、`Ordered` 设置顺序，数值越小越先执行。
- Interceptor 可以通过 `InterceptorRegistration#order()` 设置顺序；相同顺序值时按添加顺序执行。
- Filter 的普通后置代码在异常时可能被跳过，必须执行的清理逻辑应放在 `finally` 中。

### 3. Filter 如何注入 Spring Bean？

**关键是 Filter 实例由谁创建。** Servlet 容器独立创建的 Filter 不会自动获得 Spring 依赖注入；Spring 创建的 Filter 可以使用构造器注入或 `@Autowired`。

在 Spring Boot 嵌入式 Servlet 容器中，将 Filter 注册为 Spring Bean（`@Component` 或 `@Bean`）即可自动注册到过滤链；需要指定 URL、顺序等时，使用 `FilterRegistrationBean` 配置同一个实例。避免同时通过 `@WebFilter` 扫描注册另一个实例。也可以使用 `DelegatingFilterProxy` 将过滤工作委托给 Spring Bean。[Spring Boot 官方注册说明](https://docs.spring.io/spring-boot/how-to/webserver.html#howto.webserver.add-servlet-filter-listener)。

拦截器同样需要使用 Spring 创建的实例，再通过 `WebMvcConfigurer#addInterceptors()` 注册；直接 `new` 出来的实例不会自动注入依赖。

### 4. 拦截器的三个回调有什么区别？

| 回调 | 时机与用途 |
| --- | --- |
| `preHandle` | 处理器执行前调用；返回 `true` 继续，返回 `false` 中止后续执行，由当前拦截器负责响应 |
| `postHandle` | 处理器成功执行后、视图渲染前调用；可补充 `ModelAndView`，处理器抛出异常时通常不会调用 |
| `afterCompletion` | 请求处理完成后调用，用于清理资源；仅对 `preHandle` 已成功返回 `true` 的拦截器执行，按逆序回调 |

如果 Interceptor 2 的 `preHandle` 返回 `false`，Controller 和所有 `postHandle` 都不会执行，只回调 Interceptor 1 的 `afterCompletion`。异步请求的回调时机有所不同，需要结合 `AsyncHandlerInterceptor` 理解。[Spring 拦截器官方说明](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)。

对于 `@ResponseBody`，响应体通常在 HandlerAdapter 返回前已写出，修改响应数据更适合使用 `ResponseBodyAdvice`；认证授权优先交给 Spring Security 的过滤链处理。
