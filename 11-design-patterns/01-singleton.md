# 01 · 单例模式（Singleton）

> 保证一个类在整个 JVM 中**只有一个实例**，并提供一个全局访问点。面试第一高频设计模式，5 种写法、线程安全、DCL 为什么加 `volatile` 必须张口就来。面试重要度 ⭐⭐⭐。

## 📖 核心知识

单例的三要素：**私有构造器**（外部不能 `new`）+ **私有静态实例** + **公有静态获取方法**。难点全在「怎么保证线程安全地只创建一个」。

### 1. 饿汉式（Eager，线程安全）

类加载时就创建实例，天生线程安全（靠 JVM 类初始化保证），缺点是**不管用不用都占内存**（无法懒加载）。

```java
public class Eager {
    private static final Eager INSTANCE = new Eager();
    private Eager() {}
    public static Eager getInstance() { return INSTANCE; }
}
```

### 2. 懒汉式（Lazy，非线程安全 → 加锁）

用到才创建。最朴素写法多线程下会创建多个实例；加 `synchronized` 后线程安全，但**每次获取都要抢锁**，性能差。

```java
public class Lazy {
    private static Lazy instance;
    private Lazy() {}
    public static synchronized Lazy getInstance() {  // 锁整个方法，性能差
        if (instance == null) instance = new Lazy();
        return instance;
    }
}
```

### 3. 双重检查锁 DCL（Double-Checked Locking，推荐）

只在实例为 null 时才进同步块，兼顾懒加载与性能。**关键：实例必须用 `volatile` 修饰**。

```java
public class DCL {
    private static volatile DCL instance;   // volatile 不可省！
    private DCL() {}
    public static DCL getInstance() {
        if (instance == null) {              // 第一次检查：避免每次加锁
            synchronized (DCL.class) {
                if (instance == null) {      // 第二次检查：避免重复创建
                    instance = new DCL();
                }
            }
        }
        return instance;
    }
}
```

**为什么必须加 `volatile`？** `instance = new DCL()` 不是原子操作，底层分三步：①分配内存；②调用构造器初始化对象；③把引用指向内存。JVM/CPU 可能发生**指令重排序**，把顺序变成 ①→③→②。如果线程 A 执行到「③已赋值但②未初始化」时，线程 B 在第一次检查发现 `instance != null`，就会拿到一个**尚未初始化完成的半成品对象**，使用时报错（NPE 或脏数据）。`volatile` 通过内存屏障**禁止这种重排序**，保证「初始化完成」对其他线程可见。

### 4. 静态内部类（Holder，推荐）

利用 JVM **类初始化的线程安全保证**（`<clinit>` 有锁）+ **延迟加载**（内部类只有被首次引用才加载）。写法优雅、无锁、懒加载。

```java
public class Holder {
    private Holder() {}
    private static class Inner {
        private static final Holder INSTANCE = new Holder();
    }
    public static Holder getInstance() { return Inner.INSTANCE; }  // 此刻才加载 Inner
}
```

### 5. 枚举（Enum，最安全，Effective Java 推荐）

天然线程安全、天然单例，而且**能防反射攻击、防序列化破坏**——这是其他写法都做不到的。

```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
// 用法：Singleton.INSTANCE.doSomething();
```

## 🔑 面试要点

- 三要素：私有构造器 + 私有静态实例 + 公有静态获取方法。
- **五种写法线程安全对比表**：

| 写法 | 线程安全 | 懒加载 | 防反射 | 防序列化破坏 | 评价 |
| --- | --- | --- | --- | --- | --- |
| 饿汉式 | ✅（类加载保证） | ❌ | ❌ | ❌ | 简单，浪费内存 |
| 懒汉式 + synchronized | ✅ | ✅ | ❌ | ❌ | 每次加锁，性能差 |
| 双重检查 DCL | ✅（需 volatile） | ✅ | ❌ | ❌ | **推荐**，高性能 |
| 静态内部类 Holder | ✅（类初始化保证） | ✅ | ❌ | ❌ | **推荐**，优雅无锁 |
| 枚举 Enum | ✅ | ❌ | ✅ | ✅ | **最安全**，首选 |

- DCL 必须 `volatile`，否则可能拿到「半初始化对象」（指令重排序问题）。
- 静态内部类懒加载的原理：JVM 保证类的 `<clinit>()` 只执行一次且线程安全，内部类被**首次主动引用**时才初始化。
- 枚举为什么最安全：JVM 保证枚举实例唯一，反射 `newInstance` 对枚举会直接抛 `IllegalArgumentException`，序列化也由 JVM 特殊处理只返回同一实例。

## ❓ 高频面试题

**Q：DCL 里的 `volatile` 能不能去掉？为什么？**
A：不能。`instance = new DCL()` 分「分配内存→初始化→赋引用」三步，可能被重排序成「分配内存→赋引用→初始化」。此时另一线程在第一次 null 检查看到引用非空，就会拿到未初始化完的对象。`volatile` 禁止该重排序并保证可见性。

**Q：为什么两次判空？只判一次行不行？**
A：不行。第一次判空是**性能优化**（实例已存在时直接返回，不进同步块）；第二次判空是**正确性保证**（多个线程可能同时通过第一次检查排队进同步块，进去后必须再判一次，否则会重复创建）。

**Q：单例有哪些破坏方式，怎么防？**
A：①**反射**：通过 `setAccessible(true)` 强行调私有构造器——可在构造器里判断实例已存在就抛异常，或直接用枚举；②**序列化/反序列化**：反序列化会新建对象——可实现 `readResolve()` 返回单例，或用枚举；③**克隆**：不实现 `Cloneable` 即可。枚举天然免疫前两种。

**Q：饿汉式和静态内部类都靠类加载，区别是什么？**
A：饿汉式的实例是**外层类**的静态字段，外层类一加载就创建，无法懒加载；静态内部类的实例在**内部类**里，外层类加载时不会加载内部类，只有调用 `getInstance()` 首次引用内部类时才创建，实现了懒加载。

## ⚠️ 易错点 / 加分项

- **误区**：「DCL 加了 synchronized 就够了。」——不够，缺 `volatile` 在极端情况下仍有半初始化对象问题。
- **误区**：「静态内部类需要加锁保证线程安全。」——不需要，JVM 类初始化 `<clinit>` 自带锁。
- **加分**：能说出 `new` 对象的三步字节码（`new` / `invokespecial` / `astore`）和重排序场景。
- **加分**：能点出枚举防序列化的原理——`Enum` 的序列化只写枚举名，反序列化用 `valueOf` 拿回同一实例，不走普通对象反序列化流程。
- **加分**：Spring 的 Bean 默认是单例，但那是**容器级单例**（一个 Spring 容器一个实例），和这里的「JVM 级单例」不是一回事，靠的是 `ConcurrentHashMap` 缓存（三级缓存），不是私有构造器。
