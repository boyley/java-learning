# Java 面试学习合集 · 统一规范（所有 sub-agent 必须遵守）

> 本文件是本工程唯一风格标准。任何 agent 生成内容前先读本文件。目标：**系统、面试导向、一个知识点一个 md、按顺序编号**。

## 一、目录结构（主题分类 + 扁平知识点）

```
java-learning/
├── README.md                 ← 总览 + 知识地图 + 面试冲刺路线
├── _CONVENTIONS.md           ← 本文件
├── 01-java-basics/           ← 主题分类文件夹（NN-主题）
│   ├── README.md             ← 该主题的知识点索引
│   ├── 01-data-types.md      ← 一个知识点一个 md
│   └── ...
├── 02-oop/
└── ...
```

## 二、命名

- 主题目录：`NN-topic`（两位数字 + kebab-case 英文），如 `04-collections`、`09-concurrency`。
- 知识点文件：`NN-knowledge-point.md`（两位数字 + kebab-case 英文），如 `05-hashmap.md`。
- 编号 = 推荐阅读/复习顺序。

## 三、每个知识点 md 固定结构（面试导向，简洁）

```markdown
# NN · 知识点中文名（English Name）

> 一句话核心 + 面试重要度（⭐⭐⭐ 高频 / ⭐⭐ 常考 / ⭐ 了解）。

## 📖 核心知识
（中文讲清概念本质与原理，2~6 段；能配图的配 Mermaid。）

## 🔑 面试要点
（bullet 列出必须记住的关键点，5~8 条。）

## ❓ 高频面试题
**Q：……？**
A：……（2~4 个高频问题，问题加粗，答案简洁到位。）

## ⚠️ 易错点 / 加分项
（3~5 条：常见误区 + 答出来能加分的深入点。）
```

> 复杂机制（HashMap/ConcurrentHashMap 原理、AQS、synchronized 锁升级、线程池工作流程、动态代理、Stream 管道等）**建议配一个 Mermaid 图或对比表**。别为配图而配图。

## 四、内容语言与质量底线

- 讲解一律**中文**；类名、方法名、关键字、参数用英文（如 `HashMap`、`volatile`、`ReentrantLock`）。
- 基于 **JDK 8+**（主流面试基准），涉及版本差异（如 HashMap JDK7 vs 8、ConcurrentHashMap 分段锁 vs CAS）要点明。
- 面试导向：多讲「为什么」「怎么答」「易错点」，示例代码可编译、简短。
- 每个 md 聚焦**一个**知识点，篇幅适中。
- 与 JVM 底层强相关的点（类加载、GC、JMM、JVM 调优）**不重复展开**，一句话点到并引用姊妹项目 `jvm-learning`（相对链接 `../../jvm-learning/xx.md`）。

## 五、覆盖主线

基础语法 → 面向对象 → String → 集合框架 → 泛型 → 异常 → IO/NIO → 反射与代理 → 并发编程（重头）→ Java 8 特性 → 常用设计模式。
