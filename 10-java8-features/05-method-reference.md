# 05 · 方法引用（Method Reference）

> 方法引用是 Lambda 的进一步简化：当 Lambda 体只是「调用一个已有方法」时，可用 `类名::方法名` 直接引用。它是 Lambda 的语法糖，共四种形式。面试重要度：⭐⭐ 常考。

## 📖 核心知识

当一个 Lambda 表达式的唯一作用就是**调用一个已经存在的方法**时，可以用方法引用 `::` 让代码更简洁、可读。它和 Lambda 一样，目标类型也必须是**函数式接口**，本质就是 Lambda 的简写。

```java
// Lambda
Function<String, Integer> f1 = s -> Integer.parseInt(s);
// 方法引用（等价）
Function<String, Integer> f2 = Integer::parseInt;
```

**四种形式**：

| 形式 | 语法 | 等价 Lambda | 例子 |
| --- | --- | --- | --- |
| ① 静态方法引用 | `类名::静态方法` | `(args) -> 类名.静态方法(args)` | `Integer::parseInt` |
| ② 特定对象的实例方法 | `对象::实例方法` | `(args) -> 对象.方法(args)` | `System.out::println` |
| ③ 特定类型任意对象的实例方法 | `类名::实例方法` | `(a, b...) -> a.方法(b...)` | `String::toUpperCase` |
| ④ 构造方法引用 | `类名::new` | `(args) -> new 类名(args)` | `ArrayList::new` |

**逐个说明**：

```java
// ① 静态方法引用
Function<String, Integer> parse = Integer::parseInt;   // s -> Integer.parseInt(s)

// ② 特定对象的实例方法：out 是已存在的对象
Consumer<String> print = System.out::println;          // s -> System.out.println(s)

// ③ 特定类型任意对象的实例方法：第一个参数变成调用者
// String::toUpperCase 等价于 (String s) -> s.toUpperCase()
Function<String, String> upper = String::toUpperCase;
// 排序：Comparator 的两个参数，第一个当调用者，第二个当实参
list.sort(String::compareTo);   // (a, b) -> a.compareTo(b)

// ④ 构造方法引用
Supplier<List<String>> newList = ArrayList::new;       // () -> new ArrayList<>()
Function<String, User> newUser = User::new;            // name -> new User(name)
```

**第 ②③ 种最容易混**：都是「类名/对象 :: 实例方法」，区别在于 `::` 左边是**具体对象实例**（②）还是**类名**（③）。第 ③ 种的特别之处是：Lambda 的**第一个参数被当作方法的调用者**，其余参数才是方法实参。这也是为什么 `String::compareTo` 能匹配 `Comparator<String>`（两个参数）。

**与 Lambda 的关系**：方法引用不是新东西，它是 Lambda 的语法糖——凡是能写成方法引用的，都能写成 Lambda；反之不一定。当 Lambda 只是「原样转调一个方法、参数一一对应」时才能用方法引用；若还要做额外运算（如 `x -> x + 1`）就只能用 Lambda。

## 🔑 面试要点

- 方法引用是 Lambda 的简写，用 `::`，目标类型仍必须是函数式接口。
- 四种形式：静态方法 `类::静态`、实例方法（特定对象）`对象::方法`、实例方法（任意对象）`类::方法`、构造 `类::new`。
- 第三种 `类名::实例方法` 的精髓：Lambda 第一个参数充当方法调用者。
- 能用方法引用的前提：Lambda 体只是「转调一个已有方法且参数一一对应」，无额外逻辑。
- 数组构造引用 `int[]::new` 等价于 `size -> new int[size]`。
- 可读性更好，但不是必须——复杂逻辑仍用 Lambda。

## ❓ 高频面试题

**Q：方法引用和 Lambda 是什么关系？**
A：方法引用是 Lambda 表达式的语法糖，用于「Lambda 体仅仅是调用某个已有方法」的场景。凡能写成方法引用的都能写成等价 Lambda；但需要额外运算或多步逻辑的 Lambda 无法写成方法引用。两者目标类型都必须是函数式接口，底层实现机制也一致。

**Q：方法引用有哪几种形式？`String::toUpperCase` 属于哪种？**
A：四种——静态方法引用（`Integer::parseInt`）、特定对象的实例方法引用（`System.out::println`）、特定类型任意对象的实例方法引用（`String::toUpperCase`）、构造方法引用（`ArrayList::new`）。`String::toUpperCase` 属于第三种：`::` 左边是类名，实际调用时 Lambda 的第一个参数（某个 String 对象）作为方法的调用者。

## ⚠️ 易错点 / 加分项

- 混淆第②③种：区别看 `::` 左边是对象实例还是类名；类名形式下第一个参数是调用者。
- 以为所有 Lambda 都能改成方法引用——只有「纯转调、参数对应」才行，带运算的不行。
- `类名::new` 匹配哪个构造器由目标函数式接口的参数决定（编译器重载解析）。
- 加分：能举出 Stream 中常见搭配，如 `.map(String::trim)`、`.collect(Collectors.toCollection(TreeSet::new))`。
- 加分：指出方法引用和 Lambda 编译后都走 invokedynamic，性能一致，选择只看可读性。
