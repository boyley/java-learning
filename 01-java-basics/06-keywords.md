# 06 · 常见关键字（Keywords）

> `this`/`super` 指代当前对象/父类，`instanceof` 判类型，`transient` 排除序列化字段；这些高频关键字要点速览。面试重要度：⭐⭐ 常考。

## 📖 核心知识

### this 与 super

- **`this`**：指向**当前对象**。用途：区分同名的成员变量与局部变量（`this.x = x`）、在构造器中调用本类另一个构造器（`this(...)`，必须是第一行）、返回自身实现链式调用。
- **`super`**：指向**父类**部分。用途：访问被子类隐藏的父类成员（`super.field`）、调用父类被重写的方法（`super.method()`）、调用父类构造器（`super(...)`，必须是子类构造器第一行）。

```java
class Child extends Parent {
    int x;
    Child(int x) {
        super();      // 调父类构造器（不写则默认隐式调用无参）
        this.x = x;   // this 区分成员与参数
    }
}
```

注意：`this(...)` 和 `super(...)` 都必须是构造器**第一行**，因此二者不能同时出现在同一个构造器里。

### instanceof

判断对象是否为某类型（或其子类型/实现类型）的实例，返回 `boolean`。左操作数为 `null` 时恒返回 `false`，因此可安全用于空判断前置。

```java
Object o = "hello";
if (o instanceof String) {
    String s = (String) o;      // 传统写法
}
// JDK16+ 模式匹配：if (o instanceof String s) { ... 直接用 s ... }
```

### transient

修饰**成员变量**，标记该字段**不参与默认序列化**（`Serializable`）。反序列化后该字段为默认值（0/null）。常用于敏感信息（密码）或可由其他字段推导、无需持久化的字段。

```java
class User implements Serializable {
    String name;
    transient String password;  // 不会被序列化
}
```

### 其他高频关键字速览

| 关键字 | 作用 | 常见考点 |
| --- | --- | --- |
| `native` | 声明本地方法，由 C/C++ 等实现 | 如 `Object.hashCode()`、`Thread.start0` |
| `volatile` | 保证可见性、禁止指令重排 | 并发章节详解 |
| `synchronized` | 同步锁 | 并发章节详解 |
| `strictfp` | 严格浮点计算，跨平台结果一致 | 了解即可 |
| `assert` | 断言，默认关闭，`-ea` 开启 | 测试/调试用 |
| `enum` | 枚举类型 | 单例、常量集合 |
| `default` | 接口默认方法（JDK8） | 见 `../02-oop/02-abstract-vs-interface.md` |

## 🔑 面试要点

- `this` 指当前对象；`this(...)` 调本类构造器且必须第一行。
- `super` 指父类；`super(...)` 调父类构造器且必须第一行，与 `this(...)` 互斥。
- 子类构造器不写 `super(...)` 时编译器自动插入 `super()`（父类无参构造）。
- `instanceof` 判类型，`null instanceof X` 恒 false，可防空指针。
- `transient` 让字段跳过默认序列化，反序列化后为默认值。
- `native` 表示方法由本地代码实现；`Object` 的 `hashCode`、`getClass`、`notify` 等都是 native。

## ❓ 高频面试题

**Q：`this(...)` 和 `super(...)` 能同时写在一个构造器里吗？**
A：不能。两者都要求是构造器的第一条语句，位置冲突。一个构造器要么第一行 `this(...)` 转调本类构造器，要么第一行 `super(...)` 调父类构造器。

**Q：`transient` 有什么用？被它修饰的字段反序列化后是什么值？**
A：让字段不参与默认（`Serializable`）序列化，常用于敏感或可推导字段。反序列化后该字段是类型默认值（引用为 null、数值为 0、boolean 为 false）。注意 `static` 字段本就不属于对象状态，也不会被序列化。

**Q：`null instanceof String` 结果是什么？**
A：`false`。`instanceof` 左侧为 null 时一律返回 false，所以用它判断类型前无需额外判空。

**Q：子类构造器一定会调用父类构造器吗？**
A：是。若不显式写 `super(...)`，编译器自动插入无参 `super()`；若父类没有无参构造器，子类又不显式调用带参 `super(...)`，会编译报错。

## ⚠️ 易错点 / 加分项

- 误以为 `transient` 能阻止 `Externalizable` 或手写 `writeObject` 的序列化——它只对**默认**序列化生效。
- `static` 字段不会被序列化，别和 `transient` 混为"都是跳过"，原因不同（static 属于类）。
- 加分：`instanceof` 编译期就会检查左右类型是否可能有继承关系，完全无关的类型比较会编译报错。
- 加分：能提 JDK14+ 的 `instanceof` 模式匹配（`o instanceof String s`）省去强转样板代码。
- 加分：`super` 只能上溯一层，不能 `super.super.method()` 跨级访问祖父类方法。
