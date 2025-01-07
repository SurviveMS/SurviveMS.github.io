# JUC原子操作

原子操作是指在多线程环境下，某些操作能够以单一步骤完成，中途不会被其他线程中断，具有原子性。Java 提供了一系列原子类来支持高效、安全的原子操作，主要通过 `CAS` 来实现。Java 所提供的原子类全都位于包 `java.util.concurrent.atomic` 中。

<font color="orange">`CAS` 有两种不同的说法，分别是 Compare and Swap 和 Compare and Set !</font>

上述原子类主要分为五大类：

1. 基本类型：基本类型的原子包装类，有 AtomicInteger、AtomicLong、AtomicBoolean
2. 引用类型：引用类型的原子包装类，有 AtomicReference、AtomicMarkableReference、AtomicStampedReference
3. 数组类型：数组类型的原子包装类（修改数组元素而非数组对象），有 AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray
4. 字段更新器：用于修改对象字段的原子类，有 AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater
5. 原子累加器：用于计数的累加器，有 LongAdder、LongAccumulator、DoubleAdder、DoubleAccumulator

## 基本类型原子包装类

Java 提供的基本类型的原子包装类主要有：`AtomicInteger`、`AtomicBoolean`、`AtomicLong`

### AtomicInteger

`AtomicInteger` 提供对于 `Integer` 类型的原子包装，其内部维护了一个 `int` 类型值，该类提供了基于内部`int` 值的原子和非原子操作。下面是 `AtomicInteger` 常用方法：

+ get()：获取当前值
+ set(int newValue)：设置值（非原子操作，直接赋值）
+ lazySet(int newValue)：最终会设置值，但可能会延迟
+ `getAndIncrement()`：先获取当前值，然后自增
+ `incrementAndGet()`：先自增，然后获取当前值
+ `getAndDecrement()`：先获取当前值，然后自减
+ `decrementAndGet()`：先自减，然后获取当前值
+ `addAndGet(int delta)`：添加 delta，并返回更新后的值
+ `getAndAdd(int delta)`：先获取当前值，然后添加 delta
+ `compareAndSet(int expect, int update)`：如果当前值等于预期值，则设置为新值，返回是否成功（CAS 操作）
+ `getAndUpdate(IntUnaryOperator updateFunction)`：使用指定函数更新当前值，并返回更新前的值
+ `updateAndGet(IntUnaryOperator updateFunction)`：使用指定函数更新当前值，并返回更新后的值
+ `accumulateAndGet(int x, IntBinaryOperator accumulatorFunction)`：使用累加函数与指定值更新并返回更新后的值

> 提示：着色方法都是原子性的，都是基于CAS完成。`get` 方法只是直接读取当前的值，没有参与修改，因此也可认为是线程安全的！

`AtomicInteger` 使用示例：
```java
@Slf4j
@SuppressWarnings("all")
public class AtomicIntegerDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(); // 默认初始化为0
        // 自增
        int current = atomicInteger.incrementAndGet(); // 先自增再获取，类似于 ++i
        log.info("current value is {}", current); // 1
        current = atomicInteger.getAndIncrement(); // 先获取再自增，类似于 i++
        log.info("current value is {}", current); // 1

        // 自减
        current = atomicInteger.getAndDecrement(); // 先获取再自减，类似于 i--
        log.info("current value is {}", current); // 2
        current = atomicInteger.decrementAndGet(); // 先自减再后去，类似于 --i
        log.info("current value is {}", current); // 0

        // 增加，如果需要减少则传入负数
        current = atomicInteger.addAndGet(3); // 当前加上指定值在获取
        log.info("current value is {}", current); // 3
        current = atomicInteger.getAndAdd(-2); // 获取当前值3，然后把-2增加到当前值上
        log.info("current value is {}", current); // 3
        log.info("now value is {}", atomicInteger.get()); // 1

        // 修改值
        current = atomicInteger.getAndSet(10);
        log.info("current value is {}, now value is {}", current, atomicInteger.get()); // 1, 10

        // cas操作
        int prev, update;
        do {
            // 先获取原值
            prev = atomicInteger.get(); // 10
            // 更新值
            update = prev + 10;
        } while (!atomicInteger.compareAndSet(prev, update)); // 如果cas失败则重试
        log.info("now value is {}", atomicInteger.get()); // 20
    }
}
```

> `AtomicInteger` 也提供了函数式的原子修改功能！

### AtomicLong

`AtomicLong` 提供对于 `Long` 类型的原子包装，其内部维护了一个 `long` 类型值，该类提供了基于内部`long` 值的原子和非原子操作。由于
`AtomicLong` 和 `AtomicInteger` 除了内部维护的变量类型不一样，其他操作均一致，可以参考 `AtomicInteger` API学习。

`AtomicLong` 使用示例
```java
@Slf4j
@SuppressWarnings("all")
public class AtomicLongDemo {
    public static void main(String[] args) {
        AtomicLong atomicLong = new AtomicLong(); // 0L
        // 使用函数式更新
        // 使用指定的函数更新当前值并返回
        long current = atomicLong.updateAndGet(value -> value + 10);
        log.info("current value is {}", current); // 10
        // 使用指定的值和函数更新当前值并返回
        current = atomicLong.accumulateAndGet(2, (prev, x) -> prev * x); // 10 * 2 = 20
        log.info("current value is {}, now value is {}", current, atomicLong.get()); // 20, 20
    }
}
```

### AtomicBoolean

`AtomicBoolean` 表面上是提供基于 `Boolean` 类型的原子包装，<font color="orange">但内部维护的仍然是 `int` 类型值，在其内部将 1 当作 true，0 当作 false 。<font>其常用方法如下：

+ get()：返回内部 boolean 值（直接访问，非原子性）
+ `compareAndSet(boolean expect, boolean update)`：如果当前值等于预期值，则设置为新值，返回是否成功（CAS 操作）
+ set(boolean newValue)：设置值（非原子操作，直接赋值）
+ lazySet(boolean newValue)：最终会设置值，但可能会延迟
+ getAndSet(boolean newValue)：原子地返回当前值并设置为新值

> 提示：着色方法提供基于CAS原子访问，非着色方法不具备该特性，多线程访问需注意线程安全！

`AtomicBoolean` 示例
```java
@Slf4j
@SuppressWarnings("all")
public class AtomicBooleanDemo {
    public static void main(String[] args) {
        AtomicBoolean atomicBoolean = new AtomicBoolean(true);
        // cas修改值，如果当前值符合预期值，将其修改为false
        atomicBoolean.compareAndSet(true, false); // 修改成功返回true
        log.info("now value is {}", atomicBoolean.get()); // false
        // 获取原值之后在设置为新值
        boolean current = atomicBoolean.getAndSet(true);
        log.info("current value is {}", current); // false
        log.info("now value is {}", atomicBoolean.get()); // true
    }
}
```

## 引用类型原子包装类