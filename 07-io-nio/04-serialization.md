# 04 · 序列化（Serialization）

> 序列化把对象转成字节序列以便存储/传输，反序列化还原。`Serializable`、`serialVersionUID`、`transient` 是必考三件套。面试重要度 ⭐⭐。

## 📖 核心知识

**序列化（Serialization）**：把 Java 对象转换成字节序列，用于持久化到磁盘或跨网络传输；**反序列化**是逆过程。Java 原生方案：实现 `java.io.Serializable` 接口（一个**空标记接口**，无任何方法），用 `ObjectOutputStream.writeObject()` 写、`ObjectInputStream.readObject()` 读。

```java
class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private transient String password;   // 不参与序列化
    private static int count;             // 静态字段不参与序列化
}

// 写
try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("u.dat"))) {
    oos.writeObject(new User());
}
// 读
try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("u.dat"))) {
    User u = (User) ois.readObject();
}
```

**serialVersionUID 的作用**：序列化版本号，用于反序列化时**校验类版本一致性**。反序列化时比对字节流中的 UID 与当前类的 UID，不一致抛 `InvalidClassException`。若不显式声明，JVM 会**根据类结构（字段、方法等）自动计算**一个 UID——一旦类改动（哪怕加个方法），UID 变化，旧数据就无法反序列化。因此**强烈建议手动显式声明** `private static final long serialVersionUID`，让类演进（加字段）时仍能兼容老数据。

**transient**：被 `transient` 修饰的字段**不参与序列化**，反序列化后是默认值（对象为 `null`、int 为 0）。用于敏感字段（密码）或可重新计算/无需持久化的字段。

**静态字段不序列化**：序列化保存的是**对象状态**，`static` 字段属于**类**而非对象，因此不会被写入字节流。反序列化后静态字段的值取当前 JVM 中类的值，而非序列化时的值——这是常见误区。

**自定义序列化**：可实现 `writeObject`/`readObject` 私有方法精细控制，或实现 `Externalizable` 接口完全自己掌控读写（性能更高但要手写全部字段）。

## 🔑 面试要点

- `Serializable` 是空标记接口，实现它即声明「可被序列化」，配 `ObjectOutputStream`/`ObjectInputStream`。
- `serialVersionUID` 是版本校验号，反序列化时不匹配抛 `InvalidClassException`；务必手动显式声明。
- 不声明 UID，JVM 按类结构自动生成，类一改 UID 就变，破坏兼容性。
- `transient` 字段不序列化，反序列化后为默认值；用于密码等敏感或可重算字段。
- `static` 字段属于类不属于对象，不参与序列化。
- 父类未实现 `Serializable` 时，其字段不会被序列化，反序列化时父类字段靠调用父类无参构造初始化。
- 生产中 RPC/缓存多用 JSON、Protobuf、Kryo、Hessian 等替代 Java 原生序列化（更快、跨语言、体积小）。

## ❓ 高频面试题

**Q：serialVersionUID 有什么用？不写会怎样？**
A：它是序列化版本一致性校验号。反序列化时会比对字节流里的 UID 和当前类的 UID，不同则抛 `InvalidClassException`。不显式写，JVM 会依据类的字段、方法等结构自动算一个；只要类发生任何改动，算出的 UID 就变，导致之前序列化的旧数据无法反序列化。所以显式固定 UID，能在类正常演进（如新增字段）时保持向后兼容。

**Q：transient 和 static 字段会被序列化吗？**
A：都不会。`transient` 是显式声明「跳过此字段」，反序列化后取默认值。`static` 字段属于类而非对象实例，序列化保存的是对象状态，所以也不写入。注意反序列化后看到的静态字段值是当前 JVM 里类的值，不是当时序列化的值，别误以为它被恢复了。

**Q：Java 原生序列化有什么缺点？生产上一般用什么？**
A：缺点：只能 Java 用（不跨语言）、字节流体积大、性能差、有安全风险（反序列化漏洞可导致 RCE）。生产上 RPC/消息/缓存常用 JSON（可读、跨语言）、Protobuf/Thrift（体积小、快、需 IDL）、Kryo/Hessian（Java 高性能）等替代。

**Q：如何让某个字段不被序列化？有几种方式？**
A：① 用 `transient` 修饰；② 该字段所属类不实现 `Serializable`；③ 自定义 `writeObject`/`readObject` 手动跳过；④ 用 `Externalizable` 完全接管序列化逻辑，只写需要的字段。

## ⚠️ 易错点 / 加分项

- 误以为静态字段会被序列化并恢复——它取的是当前类的值。
- 父类不可序列化但子类可序列化时，父类的字段不会被保存，反序列化时父类必须有可访问的**无参构造器**，否则抛异常。
- `transient` 字段反序列化后是默认值，若业务依赖它需在 `readObject` 里重新赋值。
- 加分项：反序列化不调用构造方法（`Externalizable` 除外，它需要 public 无参构造），对象由 JVM 直接从字节流重建——这也是单例可能被反序列化破坏的原因，可用 `readResolve()` 防御。
- 加分项：Java 原生反序列化存在安全漏洞（如 Apache Commons Collections 利用链），不要反序列化不可信数据，可用 `ObjectInputFilter`（JDK 9+）做白名单过滤。
