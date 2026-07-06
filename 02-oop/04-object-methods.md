# 04 · Object 的方法（Object Methods）

> `Object` 是所有类的根父类，提供 `equals`/`hashCode`/`toString`/`getClass`/`clone`/`wait`/`notify`/`finalize` 等公共方法。面试重要度：⭐⭐ 常考。

## 📖 核心知识

`Object` 共 11 个方法（含重载），按用途分四组：

### 1. 比较与标识

- **`equals(Object)`**：默认比引用地址（`==`），常被重写为逻辑相等。
- **`hashCode()`**：native，返回哈希码，与 equals 遵守契约（详见 `../01-java-basics/04-equals-hashcode.md`）。
- **`getClass()`**：native，final，返回运行时 `Class` 对象，反射入口，不能被重写。

### 2. 字符串表示

- **`toString()`**：默认返回 `类名@十六进制hashCode`，通常重写为可读描述。`System.out.println(obj)`、字符串拼接都会隐式调用它。

### 3. 克隆

- **`clone()`**：protected，native，创建并返回对象副本。要用必须实现 `Cloneable` 标记接口，否则抛 `CloneNotSupportedException`。默认是**浅拷贝**（引用字段只拷地址，副本与原对象共享同一引用对象）；要**深拷贝**需自己递归克隆引用字段。

```java
class Person implements Cloneable {
    int[] scores;
    @Override
    protected Person clone() throws CloneNotSupportedException {
        Person p = (Person) super.clone();  // 浅拷贝
        p.scores = scores.clone();           // 手动深拷贝引用字段
        return p;
    }
}
```

### 4. 线程协作（都要在 synchronized 里调用）

- **`wait()` / `wait(long)`**：final，让当前线程释放锁并等待，直到被唤醒或超时。
- **`notify()` / `notifyAll()`**：final，唤醒在此对象上等待的一个 / 全部线程。
- 必须持有该对象监视器锁时调用，否则抛 `IllegalMonitorStateException`。原理见并发章节。

### 5. 已废弃

- **`finalize()`**：GC 回收对象前调用，JDK9 起 `@Deprecated`，**不可靠、不要用**，资源释放用 try-with-resources / `Cleaner`。

| 方法 | 修饰 | 可重写 | 用途 |
| --- | --- | --- | --- |
| `equals` | public | ✅ | 逻辑相等 |
| `hashCode` | public native | ✅ | 哈希码 |
| `toString` | public | ✅ | 字符串表示 |
| `getClass` | public final native | ❌ | 运行时类型 |
| `clone` | protected native | ✅ | 对象拷贝 |
| `wait/notify/notifyAll` | public final | ❌ | 线程协作 |
| `finalize` | protected | ✅（废弃） | 回收前回调 |

## 🔑 面试要点

- `Object` 是所有类的根，共 11 个方法。
- `getClass`、`wait`、`notify`、`notifyAll` 是 `final`，不能重写。
- `equals`/`hashCode`/`toString`/`clone`/`finalize` 可重写。
- `clone` 默认浅拷贝，深拷贝要手动克隆引用字段或用序列化。
- 用 `clone` 必须实现 `Cloneable`（标记接口），否则抛异常。
- `wait`/`notify` 必须在 `synchronized` 块内、持有对象锁时调用。
- `finalize` 已废弃且不可靠，别依赖它释放资源。

## ❓ 高频面试题

**Q：`Object` 有哪些方法？**
A：`equals`、`hashCode`、`toString`、`getClass`、`clone`、`wait`（3 个重载）、`notify`、`notifyAll`、`finalize`，共 11 个。其中 `getClass`/`wait`/`notify`/`notifyAll` 是 final 不可重写。

**Q：浅拷贝和深拷贝的区别？**
A：浅拷贝只复制对象本身，引用类型字段仍指向原对象（共享），改副本的引用字段会影响原对象；深拷贝会递归复制引用字段指向的对象，副本与原对象完全独立。`Object.clone()` 默认浅拷贝，深拷贝需手动克隆每个引用字段或借助序列化。

**Q：为什么 `wait`/`notify` 定义在 `Object` 而不是 `Thread`？**
A：因为它们是基于**对象监视器锁**的线程协作机制，等待/唤醒都针对某个对象的锁而非线程。任何对象都可以作为锁，所以这些方法必须在所有对象的根类 `Object` 上，且调用前必须持有该对象的锁。

**Q：`finalize` 能用来释放资源吗？**
A：不能依赖。它执行时机不确定（取决于 GC）、可能根本不执行、还会影响 GC 性能和对象回收，JDK9 起已废弃。释放资源应使用 try-with-resources（`AutoCloseable`）或 `Cleaner`/`PhantomReference`。

## ⚠️ 易错点 / 加分项

- `Cloneable` 是**标记接口**（无方法），`clone()` 方法其实在 `Object` 里，容易记混。
- `clone()` 不调用构造器，直接在内存层面复制，可能绕过构造器中的初始化逻辑。
- `wait` 要在循环里判断条件（防**虚假唤醒** spurious wakeup），不能用 if。
- 加分：深拷贝除手动克隆外，还可用序列化/反序列化（对象需 `Serializable`）或第三方库实现。
- 加分：`sleep` 是 `Thread` 的静态方法且不释放锁，`wait` 是 `Object` 方法且释放锁，二者常被对比。
