# 08 · ReentrantLock（可重入锁）

> `ReentrantLock` 是基于 **AQS** 的显式可重入锁，相比 `synchronized` 更灵活：**可中断、可超时、可公平、可绑定多个 Condition**。代价是必须**手动 `lock()`/`unlock()`**（配对在 `try-finally`）。面试重要度 ⭐⭐⭐（常与 synchronized 对比）。

## 📖 核心知识

**基本用法**——`unlock()` 必须放在 `finally`，否则异常时锁不释放导致死锁：

```java
private final ReentrantLock lock = new ReentrantLock(); // 默认非公平
public void doWork() {
    lock.lock();              // 加锁（或 lock.lock() 前用 tryLock 等）
    try {
        // 临界区
    } finally {
        lock.unlock();        // 必须在 finally 释放！
    }
}
```

**可重入**。同一线程可重复 `lock()`，内部 AQS 的 `state` 记重入次数，加几次就要 `unlock()` 几次，**lock/unlock 必须次数配对**，否则锁无法真正释放。

**公平锁 vs 非公平锁**。构造传 `new ReentrantLock(true)` 为公平锁。

- **非公平（默认）**：线程抢锁时**先直接 CAS 尝试抢占**，抢不到才排队。可能「插队」——刚释放锁时新来的线程可能比队列里等待的更早拿到锁。**吞吐量高**（减少线程切换），但可能导致队列线程「饥饿」。
- **公平**：严格按 FIFO，抢锁前先检查队列**是否有前驱等待**（`hasQueuedPredecessors()`），有则乖乖排队。**公平但吞吐量较低**（频繁唤醒切换）。

**可中断 / 超时获取**：

```java
lock.lockInterruptibly();          // 等待锁期间可响应 interrupt() 中断（抛 InterruptedException）
boolean ok = lock.tryLock();       // 非阻塞尝试，立即返回是否拿到
boolean ok2 = lock.tryLock(2, TimeUnit.SECONDS); // 超时等待，超时返回 false（可避免死锁）
```

`synchronized` 等锁时**无法中断、无法超时**，只能死等——这是 `ReentrantLock` 的关键优势，可用于**死锁预防**（拿不到就放弃）。

**Condition 条件变量**。`lock.newCondition()` 可创建**多个** Condition，每个是一个独立的等待队列，实现精准唤醒。相比 `synchronized` 只有一个 `wait/notify` 等待集，Condition 能把「不同条件的等待线程」分开管理（如生产者-消费者中「队列满」和「队列空」两个条件），`signal()` 只唤醒对应条件的线程。

```java
Condition notEmpty = lock.newCondition();
Condition notFull  = lock.newCondition();
// 消费者：队列空则在 notEmpty 上等；生产者放入后 notEmpty.signal()
notEmpty.await();      // 对标 wait()，会释放锁
notEmpty.signal();     // 对标 notify()，唤醒等在 notEmpty 的一个线程
```

> `await`/`signal` 同样**必须在持有 lock 的情况下调用**，`await()` 会释放锁并阻塞。

**ReentrantLock vs synchronized 对比**：

| 维度 | synchronized | ReentrantLock |
|---|---|---|
| 实现层面 | JVM 内置（关键字） | JDK API（基于 AQS） |
| 加/解锁 | 自动（进出同步块） | 手动 `lock`/`unlock`（try-finally） |
| 可重入 | ✅ | ✅ |
| 公平锁 | ❌ 只非公平 | ✅ 可选公平/非公平 |
| 可中断 | ❌ | ✅ `lockInterruptibly` |
| 超时获取 | ❌ | ✅ `tryLock(timeout)` |
| 条件变量 | 单个（`wait/notify`） | 多个 `Condition`，精准唤醒 |
| 锁状态查询 | ❌ | ✅ `isLocked`/`getHoldCount` 等 |
| 性能 | JDK6 后已优化，接近 | 高竞争下略优，功能更强 |

## 🔑 面试要点

- `ReentrantLock` 基于 **AQS**，`state` 记重入次数，独占模式。
- 必须 **`try-finally` 中 `unlock()`**，lock/unlock 次数配对。
- 默认**非公平**（吞吐高），可构造为公平锁（`new ReentrantLock(true)`）。
- 独有能力：**可中断（`lockInterruptibly`）、可超时/非阻塞（`tryLock`）、多 Condition 精准唤醒**。
- `Condition.await/signal` 对标 `Object.wait/notify`，但可有多个条件队列。
- 选型：简单同步用 `synchronized`（自动释放、不易出错）；需要中断/超时/公平/多条件用 `ReentrantLock`。

## ❓ 高频面试题

**Q：ReentrantLock 和 synchronized 的区别？**
A：① 实现——`synchronized` 是 JVM 关键字自动加解锁，`ReentrantLock` 是基于 AQS 的 API 需手动 `lock/unlock`；② 灵活性——`ReentrantLock` 支持**公平锁、可中断、可超时、多个 Condition**，`synchronized` 都不支持；③ 都可重入；④ 性能——JDK6 优化后 `synchronized` 已接近，一般优先用它（不易忘记释放），复杂场景才用 `ReentrantLock`。

**Q：公平锁和非公平锁的区别？为什么默认非公平？**
A：公平锁严格按请求顺序 FIFO 分配（抢锁前先看有没有前驱在排队），不会插队但吞吐低；非公平锁允许新线程直接 CAS 抢占，可能插队，吞吐高但可能饥饿。默认非公平是因为——**减少线程唤醒和上下文切换的开销**，刚释放锁的线程恰好被新线程接手，避免唤醒队列头节点的代价，整体吞吐更好。

**Q：ReentrantLock 的可重入是怎么实现的？**
A：AQS 的 `tryAcquire` 中：若 `state==0` 则 CAS 抢锁并用 `setExclusiveOwnerThread` 记录持有线程；若当前线程就是持有者，则 `state` 累加（重入），直接返回成功。释放时 `state` 递减，减到 0 才真正释放并清空持有线程。所以加锁几次必须解锁几次。

**Q：Condition 和 wait/notify 有什么区别？**
A：`Condition` 需配合 `Lock` 使用（`await`/`signal`），一个 Lock 可创建**多个 Condition**，每个对应独立等待队列，能实现「分组精准唤醒」；`wait/notify` 依附 `synchronized`，一个锁只有一个等待集，`notify` 只能随机唤醒。生产者-消费者用两个 Condition（notFull/notEmpty）比单一 wait/notify 更高效、无需唤醒无关线程。

## ⚠️ 易错点 / 加分项

- 忘记 `unlock()` 或没放 `finally`：临界区抛异常时锁永不释放，直接死锁。
- `lock()` 调用要放在 `try` **之外或紧邻 try 第一行**——常见写法是 `lock(); try{...}finally{unlock();}`；若把 `lock()` 写进 `try` 内且加锁失败，`finally` 里的 `unlock` 会对未持有的锁抛 `IllegalMonitorStateException`。
- `Condition.await()` 也要用 **while 循环**判断条件，防虚假唤醒。
- 加分：`ReentrantLock` 的 `Sync` 有公平/非公平两个子类，非公平的 `lock()` 会先直接 CAS 抢一次再走 AQS 流程。
- 加分：读写场景用 `ReentrantReadWriteLock`（读共享写独占）或 JDK 8 的 `StampedLock`（支持乐观读，性能更高但不可重入）。
