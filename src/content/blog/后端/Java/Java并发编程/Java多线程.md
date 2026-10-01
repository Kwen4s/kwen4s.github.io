---
title: 'Java多线程'
description: 'Java 多线程与并发编程学习笔记'
pubDate: 'Sep 29 2026'
tags: ['Java', '并发', '多线程']
---

## 一、介绍一下 Java 内存模型（JMM）

JMM 是 **Java Memory Model** 的缩写，中文叫 Java 内存模型。它不是 JVM 内存区域的划分，也不是某一块真实存在的物理内存，而是一套关于多线程如何访问共享变量的规范。

JMM 主要解决三个问题：

- **可见性**：一个线程修改共享变量后，其他线程能否及时看到最新值。
- **原子性**：一个操作是否会被其他线程打断，能否作为一个不可分割的整体完成。
- **有序性**：编译器和处理器进行指令重排序后，程序是否仍然符合预期。

### 1. JMM 中的主内存和工作内存

为了描述线程之间如何交换数据，JMM 抽象出了“主内存”和“工作内存”：

```text
                    主内存
             保存共享变量的最新值
                /           \
               /             \
       线程 A 工作内存       线程 B 工作内存
       保存变量的副本         保存变量的副本
```

- **主内存**：可以理解为所有线程共享的变量存储区域。
- **工作内存**：每个线程私有的内存区域，用来保存共享变量的副本、寄存器值或其他中间数据。

线程不能直接操作其他线程的工作内存。一个线程修改共享变量时，可能先修改自己的副本；什么时候把修改刷新到主内存，以及其他线程什么时候读取到最新值，就需要由 JMM 规定的同步机制保证。

这里的主内存和工作内存是 JMM 的抽象概念，不要简单地把它们等同于 JVM 的堆和线程栈。JVM 实际运行时可能使用 CPU 缓存、寄存器和编译器优化来实现这些规则。

### 2. 可见性

下面的代码中，线程 `main` 修改了 `running`，但工作线程不一定能及时看到这个修改：

```java
public class VisibilityDemo {
    private static boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) {
                // 执行任务
            }
            System.out.println("线程结束");
        });

        worker.start();
        Thread.sleep(100);
        running = false;
    }
}
```

可以使用 `volatile` 保证对 `running` 的修改对其他线程可见：

```java
public class VisibilityDemo {
    private static volatile boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread worker = new Thread(() -> {
            while (running) {
                // 执行任务
            }
            System.out.println("线程结束");
        });

        worker.start();
        Thread.sleep(100);
        running = false;
    }
}
```

当一个线程写入 `volatile` 变量后，其他线程读取该变量时能够看到最新值。`volatile` 还会限制相关指令重排序，但它不能保证复合操作的原子性。

### 3. 原子性

下面的 `count++` 看起来是一条语句，实际至少包含“读取、加一、写回”三个步骤：

```java
private static volatile int count;

count++; // 不是原子操作
```

即使 `count` 使用了 `volatile`，多个线程同时执行 `count++` 时仍然可能发生数据覆盖：

```text
线程 A 读取 count = 0
线程 B 读取 count = 0
线程 A 计算并写入 1
线程 B 计算并写入 1
最终结果是 1，而不是 2
```

如果需要保证自增操作的原子性，可以使用 `AtomicInteger`：

```java
import java.util.concurrent.atomic.AtomicInteger;

AtomicInteger count = new AtomicInteger();
count.incrementAndGet();
```

也可以使用 `synchronized` 或 `Lock` 保护整个读、改、写过程：

```java
private static int count;

public static synchronized void increment() {
    count++;
}
```

### 4. 有序性

为了提高性能，编译器和处理器可能对没有依赖关系的指令进行重排序。单线程环境下，重排序通常不会改变最终结果；但在多线程环境下，如果没有同步机制，一个线程观察到的执行顺序可能与代码书写顺序不同。

`volatile`、锁和其他并发工具能够建立必要的内存屏障，限制不安全的重排序，让线程按照规定的可见性和顺序观察共享变量。

### 5. happens-before 规则

JMM 使用 **happens-before** 规则描述操作之间的可见性和顺序关系。它不是指前一个操作一定先执行完，而是指前一个操作的结果对后一个操作可见，并且前一个操作的执行顺序排在后一个操作之前。

常见规则包括：

- **程序顺序规则**：同一个线程中，按照代码顺序执行的操作具有 happens-before 关系。
- **解锁规则**：对同一把锁的解锁操作，happens-before 后续对这把锁的加锁操作。
- **volatile 变量规则**：对一个 `volatile` 变量的写操作，happens-before 后续对这个变量的读操作。
- **线程启动规则**：调用线程 `start()` 前的所有操作，happens-before 新线程中的所有操作。
- **线程终止规则**：线程中的所有操作，happens-before 其他线程检测到该线程已经结束。
- **传递性**：如果操作 A happens-before 操作 B，操作 B happens-before 操作 C，那么操作 A happens-before 操作 C。

### 6. `synchronized` 为什么能保证线程安全？

进入 `synchronized` 代码块时，线程会获取锁；退出代码块时会释放锁。对同一把锁来说：

- 同一时刻只有一个线程能够进入受保护的代码。
- 一个线程释放锁前对共享变量的修改，对后续获取同一把锁的线程可见。
- 通过互斥和内存可见性，锁可以同时帮助解决原子性、可见性和有序性问题。

```java
private static int count;

public static void increment() {
    synchronized (VisibilityDemo.class) {
        count++;
    }
}
```

### 7. 总结

| 问题 | 含义 | 常见解决方式 |
| --- | --- | --- |
| 可见性 | 一个线程的修改能否被其他线程看到 | `volatile`、锁、并发容器 |
| 原子性 | 操作能否不可分割地完成 | `synchronized`、`Lock`、原子类 |
| 有序性 | 多线程观察到的执行顺序是否符合规则 | `volatile`、锁、happens-before |

可以这样记忆：`volatile` 主要解决可见性和有序性，不能保证复合操作的原子性；`synchronized` 和 `Lock` 可以通过互斥访问保护一组操作；`AtomicInteger` 等原子类适合对单个变量进行高效的原子更新。

## 二、Java 多线程是什么？使用时需要注意什么？

Java 多线程是指在同一个程序中同时运行多个线程。线程共享进程中的堆、方法区等资源，但拥有各自的程序计数器和执行路径，可以同时处理不同任务。

需要注意以下几点：

1. **线程安全**：多个线程同时修改共享数据时，可能产生数据竞争。例如 `count++` 并不是原子操作，需要使用 `synchronized`、`Lock` 或原子类保护。
2. **线程间通信**：线程需要协作时，可以使用 `wait()`、`notify()` 等机制；生产者消费者场景通常优先使用 `BlockingQueue`。调用 `wait()`、`notify()` 前必须持有对应对象的锁。
3. **线程创建成本**：创建和销毁线程会消耗系统资源。处理大量短任务时，通常使用线程池复用线程。
4. **死锁问题**：多个线程互相等待对方持有的锁就可能发生死锁。获取多把锁时应保持固定顺序，并尽量缩小锁的范围。
5. **共享状态**：尽量减少共享可变数据，优先使用局部变量、不可变对象和线程安全集合。

多线程可以提高程序的响应能力和资源利用率，但“同时运行”不一定代表真正并行：单核 CPU 主要是并发执行，多核 CPU 才可能实现并行执行。

## 三、Java 里的线程和操作系统线程一样吗？

需要区分平台线程和虚拟线程。

### 1. 平台线程

平台线程是 Java 长期以来默认的线程实现。一个 Java 平台线程通常对应一个操作系统线程，属于严格的 **1:1 模型**。线程的调度和上下文切换主要由操作系统内核负责，因此创建和维护成本相对较高，不能无限创建。

### 2. 虚拟线程

虚拟线程在 Java 21 中正式发布。它由 JVM 在用户态进行调度，采用 **M:N 模型**：大量虚拟线程可以映射到少量承载平台线程（Carrier Thread）上执行。

虚拟线程创建和切换成本较低，适合需要同时处理大量阻塞 I/O 的场景，例如网络请求、文件读写等。但它不能让 CPU 密集型任务变得更快，CPU 密集型任务仍然需要合理控制并发量。

### 3. 两者的区别

| 对比项 | 平台线程 | 虚拟线程 |
| --- | --- | --- |
| 映射关系 | Java 线程与操作系统线程通常是 `1:1` | 多个虚拟线程映射到少量平台线程，属于 `M:N` |
| 调度者 | 主要由操作系统调度 | 主要由 JVM 调度 |
| 创建成本 | 较高 | 较低 |
| 适用场景 | CPU 密集型任务、线程数量可控的场景 | 大量并发、阻塞 I/O 密集型任务 |

因此，“Java 线程等于操作系统线程”只适用于传统平台线程。使用虚拟线程后，Java 可以用较低的成本创建大量并发任务。

## 四、保证数据的一致性有哪些方案？

在多线程或并发业务中，多个操作可能同时修改同一份数据。常见的一致性保证方案有以下几种：

- **事务管理**：将一组数据库操作放在同一个事务中，要么全部提交，要么全部回滚。数据库通过 ACID 特性保证操作的原子性、一致性、隔离性和持久性。
- **锁机制**：通过互斥访问避免多个线程同时修改共享资源。Java 中可以使用 `synchronized`、`ReentrantLock` 等；跨进程或跨服务时，则需要使用数据库锁或分布式锁。
- **版本控制**：使用乐观锁记录数据版本。更新时同时检查版本号，如果版本已经变化，说明数据被其他线程修改过，当前更新失败或重新尝试。

选择方案时，需要根据数据范围和并发场景判断：数据库内部的一组操作优先考虑事务；共享资源访问冲突明显时使用锁；冲突较少、读多写少时可以考虑乐观锁。

## 五、线程的创建方式有哪些？

常见的线程创建方式有四种：继承 `Thread`、实现 `Runnable`、实现 `Callable`，以及使用线程池。

### 1. 继承 `Thread` 类

继承 `Thread` 并重写 `run()` 方法，然后调用 `start()` 启动线程：

```java
class MyThread extends Thread {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        MyThread thread = new MyThread();
        thread.start();
    }
}
```

这种方式写法直接，但 Java 类只能继承一个父类，因此会限制其他继承关系。

### 2. 实现 `Runnable` 接口

实现 `Runnable` 的 `run()` 方法，再将 Runnable 对象传给 `Thread`：

```java
class MyRunnable implements Runnable {
    @Override
    public void run() {
        // 线程执行的代码
    }
}

public class RunnableDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(new MyRunnable());
        thread.start();
    }
}
```

这种方式将任务和线程分离，任务类还可以继承其他类，也便于多个线程共享同一个任务对象，因此通常比直接继承 `Thread` 更灵活。

### 3. 实现 `Callable`，配合 `FutureTask`

`Callable` 类似于 `Runnable`，但 `call()` 方法可以有返回值，也可以抛出异常。由于 `Thread` 的构造方法接收 `Runnable`，所以通常需要使用实现了 `Runnable` 的 `FutureTask` 包装 Callable：

```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        Callable<Integer> task = () -> 10 + 20;
        FutureTask<Integer> futureTask = new FutureTask<>(task);

        new Thread(futureTask).start();

        System.out.println(futureTask.get()); // 30
    }
}
```

`FutureTask.get()` 会等待任务执行完成，并获取返回结果；如果任务执行过程中抛出异常，也可以通过 `get()` 发现。

### 4. 使用线程池

实际开发中通常不直接频繁创建线程，而是将任务提交给线程池执行。线程池可以复用线程，减少线程创建和销毁的开销，并限制同时运行的线程数量。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ExecutorDemo {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 100; i++) {
            executor.submit(() -> {
                // 线程执行的代码
            });
        }

        executor.shutdown();
    }
}
```

线程池是更常用的任务执行方式，但需要合理设置线程数量、任务队列和拒绝策略，避免线程过多或任务堆积。

### 5. 方式选择

- 继承 `Thread`：写法简单，但会占用继承关系。
- 实现 `Runnable`：任务与线程分离，适合没有返回值的任务。
- 实现 `Callable`：适合需要返回结果或抛出异常的任务。
- 使用线程池：适合管理大量并发任务，是实际项目中更常见的方式。

## 六、Java 线程的状态有哪些？

Java 通过 `Thread.State` 定义了六种线程状态：

| 状态 | 含义 |
| --- | --- |
| `NEW` | 线程已经创建，但还没有调用 `start()`，尚未启动。 |
| `RUNNABLE` | 线程已经调用 `start()`，处于就绪或正在运行状态。Java 没有单独定义 `RUNNING` 状态。 |
| `BLOCKED` | 线程正在等待获取对象监视器锁，通常发生在进入 `synchronized` 代码块时。 |
| `WAITING` | 线程无限期等待其他线程执行特定操作，例如调用 `Object.wait()`、`Thread.join()` 或 `LockSupport.park()`。 |
| `TIMED_WAITING` | 线程在指定时间内等待，例如调用 `Thread.sleep()`、带超时的 `wait()` 或 `join()`。 |
| `TERMINATED` | 线程的 `run()` 方法执行结束，线程已经终止。 |

线程状态不是严格单向变化的。例如，线程可能从 `RUNNABLE` 进入 `BLOCKED`，获取锁后重新回到 `RUNNABLE`；等待结束后，也可能从 `WAITING` 或 `TIMED_WAITING` 回到 `RUNNABLE`。

## 七、`sleep()` 和 `wait()` 的区别是什么？

| 对比项 | `sleep()` | `wait()` |
| --- | --- | --- |
| 所属类 | `Thread` 的静态方法 | `Object` 的实例方法 |
| 是否释放锁 | 不释放已经持有的锁 | 释放调用对象的监视器锁 |
| 调用位置 | 可以在任意位置调用 | 必须在持有对象锁时调用 |
| 唤醒方式 | 时间到后自动恢复，也可以被中断 | 需要其他线程调用 `notify()`/`notifyAll()`，或等待超时 |
| 主要用途 | 暂停线程执行一段时间 | 线程之间进行协调和通信 |

### 1. 所属类不同

`sleep()` 是 `Thread` 类的静态方法，可以直接调用；`wait()` 是 `Object` 类的实例方法，必须通过具体对象调用。这是因为 `wait()` 操作的是对象的监视器锁，而 Java 中每个对象都可以作为锁。

### 2. 是否释放锁不同

线程调用 `Thread.sleep()` 后会暂停执行，但不会释放已经持有的对象锁。其他线程仍然无法获取这把锁。

线程调用 `Object.wait()` 后会释放调用对象的监视器锁，并进入等待状态。其他线程获取同一把锁后，才有机会修改共享状态并调用 `notify()` 或 `notifyAll()` 唤醒它。

### 3. 调用条件不同

`sleep()` 可以在任意位置调用；`wait()` 必须在同步代码块或同步方法中调用，并且同步的对象必须与调用 `wait()` 的对象一致，否则会抛出 `IllegalMonitorStateException`。

### 4. 唤醒机制不同

`sleep()` 的等待时间结束后会自动恢复，也可能因为线程被中断而提前结束等待。

`wait()` 通常需要其他线程调用同一个对象的 `notify()` 或 `notifyAll()`，也可以使用带超时参数的 `wait()` 自动结束等待。被唤醒的线程还需要重新获得对象锁，才能继续执行。

- `notify()`：唤醒一个等待线程，具体唤醒哪个线程由 JVM 调度决定，不应依赖固定顺序。
- `notifyAll()`：唤醒所有等待该对象锁的线程，但它们仍然需要竞争锁后才能继续执行。

可以简单记忆：`sleep()` 是让自己休息一段时间，不主动释放锁；`wait()` 是释放锁并等待其他线程协作。

## 八、不同的线程之间如何通信？

线程通信的核心是让线程之间安全地交换数据、传递信号或协调执行顺序。常见方式有以下几种：

- **共享变量**：多个线程通过共享变量传递状态，通常需要配合 `volatile`、原子类或锁保证可见性和线程安全。
- **`wait()` / `notify()`**：线程在对象监视器上等待和唤醒，适合简单的线程协作，但必须在持有同一把对象锁时调用。
- **`BlockingQueue`**：适合生产者消费者场景，队列为空时消费者等待，队列已满时生产者等待，通信和同步逻辑已经封装好。
- **`LockSupport.park()` / `unpark()`**：可以阻塞或唤醒指定线程，不要求必须依赖对象监视器锁。
- **并发工具类**：`CountDownLatch` 用于等待一组任务完成，`CyclicBarrier` 用于多个线程相互等待，`Semaphore` 用于控制同时访问资源的线程数量。
- **任务结果传递**：`Future`、`CompletableFuture` 等可以用于异步任务之间传递结果和异常。

选择方式时，简单的状态通知可以使用 `volatile`；生产者消费者优先使用 `BlockingQueue`；需要控制多个线程执行顺序时，可以选择相应的并发工具类。

## 九、Java 中如何安全地停止一个线程？

Java 通常采用“协作式取消”，由线程自己检查取消信号并结束执行，不建议强制杀死正在运行的线程。

| 方法 | 适用场景 | 注意事项 |
| --- | --- | --- |
| 循环检测标志位 | 简单的非阻塞任务 | 标志位需要使用 `volatile` 或通过锁保证可见性 |
| 中断机制 | 可以被中断的阻塞操作 | 正确处理 `InterruptedException`，必要时恢复中断标志 |
| `Future.cancel(true)` | 线程池中的任务取消 | 线程池任务需要支持中断处理 |
| 关闭资源 | 不支持中断的阻塞操作，例如部分 Socket 操作 | 显式关闭资源触发异常，结合中断状态决定是否退出 |

### 1. 循环检测标志位

线程定期检查一个共享的取消标志，发现任务需要停止后主动退出。标志位必须保证线程之间的可见性，否则执行线程可能一直读取到旧值。

### 2. 中断机制

调用 `interrupt()` 不会直接杀死线程，而是向线程发出停止信号：

- 如果线程正在普通代码中运行，会设置其中断标志，需要线程主动检查并退出。
- 如果线程正在 `sleep()`、`wait()` 或可中断的阻塞方法中等待，通常会抛出 `InterruptedException`。
- 捕获异常后不能无视中断请求；如果当前方法无法直接结束任务，通常应调用 `Thread.currentThread().interrupt()` 恢复中断标志，再交给上层处理。

### 3. `Future.cancel(true)`

线程池任务可以通过 `Future.cancel(true)` 请求取消。参数为 `true` 时会尝试中断正在执行任务的线程，但任务是否真正结束，仍取决于任务是否正确处理中断。

### 4. 关闭阻塞资源

部分阻塞操作对中断的响应并不一致。对于这类任务，可以关闭正在使用的 Socket、文件或其他资源，让阻塞操作抛出异常，再根据任务状态决定是否退出。

不建议使用已经废弃的 `Thread.stop()` 强行终止线程，因为它可能在线程持有锁或修改共享数据的中途结束，导致数据不一致和锁状态异常。
