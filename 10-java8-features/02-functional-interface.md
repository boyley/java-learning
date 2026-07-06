# 02 · 函数式接口（Functional Interface）

> 只含一个抽象方法的接口叫函数式接口，是 Lambda 的「靶子」类型。`java.util.function` 包提供了四大核心接口 Function/Consumer/Supplier/Predicate 及大量变体。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

**定义**：函数式接口（Functional Interface）是**有且仅有一个抽象方法**的接口（SAM，Single Abstract Method）。Lambda 表达式和方法引用只能赋值给函数式接口类型的变量，因为编译器需要知道 Lambda 到底实现的是哪个方法。

**`@FunctionalInterface` 注解**：可选，加上它编译器会**强制检查**该接口是否只有一个抽象方法，多一个就编译报错。它是一种「契约声明」，防止后人误加抽象方法破坏函数式语义。

```java
@FunctionalInterface
interface MyFunc {
    int apply(int x);          // 唯一抽象方法
    default int twice(int x) { return apply(x) * 2; }  // default 方法不算
    static void hello() {}     // static 方法不算
    // Object 的 public 方法（如 equals/toString）也不算抽象方法
}
```

注意：**默认方法、静态方法、以及重写自 `Object` 的 public 方法（如 `equals`、`toString`）都不计入**「抽象方法」个数。所以 `Comparator` 里虽然声明了 `equals`，仍是函数式接口。

**四大核心函数式接口**（`java.util.function` 包）：

| 接口 | 抽象方法 | 含义 | 典型场景 |
| --- | --- | --- | --- |
| `Function<T,R>` | `R apply(T t)` | 一进一出，做转换 | `map`、类型转换 |
| `Consumer<T>` | `void accept(T t)` | 只进不出，消费副作用 | `forEach`、打印 |
| `Supplier<T>` | `T get()` | 不进有出，提供/生产 | 延迟求值、工厂 |
| `Predicate<T>` | `boolean test(T t)` | 一进出布尔，做判断 | `filter`、条件过滤 |

再加两个也很常用：

| 接口 | 抽象方法 | 含义 |
| --- | --- | --- |
| `UnaryOperator<T>` | `T apply(T t)` | 一元运算，`Function<T,T>` 的特例 |
| `BinaryOperator<T>` | `T apply(T,T)` | 二元运算，`reduce` 常用 |

**常见变体**（记住命名规律即可，不用死背全部）：
- **二元版**：`BiFunction<T,U,R>`、`BiConsumer<T,U>`、`BiPredicate<T,U>`（两个入参）。
- **基本类型特化版**：为避免自动装箱开销，提供 `IntFunction`、`ToIntFunction`、`IntPredicate`、`IntConsumer`、`IntSupplier`、`IntUnaryOperator` 等，`Int`/`Long`/`Double` 各一套。

**默认方法组合**：这些接口自带默认方法可链式组合，很实用：
- `Function`：`andThen`（先自己后参数）、`compose`（先参数后自己）。
- `Predicate`：`and`、`or`、`negate`。
- `Consumer`：`andThen`。

```java
Predicate<String> notEmpty = s -> !s.isEmpty();
Predicate<String> shortStr = s -> s.length() < 5;
Predicate<String> both = notEmpty.and(shortStr);   // 组合

Function<Integer,Integer> add1 = x -> x + 1;
Function<Integer,Integer> mul2 = x -> x * 2;
add1.andThen(mul2).apply(3);   // (3+1)*2 = 8
add1.compose(mul2).apply(3);   // (3*2)+1 = 7
```

## 🔑 面试要点

- 函数式接口 = 只有一个抽象方法的接口（SAM），是 Lambda / 方法引用的目标类型。
- `@FunctionalInterface` 是可选的编译期校验注解，保证接口只有一个抽象方法。
- default 方法、static 方法、Object 的 public 方法**不算**抽象方法。
- 四大核心：`Function`（转换 T→R）、`Consumer`（消费 T→void）、`Supplier`（生产 →T）、`Predicate`（判断 T→boolean）。
- 记忆口诀：有进有出 Function、只进不出 Consumer、只出不进 Supplier、进出布尔 Predicate。
- 变体规律：`Bi`+ 名 = 两参数；`Int/Long/Double`+ 名 = 基本类型特化（避免装箱）；`UnaryOperator`/`BinaryOperator` = 输入输出同类型。
- `Runnable`、`Callable`、`Comparator`、`Comparable` 也都是函数式接口。

## ❓ 高频面试题

**Q：什么是函数式接口？`@FunctionalInterface` 注解有什么用？**
A：函数式接口是只包含一个抽象方法的接口，它是 Lambda 表达式的目标类型。`@FunctionalInterface` 注解不是必需的，作用是让编译器强制校验「只有一个抽象方法」，若违反就报错，起到防止破坏契约的保护作用。default/static/Object 的方法不计入抽象方法数。

**Q：Function、Consumer、Supplier、Predicate 分别用在什么场景？**
A：`Function<T,R>` 做转换（如 Stream 的 map）；`Consumer<T>` 做消费无返回（如 forEach 打印）；`Supplier<T>` 做延迟生产（如 Optional.orElseGet、日志的懒加载）；`Predicate<T>` 做布尔判断（如 Stream 的 filter）。

**Q：为什么有 IntFunction、ToIntFunction 这些基本类型特化接口？**
A：为了**避免自动装箱/拆箱的性能开销**。如果用 `Function<Integer,Integer>` 处理大量 int，会频繁装箱成 Integer 对象产生 GC 压力；基本类型特化版直接操作 int，省去装箱，在 Stream 大数据量处理时性能提升明显（对应 IntStream 等）。

## ⚠️ 易错点 / 加分项

- 误以为函数式接口只能有一个方法——是只能有一个**抽象**方法，default/static 方法可以有多个。
- 忘了 `Comparator` 也是函数式接口（虽然它声明了 equals，但那是 Object 的方法不计数）。
- 加分：能说清 `andThen` 和 `compose` 的执行顺序差异（andThen 先执行调用者，compose 先执行参数）。
- 加分：提到基本类型特化接口是为了性能（避免装箱），并关联到 IntStream/LongStream。
- 加分：`Supplier` 在延迟求值场景（`orElseGet` vs `orElse`）能避免不必要的对象创建。
