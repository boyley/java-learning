# 08 · 反射与代理（Reflection & Proxy）

> 反射、注解、动态代理是 Java 框架（Spring、MyBatis）的三大基石。理解它们才能看懂「框架为什么能在运行时帮你做事」，是中高级面试的重点区。

## 知识点索引

| 编号 | 知识点 | 重要度 | 一句话 |
| --- | --- | --- | --- |
| [01](./01-reflection.md) | 反射机制（Reflection） | ⭐⭐⭐ | 运行时动态获取类信息并操作，获取 Class 三种方式 + 性能/安全代价 |
| [02](./02-annotation.md) | 注解（Annotation） | ⭐⭐ | 注解本质是接口、元注解 @Retention/@Target、自定义注解 + 反射读取 |
| [03](./03-dynamic-proxy.md) | 动态代理（Dynamic Proxy） | ⭐⭐⭐ | JDK 动态代理（接口）vs CGLIB（继承子类）、Spring AOP 选型 |

## 推荐复习顺序

先吃透 **反射**（01）——它是注解读取和动态代理的底层能力；再看 **注解**（02）理解框架如何声明式配置；最后 **动态代理**（03）是 Spring AOP 的核心，也是三者的综合应用。

> 反射依赖类加载与 `Class` 对象，类加载底层（双亲委派、Class 对象在方法区/元空间的存储）见姊妹项目 [`../../jvm-learning/04-class-loading-process.md`](../../jvm-learning/04-class-loading-process.md) 与 [`../../jvm-learning/05-class-loaders.md`](../../jvm-learning/05-class-loaders.md)，本主题聚焦 Java 层面用法。
