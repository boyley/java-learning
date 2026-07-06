# 01 · Java 基础语法（Java Basics）

> Java 面试的地基。数据类型、装箱缓存、值传递、equals/hashCode 等是几乎每场面试都会被追问的「送分 + 送命」题，答不清楚直接减分。

## 知识点索引

| 编号 | 知识点 | 重要度 | 一句话 |
| --- | --- | --- | --- |
| [01](./01-data-types.md) | 数据类型（Data Types） | ⭐⭐⭐ | 8 种基本类型 + 大小 + 包装类 + 栈堆存储 + 默认值 |
| [02](./02-autoboxing-cache.md) | 自动装箱与缓存池（Autoboxing & Cache） | ⭐⭐⭐ | 装箱拆箱原理 + Integer -128~127 缓存 + `==` 经典题 |
| [03](./03-value-passing.md) | 值传递（Value Passing） | ⭐⭐⭐ | Java 只有值传递，引用类型传的是引用副本 |
| [04](./04-equals-hashcode.md) | equals 与 hashCode（equals & hashCode） | ⭐⭐⭐ | == vs equals + 为何重写 equals 必须重写 hashCode |
| [05](./05-final-static.md) | final 与 static（final & static） | ⭐⭐ | final 修饰三处 + static 变量/方法/代码块 + 初始化顺序 |
| [06](./06-keywords.md) | 常见关键字（Keywords） | ⭐⭐ | this/super、instanceof、transient 等速览 |

## 推荐复习顺序

按编号 01 → 06 顺序即可。其中 01/02/03/04 是绝对高频，务必吃透；05/06 快速扫盲。

> 与 JVM 内存模型、类加载强相关的部分只点到为止，深入见姊妹项目 `../../jvm-learning/`。
