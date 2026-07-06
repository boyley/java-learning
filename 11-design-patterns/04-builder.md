# 04 · 建造者模式（Builder）

> 把一个复杂对象的**构建过程**和它的**表示**分离，用链式调用一步步装配，解决「构造器参数太多、可选参数组合爆炸」的痛点。面试重要度 ⭐⭐。

## 📖 核心知识

当一个对象有很多字段（尤其是可选字段）时，传统写法要么写一堆重叠构造器（`telescoping constructor`），要么用 setter（对象可变、无法保证一致性）。建造者模式用一个内部 `Builder` 收集参数，最后 `build()` 一次性创建对象。

```java
public class User {
    private final String name;   // 必填
    private final int age;       // 可选
    private final String email;  // 可选

    private User(Builder b) {    // 私有构造器，只能通过 Builder 创建
        this.name = b.name;
        this.age = b.age;
        this.email = b.email;
    }

    public static class Builder {
        private final String name;        // 必填放 Builder 构造器
        private int age;
        private String email;

        public Builder(String name) { this.name = name; }
        public Builder age(int age) { this.age = age; return this; }       // 返回 this 支持链式
        public Builder email(String email) { this.email = email; return this; }
        public User build() { return new User(this); }
    }
}

// 使用：可读性极强，参数顺序无关
User u = new User.Builder("Tom").age(18).email("tom@x.com").build();
```

要点：链式调用的关键是每个设置方法**返回 `this`**；`build()` 里可集中做参数校验，保证构建出的对象状态合法且**不可变**。

### 真实应用

- **`StringBuilder` / `StringBuffer`**：`append().append()...toString()`，链式构建字符串。
- **Lombok `@Builder`**：编译期自动生成 Builder 内部类，省去手写模板代码。
- **`Stream.Builder`**、**`Calendar.Builder`（JDK 8）**、**`Locale.Builder`**。
- **OkHttp / Retrofit**：`new OkHttpClient.Builder().connectTimeout(...).build()`。
- **MyBatis `SqlSessionFactoryBuilder`**、**`Thread.Builder`（JDK 21 虚拟线程）**。

## 🔑 面试要点

- 解决的问题：**构造器参数过多 + 可选参数组合爆炸 + 保证对象不可变**。
- 链式调用靠每个方法 `return this`。
- 与 JavaBean（无参构造 + setter）的区别：Builder 能保证**对象构建完就不可变**、参数校验集中在 `build()`，而 setter 方式对象长期可变、可能处于半初始化状态。
- 典型应用：`StringBuilder`、Lombok `@Builder`、各种 SDK 的 `XxxClient.Builder`。
- 出自《Effective Java》第 2 条：**面对多参数构造器，优先考虑 Builder**。

## ❓ 高频面试题

**Q：建造者模式解决什么问题？**
A：解决复杂对象「构造器参数太多、可选参数很多导致构造器重载爆炸」的问题。用链式 Builder 逐步设置参数，可读性好、参数顺序无关、能在 `build()` 里统一校验并产出不可变对象。

**Q：建造者模式和工厂模式的区别？**
A：工厂模式关注「**造出哪个**对象」（返回不同类型的产品，一步创建）；建造者模式关注「**怎么一步步造**一个复杂对象」（同一类型，多步装配）。工厂强调结果，建造者强调过程。

**Q：用 setter 也能设置多个属性，为什么还要 Builder？**
A：setter 方式对象在设置过程中一直是可变的、可能处于不完整状态，且无法做成 `final` 不可变对象；Builder 把中间状态收在 Builder 里，`build()` 时一次性构造并校验，产出的是不可变、线程安全的对象。

## ⚠️ 易错点 / 加分项

- **误区**：认为 Builder 就是「一堆 setter 返回 this」——核心价值其实是**产出不可变对象 + 集中校验**，不只是链式。
- **加分**：能说出经典 Builder（GoF）有 `Director` 指挥者角色，而日常 Java 里常用的是**简化版**（省略 Director，直接 Builder 内部类）。
- **加分**：Lombok `@Builder` 是编译期注解处理（APT）生成代码，运行时无开销。
- **加分**：链式调用（fluent API）和建造者不完全等价——`StringBuilder.append` 是链式，但严格说是「构建者思想」的体现。
