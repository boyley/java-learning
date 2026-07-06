# 02 · 工厂模式（Factory）

> 把「对象的创建」从「对象的使用」中剥离，让调用方只依赖抽象、不依赖具体类，实现解耦。工厂模式有三种粒度：简单工厂、工厂方法、抽象工厂。面试重要度 ⭐⭐。

## 📖 核心知识

三者本质都是「用一个专门的角色负责 `new`」，区别在于**扩展粒度**。

### 1. 简单工厂（Simple Factory，严格说不算 GoF 模式）

一个工厂类，用一个方法（常带 `if/switch`）根据参数返回不同产品。**缺点：加新产品要改工厂代码，违反开闭原则。**

```java
interface Product { void use(); }
class A implements Product { public void use() { System.out.println("A"); } }
class B implements Product { public void use() { System.out.println("B"); } }

class SimpleFactory {
    static Product create(String type) {
        switch (type) {
            case "A": return new A();
            case "B": return new B();
            default: throw new IllegalArgumentException();
        }
    }
}
```

### 2. 工厂方法（Factory Method，GoF）

定义一个创建产品的**抽象方法**，每种产品对应一个工厂子类。**加新产品只需加新工厂子类，不改老代码，符合开闭原则。**

```java
interface Factory { Product create(); }
class AFactory implements Factory { public Product create() { return new A(); } }
class BFactory implements Factory { public Product create() { return new B(); } }

// 新增 C 产品：加 CFactory 即可，不动老工厂
```

### 3. 抽象工厂（Abstract Factory，GoF）

一个工厂负责创建**一整族相关产品**（产品族）。适合有多个维度的产品，比如「一套 UI 组件（按钮+文本框）」的 Windows 风格和 Mac 风格。

```java
interface GUIFactory {
    Button createButton();
    TextField createTextField();
}
class WinFactory implements GUIFactory {
    public Button createButton() { return new WinButton(); }
    public TextField createTextField() { return new WinTextField(); }
}
class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public TextField createTextField() { return new MacTextField(); }
}
```

### 真实应用

- **`Calendar.getInstance()`**：内部根据 `Locale`/时区返回 `GregorianCalendar` 等具体子类，调用方拿到的是抽象 `Calendar`——典型简单工厂/工厂方法思想。
- **`DateFormat.getDateInstance()`、`NumberFormat.getInstance()`**：同理。
- **Spring 的 `BeanFactory`**：`getBean()` 就是一个大工厂，`FactoryBean` 接口更是让用户自定义 Bean 创建逻辑。
- **JDBC 的 `DriverManager.getConnection()`**：按 URL 返回不同数据库驱动的连接。
- **日志框架 `LoggerFactory.getLogger()`**。

## 🔑 面试要点

- 简单工厂：**一个**工厂类 + 参数分支，加产品要改工厂（违反开闭）。
- 工厂方法：**每个产品一个工厂子类**，加产品加子类（符合开闭）；把 new 的决定权下放给子类。
- 抽象工厂：**一个工厂造一族产品**，关注「产品族」的一致性；加新产品族容易，加新产品种类难（要改所有工厂接口）。
- 核心价值：**依赖倒置**——调用方依赖抽象产品和抽象工厂，不 new 具体类。
- 记忆口诀：简单工厂管「一个方法造多个产品」，工厂方法管「一个工厂造一个产品」，抽象工厂管「一个工厂造一族产品」。

## ❓ 高频面试题

**Q：简单工厂和工厂方法的区别？**
A：简单工厂用一个类 + `switch` 集中创建，新增产品要修改工厂类，违反开闭原则；工厂方法为每个产品定义独立工厂子类，新增产品只需加新子类不改旧代码，符合开闭原则。代价是类变多。

**Q：工厂方法和抽象工厂的区别？**
A：工厂方法一个工厂只生产**一种**产品；抽象工厂一个工厂生产**一族相关**产品（多个产品等级）。抽象工厂在「产品族维度扩展容易、产品种类维度扩展难」。

**Q：举一个 JDK 里用到工厂模式的例子？**
A：`Calendar.getInstance()`，根据地区/时区返回具体的日历实现子类（如 `GregorianCalendar`），调用方只面向 `Calendar` 抽象类编程。

## ⚠️ 易错点 / 加分项

- **误区**：把简单工厂当成 GoF 23 种之一——它不是，只是一种编程习惯（有时叫「静态工厂方法」）。
- **加分**：说清工厂模式的本质是**开闭原则 + 依赖倒置**的落地。
- **加分**：静态工厂方法（如 `Integer.valueOf()`、`List.of()`）相比构造器的好处：有名字、可缓存复用对象、可返回子类型——出自《Effective Java》第 1 条。
- **加分**：抽象工厂「新增产品族易、新增产品种类难」这个不对称性，是它最容易被追问的点。
