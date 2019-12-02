## ThreadLocal

首先 ThreadLocal 是一个泛型类，保证可以接受任何类型的对象。其能保证线程与线程之间相互隔离。

**使用场景**

如上文所述，ThreadLocal 适用于如下两种场景

- 每个线程需要有自己单独的实例
- 实例需要在多个方法中共享，但不希望被多线程共享

存储用户Session、解决线程安全的问题（Java7中的SimpleDateFormat不是线程安全的）

![image-20191111183905587](assets/image-20191111183905587.png)

## 实现原理：

因为一个线程内可以存在多个 ThreadLocal 对象，所以其实是 ThreadLocal 内部维护了一个 Map ，这个 Map 不是直接使用的 HashMap ，而是 ThreadLocal 实现的一个叫做 ThreadLocalMap 的静态内部类。而我们使用的 get()、set() 方法其实都是调用了这个ThreadLocalMap类对应的 get()、set() 方法。例如下面的 set 方法：

### set 方法

ThreadLocal的set方法，大致意思为

- 首先获取当前线程
- 利用当前线程作为句柄获取一个**ThreadLocalMap**的对象
- 如果上述ThreadLocalMap对象不为空，则设置值，否则创建这个ThreadLocalMap对象并设置值

源码如下：

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

下面是一个利用Thread对象作为句柄获取ThreadLocalMap对象的代码

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

上面的代码获取的实际上是Thread对象的threadLocals变量，可参考下面代码

```java
class Thread implements Runnable {
    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */

    ThreadLocal.ThreadLocalMap threadLocals = null;
}
```

而如果一开始设置，即ThreadLocalMap对象未创建，则新建ThreadLocalMap对象，并设置初始值。

```java
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

总结：实际上ThreadLocal的值是放入了当前线程的一个ThreadLocalMap实例中，所以只能在本线程中访问，其他线程无法访问。

从上面可以知道，ThreadLocal 实例（this）被 ThreadLocalMap 所持有，当使用线程池管理线程时，由于复用线程，就会出现 ThreadLocalMap 中存在过多的新 ThreadLocal 实例，从而导致内存泄漏。

threadLocalMap 初始大小为16，当容量超过2/3时会自动扩容。

### 真的只能被一个线程访问么

既然上面提到了 ThreadLocal 只对当前线程可见，是不是说 ThreadLocal 的值只能被一个线程访问呢？

使用 InheritableThreadLocal 对象可以实现多个线程访问 ThreadLocal 的值。

如下，我们在主线程中创建一个 InheritableThreadLocal 的实例，然后在子线程中得到这个 InheritableThreadLocal 实例设置的值。

```java
`private void testInheritableThreadLocal() {     final ThreadLocal threadLocal = new InheritableThreadLocal();     threadLocal.set("droidyue.com");     Thread t = new Thread() {         @Override         public void run() {             super.run();             Log.i(LOGTAG, "testInheritableThreadLocal =" + threadLocal.get());         }     };      t.start(); } `
```

上面的代码输出的日志信息为

```shell
`I/MainActivity( 5046): testInheritableThreadLocal =droidyue.com `
```

使用 InheritableThreadLocal 可以将某个线程的 ThreadLocal 值在其子线程创建时传递过去。因为在线程创建过程中，有相关的处理逻辑。

### 内存泄漏问题

实际上 ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用，弱引用的特点是，如果这个对象只存在弱引用，那么在下一次垃圾回收的时候必然会被清理掉。

所以如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会被清理掉的，这样一来 ThreadLocalMap中使用这个 ThreadLocal 的 key 也会被清理掉。但是，value 是强引用，不会被清理，这样一来就会出现 key 为 null 的 value。

ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。如果说会出现内存泄漏，那只有在出现了 key 为 null 的记录后，没有手动调用 remove() 方法，并且之后也不再调用 get()、set()、remove() 方法的情况下。

**如何避免：**

使用完ThreadLocal后，执行remove操作，避免出现内存溢出情况。