# 10 · Java 8 新特性（Java 8 Features）

> Java 8 是里程碑版本，把「函数式编程」带进 Java。Lambda、Stream、Optional、新时间 API 是面试必考区，尤其 Stream 几乎每场都问。掌握「怎么用 + 为什么这么设计 + 有哪些坑」是拿分关键。

## 知识点索引

| 编号 | 知识点 | 重要度 | 一句话 |
| --- | --- | --- | --- |
| [01](./01-lambda.md) | Lambda 表达式（Lambda） | ⭐⭐⭐ | 函数式接口实例的简写 + 变量捕获 effectively final + 底层 invokedynamic |
| [02](./02-functional-interface.md) | 函数式接口（Functional Interface） | ⭐⭐⭐ | @FunctionalInterface + 四大核心接口 Function/Consumer/Supplier/Predicate |
| [03](./03-stream.md) | Stream 流（Stream API） | ⭐⭐⭐ | 中间 vs 终端操作 + 惰性求值 + collect/groupingBy + 并行流 + 不可复用 |
| [04](./04-optional.md) | Optional（Optional） | ⭐⭐⭐ | 优雅避免 NPE + of/ofNullable/orElse/map + 正确用法与反模式 |
| [05](./05-method-reference.md) | 方法引用（Method Reference） | ⭐⭐ | 四种形式 + 是 Lambda 的语法糖 |
| [06](./06-datetime-api.md) | 新日期时间 API（Date/Time API） | ⭐⭐⭐ | LocalDate/LocalDateTime/Instant/Duration/Period + 线程安全替代 Date |
| [07](./07-default-method.md) | 接口默认方法（Default Method） | ⭐⭐ | default/static 方法 + 为接口演进而生 + 菱形冲突解决 |

## 推荐复习顺序

Lambda（01）与函数式接口（02）是地基，必须先吃透——它俩是 Stream、方法引用的前提。然后主攻 **Stream（03）** 这个重头戏，Optional（04）和方法引用（05）搭配 Stream 一起用。时间 API（06）和默认方法（07）相对独立，各击破即可。

> 本主题可对照姊妹项目 `../../jdk-learning/08-jdk8/`（侧重语言/JDK 视角），本目录内容独立、面向面试。与 `CompletableFuture`、并发相关的部分见 `../09-concurrency/`。
