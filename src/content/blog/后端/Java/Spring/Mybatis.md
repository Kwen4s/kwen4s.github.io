---
title: 'Mybatis'
description: 'MyBatis 学习笔记'
pubDate: 'Oct 4 2026'
tags: ['Java', 'MyBatis']
---

## 一、与传统 JDBC 相比，MyBatis 有哪些优点？

MyBatis 基于 JDBC，封装数据库访问中的重复操作，同时保留开发者对 SQL 的控制。

- **减少样板代码**：封装 SQL 执行、参数绑定和结果集处理，减少手动编写 JDBC 代码的工作量。
- **SQL 管理灵活**：支持在 XML 或注解中编写 SQL；使用 XML 可以将 SQL 与 Java 业务代码分离，便于维护和优化。
- **支持动态 SQL 和复用**：通过 `if`、`choose`、`foreach` 等标签按条件生成 SQL，并通过 `sql`、`include` 复用 SQL 片段。
- **支持结果映射**：将查询结果映射为 POJO、Map 等类型，通过 `resultMap`、`association`、`collection` 处理字段映射及一对一、一对多关联。
- **数据库适配范围广**：可使用支持 JDBC 的数据库；具体 SQL 的方言差异仍需开发者处理。
- **便于集成 Spring**：通过 MyBatis-Spring 管理 Mapper、SqlSession，并参与 Spring 事务，提高开发效率。

资源管理取决于使用方式：独立使用 MyBatis 时需要关闭 `SqlSession`，通常使用 try-with-resources；集成 Spring 后，由框架管理会话及相关资源。代码减少量取决于业务，不能统一表述为“减少 50% 以上”。[MyBatis 官方介绍](https://mybatis.org/mybatis-3/)、[入门与会话管理](https://mybatis.org/mybatis-3/getting-started.html)。

## 二、JDBC 连接数据库有哪些步骤？

主要流程是：**获取连接 → 创建语句 → 设置参数 → 执行 SQL → 处理结果 → 释放资源**。

1. **准备驱动**：引入数据库 JDBC 驱动。JDBC 4.0 起，驱动通常通过 SPI 自动加载，一般无需手动调用 `Class.forName()`。
2. **获取连接**：示例可用 `DriverManager`；实际项目通常通过 `DataSource` 和连接池获取。
3. **创建语句并设置参数**：有外部参数时使用 `PreparedStatement`，通过占位符传递数据，避免将用户输入直接拼接到 SQL 中。
4. **执行 SQL**：查询使用 `executeQuery()`；增、删、改通常使用 `executeUpdate()`，返回受影响的行数。
5. **处理结果**：通过 `ResultSet.next()` 遍历结果，并读取字段。
6. **释放资源**：使用 try-with-resources，异常时也能关闭 `ResultSet`、`Statement` 和 `Connection`。

例如：

```java
import java.sql.*;

public class Main {
    public static void main(String[] args) {
        String sql = "SELECT id, username FROM users WHERE id = ?";
        try (Connection connection = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/mydatabase", "username", "password");
             PreparedStatement statement = connection.prepareStatement(sql)) {
            statement.setLong(1, 1L);
            try (ResultSet resultSet = statement.executeQuery()) {
                while (resultSet.next()) {
                    System.out.println(resultSet.getString("username"));
                }
            }
        } catch (SQLException exception) {
            exception.printStackTrace();
        }
    }
}
```

手动管理事务时，还需要通过 `setAutoCommit(false)` 关闭自动提交，成功后 `commit()`，失败后 `rollback()`。连接池获取的连接调用 `close()` 通常是归还连接；在 Spring 管理的事务中，应通过相应的数据访问组件配合管理连接。

## 三、MyBatis 中 `#{}` 和 `${}` 有什么区别？

核心区别是：**`#{}` 是参数绑定，`${}` 是文本替换**。

- **`#{}`**：在默认的 `PreparedStatement` 执行方式下，转换成 `?` 占位符，再通过 JDBC 设置参数。输入作为数据传递，不会因包含 SQL 关键字而变成 SQL 结构，适用于用户名、id、日期等参数值。
- **`${}`**：直接将内容拼入 SQL 文本，输入可能改变 SQL 结构，存在 SQL 注入风险。即使最终仍使用 `PreparedStatement` 执行，也不能消除这部分文本拼接的风险。

参数值优先使用 `#{}`。表名、列名、排序方向等 SQL 结构不能通过 `?` 绑定，应由程序白名单选择或通过动态 SQL 分支生成；使用 `${}` 时必须限制为可信内容。[MyBatis 官方参数说明](https://mybatis.org/mybatis-3/sqlmap-xml.html#Parameters)。

## 四、MyBatis 的执行流程是什么？

MyBatis 的核心是：**把 Mapper 方法调用转换成 SQL 执行，再把结果映射为 Java 对象**。

流程分为两个阶段：

- **启动时**：读取配置和 Mapper 映射，将每条映射语句的信息封装为 `MappedStatement`，统一保存在 `Configuration` 中，再创建 `SqlSessionFactory`。
- **执行时**：获取 `SqlSession` 和 Mapper 代理。调用映射方法时，`MapperProxy` 通过 `MapperMethod` 定位对应的语句并调用 `SqlSession`，交给执行器处理。

一次未命中缓存的普通查询，主要链路是：

```text
Mapper 代理 → SqlSession → Executor → StatementHandler → JDBC
                                        ↓                 ↓
                                 ParameterHandler   ResultSetHandler
                                     设置参数          映射结果
```

核心组件的职责：

- **Executor**：负责执行调度和一级缓存；开启二级缓存支持时，由 `CachingExecutor` 包装，具体缓存使用还取决于 Mapper 与语句配置。
- **StatementHandler**：准备 JDBC Statement，协调参数设置并执行 SQL。
- **ParameterHandler**：设置 SQL 参数，结合 `TypeHandler` 完成 Java 类型到 JDBC 类型的转换。
- **ResultSetHandler**：根据 `resultType` 或 `resultMap` 处理结果集，结合 `TypeHandler` 将字段值转换并映射为对象。

查询命中缓存时，可以直接返回结果，不再执行 JDBC 查询。[MyBatis 官方配置说明](https://mybatis.org/mybatis-3/configuration.html)。

## 五、MyBatis 的一级缓存和二级缓存有什么区别？

**一级缓存属于 SqlSession，二级缓存通常属于 Mapper 的 namespace。**

- **一级缓存**：默认启用，默认作用范围为 `SESSION`。同一 SqlSession 内重复查询，缓存键一致且期间没有触发清理时，可以复用结果。缓存键包含语句标识、SQL、参数、分页信息等。更新、提交、回滚、关闭会话或显式清理等操作会清理本地缓存；配置 `localCacheScope=STATEMENT` 可以将复用范围缩小到单次语句执行。[一级缓存官方说明](https://mybatis.org/mybatis-3/java-api.html#Local_Cache)。
- **二级缓存**：需要全局缓存开关开启，并配置 Mapper 的 `<cache/>`、`@CacheNamespace` 等，才能参与查询。它可以跨 SqlSession 共享结果，缓存内容通常在会话提交后发布；关闭会话时的处理取决于是否需要提交或回滚，回滚的事务不能将其待发布结果当作已提交结果发布。[二级缓存官方说明](https://mybatis.org/mybatis-3/sqlmap-xml.html#cache)。

开启二级缓存后，典型查询顺序为：**二级缓存 → 一级缓存 → 数据库**，前提是该语句允许使用相应缓存。

和 Spring 整合时，Mapper 多次调用不一定使用同一个底层 SqlSession。同一个 Spring 事务内通常复用事务绑定的会话；没有事务时，会话可能在每次调用后关闭。因此，Mapper 对象相同不代表一级缓存一定命中。[MyBatis-Spring 会话管理说明](https://mybatis.org/spring/sqlsession.html)。

二级缓存的一致性范围受关联缓存限制：一个 namespace 的更新通常只清理其关联缓存，其他 Mapper 或外部程序修改同一张表，不一定使该缓存失效。更新频繁或涉及跨表关联的数据，需要谨慎开启；默认二级缓存也不是 Redis 那样的跨进程共享缓存。

## 六、`resultType` 和 `resultMap` 有什么区别？

**`resultType` 指定结果对象类型，`resultMap` 引用显式定义的字段与对象属性映射。**

字段和属性名一致，或通过 SQL 别名、开启下划线转驼峰就能对应时，使用 `resultType` 比较方便。MyBatis 会按自动映射规则填充属性；返回集合时，填写的是集合元素类型。

字段与属性差异较大，或涉及关联对象、集合等复杂结果时，适合使用 `resultMap`：

- **`id`、`result`**：定义字段映射；`id` 还用于标识结果对象，在嵌套结果映射中帮助识别和合并对象。
- **`association`**：映射单个关联对象。
- **`collection`**：映射关联集合，例如用户的多个角色。

`resultMap` 也可以结合自动映射处理未显式配置的字段。同一条查询选择 `resultType` 或 `resultMap` 其中一个，不同时配置。[MyBatis 官方结果映射说明](https://mybatis.org/mybatis-3/sqlmap-xml.html#Result_Maps)。

## 七、MyBatis-Plus 和 MyBatis 有什么区别？

**MyBatis 是 SQL 映射框架，MyBatis-Plus 是基于它的增强工具**，主要减少常规数据库操作的重复代码，仍然可以编写自定义 SQL。[MyBatis-Plus 官方介绍](https://baomidou.com/introduce/)。

| 方面 | MyBatis | MyBatis-Plus |
| --- | --- | --- |
| CRUD | 通常通过 XML、注解或 SQL Provider 定义 SQL | 继承 `BaseMapper<T>` 即可使用常见单表增删改查方法 |
| 条件构造 | 使用动态 SQL 等方式组织条件 | 提供 `QueryWrapper`、`LambdaQueryWrapper` 等条件构造器 |
| 分页 | 可手写分页 SQL 或使用第三方插件 | 提供 `PaginationInnerInterceptor`，配置后实现物理分页 |
| 代码生成 | 可配合 MyBatis Generator 等工具 | 提供代码生成器，按配置生成实体、Mapper、XML 等文件 |
| 实体映射 | 通过结果映射、自动映射等机制配置 | 增加 `@TableName`、`@TableId`、`@TableField` 等实体映射注解 |
| 扩展能力 | 提供插件机制，可自行扩展 | 提供多租户、乐观锁、逻辑删除、字段自动填充等能力 |

分页、多租户等能力需要相应配置，并非引入依赖后自动启用。分页插件从 3.5.9 起拆分，使用时还需按版本引入相应的 SQL 解析依赖。[分页插件官方说明](https://baomidou.com/plugins/pagination/)。

常规单表操作可以使用 MyBatis-Plus 简化；复杂联表、报表或需要精细优化的查询，仍可使用 MyBatis 的自定义 SQL。
