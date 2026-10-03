---
title: 'Spring框架'
description: 'Spring 框架学习笔记'
pubDate: 'Oct 3 2026'
tags: ['Java', 'Spring']
---

## 一、Spring 框架有哪些核心特性？

- **IoC 容器**：通过控制反转管理 Bean 的创建、依赖注入和生命周期。开发者定义对象及其依赖，由容器负责创建和组装。
- **AOP**：将日志、事务等横切关注点从业务逻辑中分离，通过切面集中处理，提高代码的复用性和可维护性。
- **事务管理**：提供统一的事务抽象，支持声明式事务和编程式事务，减少业务代码对具体事务 API 的依赖。
- **Spring MVC**：基于 Servlet API 的 Web 框架，支持请求映射、参数绑定、视图渲染以及 JSON 等响应数据的输出。

## 二、介绍一下 Spring IoC 和 AOP

### 1. IoC：控制反转

将对象的创建、依赖组装和生命周期管理交给容器，减少业务代码对具体实现的依赖。**依赖注入（DI）是实现 IoC 的一种方式**，容器通过构造方法、属性等方式提供对象所需的依赖。

IoC 并不意味着程序中不能使用 `new`，而是由容器负责管理需要作为 Bean 使用的对象及其依赖关系。

### 2. AOP：面向切面编程

将日志、事务等多个业务模块共同需要的逻辑集中到切面中，在指定方法执行前后进行增强，减少重复代码。

Spring AOP 基于代理，主要有两种方式：

- **JDK 动态代理**：基于接口创建代理。
- **CGLIB 代理**：通过生成目标类的子类实现代理，不能增强 `final` 方法和 `private` 方法，也不能继承 `final` 类。

通常在未强制类代理时，有接口使用 JDK 动态代理，没有接口使用 CGLIB。Spring Boot 的 AOP 自动配置默认使用 CGLIB；设置 `spring.aop.proxy-target-class=false` 后，有合适接口的目标可以使用 JDK 动态代理。[代理机制](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)、[Spring Boot AOP 配置](https://docs.spring.io/spring-boot/reference/features/aop.html)。

IoC 负责对象及依赖管理，AOP 负责方法增强，两者配合可以让业务逻辑与公共功能分离。需要注意，同一个对象内部通过 `this` 调用方法通常绕过代理，不会触发对应的 Spring AOP 增强。

## 三、介绍一下 Spring AOP

Spring AOP 将日志、事务、权限检查等横切关注点集中到切面中，再通过代理应用到业务方法，减少重复代码，让业务逻辑与公共功能分别维护。

### 1. 核心概念

| 概念 | 含义 |
| --- | --- |
| Aspect（切面） | 封装横切关注点，通常包含切点和通知，可用 `@Aspect` 声明 |
| Join Point（连接点） | 可以进行增强的位置；Spring AOP 中是方法执行 |
| Pointcut（切点） | 用于筛选需要增强的连接点 |
| Advice（通知） | 在匹配的连接点上执行的增强逻辑 |
| Target Object（目标对象） | 被代理、被增强的对象 |
| AOP Proxy（代理对象） | 拦截方法调用并执行通知，再按流程调用目标方法 |
| Introduction（引入） | 为代理对象增加目标类原本没有实现的接口及其实现 |
| Weaving（织入） | 将切面应用到目标对象的过程；Spring AOP 通过运行时代理实现 |

Aspect 是切面的概念，AspectJ 则是独立的 AOP 框架。Spring AOP 可以使用 AspectJ 的注解和切点表达式，但不等于使用了 AspectJ 的字节码织入。

### 2. 五种通知

- **前置通知 `@Before`**：在目标方法执行前运行。
- **返回通知 `@AfterReturning`**：在目标方法正常返回后运行。
- **异常通知 `@AfterThrowing`**：在目标方法抛出匹配的异常后运行。
- **后置通知 `@After`**：无论正常返回还是抛出异常，都在方法退出后运行，类似 `finally`。
- **环绕通知 `@Around`**：包围方法执行，可以控制是否调用目标方法，并处理参数、返回值和异常。

## 四、IoC 和 AOP 通过什么机制实现？

### 1. IoC 实现机制

- **读取 Bean 定义**：解析配置类、注解或 XML 等，将对象的类型、依赖、作用域等信息记录为 `BeanDefinition`。
- **创建与依赖注入**：根据定义，通过构造方法、工厂方法等创建 Bean，再解析并注入依赖。常规运行模式下会用到反射，但不能将 IoC 简单等同于反射。
- **生命周期管理**：通过初始化回调、`BeanPostProcessor` 等机制处理 Bean，保存需要复用的实例，并在适当时机执行销毁回调。
- **容器实现**：`BeanFactory` 提供基础 Bean 管理功能；`ApplicationContext` 在此基础上提供事件、资源访问、国际化等能力。

依赖注入是实现 IoC 的主要方式，而不是 IoC 概念本身。

### 2. AOP 实现机制

Spring AOP 主要基于运行时代理：容器中的自动代理创建器属于 `BeanPostProcessor`，它为需要增强的 Bean 创建代理。方法调用经过代理时，根据切点匹配结果执行通知组成的拦截器链，再按通知逻辑调用目标方法。

- **JDK 动态代理**：通过 `Proxy` 和 `InvocationHandler` 等机制实现，代理暴露接口。
- **CGLIB 代理**：生成目标类的子类，通过方法拦截实现增强，也可以用于已经实现接口的类。

通常业务代码取得并调用的是代理对象。绕过代理直接调用目标对象，或通过 `this` 内部调用方法，就不会经过相应的 Spring AOP 拦截器链。

## 五、什么是依赖注入？如何实现？

依赖注入（DI）是由外部提供对象需要的依赖，而不是由对象内部自行创建。Spring 容器根据 Bean 定义及注入点查找依赖并进行组装，降低代码对具体实现的耦合，便于测试。

下面的示例假设依赖对象均已注册为 Bean，且能够唯一匹配。

### 1. 构造器注入

通过构造方法传入依赖，适合必需依赖，支持 `final` 字段，便于独立测试。通常优先选择这种方式。

```java
@Service
public class UserService {
    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

Spring 4.3 起，类只有一个构造器时，可以省略构造器上的 `@Autowired`。依赖引用在构造时已传入，但不意味着所有依赖 Bean 的生命周期处理都已经结束。

### 2. Setter 注入

通过 Setter 方法设置依赖，适合可选依赖或需要后续调整的依赖。默认的 `@Autowired` 仍要求依赖存在，并不会自动变成可选注入。

```java
@Service
public class PaymentService {
    private PaymentGateway gateway;

    @Autowired
    public void setGateway(PaymentGateway gateway) {
        this.gateway = gateway;
    }
}
```

### 3. 字段注入

容器直接为标注 `@Autowired` 的字段赋值，代码简短，但依赖不体现在构造器中，不便于脱离容器测试，也不适合用 `final` 声明必需依赖。

```java
@Service
public class OrderService {
    @Autowired
    private OrderRepository orderRepository;
}
```

存在多个候选 Bean 时，可以用 `@Qualifier` 指定依赖，或用 `@Primary` 设置优先候选。

## 六、AOP 在 Spring 中有哪些应用？

- **事务管理**：声明式事务通常通过 AOP 代理实现。`@Transactional` 标记的方法经过代理调用时，Spring 根据传播行为开启或加入事务，并按执行结果和回滚规则处理事务；并非所有异常都会默认回滚。
- **日志记录**：集中记录方法参数、返回结果和异常，避免业务方法重复编写日志。可以用 `@Before` 记录入参、`@AfterReturning` 记录结果，或用 `@Around` 统计耗时。
- **权限校验**：在业务方法执行前检查调用者权限，不满足条件时阻止执行。可以通过自定义切面实现，实际项目也常使用 Spring Security 的方法级授权。

AOP 通过切点选择需要增强的方法，再通过通知执行公共逻辑，使业务代码更集中。这些增强需要调用经过代理，内部 `this` 调用等绕过代理的情况不会生效。

## 七、Spring AOP 的原理是什么？动态代理是什么？

动态代理是在运行时生成代理类并创建代理对象，通过代理拦截方法调用，在不修改目标类源码的情况下加入增强逻辑。Spring AOP 主要通过这种方式实现。

### 1. JDK 动态代理

通过 `java.lang.reflect.Proxy` 生成实现指定接口的代理类，代理对象关联一个 `InvocationHandler`。方法调用被转发到处理器的 `invoke()`，由它执行增强逻辑及目标方法调用。

**实现 `InvocationHandler` 的是调用处理器，不是生成的代理类。** 在 Spring AOP 中，JDK 代理要求目标提供合适的接口，调用者通过代理暴露的接口访问方法。

### 2. CGLIB 代理

通过生成目标类的子类，并拦截可重写的方法实现增强，不要求目标类实现接口。有接口的类也可以使用 CGLIB。

由于依赖继承，不能代理 `final` 类，也不能增强 `final`、`private` 等无法被重写的方法。

### 3. Spring AOP 调用流程

容器为需要增强的 Bean 创建代理；调用代理方法时，根据切点选择适用通知，组成拦截器链，再按通知逻辑执行目标方法、处理返回值或异常。

代理类型由接口情况及配置共同决定，不能仅凭“是否实现接口”判断。Spring Boot 的 AOP 自动配置默认使用类代理；直接调用目标对象或通过 `this` 内部调用，会绕过代理增强。

## 八、什么是反射？有哪些使用场景？

反射允许程序在运行时获取类的信息，并动态创建对象、调用方法或读写字段，即使编译时不知道具体类型，也可以进行相应操作。

主要能力包括：

- **获取类信息**：通过 `Class` 查看构造方法、字段、方法、接口及注解等。
- **创建对象**：通过 `Constructor.newInstance()` 调用构造方法；通常使用 `clazz.getDeclaredConstructor().newInstance()`，而不是已废弃的 `Class.newInstance()`。
- **调用方法**：通过 `Method.invoke()` 执行方法。
- **读写字段**：通过 `Field.get()`、`Field.set()` 访问或设置字段。

常见场景有 Spring 的 Bean 创建和依赖注入、ORM 的对象映射、序列化与反序列化，以及根据配置加载插件或调用指定方法。

反射不是无条件访问任意成员。访问私有成员可能需要调整访问检查，而且仍受 Java 模块封装等限制；也不能据此任意修改所有 `final` 字段。使用时应注意类型检查、异常处理和性能开销。

## 九、Spring 如何解决循环依赖？

循环依赖是 Bean 的依赖关系形成闭环，例如 A 依赖 B，B 又依赖 A。Spring 可以在允许循环引用时，通过**提前暴露对象引用和三级缓存**处理部分单例 Bean 的 Setter 或字段注入循环依赖。

### 1. 三级缓存

| 缓存 | 内容 |
| --- | --- |
| `singletonObjects`（一级） | 已完成初始化的单例 Bean，可能是代理对象 |
| `earlySingletonObjects`（二级） | 已获取的早期引用，可能是原始对象或早期代理 |
| `singletonFactories`（三级） | 提供早期引用的 `ObjectFactory`，按需生成引用并协调 AOP 代理 |

### 2. 处理流程

1. 实例化 A，将提供 A 早期引用的工厂放入三级缓存。
2. 为 A 注入 B，开始创建 B。
3. B 需要 A，从三级缓存工厂取得 A 的早期引用，并将其放入二级缓存。
4. B 完成初始化进入一级缓存，再注入 A。
5. A 完成初始化进入一级缓存，移除对应的二、三级缓存项。

早期引用允许 B 持有尚未完全初始化的 A；涉及 AOP 时，工厂可以通过后置处理器提供早期代理，使注入引用与最终暴露的引用一致。[单例缓存源码](https://github.com/spring-projects/spring-framework/blob/main/spring-beans/src/main/java/org/springframework/beans/factory/support/DefaultSingletonBeanRegistry.java)。

### 3. 适用范围与处理建议

- **纯构造器循环依赖**：双方都需要先构造对方，无法通过这种提前暴露机制解决。
- **原型 Bean 的循环依赖**：不使用这套单例缓存机制，通常会报错。
- **单例 Setter 或字段循环依赖**：允许循环引用且对象能够正确提前暴露时，可以解决，并非所有代理或初始化组合都支持。

Spring Boot 从 2.6 起默认禁止循环引用，`spring.main.allow-circular-references` 默认是 `false`。设为 `true` 只允许容器尝试处理，不保证所有循环依赖都能解决。[Spring Boot 配置](https://docs.spring.io/spring-boot/appendix/application-properties/)。

实际开发优先拆分职责或抽取公共组件，消除依赖闭环；必要时可通过 `@Lazy` 或 `ObjectProvider` 延迟获取依赖，但不要在构造或初始化期间立即触发对方创建。

## 十、Spring 常用注解有哪些？

| 注解 | 作用 |
| --- | --- |
| `@Autowired` | 声明依赖注入点，主要按类型匹配 Bean，可用于构造器、方法和字段 |
| `@Component` | 标记通用组件，由组件扫描发现并注册为 Bean |
| `@Service` | `@Component` 的特化，标记业务层组件 |
| `@Repository` | `@Component` 的特化，标记数据访问层组件，也可作为持久化异常翻译的识别标记 |
| `@Controller` | `@Component` 的特化，标记 MVC 控制器 |
| `@Configuration` | 声明配置类，可配合 `@Bean` 定义 Bean |
| `@Bean` | 标记 Bean 工厂方法，将其返回的对象交给容器管理 |

### 1. 组件与自动注入

```java
@Service
public class MyService {
}

@Controller
public class MyController {
    @Autowired
    private MyService myService;
}
```

需要启用相应包的组件扫描，容器才会发现这些组件。`@Autowired` 注入的是容器提供的依赖，不等于每次都执行 `new`；实际开发通常优先使用构造器注入。

`@Repository` 的异常翻译需要相应后置处理器和异常翻译器支持，并非只加注解就能转换所有异常。`@Controller` 方法若要直接返回响应数据，可以使用 `@ResponseBody`，或用组合注解 `@RestController`。

### 2. 配置类与 Bean 方法

```java
@Configuration
public class MyConfiguration {
    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}
```

配置类需要被容器注册，`myBean()` 才会作为 Bean 工厂方法处理。Bean 默认名称是方法名，也可以通过 `@Bean` 指定；这种方式适合注册第三方类或需要自定义创建逻辑的对象。

## 十一、Spring 的事务什么情况下会失效？

要区分**事务拦截没有生效**与**事务生效但没有按预期回滚**。在常见的代理式声明事务中，主要有以下情况：

| 情况 | 原因与处理 |
| --- | --- |
| 同类内部 `this` 调用 | 绕过代理，被调用方法的事务配置不生效；可以将事务方法拆到另一个 Bean，通过代理调用 |
| 对象不由容器管理 | 自行 `new` 的普通对象没有事务代理；使用容器中的 Bean，并确保启用事务管理 |
| 方法无法被代理拦截 | `private` 方法不能通过代理增强，CGLIB 也不能增强 `final` 方法；优先使用可代理的公开业务方法 |
| 异常被捕获后吞掉 | 代理可能看不到异常而正常提交；按业务需要重新抛出异常，或显式标记回滚 |
| 回滚规则不匹配 | 默认规则对 `RuntimeException`、`Error` 回滚，受检异常默认不回滚；可按需设置 `rollbackFor`，并检查全局规则及 `noRollbackFor` |
| 传播行为不符合预期 | 如 `NOT_SUPPORTED` 会暂停事务，`SUPPORTS` 在无事务时不会新建事务；根据业务选择传播行为 |
| 事务管理器与资源不匹配 | 多数据源时，选择的管理器可能未覆盖实际操作的数据源；明确指定正确的事务管理器，跨资源原子性需要相应协调方案 |
| 新线程执行数据库操作 | 普通线程绑定的事务不会自动传到新线程；在新线程中独立建立所需事务 |

如果内部调用者已有事务，被调用方法仍可能在该事务内运行，只是其自身的传播等配置未经过代理处理。异常被捕获时，也可能因事务早已被标记为 rollback-only 而最终回滚，不能断言一定提交。

方法可见性还取决于版本和代理方式：Spring 6 起，类代理默认也支持 `protected` 和包可见事务方法；接口代理要求方法是公开的并定义在代理接口中，不能概括为“所有非 public 方法都失效”。[事务注解规则](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)。

## 十二、Bean 是否是单例？

Spring Bean 默认作用域是 **singleton**：在同一个容器中，同一个 Bean 定义通常对应一个共享实例，后续获取时复用。它不是整个 JVM 中某个类只能有一个对象，不同容器或不同 Bean 定义可以创建不同实例。

也可以用 `@Scope("prototype")` 设置为原型作用域，容器每次获取该 Bean 时创建新实例。这里的“每次获取”不等于每次 HTTP 请求；原型 Bean 直接注入单例 Bean 时，通常只在注入时创建一次，之后仍使用同一个引用。

**单例不代表线程安全。** 多个线程共享有可变状态的单例 Bean 时，需要避免共享状态，或通过锁、原子操作等保证并发安全。无状态或不可变的 Bean 通常适合单例，不需要因为不可变而改为原型。

## 十三、Bean 的单例和非单例生命周期是否一样？

不完全一样。以 **singleton 和 prototype** 为例，容器都会负责创建、依赖注入和初始化，但 prototype 交给调用者后，容器不会自动执行其销毁回调。

| 阶段 | singleton | prototype |
| --- | --- | --- |
| 创建时机 | `ApplicationContext` 通常在启动时创建非懒加载单例；懒加载单例按需创建 | 每次向容器获取时创建新实例 |
| 初始化 | 执行依赖注入、相关 Aware 回调、后置处理器及初始化回调 | 每个新实例也执行相应的创建和初始化流程 |
| 使用 | 同一 Bean 定义的实例共享复用 | 创建后交给调用者，使用方式由调用者决定 |
| 销毁 | 正常关闭容器时，执行已配置的销毁回调 | 不自动执行销毁回调，需要调用者负责资源清理 |

销毁回调可以包括 `@PreDestroy`、`DisposableBean.destroy()` 或自定义销毁方法。释放文件、连接等资源与 JVM 回收对象内存是两回事；prototype 对象不可达后，仍可由 GC 回收。

“非单例”还包括 request、session 等作用域，不能都视为 prototype。例如它们可以在请求或会话结束时执行销毁回调，用户会话数据通常应考虑 session 作用域。

## 十四、Spring Bean 的作用域有哪些？

作用域决定 Bean 实例在哪个范围内复用，以及何时创建和结束其生命周期。

| 作用域 | 实例共享范围 | 常见用途 |
| --- | --- | --- |
| `singleton` | 同一容器中，同一 Bean 定义共享一个实例，默认作用域 | 无状态服务等 |
| `prototype` | 每次向容器获取时创建新实例 | 需要独立状态的临时对象 |
| `request` | 同一个 HTTP 请求内共享一个实例 | 请求相关的数据和组件 |
| `session` | 同一个 HTTP 会话内共享一个实例 | 用户会话相关状态 |
| `application` | 同一个 `ServletContext` 内共享一个实例 | Web 应用级共享组件 |
| `websocket` | 同一个 WebSocket 会话内共享一个实例 | WebSocket 会话相关状态 |

`request`、`session` 和 `application` 需要支持相应作用域的 Web 容器上下文；`websocket` 需要相应的 WebSocket 作用域支持。Bean 通常在所属范围内首次需要时创建，不代表每个请求或会话都会无条件创建所有 Bean。

`singleton` 的范围是 Spring 容器，`application` 的范围是 `ServletContext`；同一个 Web 应用可以存在多个 Spring 容器，因此两者不完全相同。会话等范围内共享的有状态 Bean 也可能被并发访问，需要考虑线程安全。

可以通过 `@Scope` 等方式指定作用域，也可以实现并注册 `Scope` 接口来扩展自定义作用域。

## 十五、如何在 Bean 初始化或销毁前后执行自定义逻辑？

可以使用生命周期回调处理单个 Bean，或使用后置处理器统一处理多个 Bean。

| 方式 | 执行时机与用途 |
| --- | --- |
| `@PostConstruct` | 依赖注入完成后执行初始化逻辑 |
| `InitializingBean.afterPropertiesSet()` | 执行初始化回调，但会使业务类依赖 Spring 接口 |
| `@Bean(initMethod = "...")` | 指定自定义初始化方法，适合第三方类 |
| `BeanPostProcessor` | 通过 `postProcessBeforeInitialization()`、`postProcessAfterInitialization()` 在初始化回调前后统一处理 Bean，也可返回包装后的对象 |
| `@PreDestroy` | 容器销毁 Bean 前执行清理逻辑 |
| `DisposableBean.destroy()` | 执行销毁回调 |
| `@Bean(destroyMethod = "...")` | 指定自定义资源清理方法 |
| `DestructionAwareBeanPostProcessor` | 在销毁回调前统一处理 Bean |

同时配置多种初始化回调时，通常按 `@PostConstruct` → `afterPropertiesSet()` → 自定义初始化方法执行；销毁回调通常按 `@PreDestroy` → `destroy()` → 自定义销毁方法执行。

这里的“初始化前”通常已经完成实例化和依赖注入。如果需要介入实例化或属性填充阶段，可以使用 `InstantiationAwareBeanPostProcessor` 等扩展点。

Spring 没有与初始化后处理完全对称的通用“销毁后”回调；需要在某个清理动作完成后执行逻辑，可以放在自定义销毁方法中。prototype Bean 的销毁回调不会由容器自动调用，进程被强制终止时也不能保证执行清理。
