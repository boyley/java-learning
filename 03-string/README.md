# 03 · String 字符串 ⭐⭐⭐

> String 是 Java 面试的绝对高频区：不可变性、可变类对比、常量池与 `intern()`。三者环环相扣，几乎必问。

## 知识点索引

| # | 知识点 | 重要度 | 一句话 |
|---|---|---|---|
| 01 | [String 为什么不可变](01-string-immutable.md) | ⭐⭐⭐ | `final class` + `private final` 数组，不可变换来安全/线程安全/hashCode 缓存/常量池复用 |
| 02 | [String / StringBuilder / StringBuffer](02-string-stringbuilder-stringbuffer.md) | ⭐⭐⭐ | 可变性、线程安全、性能三维对比，循环拼接必用 `StringBuilder` |
| 03 | [常量池与 intern()](03-string-pool-intern.md) | ⭐⭐⭐ | `new String("a")` 创建几个对象、`intern()` 作用、JDK7 常量池移入堆 |

## 学习顺序

先懂**不可变**（01）是理解一切的基础 → 再对比**可变类**（02）→ 最后深入**常量池与 intern**（03）。

> 字符串常量池的 JVM 底层布局（方法区 → 堆的迁移、GC）见姊妹项目 [`../../jvm-learning/14-string-constant-pool.md`](../../jvm-learning/14-string-constant-pool.md)，本主题只讲 Java 层面结论。
