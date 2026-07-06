# 01 · 反射机制（Reflection）

> 反射让程序在**运行时**动态获取类的结构信息并操作对象——获取 `Class`、读写 `Field`、调用 `Method`、创建实例。是框架的基石。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**反射（Reflection）**：在运行时，对任意一个类，都能知道它的所有属性和方法；对任意一个对象，都能调用它的任意方法和属性。核心是打破「编译期就要确定类型」的限制，改成**运行期动态解析**。这正是 Spring 靠字符串类名创建 Bean、MyBatis 把结果集映射到对象的底层能力。

一切从 `Class` 对象开始。类加载后，JVM 为每个类生成唯一的 `Class` 对象（存于方法区/元空间），它是反射的入口。

**获取 Class 对象的三种方式**：

```java
// 1. 类名.class —— 编译期已知类，最安全，不触发初始化
Class<?> c1 = User.class;

// 2. 对象.getClass() —— 已有实例时
Class<?> c2 = user.getClass();

// 3. Class.forName("全限定名") —— 运行时字符串，最灵活，会触发类初始化（静态块执行）
Class<?> c3 = Class.forName("com.demo.User");
```

三者返回同一个 `Class` 对象（`c1 == c3` 为 `true`）。区别：`forName` 传字符串、最灵活（配置驱动），且**默认会初始化类**（执行静态代码块，这也是 JDBC `Class.forName("com.mysql.jdbc.Driver")` 注册驱动的原理）；`.class` 不初始化。

**三大反射操作对象**：

- **`Field`**：字段。`getDeclaredField(name)` 取（含私有），`field.setAccessible(true)` 后可读写私有字段，`get(obj)`/`set(obj, val)`。
- **`Method`**：方法。`getDeclaredMethod(name, 参数类型...)`，`method.invoke(obj, args...)` 调用。
- **`Constructor`**：构造器。`getDeclaredConstructor(...)`，`newInstance(args...)` 创建实例。

```java
Class<?> clazz = Class.forName("com.demo.User");
Object user = clazz.getDeclaredConstructor().newInstance(); // 调无参构造建实例

Field f = clazz.getDeclaredField("name");
f.setAccessible(true);          // 突破 private 访问检查
f.set(user, "Tom");

Method m = clazz.getDeclaredMethod("greet", String.class);
m.setAccessible(true);
Object result = m.invoke(user, "hi");   // 等价 user.greet("hi")
```

注意 `getXxx` 只拿 public（含继承），`getDeclaredXxx` 拿本类所有（含 private，但不含继承）。

**代价**：反射灵活但有成本。① **性能**：方法调用比直接调用慢（需做安全检查、无法内联/JIT 优化，虽然 JDK 已大幅优化）；② **安全**：`setAccessible(true)` 破坏封装、绕过访问控制；③ **编译期失去检查**：类型错误、方法名拼错要到运行时才暴露。

## 🔑 面试要点

- 反射 = 运行时动态获取类信息、操作对象，是 Spring/MyBatis/JUnit 等框架的核心机制。
- 获取 Class 三种方式：`类.class`、`对象.getClass()`、`Class.forName("全限定名")`，返回同一 Class 对象。
- `Class.forName` 默认初始化类（执行静态块），`.class` 不初始化——JDBC 注册驱动就靠前者。
- 三大操作对象：`Field`、`Method`、`Constructor`；`invoke`/`get`/`set`/`newInstance` 是关键 API。
- `getXxx` 只取 public（含继承）；`getDeclaredXxx` 取本类全部含 private。
- `setAccessible(true)` 可访问私有成员，突破封装。
- 代价：性能开销、破坏封装安全性、丧失编译期检查。

## ❓ 高频面试题

**Q：获取 Class 对象有哪几种方式？有什么区别？**
A：三种——`类名.class`（编译期已知类，不触发初始化，最安全）、`实例.getClass()`（已有对象时用）、`Class.forName("全限定类名")`（传字符串最灵活，适合配置驱动，且默认会初始化类、执行静态代码块）。三者拿到的是 JVM 中同一个 Class 对象。JDBC 用 `Class.forName` 正是利用它初始化时执行驱动类静态块完成注册。

**Q：反射为什么能访问 private 成员？会有什么问题？**
A：通过 `field.setAccessible(true)` / `method.setAccessible(true)` 关闭 JVM 的访问权限检查即可读写私有成员。问题是它破坏了封装性和安全性——本该隐藏的实现细节被外部随意改动，可能绕过不变式校验，且在有 `SecurityManager` 或 JDK 9+ 模块系统（`--illegal-access`）下可能被拒绝。

**Q：反射有性能问题吗？为什么？**
A：有，但没想象中夸张。反射调用比直接调用慢，因为每次要做访问检查、参数装箱、无法像普通调用那样被 JIT 内联优化。频繁调用的热点路径应缓存 `Method`/`Field` 对象（避免重复查找）、`setAccessible(true)` 跳过检查，或用 `MethodHandle`/字节码生成（如 CGLIB、`LambdaMetafactory`）替代。

## ⚠️ 易错点 / 加分项

- `getDeclaredMethod`/`getMethod` 传参必须传**参数类型的 Class**（如 `String.class`），不是参数值。
- `getMethod` 拿不到 private 方法，需用 `getDeclaredMethod` 且 `setAccessible(true)`。
- `Class.newInstance()` 已废弃（只能调无参构造且吞异常），改用 `getDeclaredConstructor().newInstance()`。
- 反射调用要缓存 `Method`/`Field`，别每次 `getDeclaredMethod` 重新查找——查找才是主要开销。
- 加分项：能区分 `getXxx`（public + 继承）与 `getDeclaredXxx`（本类全部、不含继承），并说清 Class 对象存于方法区/元空间，底层见 [`../../jvm-learning/13-method-area-metaspace.md`](../../jvm-learning/13-method-area-metaspace.md)。
- 加分项：JDK 7+ 提供 `MethodHandle`（`java.lang.invoke`）作为反射的高性能替代，直接对接字节码指令。
