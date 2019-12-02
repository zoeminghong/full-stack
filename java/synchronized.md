# Synchronized

## 常规知识

锁分类：

- 获取对象锁
  - synchronized(this|object) {}
  - 修饰非静态方法
- 获取类锁
  - synchronized(类.class) {}
  - 修饰静态方法

小结

- 类中 synchronized(this|object) {} 代码块和 synchronized 修饰非静态方法获取的锁是同一个锁，即该**类的对象的对象锁**。

- 两个线程访问不同对象的 synchronized(类.class) {} 代码块或 synchronized 修饰静态方法还是同步的，类中 synchronized(类.class) {} 代码块和 synchronized 修饰静态方法获取的锁是类锁。对于同一个类的不同对象的类锁是同一个。

类中同时有 synchronized(类.class) {} 代码块或 synchronized 修饰静态方法和 synchronized(this|object) {} 代码块和 synchronized 修饰非静态方法时会怎样？

**对象锁和类锁是独立的，互不干扰。**

**synchronized关键字不能继承。**对于父类中的 synchronized 修饰方法，子类在覆盖该方法时，默认情况下不是同步的，必须显示的使用 synchronized 关键字修饰才行。

在定义接口方法时不能使用synchronized关键字。

构造方法不能使用synchronized关键字，但可以使用synchronized代码块来进行同步。

## 底层实现

Java中提供了两种实现同步的基础语义：`synchronized`方法和`synchronized`块， 我们来看个demo：

```
public class SyncTest {
    public void syncBlock(){
        synchronized (this){
            System.out.println("hello block");
        }
    }
    public synchronized void syncMethod(){
        System.out.println("hello method");
    }
}
复制代码
```

当SyncTest.java被编译成class文件的时候，`synchronized`关键字和`synchronized`方法的字节码略有不同，我们可以用`javap -v` 命令查看class文件对应的JVM字节码信息，部分信息如下：

```
{
  public void syncBlock();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: aload_0
         1: dup
         2: astore_1
         3: monitorenter				 	  // monitorenter指令进入同步块
         4: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         7: ldc           #3                  // String hello block
         9: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        12: aload_1
        13: monitorexit						  // monitorexit指令退出同步块
        14: goto          22
        17: astore_2
        18: aload_1
        19: monitorexit						  // monitorexit指令退出同步块
        20: aload_2
        21: athrow
        22: return
      Exception table:
         from    to  target type
             4    14    17   any
            17    20    17   any
 

  public synchronized void syncMethod();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED      //添加了ACC_SYNCHRONIZED标记
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #5                  // String hello method
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
 
}
```

每个对象有一个监视器锁(monitor)。当monitor被占用时就会处于锁定状态，线程执行monitorenter指令时尝试获取monitor的所有权，过程：

1. 如果monitor的进入数为0，则该线程进入monitor，然后将进入数设置为1，该线程即为monitor的所有者。

2. 如果线程已经占有该monitor，只是重新进入，则进入monitor的进入数加1.
3. 如果其他线程已经占用了monitor，则该线程进入阻塞状态，直到monitor的进入数为0，再重新尝试获取monitor的所有权。

#### 加锁过程

1.在线程栈中创建一个`Lock Record`，将其`obj`（即上图的Object reference）字段指向锁对象。

2.**直接通过CAS指令将`Lock Record`的地址存储在对象头的`mark word`中，如果对象处于无锁状态则修改成功，代表该线程获得了轻量级锁。**如果失败，进入到步骤3。

3.如果是当前线程已经持有该锁了，代表这是一次锁重入。设置`Lock Record`第一部分（`Displaced Mark Word`）为null，起到了一个重入计数器的作用。然后结束。

4.走到这一步说明发生了竞争，需要膨胀为重量级锁。

#### 解锁过程

1.遍历线程栈,找到所有obj字段等于当前锁对象的Lock Record。

2.如果Lock Record的Displaced Mark Word为null，代表这是一次重入，将obj设置为null后continue。

3.如果Lock Record的Displaced Mark Word不为null，则利用CAS指令将对象头的mark word恢复成为Displaced Mark Word。如果成功，则continue，否则膨胀为重量级锁。

> 加锁、解锁都使用了 CAS 操作

https://juejin.im/post/5bfe6ddee51d45491b0163eb

https://mip.yunqishi.net/dnjc/dnzx/30175.html

## 问题

**偏向锁和轻量级锁作用是什么？**

它们的引入是为了解决 synchronized 在没有多线程竞争或基本没有竞争的场景下因使用传统锁机制带来的性能开销问题。

**通过什么方式可以查看 class 文件内容？**

我们可以用`javap -v` 命令查看class文件对应的JVM字节码信息

**偏向锁的优化 synchronized 什么？**

加锁解锁都有一个CAS操作；对于重量级锁而言，加锁也会有一个或多个CAS操作（这里的’一个‘、’多个‘数量词只是针对该demo，并不适用于所有场景）。在JDK1.6中为了**提高一个对象在一段很长的时间内都只被一个线程用做锁对象场景下的性能**，引入了偏向锁，在第一次获得锁时，会有一个CAS操作，之后该线程再获取锁，只会执行几个简单的命令，而不是开销相对较大的CAS命令。

**synchronized 和 lock 的区别？**

一、synchronized和lock的用法区别

synchronized：在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

lock：需要显示指定起始位置和终止位置。一般使用ReentrantLock类做为锁，多个线程中必须要使用一个ReentrantLock类做为对象才能保证锁的生效。且在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。

用法区别比较简单，这里不赘述了，如果不懂的可以看看Java基本语法。

二、synchronized和lock性能区别

synchronized是托管给JVM执行的，而lock是java写的控制锁的代码。在Java1.5中，synchronize是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用Java提供的Lock对象，性能更高一些。但是到了Java1.6，发生了变化。synchronize在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在Java1.6上synchronize的性能并不比Lock差。官方也表示，他们也更支持synchronize，在未来的版本中还有优化余地。

说到这里，还是想提一下这2中机制的具体区别。据我所知，synchronized原始采用的是CPU悲观锁机制，即线程获得的是独占锁。独占锁意味着其他线程只能依靠阻塞来等待线程释放锁。而在CPU转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起CPU频繁的上下文切换导致效率很低。

而Lock用的是乐观锁方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是CAS操作（Compare and Swap）。我们可以进一步研究ReentrantLock的源代码，会发现其中比较重要的获得锁的一个方法是compareAndSetState。这里其实就是调用的CPU提供的特殊指令。

现代的CPU提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

我也只是了解到这一步，具体到CPU的算法如果感兴趣的读者还可以在查阅下，如果有更好的解释也可以给我留言，我也学习下。

三、synchronized和lock用途区别

synchronized原语和ReentrantLock在一般情况下没有什么区别，但是在非常复杂的同步应用中，请考虑使用ReentrantLock，特别是遇到下面2种需求的时候。

1.某个线程在等待一个锁的控制权的这段时间需要中断
2.需要分开处理一些wait-notify，ReentrantLock里面的Condition应用，能够控制notify哪个线程
3.具有公平锁功能，每个到来的线程都将排队等候