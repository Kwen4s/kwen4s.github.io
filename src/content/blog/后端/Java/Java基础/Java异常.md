---
title: 'Java异常'
description: 'Java异常学习笔记'
pubDate: 'Sep 18 2026'
tags: ['Java', '异常']
---

## 一、请介绍一下 Java 的异常体系结构？

Java 异常体系的根类是 `java.lang.Throwable`，它有两个直接子类：`Error` 和 `Exception`。

1. **Error（错误）**
   - **概念**：JVM 级别的严重错误，程序无法处理，也不应该处理。
   - **常见例子**：`OutOfMemoryError`（OOM）、`StackOverflowError`。

2. **Exception（异常）**
   - **概念**：程序级别的错误，程序可以并且应该捕获处理。
   - **分类**：分为 `Checked`（受检异常）和 `Unchecked`（非受检异常/运行时异常）。
     - **Checked Exception**：编译时异常，编译器强制要求必须处理（`try-catch` 或 `throws`），通常是外部资源问题，如 `IOException`、`SQLException`。
     - **Unchecked Exception（RuntimeException）**：运行时异常，编译器不强制要求处理，通常是代码逻辑错误，如 `NullPointerException`、`IndexOutOfBoundsException`、`IllegalArgumentException`。

## 二、`throw` 和 `throws` 的区别是什么？

| 维度 | `throw` | `throws` |
| --- | --- | --- |
| 位置 | 方法体内部 | 方法签名末尾 |
| 动作 | 抛出一个具体的异常实例 | 声明方法可能抛出的异常类型 |
| 数量 | 一次只能抛出一个异常对象 | 可以声明多个异常类型，使用逗号分隔 |
| 语义 | “我这里发生异常了，我把它抛出去” | “这个方法可能抛出异常，调用者请注意” |

### 示例

```java
import java.io.IOException;

public class ExceptionDemo {
    // throws：声明方法可能抛出 IOException
    public static void readFile(String path) throws IOException {
        if (path == null || path.isBlank()) {
            // throw：抛出一个具体的异常对象
            throw new IOException("文件路径不能为空");
        }

        System.out.println("读取文件：" + path);
    }

    public static void main(String[] args) {
        try {
            readFile(null);
        } catch (IOException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

## 三、`try-catch-finally` 的执行顺序和常见注意点是什么？

### 1. 执行顺序

- 没有发生异常：`try` → `finally`。
- 发生异常且被捕获：`try`（执行到异常处中断）→ `catch` → `finally`。
- 发生异常但没有对应的 `catch`：`try`（执行到异常处中断）→ `finally`，然后继续向上抛出异常。

### 2. `finally` 一定会执行吗？

`finally` 中的代码通常都会执行，常用于释放资源、关闭连接等清理操作。但在以下情况下可能不会执行：

- 调用了 `System.exit()`，直接终止了 JVM。
- JVM 发生崩溃或进程被强制终止。
- 程序在进入 `try` 或 `finally` 之前就异常退出。

### 3. `try` 和 `finally` 都有 `return` 时，最终返回什么？

- `finally` 中的 `return` 会覆盖 `try` 或 `catch` 中的 `return`，因此不建议在 `finally` 中使用 `return`。
- **基本类型**：`try` 中的返回值会先计算并暂存，执行完 `finally` 后再返回；如果 `finally` 中也有 `return`，则以 `finally` 的返回值为准。
- **引用类型**：如果 `finally` 修改了返回对象的属性，修改结果会保留；如果只是让局部引用变量指向新对象，通常不会改变已经计算好的返回对象。

## 四、日常开发中，异常处理有哪些最佳实践？

1. **不要捕获过于宽泛的异常**：避免直接捕获 `Exception` 或 `Throwable`。捕获范围过大会掩盖真正的 Bug（例如空指针异常），应该捕获具体的异常类型。
2. **不要吞掉异常**：不要使用空的 `catch`，也不要只调用 `e.printStackTrace()`。至少应该记录完整的异常信息（如使用 `log.error`），必要时继续向上抛出。
3. **不要用异常控制业务流程**：异常的创建和处理成本较高，性能也较差。业务条件应该使用 `if-else` 判断，而不是依靠异常进行流程跳转。
4. **尽早抛出，延迟捕获（Fail-Fast）**：在方法入口处校验参数，不合法时立即抛出 `IllegalArgumentException`；在业务层或应用层统一捕获异常。
5. **统一异常处理**：在 Spring Boot 中，可以使用 `@ControllerAdvice` 和 `@ExceptionHandler` 进行全局异常拦截，统一返回包含 `code`、`message` 等字段的 JSON，避免将 Java 堆栈信息直接返回给前端。

## 五、如何设计一个自定义异常？

1. **继承选择**：通常继承 `RuntimeException`，避免 Checked 异常在各层之间反复传递；如果是底层基础组件，也可以根据场景继承 `Exception`。
2. **命名规范**：类名以 `Exception` 结尾，并准确表达异常原因，例如 `OrderNotFoundException`、`InsufficientBalanceException`，做到见名知意。
3. **构造方法**：至少提供以下构造方法：
   - 无参构造方法。
   - 带 `message` 参数的构造方法，用于传递错误提示。
   - 同时带 `message` 和 `cause`（`Throwable`）的构造方法，用于保留原始异常堆栈信息，这对排查问题非常重要。
4. **错误码设计**：在大型项目中，自定义异常通常会结合错误码枚举使用。异常对象中同时包含 `code`（错误码）和 `message`（错误信息），方便统一处理和返回。
