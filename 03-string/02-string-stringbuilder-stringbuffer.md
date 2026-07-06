# 02 · String / StringBuilder / StringBuffer

> 三者核心区别：String 不可变、StringBuilder 可变不安全、StringBuffer 可变且线程安全；循环拼接必用 StringBuilder。面试重要度 ⭐⭐⭐。

## 📖 核心知识

三者定位：

| 类 | 可变性 | 线程安全 | 底层 | 适用场景 |
|---|---|---|---|---|
| `String` | 不可变 | 安全（因不可变） | `final` 字符数组 | 少量、几乎不变的字符串；常量、key |
| `StringBuilder` | 可变 | **不安全** | 可扩容字符数组 | 单线程下大量拼接（最常用） |
| `StringBuffer` | 可变 | 安全（`synchronized`） | 可扩容字符数组 | 多线程共享同一缓冲区（少见） |

`StringBuilder`（JDK 5 引入）和 `StringBuffer` 都继承自 `AbstractStringBuilder`，内部维护一个**会自动扩容**的字符数组，`append` 直接在原数组上追加，不产生新对象。二者代码几乎一样，唯一区别是 StringBuffer 的方法加了 `synchronized`，所以线程安全但更慢。

**字符串拼接的底层**：JDK 8 中，编译器会把源码里的 `+` 拼接优化成 `StringBuilder.append`。例如：

```java
String s = a + b + c;
```

编译后大致等价于：

```java
String s = new StringBuilder().append(a).append(b).append(c).toString();
```

> JDK 9+ 起 `+` 拼接改用 `invokedynamic` + `StringConcatFactory` 实现，运行时动态生成拼接逻辑，但结论不变：**编译期常量拼接会直接合并，运行期变量拼接才需要缓冲区**。

**为什么循环里不能用 `+`**：编译器只对**一条语句内**的 `+` 优化成一个 StringBuilder。循环体每一轮都是独立语句，于是**每次循环都 new 一个 StringBuilder**，产生大量临时对象：

```java
// 反例：每轮循环 new 一个 StringBuilder + 一个 String，O(n²) 级开销
String s = "";
for (int i = 0; i < 10000; i++) {
    s += i;                 // 等价 s = new StringBuilder().append(s).append(i).toString();
}

// 正例：全程复用同一个缓冲区
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 10000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

**编译期常量例外**：`"a" + "b" + "c"` 全是常量，编译器直接合并成 `"abc"`，不走 StringBuilder，也会进常量池。含 `final` 常量的表达式同理。

## 🔑 面试要点

- 一句话记忆：**String 不可变、StringBuilder 快但不安全、StringBuffer 安全但慢**。
- StringBuilder / StringBuffer 都继承 `AbstractStringBuilder`，共用可扩容 `char[]`。
- StringBuffer 方法加了 `synchronized` 所以线程安全，代价是性能低于 StringBuilder。
- 单条语句的 `+` 拼接会被编译器优化为 StringBuilder（JDK 8）/ `invokedynamic`（JDK 9+），**不用手动改写**。
- **循环拼接一定手动用 StringBuilder**，否则每轮 new 一个新缓冲区，性能退化到 O(n²)。
- 纯字面量常量拼接（`"a"+"b"`）在编译期就合并为一个常量，进常量池。
- 性能排序（大量拼接）：`StringBuilder > StringBuffer >> String`。

## ❓ 高频面试题

**Q：String、StringBuilder、StringBuffer 有什么区别？**
A：String 不可变，每次修改都产生新对象；StringBuilder 可变、线程不安全、性能最高；StringBuffer 可变、方法加 `synchronized` 所以线程安全但较慢。单线程大量拼接用 StringBuilder，多线程共享缓冲区才用 StringBuffer。

**Q：`String s = "a" + "b" + "c"` 会创建几个对象？**
A：只有一个 `"abc"`，且在常量池里。因为都是编译期常量，编译器直接合并，不会走 StringBuilder。

**Q：为什么循环里用 `+` 拼接字符串性能差？**
A：编译器只对单条语句的 `+` 优化成一个 StringBuilder。循环每一轮是独立语句，所以每轮都 new 一个 StringBuilder 并 `toString` 生成新 String，产生大量临时对象，复杂度退化到 O(n²)。应在循环外建一个 StringBuilder 复用。

**Q：StringBuffer 是怎么做到线程安全的？**
A：它在 `append`、`insert`、`toString` 等方法上加了 `synchronized`，同一时刻只有一个线程能操作缓冲区。代价是加锁开销，单线程下性能不如 StringBuilder。

## ⚠️ 易错点 / 加分项

- **误区**："`+` 拼接一定慢，要全部换 StringBuilder。"——单条语句的 `+` 编译器已优化，可读性优先，不必手动改；**只有循环/大量分步拼接才必须手动 StringBuilder**。
- **加分**：能说出 JDK 8 用 StringBuilder、JDK 9+ 改用 `invokedynamic`（`StringConcatFactory`）实现字符串拼接的版本差异。
- **加分**：`new StringBuilder(int capacity)` 预估容量可减少扩容次数（默认 16，扩容为 `旧容量*2+2`），大拼接场景是优化点。
- **加分**：StringBuffer 的线程安全是"单个方法级别"的，多个方法组合调用时仍需外部加锁，不是万能的。
- **易错**：`sb.append("a").append("b")` 是链式调用同一对象；而 `String` 的 `s.concat(...)` 每次返回新对象。
