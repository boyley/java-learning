# 08 · 装饰器模式（Decorator）

> 在**不修改原类、不用继承爆炸**的前提下，通过「层层包裹」动态地给对象叠加新功能。Java IO 的 `BufferedXxx` 就是它的教科书级应用。面试重要度 ⭐⭐（常与代理模式对比考）。

## 📖 核心知识

装饰器和被装饰对象实现**同一个接口**，装饰器内部**持有一个该接口的引用**，在调用被包装对象方法的前后追加增强逻辑。多个装饰器可以像套娃一样嵌套，任意组合功能。

```mermaid
graph LR
    Client --> D2[装饰器2：加牛奶] --> D1[装饰器1：加糖] --> C[被装饰对象：咖啡]
```

```java
interface Coffee { double cost(); }

class SimpleCoffee implements Coffee {           // 被装饰的核心对象
    public double cost() { return 10; }
}

// 抽象装饰器：持有同接口引用
abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;
    CoffeeDecorator(Coffee coffee) { this.coffee = coffee; }
}

class Milk extends CoffeeDecorator {
    Milk(Coffee c) { super(c); }
    public double cost() { return coffee.cost() + 3; }   // 在原功能上叠加
}
class Sugar extends CoffeeDecorator {
    Sugar(Coffee c) { super(c); }
    public double cost() { return coffee.cost() + 1; }
}

// 使用：层层包裹，自由组合
Coffee c = new Sugar(new Milk(new SimpleCoffee()));
System.out.println(c.cost());   // 10 + 3 + 1 = 14
```

### 真实应用：Java IO

Java IO 是装饰器模式最经典的落地。`InputStream` 是抽象组件，`FileInputStream` 是核心实现，`BufferedInputStream`、`DataInputStream` 都是装饰器：

```java
// 层层包装：文件流 → 加缓冲 → 加基本类型读取能力
DataInputStream in = new DataInputStream(
    new BufferedInputStream(
        new FileInputStream("a.txt")));
```

同理 `BufferedReader(new InputStreamReader(...))`、`BufferedWriter`、`BufferedOutputStream` 都是装饰器。这解释了为什么 Java IO 类看起来「一层套一层」（详见 [`07-io-nio`](../07-io-nio)）。

其他：Spring 的 `TransactionAwareCacheDecorator`、`Collections.synchronizedList()`/`unmodifiableList()`（给集合装饰上线程安全/只读能力）。

### 装饰器 vs 代理（高频对比）

| 维度 | 装饰器模式 | 代理模式 |
| --- | --- | --- |
| 目的 | **增强功能**（加缓冲、加线程安全…） | **控制访问**（权限、事务、懒加载、远程） |
| 对象来源 | 装饰对象**由外部传入**（`new Milk(coffee)`） | 目标对象通常**由代理自己创建/持有** |
| 关注点 | 我要给你**加什么新能力** | 我要不要、能不能让你**被访问** |
| 嵌套 | 天生支持多层嵌套自由组合 | 一般一层 |

> 结构上都是「持有同接口引用 + 转发调用」，几乎一样；区别在**意图**：装饰器强调「锦上添花加功能」，代理强调「把关控制访问」。

## 🔑 面试要点

- 目的：**动态、透明地给对象增加功能**，替代「为每种功能组合写一个子类」的继承爆炸。
- 结构：装饰器与被装饰者**实现同一接口** + 装饰器**持有该接口引用** + 前后增强 + 可多层嵌套。
- 遵循开闭原则：加新功能 = 加新装饰器，不改原类。
- 教科书应用：**Java IO 的 `BufferedInputStream`/`BufferedReader`/`DataInputStream`**。
- 与代理的区别：装饰器**增强功能**（对象外部传入），代理**控制访问**（对象常自己持有）。

## ❓ 高频面试题

**Q：装饰器模式解决什么问题？为什么不用继承？**
A：给对象动态叠加功能。若用继承，N 种功能的任意组合需要 2^N 个子类（类爆炸），且是编译期静态的。装饰器用组合，在运行时任意包裹组合功能，灵活且符合开闭原则。

**Q：Java IO 为什么设计成一层套一层？用了什么模式？**
A：装饰器模式。`FileInputStream` 负责读字节，`BufferedInputStream` 装饰它加缓冲，`DataInputStream` 再装饰加基本类型读取。各功能正交、可自由组合，不用为「带缓冲的文件流」「带缓冲又能读 int 的文件流」各写一个类。

**Q：装饰器模式和代理模式有什么区别？**
A：结构类似（都持有同接口引用并转发），区别在意图：装饰器为对象**增强/添加功能**，被装饰对象一般由客户端从外部传入并显式组合；代理是为对象**控制访问**（权限、事务、懒加载、远程调用），代理通常自己管理目标对象、对客户端透明。

## ⚠️ 易错点 / 加分项

- **加分**：能一句话点透——「装饰器和代理结构几乎一样，差别在**意图**：加功能 vs 控访问」，这是最能加分的回答。
- **加分**：能举出 `Collections.synchronizedList()`、`unmodifiableList()` 是给集合装饰上线程安全/只读能力。
- **易错**：装饰器要求与被装饰者**实现同一接口/抽象类**，否则无法层层嵌套和透明替换。
- **加分**：装饰器多层嵌套过深会导致调试困难、堆栈变长（Java IO 就常被吐槽包装繁琐，故有 `Files.newBufferedReader()` 等便捷工厂）。
