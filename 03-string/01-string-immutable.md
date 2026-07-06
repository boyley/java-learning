# 01 · String 为什么不可变（String Immutability）

> String 对象一旦创建，内容就不能改变；不可变换来了安全、线程安全、hashCode 缓存与常量池复用。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**「不可变」指的是**：一个 String 对象所代表的字符序列，从创建到被 GC 回收，永远不变。任何看似"修改"的操作（`substring`、`replace`、`+` 拼接）都会**返回一个新的 String 对象**，原对象纹丝不动。

String 靠三层设计保证不可变：

1. **类被 `final` 修饰**：`public final class String`，不能被继承，也就无法通过子类重写方法去破坏不可变性。
2. **底层字符数组是 `private final` 的**：
   - JDK 8 是 `private final char value[]`；
   - JDK 9+ 改为 `private final byte[] value`（Compact Strings，Latin-1 单字节存储省内存）。
   - `final` 保证引用不能改指向别的数组，`private` 保证外部拿不到这个数组。
3. **不提供任何修改内部数组的方法**：String 没有 setter，所有"修改类"方法都是生成新对象。

```java
String s = "abc";
s.concat("def");          // 返回新对象，但没接收
System.out.println(s);    // 仍是 abc

s = s.concat("def");      // 让 s 指向新对象
System.out.println(s);    // abcdef —— 变的是引用，不是原对象
```

注意：`final` 只保证 `value` 这个**引用**不可变，并不能保证数组元素不可变。真正让外界改不到元素的是 `private` + 不暴露数组。理论上通过反射能强改，但那是破坏封装，不算正常语义。

## 🔑 面试要点

- 不可变 = `final` 类 + `private final` 数组 + 无修改方法，三者缺一不可。
- `final` 修饰的是数组引用，不是数组内容；`private` 才是把数组藏起来的关键。
- 一切"修改"操作都返回新 String，原对象不变。
- **不可变的四大好处**（必背）：
  1. **安全**：String 常用作类加载路径、网络 URL、数据库连接串、Map 的 key、方法参数；不可变才能保证这些值不会被中途篡改。
  2. **线程安全**：不可变对象天然线程安全，多线程共享无需加锁。
  3. **hashCode 缓存**：String 内部用 `private int hash` 缓存哈希值，因为内容不变，算一次就能一直用，作为 HashMap key 时性能好。
  4. **支持常量池复用**：内容不变，相同字面量才能安全地共享同一个对象，节省内存。
- JDK 9 起底层由 `char[]` 变 `byte[]`（Compact Strings），不影响不可变语义。

## ❓ 高频面试题

**Q：String 是如何保证不可变的？**
A：三点。①类是 `final` 的，防止子类破坏；②内部字符数组是 `private final`，`final` 让引用不可改、`private` 让外部访问不到；③不提供任何修改内部数组的方法，所有修改型 API 都返回新对象。

**Q：`private final char[] value` 中的 `final` 能保证数组元素不被修改吗？**
A：不能。`final` 只保证 `value` 引用不能再指向别的数组，数组里的元素本身是可变的。真正让外部改不到元素的是 `private` 修饰符（不暴露引用）以及 String 不提供修改方法。

**Q：String 不可变有什么好处？**
A：安全（可安全用作 key、URL、参数等）、天然线程安全、可缓存 hashCode（提升 HashMap 性能）、可支持常量池复用节省内存。

**Q：既然不可变，为什么 `String s = "a"; s = s + "b";` 好像改了？**
A：`s + "b"` 创建了一个新 String 对象，然后把变量 `s` 指向它。变的是**引用变量的指向**，原来的 `"a"` 对象依然没变。

## ⚠️ 易错点 / 加分项

- **误区**："不可变就是加了 `final`。"——不完整，`final` 只是三要素之一，关键还有 `private` 数组和无修改方法。
- **误区**："`final char[]` 保证元素不可改。"——错，`final` 只锁引用不锁元素。
- **加分**：能点出 hashCode 缓存字段 `hash`，以及为什么它对 HashMap 场景重要（key 的哈希只算一次）。
- **加分**：能提到 JDK 9 Compact Strings 把 `char[]`（每字符 2 字节）换成 `byte[]` + 编码标志位，纯 Latin-1 字符串内存直接减半。
- **加分**：反射可以强行修改 `value` 数组破坏不可变，但这是破坏封装的非正常手段，说明"不可变"是语言层面的约定保护。
