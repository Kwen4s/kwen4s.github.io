---
title: 'Java的Object类'
description: 'Java Object类学习笔记'
pubDate: 'Sep 18 2026'
tags: ['Java', 'Object']
---

## 一、Object 类一共有哪些方法？（请分类简述）

`Object` 类的方法可以按功能分为 4 类：

1. **对象信息获取**
   - `getClass()`：获取运行时的 `Class` 对象，是反射的入口。
   - `toString()`：返回对象的字符串表示，默认格式通常是“类名@哈希值”，实际开发中通常需要重写。
   - `hashCode()`：返回对象的哈希码，主要用于哈希集合。
2. **对象比较**
   - `equals(Object obj)`：判断两个对象是否“逻辑相等”，默认等同于 `==`，实际开发中通常需要根据业务重写。
3. **对象操作与线程控制**
   - `clone()`：创建并返回当前对象的一个副本，需要实现 `Cloneable` 接口。
   - `wait()`、`notify()`、`notifyAll()`：提供线程等待与唤醒机制，通常配合 `synchronized` 使用。
4. **底层与垃圾回收**
   - `finalize()`：垃圾回收前可能调用的清理方法，Java 9 起已标记为废弃，不建议使用。
   - `registerNatives()`：私有方法，用于注册本地（C/C++）方法，通常由 JVM 内部使用。

## 二、`equals()` 和 `==` 的区别？为什么重写 `equals()` 必须重写 `hashCode()`？

### 1. `==` 和 `equals()` 的区别

- **`==`**：比较两个基本数据类型的值；比较两个引用类型时，比较的是对象的内存地址。
- **`equals()`**：`Object` 默认实现也是比较对象地址，但 `String`、`Integer` 等类通常会重写它，用于比较对象的逻辑内容是否相等。

### 2. 为什么重写 `equals()` 必须重写 `hashCode()`？

- **Java 约定**：如果两个对象通过 `equals()` 判断相等，那么它们的 `hashCode()` 必须相同。
- **集合机制**：`HashMap` 和 `HashSet` 判断对象是否存在时，会先比较 `hashCode()`，再比较 `equals()`。
- **常见问题**：如果只重写 `equals()` 而不重写 `hashCode()`，逻辑相等的对象可能拥有不同的哈希值，导致 `HashSet` 出现重复元素，或 `HashMap` 通过 `get()` 找不到之前 `put()` 的值。

### 示例

```java
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

class User {
    private final String name;

    public User(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (!(obj instanceof User)) {
            return false;
        }
        User other = (User) obj;
        return Objects.equals(name, other.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}

public class ObjectDemo {
    public static void main(String[] args) {
        User user1 = new User("Tom");
        User user2 = new User("Tom");

        System.out.println(user1 == user2);       // false：两个不同的对象
        System.out.println(user1.equals(user2)); // true：name 内容相同

        Set<User> users = new HashSet<>();
        users.add(user1);
        System.out.println(users.contains(user2)); // true
    }
}
```

## 三、`wait()`、`sleep()`、`notify()` 的区别？为什么这些方法定义在 `Object` 中而不是 `Thread` 中？

### 1. `wait()` 和 `sleep()` 的区别

| 维度 | `wait()` | `sleep()` |
| --- | --- | --- |
| 所属类 | `Object` | `Thread` |
| 是否释放锁 | 会释放锁，必须在 `synchronized` 同步块或同步方法中使用 | 不会释放锁，线程休眠期间仍然持有锁 |
| 唤醒方式 | 由其他线程调用 `notify()` 或 `notifyAll()` 唤醒 | 休眠时间结束后自动唤醒，也可以被 `interrupt()` 中断 |

### 2. 为什么定义在 `Object` 中？

- Java 中的锁（Monitor）是加在对象上的，而不是加在线程上的。
- `wait()`、`notify()` 和 `notifyAll()` 操作的是对象的锁以及与锁关联的等待集合，因此应该定义在所有对象的根类 `Object` 中。

### 示例

```java
public class WaitNotifyDemo {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            synchronized (LOCK) {
                try {
                    System.out.println("线程开始等待");
                    LOCK.wait(); // 释放 LOCK，进入等待状态
                    System.out.println("线程被唤醒");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
        });

        worker.start();
        Thread.sleep(1000); // 当前线程休眠，但不会释放它持有的其他锁

        synchronized (LOCK) {
            LOCK.notify(); // 唤醒一个正在等待 LOCK 的线程
        }
    }
}
```

调用 `wait()`、`notify()` 或 `notifyAll()` 时，必须持有同一个对象的监视器锁，否则会抛出 `IllegalMonitorStateException`。

## 四、`clone()` 方法的作用？什么是深拷贝与浅拷贝？

- **作用**：用于创建一个对象的副本，不需要通过 `new` 重新创建对象。
- **使用条件**：类需要实现 `Cloneable` 标记接口，并重写 `clone()` 方法（通常将访问修饰符改为 `public`），否则运行时会抛出 `CloneNotSupportedException`。
- **浅拷贝（Shallow Copy）**：复制对象本身，但如果对象内部包含引用类型的属性，只复制该属性的引用地址。修改副本中的引用对象，也会影响原对象。
- **深拷贝（Deep Copy）**：不仅复制对象本身，还会递归复制对象内部所有引用类型属性指向的对象。原对象和副本相互独立，修改其中一个不会影响另一个。
- **常见实现方式**：重写 `clone()` 时同时复制引用类型属性，或者使用序列化/反序列化、JSON 工具类完成转换。

### 示例

```java
class Address implements Cloneable {
    String city;

    public Address(String city) {
        this.city = city;
    }

    @Override
    public Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }
}

class User implements Cloneable {
    String name;
    Address address;

    public User(String name, Address address) {
        this.name = name;
        this.address = address;
    }

    // 浅拷贝：address 仍然指向同一个对象
    public User shallowCopy() throws CloneNotSupportedException {
        return (User) super.clone();
    }

    // 深拷贝：同时复制 address 对象
    public User deepCopy() throws CloneNotSupportedException {
        User copy = (User) super.clone();
        copy.address = address.clone();
        return copy;
    }
}

public class CloneDemo {
    public static void main(String[] args) throws CloneNotSupportedException {
        User original = new User("Tom", new Address("Beijing"));

        User shallow = original.shallowCopy();
        shallow.address.city = "Shanghai";
        System.out.println(original.address.city); // Shanghai：影响原对象

        User deep = original.deepCopy();
        deep.address.city = "Guangzhou";
        System.out.println(original.address.city); // Shanghai：不影响原对象
    }
}
```
