---
title: 'Java中的关键字'
description: 'Java关键字学习笔记'
pubDate: 'Sep 18 2026'
tags: ['Java', '基础概念']
---

## Java 中的关键字

### 一、Java 中 final 作用是什么？

`final` 关键字主要有以下三个方面的作用：用于修饰类、方法和变量。

- **修饰类**：当 `final` 修饰一个类时，表示这个类不能被继承，是类继承体系中的最终形态。例如，Java 中的 `String` 类就是用 `final` 修饰的，这保证了 `String` 类的不可变性和安全性，防止其他类通过继承来改变 `String` 类的行为和特性。
- **修饰方法**：用 `final` 修饰的方法不能在子类中被重写。比如，`java.lang.Object` 类中的 `getClass` 方法就是 `final` 的，因为这个方法的行为是由 Java 虚拟机底层实现来保证的，不应该被子类修改。
- **修饰变量**：当 `final` 修饰基本数据类型的变量时，该变量一旦被赋值就不能再改变。例如，`final int num = 10;`，这里的 `num` 就是一个常量，不能再对其进行重新赋值操作，否则会导致编译错误。对于引用数据类型，`final` 修饰意味着这个引用变量不能再指向其他对象，但对象本身的内容是可以改变的。例如，`final StringBuilder sb = new StringBuilder("Hello");`，不能让 `sb` 再指向其他 `StringBuilder` 对象，但可以通过 `sb.append(" World");` 来修改字符串的内容。

### 二、Java 中 static 的作用是什么？

`static` 关键字主要用于修饰类的成员（变量、方法、代码块）和内部类，其核心作用是将成员与类本身关联，而非与类的实例（对象）关联。具体作用如下：

#### 1、修饰变量

被 `static` 修饰的变量属于类本身，而非类的某个实例。所有对象共享同一份静态变量，内存中只存在一份副本。可以通过 `类名.变量名` 直接访问，无需创建对象（也可以通过对象访问，但不推荐）。

通常用于存储所有对象共享的数据，如常量、计数器等。

```java
public class Student {
    // 静态变量（所有学生共享一个学校名称）
    public static String schoolName = "阳光中学";
    // 实例变量（每个学生有自己的姓名）
    private String name;
}

// 访问静态变量
public class Test {
    public static void main(String[] args) {
        System.out.println(Student.schoolName); // 直接通过类名访问
    }
}
```

#### 2、修饰方法

静态方法属于类，不属于任何实例，因此不能直接访问类中的非静态成员（变量 / 方法）（因为非静态成员依赖于对象存在），但可以访问静态成员。通过 `类名.方法名` 直接调用，无需创建对象。

通常用于工具类方法（如 `Math.random()`）、工厂方法等，不需要依赖对象状态即可完成操作。

```java
public class MathUtils {
    // 静态方法（无需创建对象即可调用）
    public static int add(int a, int b) {
        return a + b;
    }
}

// 调用静态方法
public class Test {
    public static void main(String[] args) {
        int result = MathUtils.add(2, 3); // 直接通过类名调用
    }
}
```

#### 3、修饰代码块

静态代码块在类初始化阶段（即执行 `<clinit>` 时）执行，且只执行一次（优于对象构造方法），用于初始化静态变量或执行类级别的预处理操作。JVM 的类生命周期为：加载 → 链接（验证、准备、解析）→ 初始化，静态代码块属于“初始化”阶段而非“加载”阶段。

多个静态代码块按定义顺序执行，且先于非静态代码块和构造方法。

```java
public class Database {
    private static String url;

    // 静态代码块：初始化静态变量
    static {
        url = "jdbc:mysql://localhost:3306/test";
        System.out.println("数据库连接地址初始化完成");
    }
}
```

#### 4、修饰内部类

静态内部类不依赖于外部类的实例，可以独立存在，不能直接访问外部类的非静态成员（需通过外部类实例访问）。

当内部类与外部类的实例无关时，可以使用静态内部类，避免内部类持有外部类的引用导致内存泄漏。

```java
public class OuterClass {
    private static int staticVar = 10;
    private int instanceVar = 20;

    // 静态内部类
    public static class StaticInnerClass {
        public void print() {
            System.out.println(staticVar); // 可访问外部类静态变量
            // System.out.println(instanceVar); // 错误：不能直接访问非静态变量
        }
    }
}

// 使用静态内部类
public class Test {
    public static void main(String[] args) {
        OuterClass.StaticInnerClass inner = new OuterClass.StaticInnerClass();
        inner.print();
    }
}
```

### 三、常量和特殊保留字

`true`、`false` 和 `null` 是 Java 中的字面量，不能作为标识符使用，但严格来说它们不是普通关键字：

- `true`：布尔值真。
- `false`：布尔值假。
- `null`：空引用，不指向任何对象。

`goto` 和 `const` 是 Java 保留字，目前没有实际语法功能，不能作为标识符使用。

### 四、Java 新版本中的上下文关键字

部分单词只在特定语法位置具有特殊含义，称为上下文关键字。它们并不一定像传统关键字一样在所有场景中都被禁止使用。

- `var`：Java 10 引入，用于局部变量类型推断。
- `yield`：Java 14 引入，用于带返回值的 `switch` 表达式。
- `record`：Java 16 正式引入，用于声明主要用于保存数据的不可变类。
- `sealed`、`permits`、`non-sealed`：Java 17 正式引入，用于限制类或接口的继承范围。

```java
var name = "Java";

int result = switch (name) {
    case "Java" -> 1;
    default -> 0;
};

record User(String name, int age) {
}
```

### 五、关键字和标识符的区别

| 对比项 | 关键字 | 标识符 |
| --- | --- | --- |
| 定义 | Java 语言预先规定的特殊单词 | 开发者自定义的名称 |
| 是否可以自定义 | 不可以 | 可以 |
| 是否区分大小写 | 是 | 是 |
| 示例 | `class`、`public`、`return` | `User`、`age`、`getName` |

定义标识符时，除了不能使用关键字和字面量，还应该遵循 Java 的命名规范：类名使用大驼峰命名，方法名和变量名使用小驼峰命名，常量名通常使用全大写字母并用下划线分隔。
