# 02 · 自动装箱与缓存池（Autoboxing & Cache）

> 装箱=基本类型→包装类，拆箱=反向；`Integer` 缓存 -128~127，导致 `Integer a=127` 用 `==` 的经典陷阱。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

**自动装箱（autoboxing）**：编译器自动把基本类型转成对应包装类，底层调用 `Integer.valueOf(int)`。
**自动拆箱（unboxing）**：反向把包装类转回基本类型，底层调用 `Integer.intValue()`。

```java
Integer a = 100;   // 编译期变为 Integer.valueOf(100)
int b = a;         // 编译期变为 a.intValue()
```

**Integer 缓存池**：`Integer.valueOf` 对 **-128 ~ 127** 之间的值返回缓存的同一个对象（`IntegerCache`），超出这个范围才 `new Integer()`。源码逻辑：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

所以出现经典现象：

```java
Integer a = 127, b = 127;
System.out.println(a == b);   // true  —— 命中缓存，同一对象
Integer c = 128, d = 128;
System.out.println(c == d);   // false —— 超出缓存，各 new 一个
Integer e = 127;
int f = 127;
System.out.println(e == f);   // true  —— 有 int 参与，e 拆箱后比值
```

`==` 比较引用类型时比的是**地址**；只要一端是基本类型，另一端就会**拆箱**变成值比较。

**各包装类缓存范围**：
- `Integer`：-128~127（上界可通过 `-XX:AutoBoxCacheMax` 调大）。
- `Byte`/`Short`/`Long`：固定 -128~127。
- `Character`：0~127。
- `Boolean`：TRUE/FALSE 全缓存。
- `Float`/`Double`：**无缓存**（浮点值几乎不可能落在固定小范围）。

## 🔑 面试要点

- 装箱调 `valueOf`，拆箱调 `xxxValue`，是编译器的语法糖。
- `Integer` 缓存 -128~127，源自 `IntegerCache`，`valueOf` 命中缓存返回同一对象。
- `Integer a=127,b=127` → `a==b` 为 true；改成 128 就是 false。
- `Integer` 与 `int` 用 `==` 比较时，`Integer` 会拆箱，比的是值。
- 两个包装类比较值必须用 `equals`，不能用 `==`。
- `Long`/`Short`/`Byte` 缓存范围同样是 -128~127；`Character` 是 0~127；`Double`/`Float` 无缓存。
- 混合运算（如 `Integer + int`）会触发拆箱，若包装类为 null 会抛 `NullPointerException`。

## ❓ 高频面试题

**Q：`Integer a = 127; Integer b = 127; a == b` 结果？换成 128 呢？**
A：127 时为 `true`（命中 -128~127 缓存，同一对象）；128 时为 `false`（超出缓存范围，各 `new` 一个对象，地址不同）。

**Q：`Integer i = null; int j = i;` 会怎样？**
A：抛 `NullPointerException`。因为拆箱调用 `i.intValue()`，`i` 为 null 时空指针。这是集合取值后直接赋给基本类型的常见坑。

**Q：`Integer` 和 `int` 用 `==` 比较，比的是什么？**
A：比值。只要有一端是基本类型 `int`，包装类就会自动拆箱，退化为值比较，因此不受缓存影响，结果只看数值是否相等。

**Q：为什么 Java 要设计 Integer 缓存？**
A：小整数使用极其频繁，缓存复用可减少对象创建、降低 GC 压力、提升性能。这是享元模式（Flyweight）的应用。

## ⚠️ 易错点 / 加分项

- 最大坑：包装类比值用 `==`，128 以上出错。**结论：包装类比值一律用 `equals`**。
- `Double d1 = 1.0; Double d2 = 1.0; d1 == d2` 是 `false`——浮点无缓存，别混淆。
- 循环里 `Long sum = 0L; sum += i;` 会反复装箱拆箱，性能差，累加应用基本类型。
- 加分：能说出 `-XX:AutoBoxCacheMax=N` 可调整 `Integer` 缓存上界（下界固定 -128）。
- 加分：指出缓存是享元模式，`valueOf` 优于已废弃的 `new Integer()`（后者不走缓存，且 JDK 9 起标记 deprecated）。
