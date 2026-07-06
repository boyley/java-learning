# 09 · 并发编程（Concurrency）⭐⭐⭐

> Java 面试的**绝对重头戏**。从线程基础、`synchronized` 锁升级、`volatile`、CAS、AQS，到 `ReentrantLock`、线程池、`ThreadLocal`、JUC 并发容器，几乎每场中高级面试都会深挖原理。本主题侧重**使用与面试答法**，JMM/内存屏障等底层引用姊妹项目 [`jvm-learning`](../../jvm-learning)。

## 知识点索引

| 编号 | 知识点 | 重要度 | 一句话 |
| --- | --- | --- | --- |
| [01](./01-thread-basics.md) | 线程基础（Thread Basics） | ⭐⭐ | 进程vs线程、并发vs并行、sleep/join/yield/wait、用户线程vs守护线程 |
| [02](./02-thread-create-ways.md) | 创建线程的方式（Create Threads） | ⭐⭐ | 继承Thread、实现Runnable、Callable+FutureTask、线程池对比 |
| [03](./03-thread-state.md) | 线程状态与转换（Thread State） | ⭐⭐⭐ | 6 种状态 NEW/RUNNABLE/BLOCKED/WAITING/TIMED_WAITING/TERMINATED、sleep vs wait |
| [04](./04-synchronized.md) | synchronized 与锁升级（synchronized） | ⭐⭐⭐ | 用法、Mark Word、无锁→偏向→轻量级→重量级、monitor、可重入 |
| [05](./05-volatile.md) | volatile 关键字（volatile） | ⭐⭐⭐ | 可见性+有序性、不保证原子性、DCL 单例、与 synchronized 对比 |
| [06](./06-cas-atomic.md) | CAS 与原子类（CAS & Atomic） | ⭐⭐⭐ | compareAndSwap、Unsafe、ABA 问题、AtomicInteger/LongAdder、自旋 |
| [07](./07-aqs.md) | AQS 抽象队列同步器（AQS） | ⭐⭐⭐ | state + CLH 双向队列、独占/共享、模板方法、JUC 组件基石 |
| [08](./08-reentrantlock.md) | ReentrantLock（ReentrantLock） | ⭐⭐⭐ | 对比 synchronized、公平/非公平、可中断/超时、Condition |
| [09](./09-thread-pool.md) | 线程池（ThreadPoolExecutor） | ⭐⭐⭐ | 七大参数、工作流程、拒绝策略、四种内置池、参数调优 |
| [10](./10-threadlocal.md) | ThreadLocal（ThreadLocal） | ⭐⭐⭐ | 线程隔离、ThreadLocalMap、弱引用 key、内存泄漏与 remove |
| [11](./11-concurrent-collections.md) | 并发容器（Concurrent Collections） | ⭐⭐ | CopyOnWriteArrayList、BlockingQueue、ConcurrentLinkedQueue |
| [12](./12-juc-tools.md) | JUC 同步工具（JUC Tools） | ⭐⭐ | CountDownLatch、CyclicBarrier、Semaphore、CompletableFuture |
| [13](./13-deadlock.md) | 死锁（Deadlock） | ⭐⭐⭐ | 四个必要条件、检测与预防、jstack 排查 |
| [14](./14-wait-notify.md) | 线程通信 wait/notify（wait/notify） | ⭐⭐⭐ | Object 等待-唤醒、生产者消费者、必须在 synchronized 中 |

## 推荐复习顺序

1. **打基础**：01 线程基础 → 02 创建方式 → 03 线程状态，建立线程生命周期认知。
2. **啃三大基石**：04 `synchronized`（锁升级）→ 05 `volatile`（可见性/有序性）→ 06 CAS，这是并发原理的地基。
3. **锁框架**：07 AQS → 08 `ReentrantLock`，AQS 是 JUC 半壁江山的底层。
4. **实战组件**：09 线程池（必背七大参数与流程）→ 10 ThreadLocal → 11 并发容器 → 12 JUC 同步工具。
5. **收尾**：13 死锁排查 → 14 wait/notify 线程通信。

> **面试冲刺 Top 优先级**：`synchronized` 锁升级、`volatile`+JMM、AQS 原理、线程池七大参数与工作流程、ThreadLocal 内存泄漏、死锁四条件——几乎必问。
>
> 与 JVM 底层强相关的 JMM、内存屏障、`volatile` 汇编实现见 [`../../jvm-learning/30-jmm.md`](../../jvm-learning/30-jmm.md)、[`../../jvm-learning/31-volatile.md`](../../jvm-learning/31-volatile.md)，本主题只讲 Java 层结论与答法。
