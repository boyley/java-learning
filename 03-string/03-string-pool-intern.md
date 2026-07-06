# 03 · 字符串常量池与 intern()（String Pool & intern）

> 字符串字面量共享一份对象存于常量池；`new String("a")` 创建 1~2 个对象，`intern()` 手动把字符串放入/复用常量池。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**字符串常量池（String Pool）**是 JVM 维护的一块特殊存储，保证**内容相同的字符串字面量只存一份**，节省内存。

```java
String a = "hello";
String b = "hello";
System.out.println(a == b);   // true —— 指向池中同一对象
```

字面量 `"hello"` 在类加载/首次使用时进入常量池；第二个 `"hello"` 直接复用，所以 `a == b` 为 true（`==` 比引用）。

**`new String("hello")` 创建几个对象？** 这是经典题，答案是 **1 个或 2 个**：

- 字面量 `"hello"` 会在常量池中放一份（如果池里还没有，这是**第 1 个**，在常量池 / 堆）；
- `new` 关键字必然在**堆**上再建一个新对象（这是**第 2 个**）。

所以：如果常量池中原本没有 `"hello"`，共创建 **2 个**对象；如果之前已存在，则只在堆上新建 **1 个**。`new` 出来的引用指向堆对象，与池中对象不是同一个：

```java
String s1 = "hello";
String s2 = new String("hello");
System.out.println(s1 == s2);          // false —— 一个池对象、一个堆对象
System.out.println(s1 == s2.intern()); // true  —— intern 返回池中对象
System.out.println(s1.equals(s2));     // true  —— 内容相同
```

**`intern()` 的作用**：
- 调用 `s.intern()` 时，若常量池中已有内容相等的字符串，直接返回**池中那个对象的引用**；
- 若没有，则把该字符串加入常量池（JDK 7+ 是把**堆中对象的引用**登记进池）并返回。

**JDK 版本差异（关键）**：
- **JDK 6 及以前**：常量池在**永久代（PermGen，方法区）**，`intern()` 会把字符串**复制**一份到永久代。永久代空间小，大量 intern 容易 `OutOfMemoryError: PermGen`。
- **JDK 7 起**：常量池被**移到堆（Heap）**中。`intern()` 不再复制对象，而是**把堆中已有对象的引用登记到池里**。这带来一道著名区别题：

```java
// JDK 7+ 环境
String s = new String("a") + new String("b");  // 堆里得到 "ab"，池中此时没有 "ab"
System.out.println(s.intern() == s);            // true
// intern 时池中无 "ab"，于是直接把堆中 s 的引用存入池，返回的就是 s 本身

String x = new String("hello");                 // 池中已有字面量 "hello"
System.out.println(x.intern() == x);            // false
// 池里早有 "hello" 对象，intern 返回池对象，x 是堆对象，不相等
```

> 常量池的底层内存布局、从永久代到堆的迁移、以及元空间（Metaspace）的关系属于 JVM 细节，见 [`../../jvm-learning/14-string-constant-pool.md`](../../jvm-learning/14-string-constant-pool.md)，此处只讲 Java 层面结论。

## 🔑 面试要点

- 常量池保证**相同字面量共享同一对象**，`==` 比较字面量返回 true。
- `new String("x")` 创建 **1 或 2 个**对象：堆上必有 1 个；若池中原先没有 `"x"` 则还要在池里建 1 个（共 2 个）。
- `new` 出来的引用指向**堆对象**，与常量池对象 `==` 为 false。
- `intern()`：池中有则返回池中引用，没有则登记进池再返回。
- **JDK 7 常量池从永久代移到堆**：intern 从"复制对象到永久代"变成"登记堆对象引用到池"。
- 编译期常量拼接（`"a"+"b"`）在编译期合并进池；含变量或 `new` 的拼接结果在堆上，不进池。
- 判断相等：**内容比较永远用 `equals`，`==` 只比引用**。

## ❓ 高频面试题

**Q：`String s = new String("abc")` 创建了几个对象？**
A：1 个或 2 个。堆上一定 new 一个对象；字面量 `"abc"` 若常量池里还没有，则再创建一个放入池，共 2 个；若池中已存在则只新建堆上 1 个。

**Q：`intern()` 方法是干什么的？**
A：把字符串放入常量池或复用池中已有对象。若池中已有内容相等的字符串，返回池中引用；否则（JDK 7+）把当前堆对象的引用登记进池并返回。常用于大量重复字符串场景手动去重、省内存。

**Q：`new String("a") + new String("b")`，再 `.intern() == 原引用` 是 true 还是 false（JDK 7+）？**
A：true。拼接结果 `"ab"` 在堆上生成，此时常量池里没有 `"ab"`，`intern()` 直接把这个堆对象的引用存入池并返回，所以 `intern()` 返回的就是原引用。

**Q：JDK 7 对字符串常量池做了什么改动？**
A：把常量池从永久代（方法区）移到了堆中。好处是常量池能享受堆的 GC，不再受永久代大小限制，大量 `intern()` 不易再触发 PermGen 溢出；同时 `intern()` 语义从"复制到永久代"变为"登记堆对象引用"。

## ⚠️ 易错点 / 加分项

- **易错**：`String s = new String("a")` 后 `s == "a"` 是 **false**（堆对象 vs 池对象），但 `s.equals("a")` 是 true。
- **易错**：只有编译期能确定的常量表达式才进池；`"a" + someVar`、`new String(...)` 的结果在堆上，不自动进池。
- **加分**：能准确说出 JDK 6（永久代 + intern 复制）与 JDK 7+（堆 + intern 存引用）导致上面两道 `intern()` 题结果不同。
- **加分**：`intern()` 用于海量重复字符串（如日志、解析大文件）去重省内存，但滥用会增加常量池维护成本，需权衡。
- **加分**：常量池底层演进（永久代 → 堆，方法区其余部分 JDK 8 进元空间）可引用 [`../../jvm-learning/14-string-constant-pool.md`](../../jvm-learning/14-string-constant-pool.md)，不必在面试里展开 JVM 内存细节。
