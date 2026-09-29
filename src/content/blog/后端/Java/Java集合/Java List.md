---
title: 'Java List'
description: 'Java List 学习笔记'
pubDate: 'Sep 29 2026'
tags: ['Java', '集合', 'List']
---

## 一、List 可以一边遍历一边修改元素吗？

可以，但要区分“修改元素”和“修改集合结构”，还要看使用的遍历方式。

### 1. 使用普通 `for` 循环修改元素

普通 `for` 循环通过下标访问元素，可以使用 `List.set(index, value)` 修改元素。`set()` 只会替换指定位置的元素，不会改变集合的大小，因此不会触发 `ConcurrentModificationException`。

```java
import java.util.ArrayList;
import java.util.List;

public class ListTraversalAndModification {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        // 使用普通 for 循环遍历并修改元素
        for (int i = 0; i < list.size(); i++) {
            list.set(i, list.get(i) * 2);
        }

        System.out.println(list); // [2, 4, 6]
    }
}
```

### 2. 使用增强 `for` 循环修改集合结构

增强 `for` 循环底层使用迭代器。遍历过程中直接调用 `list.add()` 或 `list.remove()` 修改集合结构，通常会在迭代器下一次调用 `next()` 时抛出 `ConcurrentModificationException`。

需要注意：`list.set()` 只是替换元素，不会改变集合结构，因此通常不会触发这个异常；真正容易出问题的是新增或删除元素。

```java
import java.util.ArrayList;
import java.util.List;

public class ListTraversalAndModification {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);

        for (Integer number : list) {
            if (number == 2) {
                list.remove(number); // 修改集合结构
            }
        }
    }
}
```

上面的代码不适合在遍历时删除元素，因为增强 `for` 使用的迭代器并不知道集合已经被直接修改，后续继续遍历时可能抛出 `ConcurrentModificationException`。

### 3. 使用 `Iterator` 删除元素

如果需要在遍历过程中删除元素，应使用迭代器提供的 `remove()` 方法。这个方法会同步更新迭代器的状态，是遍历时删除元素的标准写法。

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ListTraversalAndModification {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);
        list.add(4);

        Iterator<Integer> iterator = list.iterator();
        while (iterator.hasNext()) {
            Integer number = iterator.next();
            if (number == 2) {
                iterator.remove();
            }
        }

        System.out.println(list); // [1, 3, 4]
    }
}
```

### 4. 使用 `ListIterator` 替换元素

如果需要在遍历过程中替换元素，可以使用 `ListIterator.set()`。它适合对当前遍历到的元素进行修改，而且不会改变集合的大小。

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListTraversalAndModification {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        ListIterator<Integer> iterator = list.listIterator();
        while (iterator.hasNext()) {
            Integer number = iterator.next();
            if (number.equals(2)) {
                iterator.set(4);
            }
        }

        System.out.println(list); // [1, 4, 3]
    }
}
```

### 结论

- 只修改元素值：可以使用普通 `for` 循环的 `set()`，或者使用 `ListIterator.set()`。
- 遍历时删除元素：使用 `Iterator.remove()`，不要直接调用 `List.remove()`。
- 遍历时新增元素：可以使用 `ListIterator.add()`，不要在增强 `for` 循环中直接调用 `List.add()`。

## 二、List 如何快速删除某个指定下标的元素？

`List` 提供了 `remove(int index)` 方法，可以根据下标删除元素。删除后，后面的元素会整体向前移动一位，以填补被删除的位置。

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListRemoveExample {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        // 删除下标为 1 的元素，也就是数字 2
        list.remove(1);

        System.out.println(list); // [1, 3]
    }
}
```

### `ArrayList` 删除元素的复杂度

`ArrayList` 底层使用数组保存元素：

- 删除末尾元素时，不需要移动其他元素，时间复杂度通常为 `O(1)`。
- 删除中间或开头的元素时，需要将后续元素向前移动，时间复杂度为 `O(n)`。

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);

list.remove(2); // 删除下标为 2 的元素，通常为 O(1)
list.remove(0); // 删除下标为 0 的元素，需要移动后续元素，通常为 O(n)
```

### `LinkedList` 删除元素的复杂度

`LinkedList` 底层使用双向链表保存元素。调用 `remove(int index)` 时，需要先从头部或尾部遍历到指定下标，再修改节点之间的指针：

- 删除头节点或尾节点时，可以直接修改指针，时间复杂度为 `O(1)`。
- 删除任意中间下标时，需要先查找节点，时间复杂度为 `O(n)`。

```java
import java.util.LinkedList;
import java.util.List;

public class LinkedListRemoveExample {
    public static void main(String[] args) {
        List<Integer> list = new LinkedList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        // 删除下标为 1 的元素，也就是数字 2
        list.remove(1);

        System.out.println(list); // [1, 3]
    }
}
```

### `remove(1)` 的重载问题

当 `List` 中存储的是 `Integer` 时，`remove(1)` 会优先匹配 `remove(int index)`，表示删除下标为 `1` 的元素，而不是删除数值为 `1` 的元素。

```java
List<Integer> list = new ArrayList<>();
list.add(1);
list.add(2);
list.add(3);

list.remove(1);                  // 删除下标 1 的元素，结果为 [1, 3]
list.remove(Integer.valueOf(1)); // 删除值为 1 的元素，结果为 [2, 3]
```

如果想明确删除某个元素值，可以使用 `Integer.valueOf()`，或者先把变量声明为 `Integer`：

```java
Integer value = 1;
list.remove(value); // 调用 remove(Object)，删除值为 1 的元素
```

因此，删除指定下标使用 `remove(int index)`；删除指定值则要确保调用的是 `remove(Object)`。

## 三、ArrayList 和 LinkedList 的区别，哪个集合是线程安全的？

`ArrayList` 和 `LinkedList` 都实现了 `List` 接口，因此都支持元素有序、允许重复，并且可以通过下标访问元素。它们的主要区别在于底层数据结构不同。

| 对比维度 | ArrayList | LinkedList |
| --- | --- | --- |
| 底层结构 | 动态数组 | 双向链表 |
| 随机访问 | 快，通过下标访问通常为 `O(1)` | 慢，需要遍历查找，通常为 `O(n)` |
| 尾部添加 | 通常为 `O(1)`，扩容时会变慢 | 通常为 `O(1)` |
| 中间插入、删除 | 需要移动后续元素，通常为 `O(n)` | 找到节点后修改指针，但查找位置通常为 `O(n)` |
| 空间占用 | 主要保存元素，可能有预留容量 | 每个节点还要保存前后节点的引用，额外开销较大 |
| 适用场景 | 查询多、修改少，或主要在尾部操作 | 频繁在头部或尾部插入、删除，或需要双端队列功能 |

### 1. 底层数据结构不同

- `ArrayList` 使用动态数组保存元素，可以通过下标直接定位元素，因此随机访问速度快。
- `LinkedList` 使用双向链表保存元素，每个节点保存元素本身以及前后节点的引用。访问指定位置时，需要从头部或尾部逐步查找。

### 2. 插入和删除的效率不同

`ArrayList` 在尾部插入或删除时，不需要移动其他元素，效率通常较高；但在开头或中间插入、删除时，需要移动后续元素。

`LinkedList` 在头部或尾部插入、删除时，只需要调整节点指针，效率较高；但如果操作的是中间位置，仍然需要先遍历找到目标节点，所以整体时间复杂度通常为 `O(n)`。

需要注意，`LinkedList` 的“插入、删除快”通常有一个前提：已经拿到了目标节点，或者操作的是头部、尾部。仅仅知道一个下标时，仍然需要先查找节点，并不会自动变成 `O(1)`。

### 3. 两者都不是线程安全的

`ArrayList` 和 `LinkedList` 都没有内置同步机制。在多线程环境下，如果多个线程同时修改同一个集合，需要额外进行同步处理，或者选择线程安全的集合实现。

常见的线程安全 `List` 有：

- `Vector`：方法大多使用 `synchronized` 修饰，线程安全，但锁粒度较大，性能开销也较高。除非维护旧代码，一般不作为首选。
- `Collections.synchronizedList()`：将普通 `List` 包装成同步集合，适合对线程安全有简单要求的场景。
- `CopyOnWriteArrayList`：写操作时复制底层数组，读操作通常不加锁，适合读多写少的并发场景。

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class ThreadSafeListExample {
    public static void main(String[] args) {
        List<Integer> synchronizedList = Collections.synchronizedList(new ArrayList<>());
        synchronizedList.add(1);

        List<Integer> copyOnWriteList = new CopyOnWriteArrayList<>();
        copyOnWriteList.add(2);
    }
}
```

### 4. 如何选择

- 默认优先选择 `ArrayList`，尤其是查询多、随机访问多的场景。
- 需要使用队列或双端队列功能时，可以考虑 `LinkedList`，不过实际项目中通常优先选择更明确的 `ArrayDeque`。
- 不要因为 `LinkedList` 擅长插入和删除，就认为它在所有插入、删除场景下都比 `ArrayList` 快；如果需要先遍历定位，整体复杂度仍然可能是 `O(n)`。
- 多线程读写普通集合时，选择合适的并发集合或使用明确的同步策略。

## 四、ArrayList 的扩容机制是怎样的？

`ArrayList` 底层使用数组保存元素，但数组的长度固定，所以 `ArrayList` 需要通过“创建更大的数组并复制元素”的方式实现动态扩容。

### 1. `size` 和 `capacity` 的区别

- `size`：当前集合中实际保存的元素个数，可以通过 `list.size()` 获取。
- `capacity`：底层数组当前能够容纳的元素个数，属于内部实现细节。

当 `size` 达到 `capacity` 后，再继续添加元素就会触发扩容。

### 2. 扩容过程

扩容时主要经历以下步骤：

1. 计算新的容量，通常按照旧容量的 `1.5` 倍增长。
2. 创建一个容量更大的新数组。
3. 将旧数组中的元素复制到新数组。
4. 让 `ArrayList` 的内部数组引用新数组。
5. 将新元素添加到新数组中。

```java
// 典型的扩容计算方式
int newCapacity = oldCapacity + (oldCapacity >> 1);
```

`oldCapacity >> 1` 表示旧容量右移一位，也就是旧容量除以 `2`，因此新的容量约为旧容量的 `1.5` 倍。

不同 JDK 版本的内部实现细节可能有所调整，例如扩容计算可能会复用 `ArraysSupport.newLength` 等内部方法，但整体思路没有变化：创建更大的数组并复制原有元素。

### 3. 为什么不是每次只增加一个位置？

如果每次添加元素都只扩容一个位置，就需要频繁创建新数组并复制数据，添加大量元素时性能会很差。

按一定比例扩容可以减少扩容次数。虽然某次扩容需要复制元素，时间复杂度为 `O(n)`，但从多次添加操作的整体来看，`ArrayList` 尾部添加的平均时间复杂度仍然接近 `O(1)`，这称为摊销复杂度。

### 4. 预估容量，减少扩容次数

如果能够提前估计集合需要保存的元素数量，可以在创建 `ArrayList` 时直接指定初始容量，避免频繁扩容和数组复制。

```java
import java.util.ArrayList;
import java.util.List;

public class ArrayListCapacityExample {
    public static void main(String[] args) {
        // 预计保存约 1000 个元素，提前设置容量
        List<Integer> list = new ArrayList<>(1000);

        for (int i = 0; i < 1000; i++) {
            list.add(i);
        }

        System.out.println(list.size()); // 1000
    }
}
```

也可以在已有集合上调用 `ensureCapacity()` 提前申请容量：

```java
ArrayList<Integer> list = new ArrayList<>();
list.ensureCapacity(1000);
```

这里设置的是底层数组的容量，不会直接增加集合中的元素个数，因此 `size()` 仍然是 `0`。

### 5. 扩容机制的结论

- `ArrayList` 的扩容本质是创建新数组并复制旧数组中的元素。
- 扩容会带来一次性复制开销，频繁扩容可能影响性能。
- 预先设置合理的初始容量，可以减少扩容次数。
- 不需要为了避免扩容而盲目设置很大的容量，否则会造成内存浪费。

## 五、线程安全的 List：CopyOnWriteArrayList 是如何实现线程安全的？

`CopyOnWriteArrayList` 是 `List` 的线程安全实现，适合“读操作很多、写操作很少”的场景，例如配置快照、监听器列表等。

它的核心思想是：**读操作直接读取当前数组，写操作先复制一份新数组，在新数组上完成修改后，再一次性替换旧数组**。因此，读操作通常不需要加锁。

### 1. `volatile` 保存数组引用

`CopyOnWriteArrayList` 底层仍然使用数组保存元素，内部数组引用使用 `volatile` 修饰：

```java
private transient volatile Object[] array;
```

这里的 `volatile` 主要保证数组引用的可见性：一个线程替换数组引用后，其他线程能够及时看到最新的数组。

不过，`volatile` 只负责保证可见性，并不能保证“复制数组、修改数组、替换引用”这一整套操作的原子性，所以写操作还需要使用锁保护。

### 2. 写操作：加锁并复制新数组

下面是 `add()` 操作的简化版流程，实际 JDK 源码还包含边界检查、序列化字段等细节：

```java
public boolean add(E element) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int length = elements.length;

        // 复制旧数组，并让新数组多出一个位置
        Object[] newElements = Arrays.copyOf(elements, length + 1);
        newElements[length] = element;

        // 用新数组替换旧数组
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

一次写操作大致经历以下步骤：

1. 获取写锁，避免多个线程同时复制和替换数组。
2. 读取当前数组。
3. 创建一个新数组，并复制旧数组中的元素。
4. 在新数组中完成新增、删除或修改操作。
5. 使用 `volatile` 引用发布新数组。
6. 释放写锁。

由于旧数组在发布后不会再被修改，正在读取旧数组的线程不会读到一半的新数据；写线程完成后，后续读线程会读取到新的数组。

### 3. 读操作：不加锁，直接读取数组

读取元素时，只需要获取当前数组并访问对应下标：

```java
public E get(int index) {
    return get(getArray(), index);
}
```

因此，多个线程可以同时读取同一个 `CopyOnWriteArrayList`，读取之间不会因为写锁而互相阻塞。

### 4. 迭代器读取的是快照

`CopyOnWriteArrayList` 的迭代器通常基于创建迭代器时的数组快照。迭代开始后，即使其他线程修改了集合，当前迭代也不会抛出 `ConcurrentModificationException`，并且不会看到后续新增的元素。

```java
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class CopyOnWriteExample {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        list.add("A");
        list.add("B");

        for (String value : list) {
            // 其他线程即使在此时修改 list，当前迭代仍然读取创建迭代器时的快照
            System.out.println(value);
        }
    }
}
```

由于迭代器读取的是快照，它不支持通过迭代器修改集合，调用 `iterator.remove()` 会抛出 `UnsupportedOperationException`。如果需要修改集合，应直接调用 `add()`、`remove()` 等方法。

### 5. 为什么适合读多写少？

`CopyOnWriteArrayList` 的优点是读操作简单、并发读取性能好、迭代过程稳定；缺点是每次写操作都需要复制整个数组：

- 写操作需要加锁，多个写线程之间仍然需要排队。
- 写操作会产生新的数组，元素数量较多时会带来复制和内存开销。
- 迭代器看到的是快照，不一定包含最新数据。

因此，`CopyOnWriteArrayList` 适合读多写少的场景。如果写操作频繁，通常应考虑其他并发集合或更合适的数据结构。

## 六、`List<>` 里面填基本数据类型为什么会报错？

Java 泛型的类型参数必须是**引用类型**，不能直接填写 `int`、`char`、`double`、`boolean` 等基本数据类型。

```java
// 错误：泛型中不能直接使用基本数据类型
List<int> list = new ArrayList<>(); // 编译报错
```

### 1. 使用包装类代替基本数据类型

每种基本数据类型都有对应的包装类：

| 基本数据类型 | 包装类 |
| --- | --- |
| `byte` | `Byte` |
| `short` | `Short` |
| `int` | `Integer` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `char` | `Character` |
| `boolean` | `Boolean` |

因此，保存整数时应该使用 `List<Integer>`：

```java
import java.util.ArrayList;
import java.util.List;

public class GenericWrapperExample {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();

        list.add(10);       // 自动装箱：int -> Integer
        int number = list.get(0); // 自动拆箱：Integer -> int

        System.out.println(number); // 10
    }
}
```

### 2. 为什么泛型不支持基本数据类型？

Java 泛型设计时只支持引用类型。泛型集合需要按照对象的方式处理元素，而基本数据类型不是对象，不能直接作为泛型类型参数。

此外，Java 泛型主要通过类型擦除实现。编译器会在编译期间检查泛型类型，并在需要时插入类型转换；运行时泛型信息会被擦除为原始类型或其上界，通常可以理解为按 `Object` 处理。`Object` 只能引用对象，不能直接保存 `int` 等基本数据类型。

使用包装类后，基本数据类型可以通过自动装箱和自动拆箱参与泛型集合操作：

- 自动装箱：将基本数据类型转换为对应的包装类，例如 `int` 转为 `Integer`。
- 自动拆箱：将包装类转换回基本数据类型，例如 `Integer` 转为 `int`。

### 3. 包装类允许保存 `null`

基本数据类型不能表示 `null`，但包装类可以。因此，`List<Integer>` 中既可以保存整数，也可以保存 `null`。

```java
List<Integer> list = new ArrayList<>();
list.add(null);

Integer value = list.get(0); // 可以接收 null

// int number = list.get(0);
// 发生自动拆箱时，null 无法转换为 int，会抛出 NullPointerException
```

如果确定集合中可能存在 `null`，在自动拆箱前应先进行判断：

```java
Integer value = list.get(0);
if (value != null) {
    int number = value;
    System.out.println(number);
}
```

所以，泛型集合中不能写基本数据类型，应该使用对应的包装类；实际使用时，Java 会通过自动装箱和自动拆箱简化两者之间的转换。

## 七、List 和数组如何相互转换？

### 1. List 转数组

可以调用 `List.toArray()` 方法。为了得到指定类型的数组，推荐传入一个类型数组：

```java
import java.util.ArrayList;
import java.util.List;

public class ListArrayConversion {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Java");
        list.add("集合");

        String[] array = list.toArray(new String[0]);

        for (String value : array) {
            System.out.println(value);
        }
    }
}
```

`toArray(new String[0])` 会按照集合的实际大小创建一个 `String[]`。也可以传入指定长度的数组：

```java
String[] array = list.toArray(new String[list.size()]);
```

在现代 JDK 中，`new String[0]` 写法简洁且常用，具体数组创建和复制由 JDK 完成。

### 2. 数组转 List

#### 使用 `Arrays.asList()`

```java
import java.util.Arrays;
import java.util.List;

String[] array = {"Java", "集合"};
List<String> list = Arrays.asList(array);
```

`Arrays.asList()` 返回的是一个由数组支持的固定长度 List：

- 可以使用 `set()` 修改元素；
- 修改 List 中的元素，数组中的对应元素也会改变；
- 不能调用 `add()` 或 `remove()` 改变集合大小，否则会抛出 `UnsupportedOperationException`。

```java
String[] array = {"A", "B", "C"};
List<String> list = Arrays.asList(array);

list.set(0, "X");
System.out.println(array[0]); // X

// list.add("D");    // UnsupportedOperationException
// list.remove("B"); // UnsupportedOperationException
```

#### 转换为可变 List

如果后续需要新增或删除元素，应使用 `ArrayList` 复制一份：

```java
String[] array = {"A", "B", "C"};
List<String> list = new ArrayList<>(Arrays.asList(array));

list.add("D");
list.remove("B");

System.out.println(list); // [A, C, D]
```

### 3. 使用 `List.of()` 创建 List

Java 9 及以上可以使用 `List.of()`：

```java
String[] array = {"A", "B", "C"};
List<String> list = List.of(array);
```

但 `List.of()` 返回的是不可变 List，不能调用 `add()`、`remove()` 或 `set()`。如果需要修改，应再创建一个 `ArrayList`：

```java
List<String> mutableList = new ArrayList<>(List.of(array));
mutableList.add("D");
```

### 4. 基本类型数组的注意事项

`Arrays.asList()` 只能正确处理引用类型数组。对于 `int[]` 这样的基本类型数组，它会把整个数组当成一个元素，而不是转换成 `List<Integer>`：

```java
int[] numbers = {1, 2, 3};
List<int[]> wrong = Arrays.asList(numbers);

System.out.println(wrong.size()); // 1
```

如果要把 `int[]` 转成 `List<Integer>`，可以使用 `IntStream`：

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

int[] numbers = {1, 2, 3};
List<Integer> list = Arrays.stream(numbers)
        .boxed()
        .collect(Collectors.toList());

System.out.println(list); // [1, 2, 3]
```

### 5. 常用写法总结

```java
// List -> 引用类型数组
String[] array = list.toArray(new String[0]);

// 引用类型数组 -> 固定长度 List
List<String> fixedList = Arrays.asList(array);

// 引用类型数组 -> 可变 List
List<String> mutableList = new ArrayList<>(Arrays.asList(array));

// Java 9+：引用类型数组 -> 不可变 List
List<String> immutableList = List.of(array);
```
