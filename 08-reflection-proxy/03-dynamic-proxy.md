# 03 · 动态代理（Dynamic Proxy）

> 代理模式在不改原类的前提下增强方法。JDK 动态代理基于**接口**（`Proxy`+`InvocationHandler`），CGLIB 基于**继承子类**。是 Spring AOP 的核心。面试重要度 ⭐⭐⭐。

## 📖 核心知识

**代理模式**：为目标对象提供一个代理，由代理控制对目标的访问，可在调用前后插入额外逻辑（日志、事务、权限、缓存）——即 AOP 的思想。

**静态代理**：手写一个代理类，实现和目标相同的接口，内部持有目标对象，在调用目标方法前后加逻辑。缺点：**每个被代理类都要写一个代理类**，接口方法一多、类一多，代码爆炸且难维护。

```java
interface Service { void doIt(); }
class ServiceImpl implements Service { public void doIt() { System.out.println("业务"); } }
class ServiceProxy implements Service {          // 静态代理：手写、一对一
    private final Service target;
    ServiceProxy(Service t) { this.target = t; }
    public void doIt() { System.out.println("前置"); target.doIt(); System.out.println("后置"); }
}
```

**动态代理**：运行时**动态生成**代理类，不用手写。两种主流实现：

**① JDK 动态代理——基于接口**
用 `java.lang.reflect.Proxy` 生成实现了目标接口的代理对象，所有方法调用都转发到 `InvocationHandler.invoke()`。**要求目标类必须实现接口**（生成的代理类 `extends Proxy implements 目标接口`，Java 单继承，已继承 Proxy 就无法再继承目标类，故只能面向接口）。

```java
Service target = new ServiceImpl();
Service proxy = (Service) Proxy.newProxyInstance(
    target.getClass().getClassLoader(),
    target.getClass().getInterfaces(),           // 基于接口
    (proxyObj, method, args) -> {                // InvocationHandler
        System.out.println("前置增强");
        Object result = method.invoke(target, args);  // 反射调用真实方法
        System.out.println("后置增强");
        return result;
    });
proxy.doIt();
```

**② CGLIB——基于继承（生成子类）**
通过 ASM 字节码技术**生成目标类的子类**，重写父类方法插入增强逻辑（用 `MethodInterceptor` 拦截）。不要求实现接口，但**不能代理 `final` 类和 `final`/`private` 方法**（final 无法被子类重写）。

```java
Enhancer enhancer = new Enhancer();
enhancer.setSuperclass(ServiceImpl.class);       // 基于继承目标类
enhancer.setCallback((MethodInterceptor) (obj, method, args, proxy) -> {
    System.out.println("前置增强");
    Object result = proxy.invokeSuper(obj, args);  // 调用父类原方法
    System.out.println("后置增强");
    return result;
});
ServiceImpl proxy = (ServiceImpl) enhancer.create();
proxy.doIt();
```

**对比表**：

| 维度 | JDK 动态代理 | CGLIB |
|---|---|---|
| 实现原理 | 反射，生成实现接口的代理类 | ASM 字节码，生成目标类的**子类** |
| 代理对象 | 实现同一**接口** | **继承**目标类 |
| 前提要求 | 目标类**必须有接口** | 无需接口，但类/方法不能是 `final` |
| 依赖 | JDK 自带 | 第三方库（Spring 已内置） |
| 性能 | 创建快、调用略慢（早期） | 创建慢、调用快（字节码直调，JDK 8+ 差距很小） |

## 🔑 面试要点

- 静态代理手写、一对一，类膨胀；动态代理运行时生成代理类，一套逻辑代理多个类。
- JDK 动态代理：`Proxy.newProxyInstance` + `InvocationHandler`，**基于接口**，目标类必须实现接口。
- CGLIB：`Enhancer` + `MethodInterceptor`，**基于继承生成子类**，不需接口，但不能代理 `final` 类/方法。
- JDK 代理生成的类 `extends Proxy`，因 Java 单继承所以只能实现接口不能继承目标类。
- Spring AOP 默认策略：目标类**有接口用 JDK 动态代理，无接口用 CGLIB**（可用 `proxyTargetClass=true` 强制 CGLIB）。
- Spring Boot 2.x 起默认全部用 CGLIB（`proxyTargetClass` 默认 true），避免「注入接口/实现不一致」的坑。
- 两者都在 `invoke`/`intercept` 里通过 `method.invoke`/`proxy.invokeSuper` 调真实方法并前后织入增强。

## ❓ 高频面试题

**Q：JDK 动态代理和 CGLIB 有什么区别？**
A：JDK 动态代理基于接口，用反射在运行时生成一个实现了目标接口的代理类，方法调用转发给 `InvocationHandler.invoke()`，要求目标类必须实现接口。CGLIB 基于继承，用 ASM 生成目标类的子类并重写方法插入增强（`MethodInterceptor`），不需要接口，但因为靠子类重写，无法代理 `final` 类和 `final`/`private` 方法。性能上 JDK 8 后两者差距很小。

**Q：JDK 动态代理为什么必须基于接口？**
A：`Proxy.newProxyInstance` 生成的代理类固定 `extends java.lang.reflect.Proxy`，Java 是单继承，代理类已经继承了 `Proxy`，就无法再继承目标类，只能通过「实现目标接口」来对齐方法签名。所以目标类必须提供接口，否则 JDK 动态代理无能为力，这时就得用 CGLIB。

**Q：Spring AOP 用的是哪种代理？**
A：两种都支持，默认按情况选：目标类实现了接口就用 JDK 动态代理，没有接口就用 CGLIB。可通过 `@EnableAspectJAutoProxy(proxyTargetClass = true)` 或配置强制全用 CGLIB。Spring Boot 2.x 起默认 `proxyTargetClass=true`（一律 CGLIB），以规避基于接口代理时按具体类型注入失败等问题。

**Q：静态代理和动态代理的区别？动态代理好在哪？**
A：静态代理需为每个目标类手写代理类、编译期就确定，类多则代理类爆炸、维护困难。动态代理在运行时动态生成代理类，一个 `InvocationHandler`/`MethodInterceptor` 就能通用地增强任意目标，逻辑集中、无需为每个类写代理，是 AOP、RPC 存根、MyBatis Mapper 的实现基础。

## ⚠️ 易错点 / 加分项

- CGLIB 不能代理 `final` 类和 `final`/`private`/`static` 方法（无法被子类重写）——面试高频陷阱。
- JDK 动态代理只能按接口类型接收代理对象（`(Service) proxy`），强转成实现类会 `ClassCastException`。
- Spring 中同类内部方法自调用（`this.method()`）不走代理，导致 `@Transactional`/`@Cacheable` **失效**——因为调用没经过代理对象。解决：注入自身代理或用 `AopContext.currentProxy()`。
- `InvocationHandler.invoke` 里若对 `proxy` 再调其方法会无限递归，应对 `target` 调用。
- 加分项：能说清 JDK 代理生成的类名形如 `$Proxy0`、`extends Proxy implements 接口`，并知道 `method.invoke(target, args)` 才是真正调用真实对象。
- 加分项：CGLIB 用 `MethodProxy.invokeSuper` 走 FastClass 机制（避免反射、用索引直调）比 JDK 早期反射调用快，这是 JDK 8 前 CGLIB 调用更快的原因。
