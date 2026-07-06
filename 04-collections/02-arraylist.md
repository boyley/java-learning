# 02 · ArrayList

> 基于**动态数组**实现的 List，默认容量 10、扩容 1.5 倍，随机访问 O(1) 快、中间增删 O(n) 慢，非线程安全且 fail-fast。面试重要度：⭐⭐⭐。

## 📖 核心知识

`ArrayList` 底层是一个 `Object[] elementData` 数组，用 `size` 记录实际元素个数。核心特性：**用连续内存存储，靠下标随机访问**。

**默认容量与懒加载**：JDK 8 中 `new ArrayList()` 并不会立即分配 10 个空间，`elementData` 先指向一个空数组 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA`；直到**第一次 `add`** 时才真正扩容到默认容量 **10**。这是一种延迟初始化，避免 new 出大量空数组浪费内存。

**扩容机制（1.5 倍）**：`add` 时若 `size + 1 > 数组长度`，触发 `grow()`：

```java
int newCapacity = oldCapacity + (oldCapacity >> 1); // 旧容量 + 旧容量/2 ≈ 1.5 倍
// 10 → 15 → 22 → 33 → ...
Arrays.copyOf(elementData, newCapacity); // 底层 System.arraycopy 拷贝到新数组
```

`oldCapacity >> 1` 是右移一位（除以 2），所以新容量约为原来的 1.5 倍。扩容要**新建数组 + 拷贝**，是较重的操作，因此已知数据量时应 `new ArrayList<>(初始容量)` 或用 `ensureCapacity` 预扩容，避免多次扩容。

**增删性能**：
- `get(i)`/`set(i)`：直接按下标访问，**O(1)**。
- `add(E)` 尾部添加：均摊 **O(1)**（偶尔触发扩容）。
- `add(i, E)`/`remove(i)` 中间增删：要 `System.arraycopy` 移动后续元素，**O(n)**。

**fail-fast**：`ArrayList` 有一个 `modCount` 字段记录结构性修改次数，迭代过程中若检测到 `modCount` 被意外改变（如遍历时直接 `list.remove()`），立即抛 `ConcurrentModificationException`（详见 [10-fail-fast-fail-safe](./10-fail-fast-fail-safe.md)）。

## 🔑 面试要点

- 底层是 `Object[]` 动态数组，内存连续，支持按下标 O(1) 随机访问。
- 默认容量 **10**，但 JDK 8 是**懒加载**：第一次 `add` 才分配。
- 扩容 **1.5 倍**（`oldCap + oldCap>>1`），扩容需拷贝数组，代价高。
- 尾部 add 均摊 O(1)；中间 add/remove 要移动元素，O(n)。
- 非线程安全；`modCount` 实现 fail-fast。
- `Arrays.asList()` 返回的是固定大小的视图，`add`/`remove` 会抛 `UnsupportedOperationException`。

## ❓ 高频面试题

**Q：ArrayList 默认容量是多少？无参构造会立即分配吗？**
A：默认容量 10。但 JDK 8 起无参构造只是把内部数组指向空数组，**第一次 add 时**才扩容到 10（懒加载，省内存）。

**Q：ArrayList 怎么扩容？为什么是 1.5 倍？**
A：`grow()` 里 `newCap = oldCap + (oldCap >> 1)`，即 1.5 倍。选 1.5 是**空间与时间的折中**：太小频繁扩容拷贝，太大浪费内存；1.5 倍还便于 JVM 复用之前释放的内存块。扩容底层用 `Arrays.copyOf`/`System.arraycopy`。

**Q：遍历时删除元素为什么会抛异常？怎么正确删除？**
A：增强 for 底层是迭代器，`list.remove()` 改了 `modCount` 触发 fail-fast。正确做法是用 `Iterator.remove()`，或 JDK 8 的 `list.removeIf(条件)`。

**Q：如何让 ArrayList 线程安全？**
A：`Collections.synchronizedList(list)`（方法级同步）或读多写少用 `CopyOnWriteArrayList`。

## ⚠️ 易错点 / 加分项

- 别说默认「构造时就分配 10 个」——JDK 8 是懒加载，第一次 add 才分配。
- 扩容后**旧数组会被 GC 回收**，短时间内新旧数组同时存在，大 List 扩容可能引起内存抖动，所以要预设容量。
- `remove(int index)`（按下标）和 `remove(Object o)`（按对象）是重载易混：`list.remove(1)` 删的是**下标 1**，删 Integer 值 1 要 `remove(Integer.valueOf(1))`。
- 加分：`elementData` 用 `transient` 修饰，`ArrayList` 自定义了 `writeObject`/`readObject` 只序列化实际存在的元素，避免把预留的空容量也序列化，节省空间。
