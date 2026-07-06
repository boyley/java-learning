# 02 · 注解（Annotation）

> 注解是给代码贴的「元数据标签」，本质是继承 `Annotation` 的接口，配合反射在运行时读取，是框架实现「声明式配置」的核心。面试重要度 ⭐⭐。

## 📖 核心知识

**注解（Annotation）** 是一种元数据（描述数据的数据），给类/方法/字段等打标签，本身不改变程序逻辑，而是**供编译器或框架在编译期/运行时读取并据此做事**。如 `@Override` 让编译器校验，`@Autowired` 让 Spring 注入。

**本质**：注解本质是一个**接口**——`public @interface MyAnno {}` 编译后是 `interface MyAnno extends java.lang.annotation.Annotation`。注解的「属性」实际是接口里的抽象方法，`value()`、`name()` 等；使用时的赋值就是给这些方法指定返回值。

**四个元注解（修饰注解的注解）**：

- **`@Retention`**：保留策略，决定注解活多久。
  - `SOURCE`：只存在源码，编译丢弃（如 `@Override`）。
  - `CLASS`：保留到 class 文件，运行时不可见（默认值）。
  - `RUNTIME`：保留到运行期，**可被反射读取**——框架注解都用它。
- **`@Target`**：注解能用在哪，取值 `ElementType`：`TYPE`（类/接口）、`METHOD`、`FIELD`、`PARAMETER`、`CONSTRUCTOR` 等。
- **`@Documented`**：生成 Javadoc 时包含该注解。
- **`@Inherited`**：允许子类继承父类上的该注解。

**自定义注解 + 反射读取**（框架的核心套路）：

```java
// 1. 定义注解：必须 RUNTIME 才能被反射读到
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Log {
    String value() default "";   // 属性，带默认值
}

// 2. 使用注解
class OrderService {
    @Log("下单")
    public void createOrder() { /* ... */ }
}

// 3. 反射读取并据此处理（模拟框架）
Method m = OrderService.class.getDeclaredMethod("createOrder");
if (m.isAnnotationPresent(Log.class)) {
    Log log = m.getAnnotation(Log.class);
    System.out.println("记录日志: " + log.value());   // 输出：记录日志: 下单
}
```

框架（Spring 的 `@Component`/`@Transactional`、JUnit 的 `@Test`、MyBatis 的 `@Select`）都是这个模式：定义 `RUNTIME` 注解 → 用户标注 → 框架启动时反射扫描类、发现注解、执行对应逻辑（注入、开事务、建代理）。

## 🔑 面试要点

- 注解是元数据标签，不含逻辑，供编译器/框架读取后行动，实现声明式编程。
- 注解本质是接口（`@interface` extends `Annotation`），属性即接口的抽象方法，可带 `default` 默认值。
- 四大元注解：`@Retention`（保留期）、`@Target`（作用目标）、`@Documented`、`@Inherited`。
- `@Retention` 三档：`SOURCE`（编译丢弃）、`CLASS`（默认，运行时不可见）、`RUNTIME`（可被反射读取）。
- 只有 `RUNTIME` 注解能被反射读到——框架注解必须 `RUNTIME`。
- 反射读取三 API：`isAnnotationPresent(X.class)`、`getAnnotation(X.class)`、`getAnnotations()`。
- 注解属性类型限定：基本类型、`String`、`Class`、枚举、注解及其数组；只有一个属性且叫 `value` 时使用可省略属性名。

## ❓ 高频面试题

**Q：注解的本质是什么？它是怎么起作用的？**
A：本质是继承 `java.lang.annotation.Annotation` 的接口，属性对应接口的抽象方法。注解本身不产生任何行为，它靠三种方式生效：① 编译器识别（如 `@Override` 做重写校验）；② APT 注解处理器在编译期生成代码（如 Lombok、MapStruct）；③ 运行时被反射读取后由框架执行逻辑（如 Spring 扫到 `@Component` 就注册 Bean）。

**Q：@Retention 的三个值有什么区别？框架注解为什么必须用 RUNTIME？**
A：`SOURCE` 编译后丢弃、`CLASS` 保留在字节码但运行时 JVM 不加载到内存、`RUNTIME` 保留到运行期且能被反射读取。Spring、MyBatis 等框架是在程序**运行时**通过反射扫描并读取注解来决定行为的，若注解不是 `RUNTIME`，运行时根本读不到，框架就无从感知，所以框架注解必须声明 `@Retention(RUNTIME)`。

**Q：如何自定义一个注解并读取它？**
A：用 `@interface` 定义，加 `@Retention(RUNTIME)` 和 `@Target` 元注解，声明带 `default` 的属性方法；在目标元素上使用；再通过反射拿到 `Class`/`Method`/`Field`，调 `isAnnotationPresent()` 判断、`getAnnotation()` 取出注解对象读取属性值，据此执行逻辑。这正是自研框架、拦截器、参数校验的通用做法。

## ⚠️ 易错点 / 加分项

- `@Retention` 不写默认是 `CLASS`，此时反射读不到——自定义框架注解忘写 `RUNTIME` 是高频坑。
- 注解属性类型有限制，不能用任意对象；数组默认值写 `{}`。
- 只有一个属性且名为 `value` 才能省略属性名（`@Log("x")`），否则必须写 `@Log(value="x")`。
- `@Inherited` 只对**类**上的注解生效，且不作用于接口和方法。
- 加分项：能区分注解生效的三条路径（编译校验 / APT 编译期代码生成 / 运行时反射），并举例 Lombok 是编译期 APT，Spring 是运行时反射。
- 加分项：JDK 8 新增 `@Repeatable`（可重复注解）和 `TYPE_USE`/`TYPE_PARAMETER`（类型注解，如 `@NonNull String`）。
