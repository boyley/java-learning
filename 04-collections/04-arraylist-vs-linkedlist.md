# 04 · ArrayList vs LinkedList

> 一句话：随机访问、读多写少用 `ArrayList`；纯头尾频繁增删用 `LinkedList`（但多数场景 ArrayList 更实用）。面试重要度：⭐⭐⭐（经典对比题）。

## 📖 核心知识

两者都实现 `List`、都非线程安全、都有 fail-fast，区别全在**底层数据结构**上。

### 对比表

| 维度 | ArrayList | LinkedList |
| --- | --- | --- |
| 底层结构 | 动态数组 `Object[]` | 双向链表 `Node`(prev/next) |
| 随机访问 `get(i)` | **O(1)** 下标直达 | **O(n)** 需遍历定位 |
| 尾部添加 `add(e)` | 均摊 O(1)（偶尔扩容） | O(1) |
| 头部/中间增删 | O(n) 需移动元素 | O(1) 改指针（前提已定位）；按下标仍 O(n) |
| 内存占用 | 较省，仅扩容有预留空位 | 较大，每节点多 2 个指针 + 对象头 |
| 扩容 | 有，1.5 倍需拷贝数组 | 无，按需 new 节点 |
| 缓存友好性 | 好（内存连续） | 差（节点分散） |
| 额外能力 | 仅 List | 实现 Deque，可当栈/队列/双端队列 |
| 适用场景 | 随机访问、读多写少、遍历 | 头尾频繁增删、用作队列/栈 |

### 关键澄清：LinkedList 增删并不总是快

很多人背「数组增删慢、链表增删快」，但 `LinkedList` 按下标 `add(i,e)` 要先 O(n) 遍历找位置。真正快的是**头尾操作**和**迭代器遍历中删除**。而 `ArrayList` 虽然中间增删要移动元素（`System.arraycopy` 是底层内存批量拷贝，很快），实测中小数据量下往往不比 LinkedList 慢。

## 🔑 面试要点

- 底层：ArrayList = 数组；LinkedList = 双向链表。
- 随机访问：ArrayList O(1) 完胜；LinkedList O(n)。
- 增删：LinkedList 头尾 O(1)；但按下标增删仍需 O(n) 遍历定位。
- 内存：LinkedList 每节点多两个指针，占用更大；ArrayList 扩容有预留浪费。
- 扩容：只有 ArrayList 有（1.5 倍拷贝）。
- LinkedList 额外实现 Deque，可当栈/队列。
- 实战 90% 场景选 ArrayList（更快、更省、缓存友好）。

## ❓ 高频面试题

**Q：什么时候用 LinkedList，什么时候用 ArrayList？**
A：需要按下标随机访问、遍历、读多写少 → `ArrayList`；只在头尾频繁增删、或要当队列/栈用 → `LinkedList`（或更推荐 `ArrayDeque`）。实际大多数情况 `ArrayList` 就是最优解。

**Q：都说链表增删快，为什么实际中 ArrayList 反而常更快？**
A：因为 `LinkedList` 按下标增删要先 O(n) 遍历找节点，而 `ArrayList` 的元素移动是 `System.arraycopy` 底层批量内存拷贝，配合 CPU 缓存局部性，中小数据量下常常更快。链表的优势仅在「已定位节点」或「头尾操作」时才体现。

**Q：ArrayList 和 LinkedList 内存谁大？**
A：一般 `LinkedList` 更大，每个元素额外携带 prev/next 引用和对象头；`ArrayList` 只有扩容后未填满的预留空间浪费。

## ⚠️ 易错点 / 加分项

- 别绝对化「链表增删一定快」——脱离「是否已定位节点」谈复杂度是错的。
- CPU 缓存局部性是加分点：数组连续存储命中缓存行，链表节点内存分散，遍历性能实际差距被放大。
- 加分：`ArrayList` 实现了 `RandomAccess` 标记接口，`Collections.binarySearch` 等会据此走下标遍历；`LinkedList` 未实现，走迭代器遍历。
- 加分：真要频繁头尾操作，`ArrayDeque` 通常比 `LinkedList` 更优（无节点对象开销）。
