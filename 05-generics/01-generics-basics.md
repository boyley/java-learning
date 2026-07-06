# 01 · 泛型基础（Generics Basics）

> 泛型 = 类型参数化，把类型作为参数传递，实现编译期类型安全与代码复用。面试重要度 ⭐⭐。

## 📖 核心知识

**泛型的作用**：在编译期就把类型检查提前，避免运行期 `ClassCastException`，同时省去强制类型转换、提升代码复用性与可读性。

没有泛型时集合只能存 `Object`，取出要强转，还可能在运行期才炸：

```java
List list = new ArrayList();
list.add("hello");
list.add(123);                       // 编译期不报错
String s = (String) list.get(1);     // 运行期 ClassCastException！
```

有了泛型，错误在编译期就暴露，取值也无需强转：

```java
List<String> list = new ArrayList<>();
list.add("hello");
// list.add(123);                    // 编译期直接报错
String s = list.get(0);              // 无需强转
```

**三种泛型形式**：

1. **泛型类 / 接口**：类名后声明类型参数 `<T>`，成员可用它。

```java
public class Box<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}
Box<String> box = new Box<>();   // T 被指定为 String
```

2. **泛型接口**：

```java
public interface Container<T> {
    T get();
    void put(T item);
}
```

3. **泛型方法**：**方法返回值前**单独声明 `<T>`，与类是否泛型无关，类型由调用时的实参推断。

```java
public static <T> T firstOf(List<T> list) {
    return list.get(0);
}
String s = firstOf(List.of("a", "b"));   // T 推断为 String
```

**类型参数命名惯例**：`T`(Type)、`E`(Element)、`K`/`V`(Key/Value)、`R`(Result)、`N`(Number)。

**上下界（Bounded Type Parameter）**：用 `extends` 约束类型参数的范围。

```java
// T 必须是 Number 或其子类，于是能安全调用 Number 的方法
public static <T extends Number> double sum(List<T> list) {
    double total = 0;
    for (T n : list) total += n.doubleValue();   // 可调用 Number 的方法
    return total;
}
```

- `<T extends 上界>`：T 是"上界或其子类"，这里的 `extends` 对接口也用它（`<T extends Comparable<T>>`）。
- 类型参数声明处**只有 `extends`（上界），没有 `super`**；`super` 只用于通配符 `?`（见 03）。
- 可多重边界：`<T extends A & B>`（类写前面，接口写后面）。

## 🔑 面试要点

- 泛型是**编译期**特性：作用是把类型检查提前、消除强转、提升复用。
- 三种用法：泛型类、泛型接口、泛型方法；泛型方法在返回值前声明 `<T>`，独立于类。
- 泛型方法的类型参数由**调用时的实参自动推断**，一般不用显式指定。
- 上界 `<T extends X>`：T 限定为 X 或其子类，可安全调用 X 的方法；支持多重边界 `<T extends A & B>`。
- 类型参数声明处只能用 `extends`（上界），`super`（下界）只出现在通配符 `?` 里。
- 常见命名：T、E、K/V、R、N。
- 泛型不支持基本类型（`List<int>` 非法），要用包装类 `List<Integer>`（因擦除后是 Object，见 02）。

## ❓ 高频面试题

**Q：Java 泛型有什么用？为什么要引入泛型？**
A：核心是编译期类型安全。没有泛型时集合存 `Object`、取值要强转，类型错误要到运行期才抛 `ClassCastException`；泛型把类型检查提前到编译期，同时省去强转、增强复用与可读性。

**Q：泛型类和泛型方法有什么区别？**
A：泛型类在类名后声明类型参数，作用于整个类的成员，实例化时确定类型；泛型方法在方法返回值前单独声明 `<T>`，作用域仅限该方法，类型由调用实参推断，普通类里也能有泛型方法。

**Q：`<T extends Number>` 是什么意思？**
A：这是上界通配，限定类型参数 T 必须是 `Number` 或其子类，从而能在方法内安全调用 `Number` 定义的方法（如 `doubleValue()`）。

## ⚠️ 易错点 / 加分项

- **易错**：泛型不能用基本类型，`List<int>` 编译不过，必须 `List<Integer>`。
- **易错**：类型参数声明处写 `<T super X>` 是非法的，`super` 只能用在通配符 `?` 上。
- **加分**：`<>`（钻石运算符，JDK 7+）让右边可省略类型：`new ArrayList<>()`，编译器自动推断。
- **加分**：泛型带来编译期安全，但运行期会被"类型擦除"，由此产生一系列限制（不能 `new T[]`、不能 `instanceof T` 等），见 [类型擦除](02-type-erasure.md)。
- **加分**：静态方法不能用类的类型参数（类的 `<T>` 依赖实例），静态方法要泛型必须自己声明 `<T>`。
