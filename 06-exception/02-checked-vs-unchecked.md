# 02 · 受检 vs 非受检异常（Checked vs Unchecked）

> 受检异常编译器强制处理（try 或 throws），非受检（RuntimeException/Error）不强制。分界是"是否 RuntimeException 子类"。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**受检异常（Checked Exception）**：编译器在**编译期**就检查你有没有处理它。凡是 `Exception` 但**不是** `RuntimeException` 子类的都属于受检异常。方法内可能抛出受检异常时，必须二选一：
- 用 `try-catch` 捕获处理，或
- 在方法签名上 `throws` 声明，交给调用者。

否则**编译不通过**。典型：`IOException`、`SQLException`、`ClassNotFoundException`、`InterruptedException`。

```java
// 必须处理，否则编译报错
public void read() throws IOException {      // 方式一：throws 向上抛
    Files.readAllBytes(Paths.get("a.txt"));
}
public void read2() {
    try {
        Files.readAllBytes(Paths.get("a.txt"));
    } catch (IOException e) {                 // 方式二：捕获
        // 处理
    }
}
```

**非受检异常（Unchecked Exception）**：编译器**不强制**处理，包括 `RuntimeException` 及其子类，以及 `Error` 及其子类。多由程序 bug 引起，可通过写对代码来避免，而非靠 catch。典型：`NullPointerException`、`IllegalArgumentException`、`ArrayIndexOutOfBoundsException`、`ClassCastException`。

```java
public void div(int a, int b) {
    int r = a / b;      // b==0 抛 ArithmeticException（非受检），编译期不强制处理
}
```

**设计哲学 / 如何选择**：
- **可恢复、调用者可预期并应对**的外部问题 → 用**受检异常**，强制对方处理（如文件不存在、网络中断）。
- **编程错误、调用者违反契约、通常无法当场恢复** → 用**非受检异常**（如参数非法用 `IllegalArgumentException`、状态非法用 `IllegalStateException`、空引用 `NullPointerException`）。
- 现代工程（尤其 Spring 生态）**倾向多用非受检异常**：受检异常会污染方法签名、层层 `throws` 或被迫 `try` 吞掉，破坏封装；把受检包装成运行期异常（如 Spring 把 `SQLException` 包成 `DataAccessException`）更简洁。

## 🔑 面试要点

- **受检**：编译器强制 `try` 或 `throws`；是 `Exception` 但非 `RuntimeException` 子类；例：`IOException`、`SQLException`、`ClassNotFoundException`、`InterruptedException`。
- **非受检**：编译器不强制；`RuntimeException` + `Error` 及其子类；例：`NPE`、`IllegalArgumentException`、`ClassCastException`。
- 分界口诀：**是不是 `RuntimeException` 的子类**。
- 选择原则：**可恢复/可预期外部问题用受检，编程错误用非受检**。
- 现代框架倾向非受检（避免签名污染），常把受检异常包装为运行期异常上抛。
- 受检异常是 Java 特有设计，很多语言（C#、Kotlin）没有。

## ❓ 高频面试题

**Q：受检异常和非受检异常有什么区别？**
A：受检异常在编译期被强制处理，必须 `try-catch` 或 `throws` 声明，否则编译不过（如 `IOException`）；非受检异常是 `RuntimeException` 和 `Error` 的子类，编译器不强制处理（如 `NullPointerException`）。分界标准是是否为 `RuntimeException` 的子类。

**Q：什么时候用受检异常，什么时候用非受检？**
A：如果异常是调用者可预期、可恢复的外部情况（文件不存在、网络失败），用受检强制对方处理；如果是编程错误、契约违背、通常无法恢复（参数非法、空指针），用非受检。经验上工程里更多用非受检，避免签名被 throws 层层污染。

**Q：`RuntimeException` 需要在方法签名上声明 throws 吗？**
A：不需要（也可以写但没意义）。它是非受检异常，编译器不强制声明或捕获。

**Q：受检异常是好设计吗？**
A：有争议。优点是编译期强制处理、防止遗漏；缺点是污染方法签名、鼓励空 catch 吞异常、和 Lambda/Stream 不兼容。所以现代框架常用非受检异常或做包装。答题时可辩证陈述。

## ⚠️ 易错点 / 加分项

- **易错**：把分界当成"Exception vs Error"——错，`RuntimeException` 也是 Exception 但非受检。真正分界是 `RuntimeException`。
- **易错**：以为 `throws` 就"处理"了异常——`throws` 只是把责任上抛，最终仍需有人真正处理。
- **加分**：重写方法时，子类方法抛出的受检异常**不能比父类更宽**（里氏替换），但可以更窄或不抛；非受检异常不受此限。
- **加分**：Lambda / Stream 里不能直接抛受检异常，常见做法是包装成 `RuntimeException`，这也是工程偏好非受检的原因之一。
- **加分**：能举 Spring `DataAccessException` 把 `SQLException`（受检）转成非受检的例子，体现"包装转换"实践。
