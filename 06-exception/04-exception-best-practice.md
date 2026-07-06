# 04 · 异常最佳实践（Exception Best Practices）

> 精准捕获、不吞异常、保留异常链、优先自定义业务异常、注意异常的性能开销。面试重要度 ⭐⭐。

## 📖 核心知识

**1. 自定义异常**：为业务定义清晰的异常类型，通常继承 `RuntimeException`（非受检，避免签名污染），携带错误码/上下文。

```java
public class BizException extends RuntimeException {
    private final int code;
    public BizException(int code, String message) {
        super(message);
        this.code = code;
    }
    public BizException(int code, String message, Throwable cause) {
        super(message, cause);          // 保留异常链
        this.code = code;
    }
    public int getCode() { return code; }
}
```

**2. 不要吞异常**：空 catch 块（或只 `printStackTrace` 后照常执行）会让错误无声消失，是最危险的反模式。

```java
// 反例：吞异常，出错了完全无感知
try { doSomething(); } catch (Exception e) { }

// 正例：至少记录日志，或包装后上抛
try { doSomething(); }
catch (IOException e) {
    log.error("读取失败, file={}", file, e);       // 记录含堆栈
    throw new BizException(500, "读取失败", e);     // 或转译上抛
}
```

**3. 异常链（Chained Exception）**：捕获底层异常后转成高层异常时，一定把原始异常作为 `cause` 传入（`new XxxException(msg, cause)` 或 `initCause`），否则丢失根因、排错困难。

```java
try {
    jdbc.query(...);
} catch (SQLException e) {
    throw new DataAccessException("查询用户失败", e);   // e 作为 cause，堆栈里有 Caused by
}
```

**4. 精准捕获、就近处理**：
- catch 具体异常类型，别一把 `catch (Exception e)` 甚至 `catch (Throwable)`；
- 多个类型可用多重 catch `catch (IOException | SQLException e)`（JDK 7+）；
- 捕获顺序**子类在前、父类在后**（否则父类先匹配，子类 catch 不可达，编译错误）；
- 不要用异常做流程控制（能 `if` 判断就别靠 catch）。

**5. 异常的性能注意**：
- 抛异常本身**开销大**的部分在于**构造异常时填充堆栈轨迹**（`fillInStackTrace`，遍历调用栈），而非 try 块本身。
- 没有异常发生时，`try` 几乎零开销，所以别怕包 try。
- **热点路径 / 高频循环里别用异常控制流程**；如需高频抛出可重写 `fillInStackTrace()` 返回 `this` 省略堆栈（谨慎）。

**6. 其他要点**：
- catch 里别只 `e.printStackTrace()`（输出到标准错误、无级别、生产不可控），应走日志框架。
- 抛异常时给足上下文信息（哪个参数、哪条数据）。
- `finally` 里别写 `return`、别抛新异常（会覆盖原异常）——见 [03](03-try-catch-finally.md)。
- 资源关闭优先 try-with-resources。

## 🔑 面试要点

- 自定义异常一般继承 `RuntimeException`，带错误码/上下文，提供含 `cause` 的构造器。
- **绝不吞异常**：空 catch 是重大隐患，至少记日志或包装上抛。
- 转译异常必须**保留异常链**（传 cause），堆栈里才有 `Caused by`，能定位根因。
- 精准 catch 具体类型；多重 catch 用 `|`；**子类 catch 在前、父类在后**。
- 不要用异常做正常流程控制。
- 异常性能开销主要来自**构造时填充堆栈**，热点路径慎用；无异常时 try 本身几乎无开销。
- 用日志框架而非 `printStackTrace()`；优先 try-with-resources 管资源。

## ❓ 高频面试题

**Q：自定义异常继承 Exception 还是 RuntimeException？**
A：看语义。多数业务场景继承 `RuntimeException`（非受检），避免 throws 层层污染方法签名、便于在框架里统一处理；若希望编译器强制调用方处理某类可恢复错误，才继承 `Exception`（受检）。工程实践更常用非受检。

**Q：什么是异常链？为什么重要？**
A：捕获低层异常后包装成高层异常时，把原始异常作为 `cause` 传入（构造器或 `initCause`）。这样堆栈里会出现 `Caused by: ...` 显示根本原因，不会丢失底层错误信息，极大方便排查。

**Q：捕获多个异常时顺序有讲究吗？**
A：有。子类异常的 catch 必须写在父类之前，否则父类 catch 会先匹配、子类 catch 永远进不去，编译器直接报"已被捕获"错误。也可用 `catch (A | B e)` 多重捕获同级异常。

**Q：try-catch 会影响性能吗？**
A：没有异常抛出时，try 块本身开销可以忽略。真正的开销在**抛出并构造异常对象时填充堆栈轨迹**（`fillInStackTrace`）。所以正常路径不必担心 try，但不要在高频循环里用抛异常来控制流程。

## ⚠️ 易错点 / 加分项

- **易错**：`catch (Exception e) {}` 空吞——生产环境最危险的反模式，问题被静默隐藏。
- **易错**：转译异常时 `throw new XxxException(e.getMessage())` 只带消息不带 cause，根因堆栈全丢。正确是 `new XxxException(msg, e)`。
- **易错**：父类 catch 写在子类前面导致编译错误（unreachable catch）。
- **加分**：能指出异常性能瓶颈在 `fillInStackTrace`，并知道高频场景可重写它省略堆栈的取舍。
- **加分**：`printStackTrace()` 在生产不可取——无日志级别、可能输出到丢失的标准错误流，应统一走 SLF4J/Logback。
- **加分**：不要用异常做正常控制流（如用异常判断循环结束），既慢又难读。
