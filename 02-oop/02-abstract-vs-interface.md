# 02 · 抽象类 vs 接口（Abstract Class vs Interface）

> 抽象类表 "is-a" 可有状态和实现，接口表 "can-do" 定义能力契约；JDK8 后接口能有默认方法，界限变模糊但语义定位不同。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

**抽象类（abstract class）**：用 `abstract` 修饰，不能实例化，可含抽象方法（无实现）和普通方法，可有构造器、成员变量、各种访问修饰符。子类 `extends` 且**只能继承一个**。

**接口（interface）**：定义行为契约。JDK8 前只能有抽象方法和常量；JDK8 起新增 **default 方法**（带实现）和 **static 方法**；JDK9 起可有 **private 方法**。类 `implements` 且**可实现多个**。

### 核心对比表

| 维度 | 抽象类 | 接口 |
| --- | --- | --- |
| 关键字 | `abstract class` / `extends` | `interface` / `implements` |
| 多继承 | 单继承（一个） | 可实现多个 |
| 方法 | 抽象 + 普通均可 | JDK8 前全抽象；JDK8+ 可 default/static；JDK9+ 可 private |
| 成员变量 | 普通变量、可各种修饰符 | 只能 `public static final` 常量 |
| 构造器 | 有（供子类调用，非实例化） | 无 |
| 访问修饰符 | 任意 | 方法默认 `public`，不能 private（普通抽象方法） |
| 语义 | is-a，模板 + 部分实现 | can-do，能力契约 |
| 状态 | 可持有状态 | 无实例状态 |

### JDK8 默认方法（default）

为接口新增方法而**不破坏已有实现类**，是 Lambda/Stream 能加入 `Collection` 而不改动海量实现类的关键。

```java
interface Greeter {
    void hello();                       // 抽象
    default void greet() {              // 默认方法，有实现
        System.out.println("Hi");
    }
    static Greeter create() { return () -> {}; }  // 静态方法
}
```

**菱形问题**：一个类实现的两个接口有同名 default 方法，必须**显式重写**解决冲突，可用 `接口名.super.方法()` 指定：

```java
class C implements A, B {
    public void m() { A.super.m(); }   // 明确用 A 的默认实现
}
```

### 如何选（决策）

- 有**共享状态/字段**、需要构造逻辑、子类是 is-a 关系 → **抽象类**（如 `AbstractList`）。
- 只定义**能力/契约**、需要多继承、不同类型都能实现 → **接口**（如 `Comparable`、`Runnable`）。
- 经典组合：接口定契约 + 抽象类给骨架实现（模板方法模式，如 `AbstractList implements List`）。

## 🔑 面试要点

- 抽象类单继承、接口可多实现，这是选择的硬约束。
- 抽象类可有状态、构造器、任意修饰符；接口只有常量 + 抽象/default/static/private 方法。
- JDK8 接口新增 default（带实现）和 static 方法，JDK9 加 private 方法。
- default 方法目的：向后兼容地给接口增加方法。
- 两接口同名 default 冲突需显式重写，用 `接口名.super.方法()` 指定。
- 语义：抽象类 is-a（模板），接口 can-do（能力契约）。
- 优先用接口（灵活、可多实现），需要复用实现和状态时用抽象类。

## ❓ 高频面试题

**Q：抽象类和接口的区别？**
A：见对比表核心几点——抽象类单继承、可有状态和构造器、可含普通方法；接口可多实现、只有常量和方法（JDK8 后可 default/static）。语义上抽象类是 is-a 模板，接口是能力契约。

**Q：JDK8 接口能有方法实现了，抽象类还有存在意义吗？**
A：有。抽象类能持有**实例状态（成员变量）**、有构造器、能定义 protected/private 成员并控制访问，这些接口都做不到；接口的 default 方法不能访问实例字段。二者定位不同，抽象类适合"有状态的模板"。

**Q：一个类实现两个接口，有同名 default 方法怎么办？**
A：编译器报错，强制该类**重写**这个方法解决冲突；重写里可用 `接口名.super.方法()` 选择复用某个接口的默认实现。

**Q：接口里能定义成员变量吗？**
A：能写字段，但会被隐式加上 `public static final`，本质是常量，不是实例状态。不能定义可变的实例变量。

## ⚠️ 易错点 / 加分项

- 接口字段默认 `public static final`，方法默认 `public abstract`（default/static 除外），别以为能写 private 普通字段。
- 抽象类可以没有任何抽象方法（仅为阻止实例化），也可以有构造器（供子类 super 调用）。
- default 方法不能重写 `Object` 的方法（如 `equals`），会编译报错。
- 加分：能举 JDK 实例——`Comparator` 大量 default/static 方法、`AbstractList` 模板方法，体现接口+抽象类协作。
- 加分：说清 default 方法引入的根本动机是"接口演进的向后兼容"，把 Stream 加进 `Collection` 而不改上千实现类。
