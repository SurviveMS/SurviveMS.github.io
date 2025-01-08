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

`AtomicBoolean` 表面上是提供基于 `Boolean` 类型的原子包装，<font color="orange">但内部维护的仍然是 `int` 类型值，在其内部将 1 当作 true，0 当作 false 。</font>其常用方法如下：

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

在一些特殊业务场景下（如涉及货币），除了保证数据的精确性外，还需要保证多个线程修改是安全的，我们就可能就会用到 BigDecimal 数字类型配合 JUC 提供的引用类型包装类来使用。

### AtomicReference

`AtomicReference` 专门用于原子的更新对象的引用，其接受一个范型参数表示内部维护的具体引用类型。`AtomicReference` 在构造时不传入对象则内部引用对象为 `null` 。常用方法如下：

+ get()：获取当前引用值
+ set(V newValue)：设置内部对象引用（非原子操作）
+ lazySet(V newValue)：设置内部对象引用（非原子操作，保证最终设置，可能有延时）
+ `compareAndSet(V expect, V update)`：如果当前内部引用于预期值一致，则将内部引用更新为指定值，返回是否cas成功
+ `getAndSet(V newValue)`：获取当前内部对象引用，然后将其更新为新的引用
+ `getAndUpdate(UnaryOperator<V> updateFunction)`：获取当前内部对象引用，然后使用函数将内部引用更新为指定引用
+ `updateAndGet(UnaryOperator<V> updateFunction)`：使用指定函数更新内部引用，然后获取内部对象引用
+ `getAndAccumulate(V x, BinaryOperator<V> accumulatorFunction)`：获取当前内部引用对象然后使用指定的引用对象和函数更新
+ `accumulateAndGet(V x, BinaryOperator<V> accumulatorFunction)`：使用指定的引用对象和函数更新当前内部对象引用并返回

> 提示：着色方法均通过 `CAS` 原子性

`AtomicReference` 示例

```java
@Slf4j
@SuppressWarnings("all")
public class AtomicReferenceDemo {
    public static void main(String[] args) {
        // 使用BigDecimal作为内部对象
        BigDecimal bigDecimal = new BigDecimal("23.12");
        AtomicReference<BigDecimal> atomicReference = new AtomicReference<>(bigDecimal);
        // CAS操作
        BigDecimal prev, update;
        do {
            // 获取值
            prev = atomicReference.get();
            update = prev.subtract(new BigDecimal("12.99"));
            // 没有别的线程修改内部引用，因此一次cas即可修改成功，不会出现重试
        } while(!atomicReference.compareAndSet(prev, update));

        // 原子的获取当前引用并设置为指定值
        BigDecimal current = atomicReference.getAndSet(prev); // 10.13
        log.info("current value is :{}", current);

        // 函数式更新
        current = atomicReference.accumulateAndGet(new BigDecimal("32.12"),
            (pre, x) -> x.add(pre));
        log.info("current value is :{}", current); // 55.24
        log.info("now value is :{}", atomicReference.get()); // 55.24
    }
}
```

### AtomicStampedReference

`AtomicStampedReference` 的作用与 `AtomicReference` 类似，不过 `AtomicStampedReference` 增加了一个标记字段（`stamp`）来解决 `ABA` 问题。在 `CAS` 操作时，除了比较当前值与预期值之外，还会比较标记字段，仅当两者都符合预期时，才会 `CAS` 成功。并且这个标记需要按照一定的业务规则更新。标记通常也称为版本号。

> `ABA` 问题：即一个线程可能读取到某个值 A，然后进行修改。与此同时，另一个线程可能先将这个值 A 修改为 B，再修改回 A。在这种情况下，<font color="orange">原本的线程并不会意识到这个值的变化</font>，通常情况下 ABA 问题不会对业务造成影响。

`AtomicStampedReference` 构造函数接收两个参数，其一是内部对象引用，其二是初始版本号。`AtomicStampedReference` 内部将引用对象和版本号定义为 `Pair` 类型，内部维护 `Pair` 实例。`AtomicStampedReference` 常用方法：

+ get(int[] stampHolder)：传入数组接收当前版本号，方法返回内部对象引用
+ `compareAndSet(V expectedReference, V newReference, int expectedStamp, int newStamp)`：CAS操作，更新成功返回true，否则false
+ set(V newReference, int newStamp)：设置内部对象引用和版本号
+ `attemptStamp(V expectedReference, int newStamp)`：原子的修改版本号

> 着色方法保证原子性操作！

`AtomicStampedReference` 示例

```java
@Slf4j
@SuppressWarnings("all")
public class AtomicStampedReferenceDemo {
    public static void main(String[] args) {
        BigDecimal bigDecimal = new BigDecimal("100.00");
        int stampe = 1;
        AtomicStampedReference<BigDecimal> atomicStampedReference 
            = new AtomicStampedReference<>(bigDecimal, stampe);
        // cas
        BigDecimal prev, update;
        int prevStamp, updateStamp;
        // 启动线程修改
        new Thread(() -> {
            LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(1));
            log.info("A：BigDecimal = {}，stamp = {}", atomicStampedReference.getReference(),
                atomicStampedReference.getStamp());
            BigDecimal dest = new BigDecimal("500.00");
            int newStamp = stampe + 1;
            atomicStampedReference.compareAndSet(bigDecimal, dest, stampe, newStamp);
            log.info("A -> B：BigDecimal = {}，stamp = {}", atomicStampedReference.getReference(),
                atomicStampedReference.getStamp());
            // 修改为原来的引用对象
            atomicStampedReference.compareAndSet(dest, bigDecimal, newStamp, newStamp + 1);
            log.info("B -> A：BigDecimal = {}，stamp = {}", atomicStampedReference.getReference(),
                atomicStampedReference.getStamp());
        }, "t1").start();
        do {
            prev = atomicStampedReference.getReference();
            update = prev.subtract(new BigDecimal("37.50"));
            prevStamp = atomicStampedReference.getStamp();
            updateStamp = prevStamp + 1;
            // 模拟其他线程修改
            LockSupport.parkNanos(TimeUnit.SECONDS.toNanos(3));
        } while (!atomicStampedReference.compareAndSet(prev, update, prevStamp, updateStamp));
    }
}
```

### AtomicMarkableReference

`AtomicMarkableReference` 与 `AtomicStampedReference` 类似，不过 `AtomicMarkableReference` 不关注 `ABA` 问题的修改次数，只关心是否被修改过，`AtomicMarkableReference` 只使用一个 布尔值标记（mark） 来表示引用的状态。

`AtomicMarkableReference` 允许支持对一个引用对象加上一个布尔值的标记。在多线程环境下，这种设计可以避免 ABA 问题，因为即使引用对象的值相同，但标记值发生了变化，CAS 操作也会失败，从而避免了不一致的操作。其API与 `AtomicStampedReference` 类似。

`AtomicMarkableReference` 示例

```java
@Slf4j
@SuppressWarnings("all")
public class AtomicMarkableReferenceDemo {

    public static void main(String[] args) {
        // 创建一个 AtomicMarkableReference，初始化引用为 "A"，标记为 false
        AtomicMarkableReference<String> reference
            = new AtomicMarkableReference<>("A", false);

        // 获取当前引用的对象和标记
        String currentReference = reference.getReference();
        boolean currentMark = reference.isMarked();
        log.info("Initial Reference: {}， Initial Mark: ", currentReference, currentMark);

        // 线程 A 通过 CAS 更新引用对象和标记
        boolean success = reference.compareAndSet("A", "B", false, true);
        log.info("CAS Update Success: {}", success);

        // 线程 A 再次尝试将引用更新为 "C" 并将标记设置为 false
        success = reference.compareAndSet("B", "C", true, false);
        log.info("CAS Update Success: {}", success);

        // 获取更新后的引用和标记
        currentReference = reference.getReference();
        currentMark = reference.isMarked();
        log.info("Final Reference: {}, Final Mark: {}", currentReference, currentMark);
    }
}
```

## 数组类型原子包装类

## 字段更新器

## 原子累加器
