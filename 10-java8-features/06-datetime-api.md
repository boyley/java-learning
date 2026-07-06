# 06 · 新日期时间 API（Date/Time API）

> Java 8 用 `java.time` 包全面替代老旧的 `Date`/`Calendar`/`SimpleDateFormat`。新 API 不可变、线程安全、职责清晰，彻底解决了老 API 线程不安全、设计混乱的问题。面试重要度：⭐⭐⭐ 高频。

## 📖 核心知识

老的 `java.util.Date` 和 `SimpleDateFormat` 有一堆历史包袱：设计混乱（月份从 0 开始、年份从 1900 起算）、`Date` 可变、`SimpleDateFormat` **非线程安全**、时区处理反人类。Java 8 借鉴 Joda-Time 推出 `java.time` 包，一次性解决。

**核心类（都不可变、线程安全）**：

| 类 | 表示 | 说明 |
| --- | --- | --- |
| `LocalDate` | 日期 | 年-月-日，无时间无时区，如 `2026-07-06` |
| `LocalTime` | 时间 | 时:分:秒，无日期 |
| `LocalDateTime` | 日期+时间 | 无时区，最常用 |
| `Instant` | 时间戳 | 从 1970-01-01 起的机器时间点（UTC），对应老的毫秒时间戳 |
| `ZonedDateTime` | 带时区日期时间 | 含 `ZoneId`，处理跨时区 |
| `Duration` | 时间段（基于时间） | 两个时间点的差，精确到纳秒（时/分/秒） |
| `Period` | 时间段（基于日期） | 两个日期的差（年/月/日） |
| `DateTimeFormatter` | 格式化器 | **不可变、线程安全**，替代 SimpleDateFormat |

**常用操作示例**：

```java
// 创建
LocalDate today = LocalDate.now();
LocalDate d = LocalDate.of(2026, 7, 6);        // 月份就是 7，不再是 0 开始！
LocalDateTime dt = LocalDateTime.now();
Instant ins = Instant.now();                   // 机器时间戳

// 取值
int year = d.getYear();
Month month = d.getMonth();                    // 枚举
int day = d.getDayOfMonth();
DayOfWeek dow = d.getDayOfWeek();

// 加减（返回新对象，因为不可变）
LocalDate tomorrow = today.plusDays(1);
LocalDate lastMonth = today.minusMonths(1);
LocalDateTime later = dt.plusHours(2).plusMinutes(30);  // 链式

// 比较
boolean before = d.isBefore(today);
boolean after  = d.isAfter(today);

// Duration：两个时间之间（时分秒）
Duration du = Duration.between(t1, t2);
long seconds = du.getSeconds();
// Period：两个日期之间（年月日）
Period p = Period.between(birthday, today);
int years = p.getYears();

// 格式化 / 解析（DateTimeFormatter 线程安全）
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String s = dt.format(fmt);
LocalDateTime parsed = LocalDateTime.parse("2026-07-06 12:00:00", fmt);
```

**为什么替代 Date / SimpleDateFormat**（面试核心）：

1. **线程安全**：新 API 的对象都是**不可变（immutable）**的，每次运算返回新对象，天然线程安全，可作为常量共享。而 `SimpleDateFormat` 内部用可变的 `Calendar` 成员做中间状态，多线程共用同一个实例会导致解析/格式化出错甚至抛异常——这是老 API 最著名的坑，以前只能靠 `ThreadLocal` 或加锁规避。`DateTimeFormatter` 无此问题。
2. **设计清晰**：`Date` 既表日期又表时间还掺时区；新 API 用 `LocalDate`/`LocalTime`/`LocalDateTime`/`ZonedDateTime` 各司其职。月份、星期用枚举，不再 0~11。
3. **API 友好**：`plusDays`/`minusMonths`/`isBefore` 等语义化方法，链式调用，可读性强。
4. **时区完善**：`ZoneId`/`ZonedDateTime` 规范处理时区与夏令时。

**与老 API 互转**（遗留系统对接常问）：

```java
// Date <-> Instant <-> LocalDateTime
Date date = new Date();
Instant instant = date.toInstant();
LocalDateTime ldt = LocalDateTime.ofInstant(instant, ZoneId.systemDefault());
Date back = Date.from(ldt.atZone(ZoneId.systemDefault()).toInstant());
```

## 🔑 面试要点

- `java.time` 所有核心类**不可变、线程安全**，运算返回新对象。
- `LocalDate`（日期）/`LocalTime`（时间）/`LocalDateTime`（日期时间，无时区）/`ZonedDateTime`（带时区）/`Instant`（时间戳）。
- `Duration` 面向「时间」（时分秒纳秒），`Period` 面向「日期」（年月日），别记反。
- `DateTimeFormatter` 线程安全，可声明为 `static final` 常量；`SimpleDateFormat` 线程不安全。
- 月份从 1 开始（老 `Calendar` 从 0），星期/月份是枚举。
- 替代原因一句话：老 API **线程不安全 + 设计混乱 + 可变**，新 API **不可变 + 线程安全 + 职责清晰**。
- 与老 API 通过 `Instant` 作为桥梁互转。

## ❓ 高频面试题

**Q：为什么 Java 8 要引入新的日期时间 API？`SimpleDateFormat` 有什么问题？**
A：老 API 有三大问题：①`SimpleDateFormat` 线程不安全，它内部复用可变的 Calendar 存中间状态，多线程共享一个实例会导致格式化/解析结果错乱甚至抛异常，只能靠 ThreadLocal 或加锁；②`Date` 是可变类，且设计混乱（月份 0 开始、年份从 1900 起）；③时区处理困难。新 `java.time` API 所有对象不可变因而线程安全，用 LocalDate/LocalDateTime 等职责分离的类，方法语义化，`DateTimeFormatter` 也是线程安全的，彻底解决了这些问题。

**Q：`Duration` 和 `Period` 有什么区别？**
A：都表示「一段时间」，但维度不同。`Duration` 基于**时间**，用于计算两个时间点之间的时/分/秒/纳秒差（如两个 Instant 或 LocalTime）；`Period` 基于**日期**，用于计算两个日期之间的年/月/日差（如两个 LocalDate，算年龄）。简单记：Duration 管「时间」，Period 管「日期」。

**Q：`LocalDateTime` 和 `Instant` 有什么区别？**
A：`LocalDateTime` 是「人类可读」的本地日期时间（年月日时分秒），**不含时区**，不能唯一确定一个绝对时刻；`Instant` 是「机器时间戳」，表示 UTC 时间轴上一个确定的点（对应毫秒/纳秒时间戳），常用于记录事件发生时刻、计算时间差。二者可借助 `ZoneId` 相互转换。

## ⚠️ 易错点 / 加分项

- 把 `SimpleDateFormat` 定义成静态共享变量在多线程下用——经典线程安全 bug，新代码应换 `DateTimeFormatter`。
- `Duration`/`Period` 用反：算年龄用 Period，算耗时用 Duration。
- 新 API 对象不可变，`today.plusDays(1)` 的返回值必须接收，写成 `today.plusDays(1);` 不赋值等于没改。
- 老 `Calendar.MONTH` 从 0 开始，新 `LocalDate` 月份从 1 开始，迁移时注意。
- 加分：说明不可变设计既保证线程安全，又能安全地作为 `static final` 常量复用 formatter。
- 加分：知道 `Instant` 对应绝对时刻、`LocalDateTime` 无时区不能定位绝对时刻，跨时区场景用 `ZonedDateTime`。
