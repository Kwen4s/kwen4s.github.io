
---
title: 'Java Map'
description: 'Java Map 学习笔记'
pubDate: 'Sep 29 2026'
tags: ['Java', '集合', 'Map']
---

## 一、如何对 Map 进行快速遍历？

`Map` 不是直接实现 `Iterable` 的集合，不能直接使用增强 `for` 遍历。通常需要先通过 `entrySet()`、`keySet()` 或 `values()` 获取视图，再进行遍历。

### 1. 使用 `entrySet()` 遍历键值对

如果同时需要键和值，推荐使用 `entrySet()`。每个 `Map.Entry` 都保存一组键值对，可以直接调用 `getKey()` 和 `getValue()`，不需要再次通过键查找值。

```java
import java.util.HashMap;
import java.util.Map;

public class MapTraversalExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        map.put("key1", 1);
        map.put("key2", 2);
        map.put("key3", 3);

        for (Map.Entry<String, Integer> entry : map.entrySet()) {
            System.out.println("Key: " + entry.getKey()
                    + ", Value: " + entry.getValue());
        }
    }
}
```

这是遍历 Map 键值对最常用、最直接的方式。

### 2. 使用 `keySet()` 遍历键

如果只需要 Map 中的键，可以使用 `keySet()`：

```java
for (String key : map.keySet()) {
    System.out.println("Key: " + key);
}
```

如果还需要值，可以调用 `map.get(key)`：

```java
for (String key : map.keySet()) {
    System.out.println("Key: " + key + ", Value: " + map.get(key));
}
```

不过，如果本来就需要键和值，优先使用 `entrySet()`，这样可以直接从 Entry 中获取值，写法更清晰。

### 3. 使用迭代器遍历

如果需要在遍历过程中删除元素，可以使用 `entrySet()` 的迭代器，并通过 `iterator.remove()` 删除当前元素：

```java
import java.util.Iterator;
import java.util.Map;

Iterator<Map.Entry<String, Integer>> iterator = map.entrySet().iterator();
while (iterator.hasNext()) {
    Map.Entry<String, Integer> entry = iterator.next();

    if (entry.getValue() < 2) {
        iterator.remove();
    }
}
```

遍历过程中不要直接调用 `map.remove(key)` 修改 Map 的结构，否则可能抛出 `ConcurrentModificationException`。使用迭代器自己的 `remove()` 方法可以正确同步迭代器状态。

### 4. 使用 `Map.forEach()`

Java 8 提供了 `Map.forEach()` 方法，可以通过 Lambda 表达式简洁地遍历键值对：

```java
map.forEach((key, value) ->
        System.out.println("Key: " + key + ", Value: " + value));
```

如果只是读取并处理每个键值对，这种写法比较简洁；如果需要在遍历过程中删除元素，应使用迭代器或 `removeIf()` 等专门方法。

### 5. 使用 Stream API 遍历和处理

可以将 `entrySet()` 转换为 Stream，再进行过滤、映射和收集等操作：

```java
import java.util.Map;
import java.util.stream.Collectors;

map.entrySet()
        .stream()
        .forEach(entry -> System.out.println(
                "Key: " + entry.getKey() + ", Value: " + entry.getValue()));

Map<String, Integer> filteredMap = map.entrySet()
        .stream()
        .filter(entry -> entry.getValue() > 1)
        .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue
        ));
```

Stream 更适合在遍历的同时进行过滤、转换或汇总。如果只是简单打印键值对，`entrySet()` 配合增强 `for` 或 `forEach()` 就足够了。

### 6. 选择建议

- 同时需要键和值：优先使用 `entrySet()`。
- 只需要键：使用 `keySet()`。
- 只需要值：使用 `values()`。
- 遍历时删除元素：使用迭代器的 `remove()`。
- 需要过滤、转换、分组或汇总：使用 Stream API。
- 只做简单遍历：使用 `Map.forEach()` 或增强 `for`，代码更直观。

## 二、HashMap 的实现原理是什么？

`HashMap` 的核心数据结构是“数组 + 链表 + 红黑树”。它通过键的 `hashCode()` 计算哈希值，再根据哈希值确定元素应该存放在哪个数组下标，也就是哪个桶（Bucket）中。

### 1. JDK 1.7 及以前：数组加链表

在 JDK 1.7 及以前，`HashMap` 的底层主要由数组和链表组成：

1. 对 Key 调用 `hashCode()`，得到哈希值。
2. 根据哈希值计算数组下标。
3. 如果该下标没有元素，直接保存新节点。
4. 如果该下标已经有元素，说明发生哈希冲突，就将多个节点连接成链表。

如果大量 Key 被定位到同一个桶中，链表会变得很长，查找时需要依次比较节点，时间复杂度可能退化到 `O(n)`。

### 2. JDK 1.8：链表过长时转换为红黑树

JDK 1.8 对冲突处理进行了优化：当同一个桶中的链表过长时，会将链表转换为红黑树，使查找时间复杂度从 `O(n)` 降低到接近 `O(log n)`。

常见的相关阈值如下：

- `TREEIFY_THRESHOLD = 8`：桶中节点数量达到这个阈值时，具备树化条件。
- `MIN_TREEIFY_CAPACITY = 64`：哈希表容量至少达到 `64` 才会树化。
- `UNTREEIFY_THRESHOLD = 6`：扩容拆分树桶时，如果节点数量较少，可能退化回链表。

当链表长度达到 `8`，但当前数组长度还小于 `64` 时，`HashMap` 通常会优先扩容，而不是立即树化。这样可以通过扩大数组容量来减少哈希冲突。

### 3. `put()` 的大致流程

向 `HashMap` 中添加键值对时，大致经过以下过程：

```text
计算 Key 的 hashCode
        ↓
对哈希值进行扰动，减少冲突
        ↓
根据哈希值计算桶下标
        ↓
桶为空：直接插入新节点
桶不为空：比较 hash 和 equals
        ↓
Key 相同：替换旧值
Key 不同：追加到链表或红黑树
        ↓
元素数量超过扩容阈值：扩容
```

可以用下面的简化代码理解桶下标的计算方式。`HashMap` 的数组长度通常保持为 `2` 的幂，这样可以使用位运算快速计算下标：

```java
int index = (table.length - 1) & hash;
```

这段代码只是帮助理解的简化写法，实际 JDK 源码还会进行哈希扰动、空表判断、扩容判断以及链表或红黑树处理。

### 4. `get()` 的大致流程

调用 `get(key)` 时，`HashMap` 会：

1. 根据 Key 计算哈希值。
2. 根据哈希值定位到对应桶。
3. 检查桶中的第一个节点。
4. 如果 Key 不相等，就继续遍历链表，或在红黑树中查找。
5. 找到相同 Key 后返回对应的 Value；找不到则返回 `null`。

判断两个 Key 是否相同时，不能只比较哈希值，还需要继续比较 `equals()`。因为不同对象可能产生相同的哈希值，这种情况称为哈希冲突。

### 5. 扩容机制

`HashMap` 默认负载因子通常为 `0.75`。当元素数量超过：

```text
扩容阈值 = 当前容量 × 负载因子
```

就会触发扩容，通常将数组容量扩大为原来的 `2` 倍，并重新计算节点在新数组中的位置。

扩容需要创建新数组并重新分配节点，因此会产生一定的性能开销。如果能够预估 Map 的元素数量，可以在创建时指定合理的初始容量，减少扩容次数：

```java
Map<String, Integer> map = new HashMap<>(128);
```

这里的初始容量不是最终元素数量，实际使用时还要结合负载因子计算扩容阈值。

### 6. 为什么 Key 要正确重写 `equals()` 和 `hashCode()`？

`HashMap` 先根据 `hashCode()` 定位桶，再通过 `equals()` 判断 Key 是否相同。因此，自定义对象作为 Key 时，必须同时重写 `equals()` 和 `hashCode()`，并且要保证：

```text
如果两个对象 equals() 返回 true，两个对象的 hashCode() 必须相同。
```

```java
class User {
    private final Long id;

    User(Long id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object object) {
        if (this == object) {
            return true;
        }
        if (!(object instanceof User other)) {
            return false;
        }
        return id.equals(other.id);
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }
}
```

如果只重写 `equals()` 而没有重写 `hashCode()`，逻辑上相等的两个对象可能被定位到不同的桶中，导致 `get()` 查找不到已经放入的值。

## 三、什么是红黑树？它有哪些特性？

红黑树（Red-Black Tree）是一种带颜色标记的自平衡二叉搜索树。它通过一组颜色规则限制树的高度，使查找、插入和删除操作在最坏情况下都能保持 `O(log n)` 的时间复杂度。

红黑树中的每个节点都有两种颜色：红色或黑色。节点之间仍然遵循二叉搜索树的大小关系：左子树中的元素小于当前节点，右子树中的元素大于当前节点。

### 红黑树的五条性质

1. 每个节点必须是红色或黑色。
2. 根节点必须是黑色。
3. 每个叶子节点（通常用表示空节点的 `NIL` 节点表示）必须是黑色。
4. 红色节点不能有红色子节点，也就是不能出现两个连续的红色节点。
5. 从任意节点到它的所有后代 `NIL` 节点的每条路径，都必须包含相同数量的黑色节点，这个数量称为黑高。

插入或删除节点后，如果树的颜色规则被破坏，就需要通过重新染色和旋转恢复平衡。红黑树不要求左右子树高度完全相同，而是通过这些规则保证树不会退化成很长的链表。

### 红黑树为什么能保持较好的性能？

红黑树属于“弱平衡”结构，平衡要求没有 AVL 树那么严格，但调整成本更低。它的树高仍然与 `log n` 成正比，因此查找、插入和删除的最坏时间复杂度都是 `O(log n)`。

```text
查找：沿着树的大小关系向左或向右查找
插入：插入节点后，通过染色和旋转恢复规则
删除：删除节点后，通过染色和旋转恢复规则
```

红黑树中的旋转只会改变局部节点的连接关系，不会破坏二叉搜索树的有序性。正是通过“颜色约束 + 局部旋转”，红黑树在更新操作和查询性能之间取得了平衡。

## 四、为什么 HashMap 使用红黑树，而不是 AVL 树或 B+ 树？

`HashMap` 使用红黑树，主要是因为它需要在哈希冲突严重时降低桶内查找的最坏复杂度，同时还要频繁处理插入、删除和扩容。红黑树在这些操作之间取得了比较合适的平衡。

### 1. 为什么不使用 AVL 树？

AVL 树是严格平衡的二叉搜索树，任意节点的左右子树高度差不会超过 `1`。因此，AVL 树的查询路径通常更短，查询性能略好一些。

但 AVL 树的问题是：每次插入或删除后，都可能需要从底部向上更新高度并进行多次旋转，维护成本较高。

红黑树的平衡条件相对宽松，查询路径可能比 AVL 树稍长，但插入和删除时通常只需要较少的染色和旋转操作。`HashMap` 既要处理插入，也要处理删除，因此更看重整体更新成本，而不是只追求极致的查询路径长度。

| 对比维度 | AVL 树 | 红黑树 |
| --- | --- | --- |
| 平衡要求 | 严格平衡 | 弱平衡 |
| 查询 | 通常略快 | 也能保持 `O(log n)` |
| 插入、删除 | 调整可能更频繁 | 调整通常更少 |
| 适用场景 | 查询远多于更新 | 查询和更新都较频繁 |

另外，`HashMap` 中的红黑树只用于冲突严重的桶，并不是所有键值对都会进入红黑树。正常情况下，哈希表仍然主要依靠数组实现接近 `O(1)` 的平均查找性能。

### 2. 为什么不使用 B/B+ 树？

B 树和 B+ 树是多路平衡搜索树，一个节点可以保存多个 Key 和多个子节点。它们的主要目标是降低磁盘或其他外部存储中的 I/O 次数，常见于数据库索引和文件系统。

`HashMap` 是纯内存数据结构，不需要通过减少磁盘 I/O 来提升性能。对于内存中的哈希冲突桶来说，红黑树的结构更简单，节点调整成本更低，也更适合直接使用对象引用进行查找。

| 对比维度 | 红黑树 | B/B+ 树 |
| --- | --- | --- |
| 节点结构 | 一个节点通常保存一个 Key | 一个节点可以保存多个 Key |
| 主要目标 | 平衡内存中的查找与更新 | 减少磁盘 I/O 和树的层数 |
| 常见场景 | 内存数据结构、冲突桶 | 数据库索引、文件系统 |
| 是否适合 HashMap | 适合 | 结构和目标都偏复杂 |

### 3. 总结

- `HashMap` 平均情况下依靠哈希定位，查找时间接近 `O(1)`。
- 只有当大量 Key 发生哈希冲突、同一个桶中的链表过长时，才可能树化为红黑树。
- 红黑树比 AVL 树更适合频繁插入、删除和查询的综合场景。
- B/B+ 树主要为磁盘或外部存储设计，`HashMap` 的数据位于内存中，不需要优先考虑磁盘 I/O。

## 五、HashMap 的 `put(key, value)` 和 `get(key)` 过程

### 1. `put(key, value)` 的过程

向 `HashMap` 中存储键值对时，大致会经历以下步骤：

1. 根据 Key 计算哈希值。JDK 会对 `hashCode()` 的结果进行扰动，尽量让高位信息也参与桶下标计算。
2. 根据哈希值计算桶下标。数组长度为 `n` 时，通常使用 `(n - 1) & hash` 计算位置。
3. 如果哈希表还没有初始化，先创建底层数组。
4. 如果目标桶为空，直接创建节点保存键值对。
5. 如果目标桶不为空，先比较哈希值，再通过 `equals()` 判断 Key 是否相同：
   - Key 相同：替换旧 Value，并返回旧 Value。
   - Key 不同：继续在链表或红黑树中查找合适位置。
6. 如果发生冲突，将新节点放入链表或红黑树中。
7. 新增元素后，如果元素数量超过扩容阈值，就进行扩容。

```text
put(key, value)
      ↓
计算 hash
      ↓
计算桶下标
      ↓
桶为空？──── 是 ────> 创建新节点
  否
      ↓
Key 已存在？── 是 ───> 替换 Value
  否
      ↓
放入链表或红黑树
      ↓
超过扩容阈值？── 是 ─> 扩容
```

```java
Map<String, Integer> map = new HashMap<>();

map.put("Java", 100);
map.put("Java", 200);

System.out.println(map.get("Java")); // 200
```

### 2. `get(key)` 的过程

从 `HashMap` 中读取 Value 时，主要过程如下：

1. 根据 Key 计算哈希值。
2. 根据哈希值定位到目标桶。
3. 检查桶中的第一个节点。
4. 如果第一个节点的 Key 不匹配，就继续遍历链表，或者在红黑树中查找。
5. 找到相同的 Key 后返回对应的 Value；找不到则返回 `null`。

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", 200);

Integer value = map.get("Java");
System.out.println(value); // 200
```

### 3. 哈希冲突时如何查找？

哈希值相同并不代表 Key 一定相同。`HashMap` 需要先比较哈希值，再调用 `equals()` 进行最终判断：

```text
hashCode() 相同？
      ↓ 是
equals() 返回 true？
      ↓ 是
判定为同一个 Key
```

因此，自定义对象作为 Key 时，必须同时正确重写 `equals()` 和 `hashCode()`。

### 4. 扩容过程

以默认配置为例，`HashMap` 的初始容量通常为 `16`，负载因子为 `0.75`，扩容阈值为：

```text
16 × 0.75 = 12
```

当元素数量超过扩容阈值时，底层数组通常扩容为原来的 `2` 倍，并重新调整节点位置。数组容量保持为 `2` 的幂，既方便使用位运算计算桶下标，也便于根据高位信息快速判断节点是否需要移动到新的位置。

扩容会创建新数组并重新分配节点，因此会带来一次性性能开销。可以根据预计的元素数量设置合理的初始容量，减少扩容次数：

```java
Map<String, Integer> map = new HashMap<>(128);
```

### 5. 常见集合的扩容对比

| 集合类 | 常见扩容倍数 | 主要原因 |
| --- | --- | --- |
| `HashMap` | `2` 倍 | 容量保持为 `2` 的幂，便于使用 `(n - 1) & hash` 计算桶下标 |
| `ArrayList` | 约 `1.5` 倍 | 底层是动态数组，在空间利用率和复制成本之间取平衡 |
| `ConcurrentHashMap` | 通常按 `2` 倍扩展 | 底层结构与 `HashMap` 类似，也需要保持合适的桶数量 |

### 6. 时间复杂度

- 没有严重哈希冲突时，`put()` 和 `get()` 的平均时间复杂度接近 `O(1)`。
- 冲突节点较少时，需要遍历链表，最坏可能达到 `O(n)`。
- 冲突桶树化后，桶内查找的最坏复杂度可以降低到 `O(log n)`。
- 扩容需要重新分配节点，单次操作可能产生 `O(n)` 的开销，但正常情况下不会频繁发生。

## 六、HashMap 的 Key 可以为 `null` 吗？

可以。`HashMap` 允许一个 `null` Key，也允许多个 `null` Value。

### 1. `null` Key 的哈希值如何计算？

在计算哈希值时，如果 Key 是 `null`，`HashMap` 不会调用 `key.hashCode()`，而是直接把哈希值当作 `0`：

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

因此，`null` Key 会被定位到哈希表的特定桶中，通常是下标为 `0` 的桶。

### 2. `null` Key 只能有一个

`Map` 中 Key 不能重复。多次使用 `null` 作为 Key 时，后一次 `put()` 会覆盖前一次保存的 Value：

```java
import java.util.HashMap;
import java.util.Map;

public class HashMapNullKeyExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();

        map.put(null, 1);
        map.put(null, 2);      // 覆盖之前的 Value
        map.put("Java", null);
        map.put("集合", null);

        System.out.println(map.get(null)); // 2
        System.out.println(map.size());    // 3
    }
}
```

这里的三个键分别是 `null`、`"Java"` 和 `"集合"`，所以集合大小为 `3`。虽然 `null` Value 可以有多个，但每个 Key 仍然只能对应一个 Value。

### 3. `get()` 返回 `null` 时要注意

`get(key)` 返回 `null` 可能有两种情况：

1. Map 中不存在这个 Key。
2. Map 中存在这个 Key，但对应的 Value 就是 `null`。

如果需要区分这两种情况，应使用 `containsKey()`：

```java
Map<String, Integer> map = new HashMap<>();
map.put("Java", null);

System.out.println(map.get("Java"));       // null
System.out.println(map.get("集合"));       // null

System.out.println(map.containsKey("Java")); // true
System.out.println(map.containsKey("集合")); // false
```

### 4. 其他 Map 实现的限制

不同 Map 实现对 `null` 的支持不同：

- `HashMap`：允许一个 `null` Key 和多个 `null` Value。
- `Hashtable`：不允许 `null` Key，也不允许 `null` Value。
- `ConcurrentHashMap`：不允许 `null` Key，也不允许 `null` Value，避免并发场景下无法区分“没有映射”和“映射值为 null”。
- `TreeMap`：使用自然排序时通常不允许 `null` Key；如果自定义比较器明确支持 `null`，则需要根据比较器的实现判断。

## 七、重写 HashMap 的 `equals()` 方法不重写 `hashCode()` 会有什么问题？

`HashMap` 查找键值对时，会先根据 Key 的 `hashCode()` 定位桶，再使用 `equals()` 判断两个 Key 是否相同。因此，自定义对象作为 Key 时，必须同时正确重写 `equals()` 和 `hashCode()`。

### 1. `equals()` 和 `hashCode()` 的约定

Java 对这两个方法有一个重要约定：

```text
如果两个对象 equals() 返回 true，
那么它们的 hashCode() 必须相同。
```

反过来，两个对象的 `hashCode()` 相同，并不能说明它们一定相等，因为不同对象可能发生哈希冲突。

`HashMap` 的查找过程可以简化为：

```text
先比较 hashCode()
      ↓
哈希值相同后，再比较 equals()
      ↓
两个条件都满足，才认为是同一个 Key
```

### 2. 只重写 `equals()` 的问题

如果只重写 `equals()`，没有重写 `hashCode()`，对象会继续使用 `Object.hashCode()`。即使两个对象的业务属性相同，它们也可能得到不同的哈希值，被定位到不同的桶中。

```java
import java.util.HashMap;
import java.util.Map;

class User {
    private final Long id;

    User(Long id) {
        this.id = id;
    }

    @Override
    public boolean equals(Object object) {
        if (this == object) {
            return true;
        }
        if (!(object instanceof User other)) {
            return false;
        }
        return id.equals(other.id);
    }

    // 故意没有重写 hashCode()
}

public class HashMapEqualsExample {
    public static void main(String[] args) {
        Map<User, String> map = new HashMap<>();
        map.put(new User(1L), "Kevin");

        User key = new User(1L);
        System.out.println(key.equals(new User(1L))); // true
        System.out.println(map.get(key));             // 可能是 null
    }
}
```

虽然两个 `User` 的 `id` 相同，`equals()` 返回了 `true`，但它们可能因为哈希值不同而被定位到不同的桶中。`HashMap` 找不到目标桶中的节点，就无法继续通过 `equals()` 完成匹配。

### 3. 可能出现的具体问题

- 使用逻辑上相同的对象作为 Key 查询时，`get()` 返回 `null`。
- `containsKey()` 判断结果为 `false`。
- 两个逻辑上相同的 Key 可能被当成不同 Key 保存，导致 Map 中出现重复数据。
- 后续 `put()` 无法按照预期覆盖原来的 Value。

### 4. 正确写法

重写 `equals()` 的同时，使用参与相等性判断的字段重写 `hashCode()`：

```java
@Override
public int hashCode() {
    return id.hashCode();
}
```

完整的 Key 类应当保证：只要两个对象的 `id` 相同，`equals()` 就返回 `true`，并且 `hashCode()` 也返回相同结果。

```java
User first = new User(1L);
User second = new User(1L);

System.out.println(first.equals(second));
System.out.println(first.hashCode() == second.hashCode());
```

实际开发中，可以使用 IDE 自动生成 `equals()` 和 `hashCode()`，并且尽量使用不可变字段作为 Key。对象放入 `HashMap` 后，不要修改参与 `equals()` 和 `hashCode()` 计算的字段，否则对象可能仍在原来的桶中，但已经无法按照新的哈希值找到。

## 八、ConcurrentHashMap 是怎么实现线程安全的？

`ConcurrentHashMap` 通过限制并发修改的范围来实现线程安全。不同 JDK 版本的实现方式不同，最重要的区别是：JDK 1.7 使用分段锁，JDK 1.8 以后取消了 `Segment`，改为数组桶级别的控制。

### 1. JDK 1.7：分段锁

JDK 1.7 的 `ConcurrentHashMap` 主要由 `Segment[]` 和每个 Segment 内部的 `HashEntry[]` 组成：

```text
ConcurrentHashMap
        ↓
Segment[]
        ↓
每个 Segment 内部有一个 HashEntry[]
        ↓
HashEntry 链表保存键值对
```

每个 `Segment` 都是一把可重入锁，可以理解为一个较小的 HashMap。多个线程访问不同 Segment 时，可以同时执行；只有访问同一个 Segment 时，才需要竞争同一把锁。

例如，一个线程正在修改 Segment 1，另一个线程仍然可以访问 Segment 2，因此并发度比给整个 Map 加一把大锁更高。

```text
线程 A ──> Segment 1：加锁修改
线程 B ──> Segment 2：可以并发访问
线程 C ──> Segment 1：等待线程 A 释放锁
```

这种方式的优点是实现清晰、并发性能比整体加锁好；缺点是锁的粒度仍然比较大，同一个 Segment 中的操作会互相影响，而且 Segment 数量在创建后通常不能动态改变。

### 2. JDK 1.8：CAS 加 synchronized

JDK 1.8 取消了 `Segment`，底层结构更接近 `HashMap`：

```text
ConcurrentHashMap
        ↓
Node[] 数组
        ↓
链表或红黑树
```

JDK 1.8 主要通过 `volatile`、CAS 和 `synchronized` 配合实现线程安全：

1. 如果目标桶为空，使用 CAS 尝试把新节点放入桶中。
2. 如果目标桶不为空，则锁住当前桶的头节点，再遍历链表或红黑树完成新增、修改和删除。
3. 如果多个线程操作不同的桶，它们通常可以并发执行。
4. 如果同一个桶被多个线程同时修改，只有竞争到该桶锁的线程可以执行修改操作。
5. 如果链表过长并且数组容量达到要求，会将链表转换为红黑树。

可以把写操作简化理解为：

```text
目标桶为空？
    是：CAS 设置新节点
    否：synchronized 锁住桶头节点
            ↓
        遍历链表或红黑树
            ↓
        新增、修改或删除节点
```

### 3. `volatile`、CAS 和 `synchronized` 分别做什么？

- `volatile`：保证数组引用、节点引用等共享状态的可见性，让其他线程能够及时看到更新结果。
- CAS：在不加锁的情况下，尝试安全地完成空桶的初始化或节点写入；如果同时有其他线程修改，CAS 会失败并重试。
- `synchronized`：保护已经存在节点的桶，保证同一个桶中的链表或红黑树修改过程不会被多个线程同时破坏。

`volatile` 本身不能保证复合操作的原子性，所以不能只依靠 `volatile` 完成并发写入；CAS 和 `synchronized` 分别负责不同情况下的原子更新和互斥修改。

### 4. JDK 1.8 为什么比 JDK 1.7 更细粒度？

JDK 1.7 的锁对象是 `Segment`，一个 Segment 负责一部分桶；JDK 1.8 的锁对象通常是发生冲突的桶头节点。也就是说，JDK 1.8 把锁的范围从 Segment 缩小到了桶级别：

| 版本 | 主要结构 | 加锁粒度 | 空桶写入 |
| --- | --- | --- | --- |
| JDK 1.7 | `Segment[]` + `HashEntry[]` | Segment | 通过 Segment 锁保护 |
| JDK 1.8+ | `Node[]` + 链表/红黑树 | 冲突桶 | 使用 CAS 尝试写入 |

因此，JDK 1.8 可以减少不同数据之间的锁竞争。读取操作通常不需要加锁，多个线程可以并发读取；只有发生冲突的写操作才会在同一个桶上竞争。

### 5. 为什么还要使用红黑树？

`ConcurrentHashMap` 在 JDK 1.8 中也使用红黑树处理严重的哈希冲突。当某个桶中的链表过长时，树化可以将桶内查询的最坏复杂度从 `O(n)` 降低到 `O(log n)`，避免冲突严重时查询效率大幅下降。

### 6. 使用时的注意事项

- `ConcurrentHashMap` 不允许 `null` Key 和 `null` Value。
- 它保证单次操作的线程安全，但复杂的多步业务逻辑仍然需要额外设计。
- `size()`、遍历等操作在并发修改期间反映的是某个时刻的状态，不应简单理解为全局静态快照。
- 如果只是单线程使用，`HashMap` 通常更简单；只有在确实存在并发访问需求时，才选择 `ConcurrentHashMap`。

## 九、CAS 是什么？

CAS 是 **Compare-And-Swap** 的缩写，中文通常称为“比较并交换”。它是一种并发更新机制：只有当共享变量当前的值仍然等于预期值时，才会把它修改为新值；如果当前值已经被其他线程修改，更新就会失败。

### 1. CAS 的三个参数

一次 CAS 操作通常包含三个值：

- 内存位置：需要修改的共享变量。
- 预期值：线程认为变量当前应该具有的值。
- 新值：比较成功后要写入的值。

可以把 CAS 理解为下面的原子操作：

```text
如果 当前值 == 预期值：
    当前值 = 新值
    返回 true
否则：
    不修改当前值
    返回 false
```

比较和替换是一个不可分割的整体，中间不会被其他线程插入修改。

### 2. Java 中的简单示例

`AtomicInteger` 提供了 CAS 操作：

```java
import java.util.concurrent.atomic.AtomicInteger;

public class CasExample {
    public static void main(String[] args) {
        AtomicInteger number = new AtomicInteger(0);

        boolean first = number.compareAndSet(0, 1);
        System.out.println(first);       // true
        System.out.println(number.get()); // 1

        // 当前值已经是 1，不再等于预期值 0，所以更新失败
        boolean second = number.compareAndSet(0, 2);
        System.out.println(second);      // false
        System.out.println(number.get()); // 1
    }
}
```

### 3. CAS 如何实现自增？

自增不是单独的一步，而是“读取旧值、计算新值、写回新值”三个步骤。CAS 通常会配合循环使用：如果写回时发现旧值已经被其他线程修改，就重新读取并重试。

```java
AtomicInteger counter = new AtomicInteger(0);

int oldValue;
int newValue;
do {
    oldValue = counter.get();
    newValue = oldValue + 1;
} while (!counter.compareAndSet(oldValue, newValue));
```

多个线程同时执行时，只有一个线程能成功把同一个旧值替换掉；失败的线程会重新读取最新值，再次尝试。

### 4. CAS 在 ConcurrentHashMap 中的作用

JDK 1.8 的 `ConcurrentHashMap` 在向空桶写入第一个节点时，可以使用 CAS：

```text
桶为空
    ↓
CAS 尝试把新节点放入桶中
    ↓
成功：直接完成写入
失败：说明其他线程已经修改，重新检查或竞争桶锁
```

这样，不同线程向不同空桶写入时不需要竞争一把全局锁，可以提高并发性能。对于已经存在链表或红黑树的桶，则通常使用 `synchronized` 保护后续修改。

### 5. CAS 的优点和限制

优点：

- 不需要阻塞线程，低竞争场景下效率较高。
- 可以减少锁竞争和线程上下文切换。
- 适合更新单个共享变量或单个引用。

限制：

- 高竞争场景下，失败线程会不断重试，浪费 CPU 资源。
- CAS 一次主要只能保证一个共享变量的原子更新，涉及多个变量的一致性时，通常仍需要锁。
- 可能遇到 ABA 问题：变量从 `A` 变成 `B`，又变回 `A`，CAS 只比较最终值时可能无法发现中间发生过变化。

如果需要同时记录版本变化，可以使用 `AtomicStampedReference` 为值增加版本号，避免只比较值而忽略中间变化。
