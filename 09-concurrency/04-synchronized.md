# 04 · synchronized 与锁升级（synchronized）

> `synchronized` 是 JVM 内置锁，保证**原子性、可见性、有序性**；JDK 6 后引入**偏向锁→轻量级锁→重量级锁**的锁升级优化，不再是「重量级」代名词。理解 Mark Word 与锁升级是并发面试的**核心必考点**。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**三种用法**：锁的粒度取决于锁对象。

```java
// ① 修饰实例方法：锁当前实例 this
public synchronized void m1() {}
// ② 修饰静态方法：锁当前类的 Class 对象（全局唯一）
public static synchronized void m2() {}
// ③ 修饰代码块：锁括号里指定的对象
public void m3() { synchronized (lock) { /* ... */ } }
```

**底层：monitor（管程/监视器）**。`synchronized` 依赖对象的**监视器锁 monitor**。同步代码块编译后前后是 **`monitorenter`** 和 **`monitorexit`** 字节码指令（同步方法则是 `ACC_SYNCHRONIZED` 标志）。每个对象关联一个 monitor，`monitorenter` 时计数器 +1（0→1 获取成功），可重入则继续 +1，`monitorexit` 时 -1，减到 0 释放。异常也会自动 `monitorexit`（多一个异常处理的 exit），保证锁释放。

**对象头与 Mark Word**。对象在堆中由**对象头 + 实例数据 + 对齐填充**组成。对象头含 **Mark Word**（存哈希码、GC 分代年龄、**锁状态标志**、线程 ID/指针等）和类型指针。锁信息就存在 Mark Word 里，通过 2 位「锁标志位」+ 1 位「偏向标志」区分状态：

| 锁状态 | Mark Word 主要内容 | 标志位 |
|---|---|---|
| 无锁 | hashCode、分代年龄 | `01`（偏向位 0） |
| 偏向锁 | 线程 ID、epoch、分代年龄 | `01`（偏向位 1） |
| 轻量级锁 | 指向栈中锁记录 Lock Record 的指针 | `00` |
| 重量级锁 | 指向 monitor（重量级锁）的指针 | `10` |
| GC 标记 | —— | `11` |

**锁升级（膨胀）过程**（JDK 6 引入，只能升级不能降级）：

```mermaid
flowchart LR
    A["无锁<br/>Mark Word: hashCode"] -->|第一个线程访问| B["偏向锁<br/>记录线程ID"]
    B -->|"另一线程竞争<br/>(CAS 撤销偏向)"| C["轻量级锁<br/>CAS 自旋，栈帧 Lock Record"]
    C -->|"自旋失败/竞争加剧<br/>(多线程争抢)"| D["重量级锁<br/>阻塞排队，OS 互斥量"]
    style A fill:#e8ffe8
    style B fill:#fff9e0
    style C fill:#ffeccc
    style D fill:#ffe0e0
```

- **偏向锁**：假设「大多数情况锁只被同一线程持有」。第一次获取时用 CAS 把线程 ID 记入 Mark Word，之后**同一线程再进入无需任何同步操作**（连 CAS 都省），开销最小。有其他线程竞争时触发**偏向撤销**，升级为轻量级锁。（注：偏向锁在 JDK 15 后默认禁用、JDK 18 移除，因维护成本高。）
- **轻量级锁**：适用「多线程交替执行、几乎无真正竞争」。线程在自己栈帧建 **Lock Record**，用 **CAS** 把对象 Mark Word 替换为指向 Lock Record 的指针，成功即获锁。失败则**自旋（忙等）** 重试。
- **重量级锁**：自旋超过阈值或竞争激烈时膨胀。依赖操作系统的**互斥量（mutex）**，未获锁线程进入 `BLOCKED` **阻塞**（涉及用户态→内核态切换，开销大），由 monitor 的 EntryList 排队。

**可重入性**。`synchronized` 是**可重入锁**：同一线程可重复获取同一把锁（如递归、同步方法调另一同步方法），monitor 计数器累加，避免自己把自己锁死。

## 🔑 面试要点

- `synchronized` 保证原子性、可见性（`monitorexit` 前刷主存）、有序性。
- 修饰实例方法锁 `this`，静态方法锁 `Class`，代码块锁指定对象——**注意锁的是不同对象**。
- 底层靠对象 monitor + `monitorenter`/`monitorexit` 字节码（同步方法用 `ACC_SYNCHRONIZED`）。
- 锁状态存在**对象头 Mark Word**，锁升级路径：无锁→偏向锁→轻量级锁→重量级锁，**单向不可逆**。
- 偏向锁省同步、轻量级锁靠 CAS 自旋、重量级锁靠 OS 互斥量阻塞。
- `synchronized` 是可重入、非公平锁，JVM 层实现，自动加解锁。
- JDK 6 的锁升级优化让 `synchronized` 性能已接近 `ReentrantLock`。

## ❓ 高频面试题

**Q：synchronized 的锁升级过程？为什么要这样设计？**
A：无锁→偏向锁→轻量级锁→重量级锁。设计目的是**根据竞争程度选用最省的策略**：无竞争时用偏向锁（几乎零开销）；轻度竞争（交替执行）用轻量级锁 CAS 自旋，避免线程阻塞的内核态切换；重度竞争才升级重量级锁阻塞排队。早期 `synchronized` 一上来就是重量级锁，性能差，JDK 6 引入升级机制大幅优化。锁只升不降。

**Q：synchronized 和 volatile 的区别？**
A：① `volatile` 只保证可见性和有序性、**不保证原子性**；`synchronized` 三者都保证。② `volatile` 只能修饰变量，`synchronized` 修饰方法/代码块。③ `volatile` 不阻塞线程、无锁；`synchronized` 会阻塞。④ `volatile` 更轻量，适合状态标志；`synchronized` 适合复合操作。

**Q：synchronized 底层是怎么实现的？**
A：基于对象的 **monitor**。同步代码块编译为 `monitorenter`/`monitorexit` 指令，同步方法用方法表的 `ACC_SYNCHRONIZED` 标志。monitor 内有计数器和 EntryList/WaitSet。线程执行 `monitorenter` 尝试获取 monitor，成功计数 +1（可重入再 +1），`monitorexit` 减到 0 释放。重量级锁的 monitor 对应操作系统的 mutex。

## ⚠️ 易错点 / 加分项

- **锁对象要一致才互斥**：两个方法分别 `synchronized(this)` 和 `synchronized(A.class)`，锁的不是同一对象，不会互斥。
- 别用 `String` 常量或 `Integer` 等**可能被复用/缓存的对象**当锁，易与其他代码意外共享同一锁。
- 一旦对象调用了 `hashCode()`（Mark Word 存了 hashCode），**该对象无法进入偏向锁**（无处存线程 ID）。
- 加分：能说清**偏向锁在 JDK 15+ 默认禁用、JDK 18 移除**（`-XX:-UseBiasedLocking`），原因是撤销成本高、现代应用收益小。
- 加分：自旋是「忙等」消耗 CPU，JDK 引入**自适应自旋**——根据上次自旋成功率动态调整自旋次数。
- `synchronized` 不可中断、不能设超时、不能实现公平锁，这些正是 `ReentrantLock` 的优势（见 08 篇）。
