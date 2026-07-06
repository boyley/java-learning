# 01 · Lambda 表达式（Lambda Expression）

> Lambda 是「函数式接口实例」的简写，让你把行为（代码块）当参数传递。本质是函数式接口的一个实现，底层用 `invokedynamic` 实现而非匿名内部类。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

Lambda 表达式是 Java 8 引入的语法糖，用于**简洁地创建函数式接口的实例**。它把「一段可执行的行为」当作数据传递，是函数式编程的入口。

**基本语法**：`(参数列表) -> {方法体}`。编译器能根据上下文推断类型，所以写法很灵活：

```java
// 完整写法
Runnable r1 = () -> { System.out.println("run"); };
// 单条语句可省略大括号和分号
Runnable r2 = () -> System.out.println("run");
// 单参数可省略括号；参数类型可省略（类型推断）
Consumer<String> c = s -> System.out.println(s);
// 多参数
Comparator<Integer> cmp = (a, b) -> a - b;
// 方法体只有一个返回表达式时，省略 return 和大括号
BinaryOperator<Integer> add = (a, b) -> a + b;
```

**本质：Lambda 不是匿名内部类**。很多人以为 Lambda 就是匿名内部类的语法糖，这不准确。Lambda 的目标类型必须是一个**函数式接口**（只有一个抽象方法的接口），编译后：
- 匿名内部类会生成一个独立的 `.class` 文件（如 `Outer$1.class`）。
- Lambda **不会**生成额外的 class 文件，而是编译成一个私有静态方法，运行时通过 `invokedynamic` 字节码指令 + `LambdaMetafactory` 动态生成实现类。这样避免了类爆炸，性能也更好。

**变量捕获（closure）与 effectively final**：Lambda 可以访问外层方法的局部变量，但这些变量必须是 `final` 或 **effectively final**（事实上不可变——赋值后再没被修改过，即使没写 `final` 关键字）。

```java
int base = 10;            // 没写 final，但之后没再改，是 effectively final
Function<Integer, Integer> f = x -> x + base;  // OK
// base = 20;             // 如果取消注释，上面这行会编译报错
```

为什么必须 effectively final？因为局部变量存在**栈**上，方法结束就销毁，而 Lambda 实例可能在方法返回后才执行。JVM 的做法是把捕获的变量**值拷贝**一份到 Lambda 实例里，若允许修改就会出现「拷贝的值」和「原变量」不一致的歧义，因此干脆禁止修改。注意：捕获的如果是引用类型，引用本身不能变，但引用指向的对象内部状态可以改。

**`this` 指向不同**：在 Lambda 内 `this` 指向**外层类**实例；而匿名内部类的 `this` 指向匿名类自己。这是二者的重要区别。

## 🔑 面试要点

- Lambda = 函数式接口实例的简写，目标类型必须是函数式接口（有且仅有一个抽象方法）。
- 语法 `(参数) -> 表达式/{语句块}`；参数类型、单参数括号、单语句大括号和 return 都可省。
- 底层是 `invokedynamic` + `LambdaMetafactory` 动态生成，**不生成独立 class 文件**，区别于匿名内部类。
- 捕获的局部变量必须 `final` 或 effectively final；根本原因是局部变量在栈上、Lambda 捕获的是值拷贝。
- Lambda 中 `this` 指向外层类实例；匿名内部类的 `this` 指向自身。
- Lambda 没有独立作用域，和外层是同一作用域，不能声明与外层同名的局部变量；匿名内部类是独立作用域。
- Lambda 只能用于函数式接口，不能用于抽象类。

## ❓ 高频面试题

**Q：Lambda 表达式和匿名内部类有什么区别？**
A：主要四点：①编译产物——匿名内部类生成独立 class 文件，Lambda 用 invokedynamic 动态生成，不产生额外 class；②`this` 指向——Lambda 中指向外层类，匿名内部类指向自身；③作用域——Lambda 与外层同作用域（不能定义同名变量），匿名内部类是独立作用域；④适用范围——匿名内部类可以实现任意接口/抽象类，Lambda 只能用于函数式接口。

**Q：为什么 Lambda 捕获的局部变量必须是 final 或 effectively final？**
A：局部变量存在栈上，方法执行完就出栈销毁，而 Lambda 实例的生命周期可能更长。JVM 实际是把变量值**拷贝**进 Lambda，如果允许原变量修改，就会出现拷贝值与原值不一致的二义性，为保证数据一致性和线程安全，语言规定被捕获的变量不可再修改。

**Q：Lambda 底层是怎么实现的？**
A：编译期把 Lambda 体转成当前类的一个私有方法，调用处生成一条 `invokedynamic` 指令，运行时首次执行时由 `LambdaMetafactory.metafactory` 引导，动态生成一个实现了目标函数式接口的类并缓存复用。好处是延迟到运行时生成、避免匿名类的类爆炸、便于 JVM 优化。

## ⚠️ 易错点 / 加分项

- 误以为 Lambda 就是匿名内部类的语法糖——底层机制完全不同（invokedynamic vs 生成 class）。
- 在 Lambda 里想修改捕获的局部变量——编译报错；如需「可变」可用长度为 1 的数组或原子类绕过（但要注意线程安全）。
- Lambda 内直接写 `this` 以为指向 Lambda 本身——实际指向外层类实例。
- 加分：能说出 `invokedynamic` 的意义（延迟绑定、避免类爆炸），面试官会眼前一亮。
- 加分：捕获引用类型时，「引用不可变」不等于「对象不可变」，对象内部字段仍可修改。
