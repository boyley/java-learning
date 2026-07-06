# 03 · 通配符与 PECS 原则（Wildcards & PECS）

> `? extends` 只读（生产者）、`? super` 只写（消费者），记忆口诀 PECS：Producer-Extends，Consumer-Super。面试重要度 ⭐⭐。

## 📖 核心知识

**为什么需要通配符**：泛型是**不可变（invariant）**的——`List<Integer>` 不是 `List<Number>` 的子类型，尽管 `Integer` 是 `Number` 的子类。

```java
List<Integer> ints = new ArrayList<>();
List<Number> nums = ints;   // 编译错误！List<Integer> 不是 List<Number>
```

这是为了类型安全（否则能往 `nums` 里塞 `Double` 破坏 `ints`）。要写通用方法处理"某种 Number 的 List"，就需要通配符 `?`。

**三种通配符**：

1. **无界 `<?>`**：表示"未知类型"。只能读出 `Object`，除了 `null` 什么都不能写入。适合只关心结构不关心元素类型的场景（如 `list.size()`、打印）。

2. **上界 `<? extends T>`**：表示"T 或 T 的某个子类型"。
   - **能读**：取出来的一定是 T（或子类），可当 T 用。
   - **不能写**（除 `null`）：因为不知道确切是哪个子类型，塞任何具体元素都可能类型不符。
   - → **只读、生产者（Producer）**。

3. **下界 `<? super T>`**：表示"T 或 T 的某个父类型"。
   - **能写**：T 及其子类都能安全放进去（一定 is-a 那个父类型）。
   - **读**：取出来只能当 `Object`（无法确定具体父类型）。
   - → **可写、消费者（Consumer）**。

```java
// ? extends Number：从中读取数字（生产者），不能往里加
double sum(List<? extends Number> src) {
    double total = 0;
    for (Number n : src) total += n.doubleValue();  // 读 OK
    // src.add(1);   // 编译错误！不能写
    return total;
}

// ? super Integer：往里写整数（消费者），读只能当 Object
void fill(List<? super Integer> dst) {
    dst.add(1);           // 写 OK：Integer 一定 is-a 上界的父类型
    dst.add(2);
    // Integer x = dst.get(0);  // 编译错误！读出来只能是 Object
    Object o = dst.get(0);      // OK
}
```

**PECS 原则**（Producer-Extends, Consumer-Super）：
- 如果一个参数是**生产者**（你从它里面**读取/取出**数据）→ 用 `? extends T`；
- 如果一个参数是**消费者**（你往它里面**写入/放入**数据）→ 用 `? super T`；
- 既读又写 → 不用通配符，用确定类型 `T`。

**经典例子**：JDK `Collections.copy`：

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    //                       消费者(写入): super      生产者(读取): extends
    for (int i = 0; i < src.size(); i++) {
        dest.set(i, src.get(i));   // 从 src 读，往 dest 写
    }
}
```

`src` 是数据来源（生产者）→ `? extends T`；`dest` 是写入目标（消费者）→ `? super T`。完美体现 PECS。

## 🔑 面试要点

- 泛型**不可变**：`List<Integer>` ≠ `List<Number>` 的子类型，需通配符解决协变/逆变需求。
- `<? extends T>`：**上界**，只读（生产者），除 `null` 外不能写。
- `<? super T>`：**下界**，可写 T 及其子类（消费者），读出来只能当 `Object`。
- `<?>`：无界，只能读 `Object`、只能写 `null`。
- **PECS**：Producer-Extends（读用 extends）、Consumer-Super（写用 super）；既读又写用确定类型。
- 记忆经典签名 `Collections.copy(List<? super T> dest, List<? extends T> src)`。

## ❓ 高频面试题

**Q：`List<? extends Number>` 和 `List<? super Number>` 有什么区别？**
A：`? extends Number` 是上界，表示"Number 或其子类的 List"，只能安全读取（取出当 Number 用），不能写入（除 null）；`? super Number` 是下界，表示"Number 或其父类的 List"，可以安全写入 Number 及其子类，读取只能当 Object。前者用于取数据，后者用于放数据。

**Q：什么是 PECS 原则？**
A：Producer-Extends, Consumer-Super。当泛型参数作为数据生产者（你从它读）时用 `? extends`；作为消费者（你往它写）时用 `? super`。它指导通配符方向的选择，`Collections.copy(dest, src)` 中 src 用 extends、dest 用 super 是标准范例。

**Q：为什么 `? extends T` 的集合不能 add 元素？**
A：`? extends T` 表示元素是"T 的某个未知子类型"，编译器无法确定到底是哪个子类，若允许 add 具体元素就可能类型不匹配、破坏类型安全，所以除了 `null` 一律禁止写入。

**Q：为什么 `List<Integer>` 不能赋值给 `List<Number>`？**
A：因为 Java 泛型是不可变的。若允许赋值，就能通过 `List<Number>` 引用往里 add 一个 `Double`，而底层其实是 `List<Integer>`，破坏类型安全。通配符 `List<? extends Number>` 才能接收 `List<Integer>`。

## ⚠️ 易错点 / 加分项

- **易错**：以为 `List<? extends Number>` 能 add Number——不能，上界集合只读。
- **易错**：把 `extends`/`super` 方向记反。口诀：**你要拿（读）就 extends，你要给（写）就 super**。
- **加分**：能解释背后是**协变（extends，安全读）**与**逆变（super，安全写）**，通配符是给不可变泛型补上有限的型变能力。
- **加分**：数组是协变的（`Integer[]` 是 `Object[]` 子类型）但不安全（运行期 `ArrayStoreException`）；泛型不可变 + 通配符是编译期安全的替代方案，可对比作答。
- **加分**：无界 `<?>` 与 `<Object>` 不同——`List<?>` 能接收任意 `List<X>`，而 `List<Object>` 只接收 `List<Object>`。
