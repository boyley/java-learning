# 04 · equals 与 hashCode（equals & hashCode）

> `==` 比地址/值，`equals` 比逻辑相等；重写 `equals` 必须同时重写 `hashCode`，否则破坏"相等对象哈希码必相等"契约，导致 HashMap 出错。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

**`==` vs `equals`**：
- `==`：基本类型比**值**；引用类型比**地址**（是否同一对象）。
- `equals`：`Object` 默认实现就是 `==`（比地址）；`String`、包装类等重写后比**内容**。

```java
String a = new String("hi"), b = new String("hi");
System.out.println(a == b);       // false，两个不同对象
System.out.println(a.equals(b));  // true，内容相同
```

**为什么重写 `equals` 必须重写 `hashCode`？**

`hashCode` 返回对象哈希码，`HashMap`/`HashSet` 先用它定位桶（bucket），再用 `equals` 在桶内比较。Java 规定的**契约**：

1. `equals` 相等的两个对象，`hashCode` **必须相等**。
2. `hashCode` 相等的对象，`equals` 不一定相等（哈希冲突允许）。
3. 同一对象多次调用 `hashCode` 应返回一致的值。

如果只重写 `equals` 不重写 `hashCode`，两个"逻辑相等"的对象可能有不同哈希码，被分到不同桶，`HashMap` 会认为它们是不同 key：

```java
class Point {
    int x, y;
    // 只重写 equals，没重写 hashCode ——错误示范
    public boolean equals(Object o) { /* 比较 x,y */ }
}
Map<Point, String> map = new HashMap<>();
map.put(new Point(1,1), "A");
map.get(new Point(1,1)); // 返回 null！哈希码不同，定位到不同桶
```

**正确重写（IDE 自动生成即可）**：

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Point p = (Point) o;
    return x == p.x && y == p.y;
}
@Override
public int hashCode() {
    return Objects.hash(x, y);  // JDK7+ 工具方法
}
```

`equals` 五大特性：自反性、对称性、传递性、一致性、非空性（`x.equals(null)` 恒 false）。

## 🔑 面试要点

- `==` 引用类型比地址，`equals` 默认也比地址，`String`/包装类重写后比内容。
- `hashCode` 用于哈希表快速定位桶，`equals` 用于桶内精确比较。
- 契约核心：**equals 相等 ⇒ hashCode 必相等**（反之不成立）。
- 重写 `equals` 不重写 `hashCode` → `HashMap`/`HashSet` 存取异常（查不到、重复存入）。
- 重写 `equals` 要满足自反、对称、传递、一致、非空五性。
- 用 `Objects.equals(a,b)` 可避免空指针；`Objects.hash(...)` 简化 hashCode 编写。
- 作为 HashMap 的 key 或放入 HashSet 的对象，务必正确重写这两个方法，且最好用不可变字段计算。

## ❓ 高频面试题

**Q：`==` 和 `equals` 的区别？**
A：`==` 对基本类型比值、对引用类型比地址；`equals` 是 `Object` 的方法，默认比地址，`String`、`Integer` 等重写后比内容。想比逻辑相等用 `equals`。

**Q：为什么重写 `equals` 一定要重写 `hashCode`？**
A：因为哈希集合先按 `hashCode` 定位桶再用 `equals` 比较。若只重写 `equals`，两个逻辑相等对象哈希码可能不同，落到不同桶，导致 `HashMap` 认为它们不相等，出现查不到、重复插入等 bug。这违反"equals 相等则 hashCode 必相等"的契约。

**Q：两个对象 hashCode 相同，它们一定 equals 吗？**
A：不一定。哈希冲突时不同对象可能有相同 hashCode，还需 `equals` 进一步判断。反过来，equals 相等则 hashCode 必须相等。

**Q：`hashCode` 可以恒返回同一个常数吗？**
A：语法上可以且不违反契约，但所有对象挤进同一个桶，哈希表退化成链表/红黑树，查询从 O(1) 变 O(n)/O(log n)，性能极差。应让哈希码尽量均匀分布。

## ⚠️ 易错点 / 加分项

- 最大坑：自定义对象当 `HashMap` key 却忘重写 `hashCode`，`get` 返回 null。
- 对称性易错：`a.equals(b)` 与 `b.equals(a)` 结果必须一致，`getClass()` 比 `instanceof` 更严格（后者在继承下可能破坏对称性）。
- 用于计算 hashCode/equals 的字段最好**不可变**，否则对象入表后字段变了会永远找不到。
- 加分：`String` 的 hashCode 有缓存（首次计算后存 `hash` 字段），空串为 0。
- 加分：能提 `Objects.equals`/`Objects.hash` 简化写法，以及 `equals` 用 `getClass()` vs `instanceof` 的对称性权衡。
