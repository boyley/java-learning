# 07 · IO / NIO（IO & NIO）

> Java 输入输出体系。字节流 vs 字符流、装饰器模式、BIO/NIO/AIO 三种模型、NIO 三大件与序列化，是中高级面试考察「设计模式 + 网络编程 + 底层原理」的综合区。

## 知识点索引

| 编号 | 知识点 | 重要度 | 一句话 |
| --- | --- | --- | --- |
| [01](./01-io-streams.md) | IO 流分类（IO Streams） | ⭐⭐⭐ | 字节流 vs 字符流、节点流 vs 处理流、装饰器模式与 BufferedXxx |
| [02](./02-bio-nio-aio.md) | BIO / NIO / AIO（IO Models） | ⭐⭐⭐ | 同步阻塞 / 同步非阻塞多路复用 / 异步，对比表 + 适用场景 |
| [03](./03-nio-components.md) | NIO 三大件（Channel / Buffer / Selector） | ⭐⭐⭐ | Buffer 的 position/limit/capacity 与 flip、Selector 多路复用原理 |
| [04](./04-serialization.md) | 序列化（Serialization） | ⭐⭐ | Serializable、serialVersionUID、transient、静态字段不序列化 |

## 推荐复习顺序

先掌握经典 **IO 流分类与装饰器模式**（01），理解「阻塞」痛点后进入 **BIO/NIO/AIO 模型对比**（02），再深入 **NIO 三大件**（03）搞懂多路复用；**序列化**（04）相对独立，可最后看。

> NIO 底层依赖操作系统 `select`/`poll`/`epoll` 多路复用与 DirectByteBuffer 堆外内存，涉及 JVM 内存布局的部分见姊妹项目 [`../../jvm-learning/`](../../jvm-learning/)，本主题聚焦 Java 层面。
