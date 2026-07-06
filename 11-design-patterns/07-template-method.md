# 07 · 模板方法模式（Template Method）

> 在父类里定义一个算法的**骨架**（固定流程），把某些可变步骤延迟到子类实现，从而「流程不变、细节可换」。是框架设计中最常见的模式。面试重要度 ⭐⭐。

## 📖 核心知识

父类定义一个 `final` 的**模板方法**，按固定顺序调用若干步骤；其中通用步骤父类实现，可变步骤定义为 `abstract` 由子类实现，另有**钩子方法（hook）**提供默认空实现让子类可选覆盖。核心思想：**好莱坞原则——「Don't call us, we'll call you」**，流程由父类掌控，子类只填空。

```java
abstract class Game {
    // 模板方法：定义骨架，用 final 防止子类改流程
    public final void play() {
        init();
        startPlay();
        if (showResult()) {   // 钩子：子类可决定是否展示
            end();
        }
    }
    protected abstract void init();       // 可变步骤，子类必须实现
    protected abstract void startPlay();
    protected void end() { System.out.println("游戏结束"); }  // 通用步骤
    protected boolean showResult() { return true; }           // 钩子，默认实现
}

class Chess extends Game {
    protected void init() { System.out.println("摆棋子"); }
    protected void startPlay() { System.out.println("走棋"); }
}
// new Chess().play(); 流程被父类锁定，Chess 只填自己的步骤
```

- **模板方法**：`final` 修饰，锁死流程不让子类改。
- **抽象方法**：必须由子类实现的可变步骤。
- **钩子方法**：有默认实现，子类可选择性覆盖以微调流程（如决定某步是否执行）。

### 真实应用

- **AQS（`AbstractQueuedSynchronizer`）**：定义了获取/释放锁的完整排队骨架（`acquire`/`release`），把 `tryAcquire`/`tryRelease` 留给 `ReentrantLock`、`Semaphore` 等子类实现——并发框架的模板方法典范（详见 [`09-concurrency`](../09-concurrency)）。
- **`HttpServlet`**：`service()` 是模板方法，按请求类型分发到 `doGet()`/`doPost()`，子类只重写关心的方法。
- **Spring `JdbcTemplate`**：`execute()` 封装了「获取连接→创建 Statement→执行→处理结果→释放连接」的固定流程，把「怎么处理结果集」通过 `RowMapper` 回调交给用户。
- **`AbstractList`**、**Spring 的 `AbstractApplicationContext.refresh()`**、MyBatis `BaseExecutor`。

## 🔑 面试要点

- 一句话：**父类定骨架（固定流程），子类填细节（可变步骤）**。
- 三种方法：模板方法（`final` 锁流程）、抽象方法（子类必填）、钩子方法（默认实现、可选覆盖）。
- 靠**继承**复用流程，符合「好莱坞原则」（父类调子类，控制反转）。
- 模板方法要用 `final` 修饰，防止子类破坏算法骨架。
- 典型应用：AQS、`HttpServlet.service`、`JdbcTemplate`。

## ❓ 高频面试题

**Q：模板方法模式的核心思想是什么？**
A：把一个算法的固定流程（骨架）放在父类的模板方法里，用 `final` 锁定步骤调用顺序；把其中会变化的步骤声明为抽象方法交给子类实现。实现「流程统一、细节多样」，避免每个子类重复写流程代码。

**Q：什么是钩子方法（hook）？**
A：父类中提供**默认实现（常为空或返回默认值）**的方法，子类可以选择性覆盖它来干预流程走向（比如返回 boolean 决定模板里某个步骤是否执行）。它让模板既固定又保留弹性。

**Q：AQS 用了哪种设计模式？**
A：模板方法模式。AQS 把「入队、阻塞、唤醒」的排队骨架实现好，`tryAcquire`/`tryRelease`/`tryAcquireShared` 等留成待子类实现的方法，`ReentrantLock`、`CountDownLatch`、`Semaphore` 通过实现这些方法定制自己的同步语义。

**Q：模板方法和策略模式有什么区别？**
A：模板方法靠**继承**，在编译期通过子类固定某些步骤，整体流程在父类；策略模式靠**组合**，运行时注入不同策略对象来替换整个算法。模板方法换「步骤」，策略换「整个算法」。

## ⚠️ 易错点 / 加分项

- **加分**：能对比模板方法（继承、编译期）和策略模式（组合、运行时）——「组合优于继承」，策略更灵活，模板方法更省代码。
- **加分**：能点出 `JdbcTemplate` 是「模板方法 + 回调」的结合（`RowMapper`/`StatementCallback` 是回调，弥补 Java 无函数指针）。
- **易错**：模板方法忘记加 `final`，子类可以重写整个流程，破坏模式初衷。
- **加分**：好莱坞原则（控制反转 IoC）——不是子类调父类，而是父类框架调子类填空，和 Spring 的 IoC 思想一脉相承。
