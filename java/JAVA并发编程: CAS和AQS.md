# JAVA并发编程: CAS和AQS

说起JAVA并发编程，就不得不聊聊CAS(Compare And Swap)和AQS了(`AbstractQueuedSynchronizer`)。

## CAS(Compare And Swap)

### 什么是CAS

> CAS(Compare And Swap)，即比较并交换。是解决多线程并行情况下使用锁造成性能损耗的一种机制，CAS操作包含三个操作数——内存位置（V）、预期原值（A）和新值(B)。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。无论哪种情况，它都会在CAS指令之前返回该位置的值。CAS有效地说明了“我认为位置V应该包含值A；如果包含该值，则将B放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。

在JAVA中，`sun.misc.Unsafe` 类提供了硬件级别的原子操作来实现这个CAS。 `java.util.concurrent` 包下的大量类都使用了这个 `Unsafe.java` 类的CAS操作。至于 `Unsafe.java` 的具体实现这里就不讨论了。

### CAS典型应用

`java.util.concurrent.atomic` 包下的类大多是使用CAS操作来实现的(eg. `AtomicInteger.java`,`AtomicBoolean`,`AtomicLong`)。下面以 `AtomicInteger.java` 的部分实现来大致讲解下这些原子类的实现。

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();

    private volatile int value;// 初始int大小
    // 省略了部分代码...

    // 带参数构造函数，可设置初始int大小
    public AtomicInteger(int initialValue) {
        value = initialValue;
    }
    // 不带参数构造函数,初始int大小为0
    public AtomicInteger() {
    }

    // 获取当前值
    public final int get() {
        return value;
    }

    // 设置值为 newValue
    public final void set(int newValue) {
        value = newValue;
    }

    //返回旧值，并设置新值为　newValue
    public final int getAndSet(int newValue) {
        /**
        * 这里使用for循环不断通过CAS操作来设置新值
        * CAS实现和加锁实现的关系有点类似乐观锁和悲观锁的关系
        * */
        for (;;) {
            int current = get();
            if (compareAndSet(current, newValue))
                return current;
        }
    }

    // 原子的设置新值为update, expect为期望的当前的值
    public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
    }

    // 获取当前值current，并设置新值为current+1
    public final int getAndIncrement() {
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return current;
        }
    }

    // 此处省略部分代码，余下的代码大致实现原理都是类似的
}
```

一般来说在竞争不是特别激烈的时候，使用该包下的原子操作性能比使用 synchronized 关键字的方式高效的多(查看getAndSet()，可知如果资源竞争十分激烈的话，这个for循环可能换持续很久都不能成功跳出。不过这种情况可能需要考虑降低资源竞争才是)。
在较多的场景我们都可能会使用到这些原子类操作。一个典型应用就是计数了，在多线程的情况下需要考虑线程安全问题。通常第一映像可能就是:

```java
public class Counter {
    private int count;
    public Counter(){}
    public int getCount(){
        return count;
    }
    public void increase(){
        count++;
    }
}

```

上面这个类在多线程环境下会有线程安全问题，要解决这个问题最简单的方式可能就是通过加锁的方式,调整如下:

```java
public class Counter {
    private int count;
    public Counter(){}
    public synchronized int getCount(){
        return count;
    }
    public synchronized void increase(){
        count++;
    }
}
```

这类似于悲观锁的实现，我需要获取这个资源，那么我就给他加锁，别的线程都无法访问该资源，直到我操作完后释放对该资源的锁。我们知道，悲观锁的效率是不如乐观锁的，上面说了Atomic下的原子类的实现是类似乐观锁的，效率会比使用 synchronized 关系字高，推荐使用这种方式，实现如下:

```java
public class Counter {
    private AtomicInteger count = new AtomicInteger();
    public Counter(){}
    public int getCount(){
        return count.get();
    }
    public void increase(){
        count.getAndIncrement();
    }
}
```

## AQS(AbstractQueuedSynchronizer)

### 什么是AQS

AQS(AbstractQueuedSynchronizer)，AQS是JDK下提供的一套用于实现基于FIFO等待队列的阻塞锁和相关的同步器的一个同步框架。这个抽象类被设计为作为一些可用原子int值来表示状态的同步器的基类。如果你有看过类似 CountDownLatch 类的源码实现，会发现其内部有一个继承了 AbstractQueuedSynchronizer 的内部类 Sync 。可见 CountDownLatch 是基于AQS框架来实现的一个同步器.类似的同步器在JUC下还有不少。(eg. Semaphore )

### AQS用法

如上所述，AQS管理一个关于状态信息的单一整数，该整数可以表现任何状态。比如， Semaphore 用它来表现剩余的许可数，ReentrantLock 用它来表现拥有它的线程已经请求了多少次锁；FutureTask 用它来表现任务的状态(尚未开始、运行、完成和取消)

```java
 To use this class as the basis of a synchronizer, redefine the
 * following methods, as applicable, by inspecting and/or modifying
 * the synchronization state using {@link #getState}, {@link
 * #setState} and/or {@link #compareAndSetState}:
 *
 * <ul>
 * <li> {@link #tryAcquire}
 * <li> {@link #tryRelease}
 * <li> {@link #tryAcquireShared}
 * <li> {@link #tryReleaseShared}
 * <li> {@link #isHeldExclusively}
 * </ul>
```

如JDK的文档中所说，使用AQS来实现一个同步器需要覆盖实现如下几个方法，并且使用`getState`,`setState`,`compareAndSetState`这几个方法来设置获取状态

1. boolean tryAcquire(int arg)
2. boolean tryRelease(int arg)
3. int tryAcquireShared(int arg)
4. boolean tryReleaseShared(int arg)
5. boolean isHeldExclusively()

以上方法不需要全部实现，根据获取的锁的种类可以选择实现不同的方法，支持独占(排他)获取锁的同步器应该实现tryAcquire、 tryRelease、isHeldExclusively而支持共享获取的同步器应该实现tryAcquireShared、tryReleaseShared、isHeldExclusively。下面以 CountDownLatch 举例说明基于AQS实现同步器, CountDownLatch 用同步状态持有当前计数，countDown方法调用 release从而导致计数器递减；当计数器为0时，解除所有线程的等待；await调用acquire，如果计数器为0，acquire 会立即返回，否则阻塞。通常用于某任务需要等待其他任务都完成后才能继续执行的情景。源码如下:

```java
public class CountDownLatch {
    /**
     * 基于AQS的内部Sync
     * 使用AQS的state来表示计数count.
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            // 使用AQS的getState()方法设置状态
            setState(count);
        }

        int getCount() {
            // 使用AQS的getState()方法获取状态
            return getState();
        }

        // 覆盖在共享模式下尝试获取锁
        protected int tryAcquireShared(int acquires) {
            // 这里用状态state是否为0来表示是否成功，为0的时候可以获取到返回1，否则不可以返回-1
            return (getState() == 0) ? 1 : -1;
        }

        // 覆盖在共享模式下尝试释放锁
        protected boolean tryReleaseShared(int releases) {
            // 在for循环中Decrement count直至成功;
            // 当状态值即count为0的时候，返回false表示 signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }

    private final Sync sync;

    // 使用给定计数值构造CountDownLatch
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);
    }

    // 让当前线程阻塞直到计数count变为0，或者线程被中断
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    // 阻塞当前线程，除非count变为0或者等待了timeout的时间。当count变为0时，返回true
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    // count递减
    public void countDown() {
        sync.releaseShared(1);
    }

    // 获取当前count值
    public long getCount() {
        return sync.getCount();
    }

    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```

### AQS实现原理浅析

AQS的实现主要在于维护一个volatile int state（代表共享资源）和一个FIFO线程等待队列（多线程争用资源被阻塞时会进入此队列）。队列中的每个节点是对线程的一个封装，包含线程基本信息，状态，等待的资源类型等。

CLH结构如下:

```java
 *      +------+  prev +-----+       +-----+
 * head |      | <---- |     | <---- |     |  tail
 *      +------+       +-----+       +-----+
```

下面简单看下获取资源的代码:

```java
    public final void acquire(int arg) {
        // 首先尝试获取,不成功的话则将其加入到等待队列，再for循环获取
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    
    // 从clh中选一个线程获取占用资源
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                // 当节点的先驱是head的时候，就可以尝试获取占用资源了tryAcquire
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    // 如果获取到资源，则将当前节点设置为头节点head
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
                // 如果获取失败的话，判断是否可以休息，可以的话就进入waiting状态，直到被unpark()
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
  
   private Node addWaiter(Node mode) {
        // 封装当前线程和模式为新的节点,并将其加入到队列中
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }  
    
    private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { 
                // tail为null，说明还没初始化，此时需进行初始化工作
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                // 否则的话，将当前线程节点作为tail节点加入到CLH中去
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }

```

详细参考:[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

