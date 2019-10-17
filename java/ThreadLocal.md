## ThreadLocal

## 实现原理：

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

### 真的只能被一个线程访问么

既然上面提到了 ThreadLocal 只对当前线程可见，是不是说 ThreadLocal 的值只能被一个线程访问呢？

使用 InheritableThreadLocal 可以实现多个线程访问 ThreadLocal 的值。

如下，我们在主线程中创建一个 InheritableThreadLocal 的实例，然后在子线程中得到这个 InheritableThreadLocal 实例设置的值。

```java
`private void testInheritableThreadLocal() {     final ThreadLocal threadLocal = new InheritableThreadLocal();     threadLocal.set("droidyue.com");     Thread t = new Thread() {         @Override         public void run() {             super.run();             Log.i(LOGTAG, "testInheritableThreadLocal =" + threadLocal.get());         }     };      t.start(); } `
```

上面的代码输出的日志信息为

```shell
`I/MainActivity( 5046): testInheritableThreadLocal =droidyue.com `
```

使用 InheritableThreadLocal 可以将某个线程的 ThreadLocal 值在其子线程创建时传递过去。因为在线程创建过程中，有相关的处理逻辑。

