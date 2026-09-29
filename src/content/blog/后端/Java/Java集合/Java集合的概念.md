---
title: 'Java集合的概念'
description: 'Java集合概念学习笔记'
pubDate: 'Sep 29 2026'
tags: ['Java', '集合']
---

## 一、数组和集合的区别，适用场景有哪些？

### 1. 数组和集合的区别

- **长度不同**：数组是固定长度的数据结构，创建后长度不能改变；集合通常是动态长度的数据结构，可以根据需要增加或删除元素。
- **元素类型不同**：数组既可以存储基本数据类型，也可以存储对象；集合只能存储对象，但基本数据类型会自动装箱成对应的包装类。
- **访问方式不同**：数组可以通过下标直接访问元素；集合通常通过迭代器、增强 `for` 循环或集合提供的方法访问元素。
- **功能不同**：数组结构简单、访问速度快；集合提供了增删、查找、排序和去重等更丰富的操作。

### 2. 数组示例

```java
public class ArrayDemo {
    public static void main(String[] args) {
        int[] numbers = {10, 20, 30};

        System.out.println(numbers[0]); // 10
        System.out.println(numbers.length); // 3

        for (int number : numbers) {
            System.out.println(number);
        }
    }
}
```

数组适合存储元素数量固定、需要频繁通过下标访问的数据，例如一周七天、固定大小的成绩表等。

### 3. 集合示例

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class CollectionDemo {
    public static void main(String[] args) {
        // ArrayList：有序、可重复、长度可动态变化
        List<String> names = new ArrayList<>();
        names.add("Tom");
        names.add("Jerry");
        names.remove("Tom");

        // HashSet：无序、元素不能重复
        Set<String> tags = new HashSet<>();
        tags.add("Java");
        tags.add("Java"); // 重复元素不会被再次添加

        // HashMap：以键值对的形式保存数据，键不能重复
        Map<String, Integer> scores = new HashMap<>();
        scores.put("Tom", 90);
        scores.put("Jerry", 85);

        System.out.println(names); // [Jerry]
        System.out.println(tags); // [Java]
        System.out.println(scores.get("Tom")); // 90
    }
}
```

### 4. 常用集合类

1. **`ArrayList`**：基于动态数组实现的 `List`，查询快，适合读多写少的场景。
2. **`LinkedList`**：基于双向链表实现的 `List`，适合频繁在首尾插入和删除元素的场景。
3. **`HashMap`**：基于哈希表实现的 `Map`，通过键快速查找值。
4. **`HashSet`**：基于哈希表实现的 `Set`，用于存储不重复的元素。
5. **`TreeMap`**：基于红黑树实现的有序 `Map`，可以按照键进行排序。
6. **`LinkedHashMap`**：在哈希表的基础上维护插入顺序或访问顺序。
7. **`PriorityQueue`**：优先队列，按照元素的自然顺序或比较器确定出队顺序。

## 二、讲一下 Java 中的集合

Java 集合框架主要分为 `Collection` 和 `Map` 两大体系：`Collection` 用于存储单个元素，`Map` 用于存储键值对。`List`、`Set` 都属于 `Collection`，而 `Map` 不继承 `Collection` 接口。

### 1. List

`List` 是有序的 `Collection`，可以精确控制元素的插入位置，并且允许元素重复。使用者可以通过索引访问其中的元素。

常见实现有 `ArrayList`、`LinkedList`、`Vector` 和 `Stack`。

- **`ArrayList`**：基于动态数组实现的非线程安全 `List`。随机访问速度快，尾部添加和删除元素效率较高；在中间位置插入或删除元素时，需要移动后续元素，代价相对较高。扩容时会创建更大的数组，并复制原数组中的元素。
- **`LinkedList`**：本质上是双向链表，也实现了 `Deque` 接口，适合在首尾插入、删除元素以及作为双端队列使用。需要注意的是，只有在已经持有目标节点引用时，插入和删除才是 `O(1)`；如果先通过索引查找位置，仍然需要 `O(n)` 遍历。由于节点需要额外分配内存，且访问连续性较差，很多实际场景下 `LinkedList` 反而比 `ArrayList` 慢。
- **`Vector`**：早期的线程安全动态数组，方法通常使用同步机制保护。由于同步开销和设计较老，现代开发中通常优先使用 `ArrayList`；需要并发容器时，应根据场景选择更合适的实现。
- **`Stack`**：`Vector` 的子类，表示后进先出（LIFO）的栈。现代代码通常推荐使用 `Deque` 的实现，例如 `ArrayDeque`。

### 2. Set

`Set` 不允许存储重复元素。元素是否重复，通常由 `equals()` 和 `hashCode()` 共同决定。不同实现对元素顺序的保证不同。

常见实现有 `HashSet`、`LinkedHashSet` 和 `TreeSet`。

- **`HashSet`**：底层基于 `HashMap` 实现，通过 HashMap 的 key 保存元素，value 使用同一个占位对象。它可以保证元素不重复，但不保证遍历顺序，并且本身不是线程安全的。
- **`LinkedHashSet`**：继承自 `HashSet`，底层使用 `LinkedHashMap`，在去重的同时维护元素的插入顺序。
- **`TreeSet`**：底层基于 `TreeMap` 的红黑树实现，元素会按照自然顺序或自定义比较器排序。

### 3. Map

`Map` 是键值对集合，用于建立 key 和 value 之间的映射关系。key 不能重复，value 可以重复。通过 key 查找元素时，会返回与之对应的 value。

常见实现有 `HashMap`、`LinkedHashMap`、`Hashtable`、`TreeMap` 和 `ConcurrentHashMap`。

- **`HashMap`**：JDK 1.8 之后主要由数组、链表和红黑树组成。数组是主体，链表用于解决哈希冲突；当某个桶中的链表长度达到树化条件，且哈希表数组长度至少为 64 时，链表可能转换为红黑树，以减少查找时间。如果数组长度不足 64，通常会优先扩容而不是树化。`HashMap` 允许一个 `null` key 和多个 `null` value，但不是线程安全的。
- **`LinkedHashMap`**：继承自 `HashMap`，在哈希表结构的基础上增加双向链表，用于维护插入顺序或访问顺序，常用于实现需要顺序的 Map 和简单的 LRU 缓存。
- **`Hashtable`**：早期的线程安全 Map，底层主要由数组和链表组成，不允许使用 `null` key 或 `null` value。现代并发场景通常优先考虑 `ConcurrentHashMap`。
- **`TreeMap`**：基于红黑树实现的有序 Map，会按照 key 的自然顺序或自定义比较器维护元素顺序。
- **`ConcurrentHashMap`**：线程安全的 Map。JDK 1.7 主要采用分段锁，JDK 1.8 之后采用数组、链表或红黑树，并结合 CAS 和局部同步机制提高并发性能。

### 4. 如何选择？

- 需要有序、允许重复并且经常按索引查询：优先考虑 `ArrayList`。
- 需要去重：选择 `HashSet`；还需要保持插入顺序时选择 `LinkedHashSet`；需要排序时选择 `TreeSet`。
- 需要保存键值对：通常选择 `HashMap`；需要保持顺序时选择 `LinkedHashMap`；需要按 key 排序时选择 `TreeMap`。
- 多线程环境下需要使用 Map：优先考虑 `ConcurrentHashMap`，不要直接把 `HashMap` 用在并发读写场景中。

## 三、Java 中的线程安全集合有哪些？

线程安全集合主要分为两类：`java.util` 中早期通过同步方法实现的集合，以及 `java.util.concurrent` 包中针对并发场景设计的集合。

### 1. `java.util` 中的线程安全集合

- **`Vector`**：线程安全的动态数组，许多方法使用 `synchronized` 修饰。由于同步粒度较粗，存在额外开销，现代开发中通常优先使用 `ArrayList`；如果确实需要并发访问，应根据实际场景选择更合适的并发容器。
- **`Hashtable`**：线程安全的哈希表，通常通过同步整个方法或对象来保证安全，不允许 `null` key 和 `null` value。由于并发性能有限，现代开发中通常使用 `ConcurrentHashMap`。

需要注意的是，单个方法线程安全并不代表多个操作组合起来也是原子的。复杂的复合操作仍然需要额外的同步或使用专门的并发方法。

### 2. 并发 Map

- **`ConcurrentHashMap`**：高并发场景下常用的线程安全 Map。JDK 1.7 主要使用分段锁；JDK 1.8 之后采用数组、链表或红黑树，并结合 CAS 和局部同步机制降低锁竞争。
- **`ConcurrentSkipListMap`**：基于跳表实现的有序并发 Map，能够按照 key 排序，适合既需要线程安全又需要有序访问的场景。

### 3. 并发 Set

- **`ConcurrentSkipListSet`**：线程安全的有序 Set，底层基于 `ConcurrentSkipListMap` 实现。
- **`CopyOnWriteArraySet`**：基于 `CopyOnWriteArrayList` 实现的线程安全 Set。写操作会复制底层数组，适合读多写少、集合规模不大的场景；写操作频繁时成本较高。

### 4. 并发 List

- **`CopyOnWriteArrayList`**：线程安全的 List。写操作会复制一份新的底层数组，修改完成后再替换旧数组；读操作通常不需要加锁，适合读多写少的场景。它允许 `null` 元素，但不适合频繁写入或数据量很大的场景。

### 5. 并发 Queue

- **`ConcurrentLinkedQueue`**：基于 CAS 实现的无界、非阻塞并发队列，适合多个线程并发入队和出队的场景。
- **`BlockingQueue`**：阻塞队列接口，重点不在于单纯提高并发性能，而在于提供生产者—消费者之间的阻塞协调机制。队列为空时，消费者调用 `take()` 会等待；队列已满时，生产者调用 `put()` 会等待。常见实现有 `ArrayBlockingQueue` 和 `LinkedBlockingQueue`。

### 6. 并发 Deque

- **`LinkedBlockingDeque`**：线程安全的阻塞双端队列，支持从队列两端插入和取出元素，适合需要生产者—消费者协调的场景。
- **`ConcurrentLinkedDeque`**：基于链表的无界、非阻塞并发双端队列，适合多个线程并发进行插入、删除和访问操作的场景。

### 7. 简单选择建议

- 高并发键值对：优先选择 `ConcurrentHashMap`。
- 读多写少的 List：可以考虑 `CopyOnWriteArrayList`。
- 需要生产者—消费者阻塞协调：选择 `BlockingQueue`。
- 需要有序并发 Map 或 Set：选择 `ConcurrentSkipListMap` 或 `ConcurrentSkipListSet`。
- 需要非阻塞并发队列：选择 `ConcurrentLinkedQueue` 或 `ConcurrentLinkedDeque`。

## 四、`Collections` 和 `Collection` 的区别？

- **`Collection`**：Java 集合框架中的接口，是 `List`、`Set` 和 `Queue` 等集合类型的基础接口。它定义了添加、删除、遍历、判断是否包含元素等通用操作，用于管理一组单独的元素。
- **`Collections`**：`java.util` 包中的工具类，提供了大量静态方法，用于操作和处理集合，例如排序、查找、替换、反转、随机打乱，以及创建同步集合和不可修改集合等。

可以简单记忆为：**`Collection` 是集合的规范，`Collections` 是操作集合的工具。**

### 示例

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CollectionsDemo {
    public static void main(String[] args) {
        List<Integer> numbers = new ArrayList<>();
        numbers.add(3);
        numbers.add(1);
        numbers.add(2);

        Collections.sort(numbers); // 排序：[1, 2, 3]
        Collections.reverse(numbers); // 反转：[3, 2, 1]
        System.out.println(numbers);
    }
}
```

## 五、集合遍历的方法有哪些？

Java 中常见的集合遍历方式有普通 `for` 循环、增强 `for` 循环、`Iterator`、`ListIterator`、`forEach` 方法和 Stream API。应根据集合类型和具体需求选择合适的方式。

### 1. 普通 `for` 循环

适合 `List` 这类可以通过索引访问的集合，可以在遍历过程中获取元素下标。

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

for (int i = 0; i < list.size(); i++) {
    String element = list.get(i);
    System.out.println(element);
}
```

### 2. 增强 `for` 循环（for-each）

适合简单地遍历数组或集合，语法简洁，但不能直接获取集合下标，也不适合在遍历过程中进行结构性修改。

```java
for (String element : list) {
    System.out.println(element);
}
```

### 3. `Iterator` 迭代器

`Iterator` 适用于大多数集合，提供了 `hasNext()`、`next()` 和 `remove()` 等方法。需要在遍历过程中删除元素时，应优先使用迭代器提供的 `remove()`，避免直接调用集合的 `remove()` 导致 `ConcurrentModificationException`。

```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String element = iterator.next();
    if ("B".equals(element)) {
        iterator.remove();
    }
}
```

### 4. `ListIterator` 列表迭代器

`ListIterator` 是 `Iterator` 的子接口，只适用于 `List`。它支持双向遍历，并且可以在遍历过程中使用 `add()`、`set()` 和 `remove()` 修改列表。

```java
ListIterator<String> listIterator = list.listIterator();
while (listIterator.hasNext()) {
    String element = listIterator.next();
    if ("A".equals(element)) {
        listIterator.set("a");
    }
}
```

### 5. `forEach` 方法

Java 8 为集合增加了 `forEach` 方法，可以使用 Lambda 表达式快速遍历集合。

```java
list.forEach(element -> System.out.println(element));
```

### 6. Stream API

Stream API 适合对集合进行过滤、映射、排序和聚合等函数式操作。它不仅可以遍历集合，还能将多个处理步骤串成一条数据处理流水线。

```java
list.stream()
        .filter(element -> !"B".equals(element))
        .forEach(System.out::println);
```

需要注意，`forEach` 和 Stream API 不适合直接控制循环流程，也不建议在其中修改原集合的结构。
