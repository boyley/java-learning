# 02 · 类型擦除（Type Erasure）

> Java 泛型只存在于编译期，编译后类型参数被"擦除"为原始类型（Object 或上界），运行期没有泛型信息。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**类型擦除**是 Java 泛型的实现方式：泛型信息只用于**编译期类型检查**，编译通过后，编译器会把所有类型参数**擦除**掉，替换为其**上界**（无上界则为 `Object`），并在需要处自动插入强制类型转换。这样字节码里就没有泛型了，是为了兼容没有泛型的旧版本 JDK。

```java
// 源码
List<String> list = new ArrayList<>();
list.add("a");
String s = list.get(0);

// 擦除后（运行期等价于）
List list = new ArrayList();
list.add("a");
String s = (String) list.get(0);   // 编译器自动插入强转
```

擦除规则：
- `<T>` 无界 → 擦除为 `Object`；
- `<T extends Number>` → 擦除为上界 `Number`；
- 编译器在取值等处自动插入到目标类型的强转。

**一个有力证据**——不同泛型参数的同一个类，运行期是同一个 `Class`：

```java
List<String> ls = new ArrayList<>();
List<Integer> li = new ArrayList<>();
System.out.println(ls.getClass() == li.getClass());  // true！都是 ArrayList.class
```

**桥接方法（Bridge Method）**：擦除会破坏多态，编译器用桥接方法弥补。当泛型类被具体化继承并重写方法时：

```java
class Node<T> {
    public void set(T t) { }        // 擦除后 set(Object)
}
class StringNode extends Node<String> {
    @Override
    public void set(String s) { }   // 想重写 set(Object)，但签名是 set(String)
}
```

擦除后父类是 `set(Object)`，子类是 `set(String)`——签名不同，本不构成重写。编译器于是在 `StringNode` 里自动**生成一个桥接方法** `set(Object o){ set((String)o); }`，它重写父类的 `set(Object)` 并转发到 `set(String)`，从而保证多态正确。桥接方法由编译器合成、对开发者不可见。

**擦除带来的限制**（高频）：
- **不能 `new T[]`**：运行期不知道 T 的真实类型，无法创建数组 → `new T[10]` 编译错误。变通：`(T[]) new Object[10]` 或传入 `Class<T>` 反射创建。
- **不能 `new T()`**：同理无法实例化类型参数。
- **不能对类型参数用 `instanceof`**：`obj instanceof T` 非法，因为运行期 T 已擦除。
- **静态成员不能用类的类型参数**：`static T field;` 非法，静态属于类而非实例。
- **泛型异常受限**：不能 `catch (T e)`，泛型类不能继承 `Throwable`。
- **不能有"仅泛型参数不同"的重载**：`void f(List<String>)` 和 `void f(List<Integer>)` 擦除后签名相同，编译冲突。
- **基本类型不能作类型参数**：擦除为 `Object`，`int` 无法当 `Object`，只能用包装类。

## 🔑 面试要点

- 类型擦除 = 泛型只在编译期检查，编译后擦除为**上界 / Object**，运行期无泛型信息，目的是兼容旧版本。
- 编译器在取值等处**自动插入强制类型转换**。
- `List<String>` 与 `List<Integer>` 运行期是**同一个 Class**（`getClass()` 相等）。
- **桥接方法**：编译器为保证擦除后仍满足重写多态而自动合成的方法。
- 擦除导致的限制要能列举：**不能 `new T[]` / `new T()` / `T.class` / `instanceof T`**、静态字段不能用类型参数、不能按泛型参数重载、不能用基本类型。
- 想在运行期保留类型信息，需自己传入 `Class<T>`（Type Token 模式）。

## ❓ 高频面试题

**Q：什么是类型擦除？Java 为什么这样实现泛型？**
A：泛型信息只用于编译期类型检查，编译后类型参数被替换为其上界（无界则 Object），并自动插入强转，字节码中不含泛型。这样做是为了**向后兼容**没有泛型的旧代码和旧 JVM，属于"伪泛型"。

**Q：`List<String>` 和 `List<Integer>` 在运行期是同一个类吗？**
A：是。擦除后都是 `List`，`new ArrayList<String>().getClass() == new ArrayList<Integer>().getClass()` 返回 true，运行期没有 String/Integer 的区分。

**Q：为什么不能 `new T[]`？**
A：数组在运行期需要知道确切的元素类型来做存储和 `ArrayStoreException` 检查，但 T 已被擦除，运行期拿不到真实类型，所以 `new T[]` 被禁止。常见变通是 `(T[]) new Object[n]` 或传 `Class<T>` 用 `Array.newInstance` 反射创建。

**Q：桥接方法是什么？**
A：泛型类被具体类型继承并重写方法时，擦除会使父子方法签名不一致（如父 `set(Object)`、子 `set(String)`），编译器会自动在子类合成一个 `set(Object)` 桥接方法转发到 `set(String)`，以保证擦除后重写多态仍然成立。

## ⚠️ 易错点 / 加分项

- **易错**：以为运行期能拿到泛型类型——不能，`obj instanceof List<String>` 非法，只能 `instanceof List`。
- **易错**：`void f(List<String>)` 与 `void f(List<Integer>)` 不能构成重载，擦除后签名相同会编译报错。
- **加分**：能说清桥接方法产生的场景和作用，是区分候选人深度的点。
- **加分**：虽然变量/参数上的泛型被擦除，但**字段、方法参数、返回值、父类**上的泛型签名会以 Signature 属性保留在字节码里，可通过反射（`getGenericSuperclass` 等）读取——这正是 Gson/Jackson 的 `TypeToken` 能拿到泛型的原理。
- **加分**：对比 C# 的"真泛型"（运行期保留类型、可 `new T()`），说明 Java 是"伪泛型"设计取舍。
