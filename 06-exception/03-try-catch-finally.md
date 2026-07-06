# 03 · try-catch-finally 执行顺序

> finally 几乎总会执行（除 JVM 退出/线程杀死等极端情况）；finally 里 return 会覆盖 try 的返回值。优先用 try-with-resources。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**基本执行顺序**：
- 无异常：`try` 全部执行 → `finally` → 返回。
- 有异常：`try` 出错点中断 → 匹配的 `catch` → `finally` → （无匹配 catch 则 finally 后异常继续上抛）。
- 无论有无异常、无论是否 return，**`finally` 都会在方法真正返回/抛出前执行**。

```java
try {
    System.out.println("1 try");
    int x = 1 / 0;                 // 抛异常
    System.out.println("不会执行");
} catch (ArithmeticException e) {
    System.out.println("2 catch");
} finally {
    System.out.println("3 finally");
}
// 输出：1 try → 2 catch → 3 finally
```

**finally 一定执行吗？** 绝大多数情况是，但有例外——`finally` **不执行**的情况：
- 进入 try 之前就返回/抛异常（根本没进 try 块）；
- `try`/`catch` 中调用了 `System.exit(n)`（JVM 直接退出）；
- 线程被强制杀死、机器断电、JVM 崩溃等。
> 结论：只要进入了 try 块且 JVM 正常存活，finally 就会执行。

**finally 中 return / 修改返回值（高频坑）**：
- `finally` 里有 `return`，会**覆盖** try/catch 里的 return，最终返回 finally 的值（并且会吞掉 try 中未处理的异常，危险）。

```java
static int f() {
    try {
        return 1;         // 先计算返回值 1，暂存
    } finally {
        return 2;         // 覆盖！最终返回 2
    }
}   // f() == 2
```

- 但如果 finally 只是**修改**基本类型的返回变量，**不影响**已确定的返回值（返回值在执行 return 时已被复制暂存）：

```java
static int g() {
    int r = 1;
    try {
        return r;         // 返回值 1 已被暂存
    } finally {
        r = 2;            // 改的是变量 r，不影响已暂存的返回值
    }
}   // g() == 1
```

> 记忆：`return` 时返回值就**定格**了；finally 改变量不影响它，但 finally 里再 `return` 会整体覆盖。**永远不要在 finally 里写 return**。

**try-with-resources（JDK 7+，推荐）**：自动关闭资源，替代手写 finally close。资源类需实现 `AutoCloseable`。

```java
// 传统写法：繁琐，还容易漏 close 或被异常掩盖
BufferedReader br = null;
try {
    br = new BufferedReader(new FileReader("a.txt"));
    return br.readLine();
} finally {
    if (br != null) br.close();
}

// try-with-resources：自动 close，代码简洁
try (BufferedReader br = new BufferedReader(new FileReader("a.txt"))) {
    return br.readLine();
}   // 无论正常还是异常，br.close() 自动调用，且在 catch/finally 之前
```

优点：
- 自动关闭，无需手写 finally；
- 多个资源按声明的**逆序**关闭；
- **异常抑制**：若 try 体和 close 都抛异常，主异常保留，close 的异常被"抑制"并记录在 `getSuppressed()` 里，不会像手写 finally 那样把原始异常覆盖掉。

## 🔑 面试要点

- 执行顺序：try（到出错点）→ 匹配 catch → finally → 返回/上抛。
- **finally 总会执行**，除非：没进 try、`System.exit()`、JVM 崩溃/线程被杀。
- **finally 里 return 会覆盖 try/catch 的 return，并吞掉异常** → 严禁在 finally 写 return。
- `return` 执行时返回值已被暂存；finally 改**基本类型变量**不影响返回值，但改**引用对象的内部状态**会生效。
- try-with-resources：资源实现 `AutoCloseable`，自动关闭、逆序关闭、异常抑制（`getSuppressed`），优于手写 finally。
- close 在 catch/finally 之前执行（try-with-resources 场景）。

## ❓ 高频面试题

**Q：finally 一定会执行吗？**
A：只要进入了 try 块且 JVM 正常运行，finally 一定执行。例外是：还没进 try 就返回、try/catch 里调用了 `System.exit()`、或 JVM 崩溃/线程被强杀。

**Q：try 里 return 了，finally 还会执行吗？先后顺序？**
A：会。finally 在方法真正返回之前执行。return 语句会先把返回值计算并暂存，然后执行 finally，最后才真正返回那个暂存值。

**Q：finally 里有 return 会怎样？**
A：finally 的 return 会覆盖 try/catch 里的 return，方法最终返回 finally 的值；而且它会吞掉 try 中尚未处理的异常，非常危险。所以绝不要在 finally 里写 return。

**Q：finally 里修改返回变量，返回值会变吗？**
A：看类型。返回值在 return 时已被复制暂存，finally 里改基本类型的局部变量不影响已暂存的返回值；但若返回的是对象引用、finally 修改对象内部字段，则调用方能看到修改（引用指向同一对象）。

**Q：try-with-resources 是什么？比 finally 好在哪？**
A：JDK 7 引入的自动资源管理语法，在 `try(...)` 括号里声明实现了 `AutoCloseable` 的资源，块结束自动 close，无需手写 finally；且多资源逆序关闭，并通过异常抑制机制保留原始异常（close 的异常进 `getSuppressed`），避免手写 finally 中 close 异常覆盖业务异常的坑。

## ⚠️ 易错点 / 加分项

- **易错**：以为 finally 改基本类型变量能改返回值——不能，返回值早已暂存。
- **易错**：手写 finally 里 `close()` 若抛异常，会**覆盖** try 中的原始异常，导致真正的错误信息丢失；try-with-resources 用异常抑制解决了这点。
- **加分**：能说出 return 时"返回值暂存"的机制，解释 `g()` 返回 1 而 `f()` 返回 2 的原因。
- **加分**：try-with-resources 中资源变量是隐式 final 的，关闭顺序与声明顺序相反。
- **加分**：`System.exit()` 会跳过 finally，是唯一"进了 try 还不执行 finally"的常规代码情形。
