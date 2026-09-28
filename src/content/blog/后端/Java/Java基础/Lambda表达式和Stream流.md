---
title: 'Lambda表达式和Stream流'
description: 'Java Lambda表达式和Stream流学习笔记'
pubDate: 'Sep 28 2026'
tags: ['Java', 'Lambda', 'Stream']
---

## 一、什么是 Lambda 表达式？它的本质是什么？

- **概念**：Lambda 表达式是一种匿名函数，可以把函数作为参数传递给方法，也可以把代码本身当作数据进行处理，让 Java 具备了函数式编程的能力。
- **语法**：`(parameters) -> expression` 或 `(parameters) -> { statements; }`。
- **使用前提**：Lambda 表达式通常赋值给函数式接口。函数式接口只能有一个抽象方法，例如 `Runnable`、`Comparator` 和自定义的 `@FunctionalInterface` 接口。

### Lambda 表达式的本质

- Lambda 表达式可以看作匿名内部类的一种简洁写法，但它并不是简单地直接生成一个匿名内部类的 `.class` 文件。
- **编译阶段**：编译器通常将 Lambda 转换为 `invokedynamic` 指令（Java 7 引入），而不是像匿名内部类那样直接生成独立的类文件。
- **运行阶段**：JVM 通过 `LambdaMetafactory` 动态生成函数式接口的实现。对于无状态、未捕获外部变量的 Lambda，JVM 还可能复用同一个实例。

### 示例

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

public class LambdaDemo {
    public static void main(String[] args) {
        Calculator calculator = (a, b) -> a + b;
        System.out.println(calculator.add(2, 3)); // 5
    }
}
```

## 二、什么是函数式接口？

函数式接口（Functional Interface）是**有且只有一个抽象方法**的接口，也称为 SAM（Single Abstract Method）接口。Lambda 表达式必须依赖函数式接口才能使用，因为接口中的唯一抽象方法就是 Lambda 表达式需要实现的目标方法。

- 接口中可以有多个 `default` 方法和 `static` 方法，它们不属于抽象方法，不影响函数式接口的定义。
- 从父接口继承来的 `Object` 类方法（如 `equals()`、`toString()`）不计入抽象方法数量。
- `@FunctionalInterface` 不是必须的，但建议使用。它可以让编译器检查接口是否确实只包含一个抽象方法。
- Java 常用的函数式接口包括 `Runnable`、`Comparator<T>`、`Consumer<T>`、`Function<T, R>`、`Predicate<T>` 和 `Supplier<T>`。

### 示例

```java
@FunctionalInterface
interface Printer {
    void print(String message);

    default void printDefault() {
        print("default message");
    }

    static void printStatic(String message) {
        System.out.println(message);
    }
}

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        Printer printer = message -> System.out.println(message);
        printer.print("Hello, Lambda");
        printer.printDefault();
        Printer.printStatic("Hello, static method");
    }
}
```

## 三、Lambda 表达式中引用外部变量有什么限制？为什么？

### 1. 先记住结论

Lambda 表达式可以使用方法中的局部变量，但这个局部变量必须满足以下条件之一：

- 使用 `final` 修饰；
- 没有使用 `final`，但初始化后从未被重新赋值，也就是“实际上是 final”（effectively final）。

简单来说：**Lambda 可以读取外部局部变量，但不能修改它，也不能在 Lambda 创建后再给它重新赋值。**

### 2. 为什么有这个限制？

Lambda 可能不会马上执行，例如把它交给另一个线程，或者保存起来稍后执行。方法执行结束后，局部变量已经离开了原来的栈空间。

因此，Java 在创建 Lambda 时会保存局部变量当前的值，而不是保存这个局部变量本身。如果允许外部代码继续修改变量，就会产生“Lambda 使用旧值还是新值”的问题，也会增加并发场景下的不确定性。

### 3. 示例

```java
import java.util.function.IntUnaryOperator;

public class LambdaVariableDemo {
    public static void main(String[] args) {
        int base = 10;
        IntUnaryOperator operator = number -> number + base;

        System.out.println(operator.applyAsInt(5)); // 15

        // base = 20; // 编译错误：base 被 Lambda 使用后不能重新赋值
    }
}
```

下面的写法也会编译失败，因为 `count++` 修改了局部变量：

```java
int count = 0;
// numbers.forEach(number -> count++); // 编译错误
```

如果确实需要在 Lambda 中修改共享状态，可以使用线程安全的对象，例如 `AtomicInteger`，但业务代码中应优先考虑使用返回值、`map`、`reduce` 等方式处理数据：

```java
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger count = new AtomicInteger();
numbers.forEach(number -> count.incrementAndGet());
```

## 四、Stream 流是什么？它和集合（Collection）有什么本质区别？

Stream 是 Java 8 引入的用于处理集合数据的抽象概念，可以对数据进行过滤、映射、排序、聚合等操作。它关注的是“如何处理数据”，而集合关注的是“如何存储数据”。

### Stream 和集合的三个本质区别

1. **Stream 不负责存储数据**
   - 集合是内存中的数据结构，负责保存数据。
   - Stream 不保存数据，而是作为数据的计算管道，从集合、数组等数据源读取数据并进行处理。
2. **Stream 通常不会直接修改源数据**
   - `filter`、`map` 等操作会返回新的 Stream，终端操作会生成新的结果，通常不会改变底层集合。
   - 如果确实需要修改集合，应明确使用集合自身的修改方法，避免在 Stream 中产生副作用。
3. **Stream 具有惰性执行和一次性消费的特点**
   - `filter`、`map` 等中间操作不会立即执行，只有遇到 `collect`、`forEach`、`count` 等终端操作时，整个流水线才会开始计算。
   - Stream 类似一条流水线，执行一次终端操作后就不能再次使用，否则会抛出 `IllegalStateException`。

### 示例

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class StreamDemo {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Tom", "Jerry", "Bob");

        // 中间操作只是在构建处理流程，此时还没有真正执行
        List<String> result = names.stream()
                .filter(name -> name.length() > 3)
                .map(String::toUpperCase)
                .collect(Collectors.toList()); // 终端操作，开始执行

        System.out.println(names);  // [Tom, Jerry, Bob]，源集合没有被修改
        System.out.println(result); // [JERRY]
    }
}
```

## 五、Stream 的常用操作

Stream 的操作主要分为两类：**中间操作**和**终端操作**。

### 1. 常用中间操作

中间操作会返回一个新的 Stream，可以连续调用，并且具有惰性，只有遇到终端操作时才会真正执行。

- **`filter`**：按照条件过滤数据。
- **`map`**：将每个元素转换成另一种形式。
- **`flatMap`**：将多个 Stream 合并成一个 Stream，常用于处理嵌套集合。
- **`distinct`**：去除重复元素，依赖元素的 `equals()` 和 `hashCode()`。
- **`sorted`**：对元素排序，可以使用自然顺序或自定义比较器。
- **`limit`**：截取前几个元素。
- **`skip`**：跳过前几个元素。
- **`peek`**：查看处理过程中的元素，主要用于调试，不建议用它修改数据。

### 2. 常用终端操作

终端操作会触发 Stream 执行，并产生最终结果或副作用。一个 Stream 只能执行一次终端操作。

- **`forEach`**：遍历元素并执行指定操作。
- **`collect`**：将元素收集成集合、Map 或其他结果。
- **`count`**：统计元素数量。
- **`anyMatch`**：判断是否至少有一个元素满足条件。
- **`allMatch`**：判断是否所有元素都满足条件。
- **`noneMatch`**：判断是否没有元素满足条件。
- **`findFirst`**：获取第一个元素，返回 `Optional`。
- **`findAny`**：获取任意一个元素，适合并行流场景。
- **`reduce`**：将多个元素聚合成一个结果。
- **`min`、`max`**：获取最小值或最大值，返回 `Optional`。

### 3. 综合示例

```java
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class StreamOperationDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5, 2);

        // filter：过滤偶数；distinct：去重；sorted：排序；collect：收集结果
        List<Integer> evenNumbers = numbers.stream()
                .filter(number -> number % 2 == 0)
                .distinct()
                .sorted()
                .collect(Collectors.toList());
        System.out.println(evenNumbers); // [2, 4]

        long count = numbers.stream()
                .filter(number -> number > 2)
                .count();
        System.out.println(count); // 3

        boolean containsFive = numbers.stream()
                .anyMatch(number -> number == 5);
        System.out.println(containsFive); // true

        int sum = numbers.stream()
                .reduce(0, Integer::sum);
        System.out.println(sum); // 16

        Optional<Integer> first = numbers.stream().findFirst();
        first.ifPresent(value -> System.out.println("第一个元素：" + value));
    }
}
```

## 六、Stream 的 `map` 和 `flatMap` 有什么区别？

### 1. `map`：一对一映射

`map(Function<T, R>)` 会把 Stream 中的每个元素 `T` 转换成一个新的元素 `R`。输入有多少个元素，处理后通常仍然有多少个元素。

例如，将 `List<String>` 中的每个字符串转换成大写，或者将 `List<User>` 转换成 `List<String>`，只提取用户名。

```java
List<String> names = Arrays.asList("Tom", "Jerry", "Bob");

List<Integer> lengths = names.stream()
        .map(String::length)
        .collect(Collectors.toList());

System.out.println(lengths); // [3, 5, 3]
```

### 2. `flatMap`：一对多映射并扁平化

`flatMap(Function<T, Stream<R>>)` 会把每个元素 `T` 转换成一个 Stream，再将所有生成的小 Stream 合并成一个大 Stream。

它适合处理嵌套集合、数组或字符串拆分等场景。例如，一句话可以拆成多个单词，多句话最终合并成一个单词流。

```java
List<String> sentences = Arrays.asList("Hello World", "Java Stream");

List<String> words = sentences.stream()
        .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
        .collect(Collectors.toList());

System.out.println(words); // [Hello, World, Java, Stream]
```

### 3. 直观对比

| 操作 | 转换关系 | 结果特点 | 常见场景 |
| --- | --- | --- | --- |
| `map` | 一个元素转换成一个元素 | 保持一层结构 | 字符串转大写、对象转字段 |
| `flatMap` | 一个元素转换成多个元素，再合并 | 拍平嵌套结构 | 嵌套集合、拆分句子、合并多个 Stream |

如果使用 `map` 将每个句子转换成一个单词 Stream，结果会是 `Stream<Stream<String>>`；使用 `flatMap` 后，结果会直接变成 `Stream<String>`。

## 七、什么是并行流？

并行流（`parallelStream()`）是 Java 8 提供的一种并发处理数据的方式。它会将数据拆分成多个部分，交给多个线程同时处理，最后合并处理结果。

### 1. 普通流和并行流的区别

| 类型 | 创建方式 | 执行方式 |
| --- | --- | --- |
| 普通流 | `list.stream()` | 通常由一个线程按顺序处理 |
| 并行流 | `list.parallelStream()` | 拆分数据后由多个线程并行处理 |

并行流默认使用 `ForkJoinPool.commonPool()` 中的线程执行任务。

### 2. 示例

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

        long evenCount = numbers.parallelStream()
                .filter(number -> number % 2 == 0)
                .count();

        System.out.println(evenCount); // 3
    }
}
```

### 3. 使用注意事项

- 数据量较小时，并行处理的线程调度成本可能高于收益，不一定比普通流更快。
- 每个元素的处理逻辑应尽量相互独立，避免多个线程同时修改共享变量。
- `forEach()` 不保证执行顺序；如果需要保持顺序，可以使用 `forEachOrdered()`，但可能会损失部分并行性能。
- 并行流更适合数据量较大、计算密集且任务之间相互独立的场景。
