# 04 · Optional（Optional）

> Optional 是一个「可能为空」的容器，用来显式表达「值可能不存在」，把 null 检查从散落的 if 变成链式调用，优雅规避 NPE。但用错反而更啰嗦。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

`Optional<T>` 是对一个可能为 `null` 的值的封装。它的价值不是「消灭 null」，而是**在类型层面强制调用者意识到值可能为空**，从而写出更安全的代码，避免 `NullPointerException`。

**创建**：

```java
Optional<String> a = Optional.of("hi");       // 值非 null，传 null 会立刻抛 NPE
Optional<String> b = Optional.ofNullable(x);   // x 可能为 null，为 null 则得到空 Optional
Optional<String> c = Optional.empty();         // 空 Optional
```

- `of(value)`：值**确定非 null** 时用，传 null 直接抛 NPE。
- `ofNullable(value)`：值**可能为 null** 时用，最常用。
- `empty()`：显式创建空容器。

**取值 / 兜底**：

```java
opt.get();                    // 有值返回，空则抛 NoSuchElementException —— 不推荐直接用
opt.orElse("default");        // 空时返回默认值（无论如何都会构造 "default"）
opt.orElseGet(() -> compute()); // 空时才调用 Supplier 生产默认值（惰性，推荐）
opt.orElseThrow(() -> new BizException("not found")); // 空时抛自定义异常
```

**判断 / 消费**：

```java
opt.isPresent();                       // 是否有值
opt.ifPresent(v -> System.out.println(v)); // 有值才执行，替代 if 判断
```

**链式转换（Optional 的精髓）**：`map`、`flatMap`、`filter` 让你安全地链式取值，任何一环为空整条链得到空 Optional，不会 NPE：

```java
// 传统写法：层层 null 判断
String city = null;
if (user != null && user.getAddress() != null) {
    city = user.getAddress().getCity();
}
// Optional 写法：链式、无 NPE
String city2 = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("Unknown");
```

`map` 与 `flatMap` 区别：若映射函数本身返回的是 `Optional`，用 `flatMap` 避免出现 `Optional<Optional<T>>` 嵌套。

**`orElse` vs `orElseGet` 的关键差异**：`orElse` 的参数是**已经算好的值**，即使 Optional 有值、默认值也会被构造（可能浪费/有副作用）；`orElseGet` 传的是 `Supplier`，**只有为空时才调用**，更高效，尤其默认值构造昂贵（查库、new 大对象）时。

```java
opt.orElse(createExpensive());     // createExpensive() 总会执行！
opt.orElseGet(() -> createExpensive()); // 只有 opt 为空才执行
```

## 🔑 面试要点

- Optional 是显式表达「值可能不存在」的容器，目的是让调用方主动处理空、减少 NPE。
- 创建：`of`（非 null，否则 NPE）、`ofNullable`（可能 null，最常用）、`empty`。
- 取值：优先 `orElse/orElseGet/orElseThrow/ifPresent`，**少用 `get()`**（空会抛异常，等于没解决问题）。
- `orElse` 参数总会被求值；`orElseGet` 惰性、只在空时求值——默认值昂贵时用后者。
- `map/flatMap/filter` 支持安全链式取值，中途为空自动短路成空 Optional。
- `flatMap` 用于映射函数返回 Optional 的情况，避免 `Optional<Optional<T>>` 嵌套。
- 设计定位：主要用作**方法返回值**，不推荐做字段和方法参数。

## ❓ 高频面试题

**Q：Optional 是怎么帮助避免 NPE 的？它真能消灭 null 吗？**
A：它把「可能为空」显式编码进返回类型，强制调用者用 `orElse/ifPresent/map` 等安全方式处理，而不是不假思索地解引用。它并没有从物理上消灭 null（内部仍可能是空容器），而是把空值处理从容易遗漏的散乱 if 判断，变成编译期可见、链式且短路安全的调用，从而减少 NPE。

**Q：`orElse` 和 `orElseGet` 有什么区别？**
A：`orElse(T)` 的参数是一个已经计算好的值，无论 Optional 是否有值，这个默认值表达式都会被求值；`orElseGet(Supplier)` 传的是一个函数，只有 Optional 为空时才会调用它生成默认值。因此当默认值的构造开销大或有副作用时，应优先用 `orElseGet`。

**Q：Optional 有哪些反模式（错误用法）？**
A：①上来就 `optional.get()` 而不判断，本质和直接解引用没区别，还多套一层；②用 `isPresent()` + `get()` 两步写法，不如直接 `map/ifPresent/orElse`；③把 Optional 当字段或方法参数（它没实现 Serializable，也让 API 变复杂），它设计上主要用于返回值；④为基本类型用 `Optional<Integer>` 而不用 `OptionalInt`（多余装箱）；⑤在 Optional 里包裹后又 `orElse(null)` 绕回 null。

## ⚠️ 易错点 / 加分项

- `Optional.of(null)` 会立刻抛 NPE——值可能为 null 时必须用 `ofNullable`。
- 直接 `get()` 空 Optional 抛 `NoSuchElementException`，把 NPE 换成了另一种异常，没解决问题。
- 用 `isPresent()`+`get()` 是「换汤不换药」，应改用 `ifPresent`/`map`/`orElse` 的函数式写法。
- Optional 不该做类字段和方法入参：它未实现 `Serializable`，且徒增复杂度，官方定位就是返回值。
- 加分：知道有 `OptionalInt/OptionalLong/OptionalDouble` 避免基本类型装箱。
- 加分：`orElseGet` 的惰性求值本质是 `Supplier`，能避免不必要的对象创建，性能敏感处更优。
