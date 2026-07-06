# 06 · 异常 Exception ⭐⭐

> 异常体系（Throwable/Error/Exception）、受检 vs 非受检、try-catch-finally 执行顺序、try-with-resources 与最佳实践，都是高频基础题。

## 知识点索引

| # | 知识点 | 重要度 | 一句话 |
|---|---|---|---|
| 01 | [异常体系](01-exception-hierarchy.md) | ⭐⭐⭐ | Throwable → Error / Exception → RuntimeException 继承图与常见异常 |
| 02 | [受检 vs 非受检](02-checked-vs-unchecked.md) | ⭐⭐⭐ | 编译期强制处理 vs 运行期，如何选择 |
| 03 | [try-catch-finally](03-try-catch-finally.md) | ⭐⭐⭐ | 执行顺序、finally 是否一定执行、finally return 覆盖、try-with-resources |
| 04 | [异常最佳实践](04-exception-best-practice.md) | ⭐⭐ | 自定义异常、不吞异常、异常链、性能注意 |

## 学习顺序

先建立**体系认知**（01）→ 分清**受检/非受检**（02）→ 吃透**执行流程**（03）→ 落到**工程实践**（04）。
