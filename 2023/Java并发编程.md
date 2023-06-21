# Java并发编程基础

## 上下文切换

**减少上下文切换的方法**

* **无锁并发编程**

  多线程竞争锁时，会引起上下文切换，所以可以**尽量避免使用锁**

* **CAS算法**

  Java的Atomic包使用CAS算法来更新数据，而不需要加锁

* **使用最少线程**

  **避免创建不需要的线程**

* **携程**

  在单线程中**实现多任务的调度**，并**在单线程中维持多个任务的切换**

## 死锁

**避免死锁的方法**

* 避免一个线程**同时获取多个锁**
* 避免一个线程**在锁内同时占用多个资源**，尽量保证每个锁**只占用一个资源**
* 尝试使用**定时锁**，使用`lock.tryLock(timeout)`来代替使用内部锁机制
* 对于数据库锁，**加锁和解锁必须在一个数据库连接里**，否则会出现解锁失败的问题

# 并发编程底层实现

## volatile应用

volatile是**轻量级**的synchronized，在多处理器开发中保证**共享变量**的**可见性**

**可见性**是当一个线程修改一个共享变量时，**另外一个线程能读到这个修改值**

它的使用和执行成本可能会比synchronized低，因为**不会引起线程上下文切换和调度**

### volatile的定义和实现原理

Java编程语言允许线程访问共享变量，为了保证共享变量能被准确和一致地更新，线程应该**确保通过排他锁单独获取这个变量**

**Java线程内存模型确保所有线程看到这个变量是一致的**

通过volatile修饰的共享变量进行写操作时，会多出一句汇编代码

`0x01a3de24: lock addl $ 0 × 0,(% esp);`

Lock前缀的指令会在多核处理器引发——

1. **将当前处理器缓存行的数据写回到系统的内存**

   Lock前缀指令在执行指令时，将声言处理器的LOCK#信号

   在多处理器环境中，LOCK#信号**确保处理器可以独占任何共享内存**

   但目前的处理器中，如果**访问的内存区域已经缓存在处理器内部**，则不会声言LOCK#信号

   它会**锁定这块内存区域的缓存**并回写到内存，并使用**缓存一致性机制**来确保修改的原子性

   缓存一致性机制会**会阻止两个以上处理器缓存的内存区域数据同时修改**

1. **写回内存的操作会使其他CPU缓存该内存地址的数据无效**

   处理器通过嗅探来检测其他处理器准备写的内存地址，**处于共享状态**，那么这个处理器的缓存行将无效，下次访问相同内存地址时，**强制执行缓存行填充**

### volatile的使用优化

**追加字节优化性能**

因为处理器的L1、L2或L3缓存的**高速缓存行是64个字节宽**，如果头尾节点都不足64字节，处理器会将他们读到同一个高速缓存行中，当处理器试图修改头节点时，会将整个缓存行锁定，在缓存一致性机制的作用下，会导致其他处理器不能访问高速缓存中的尾节点，入队出队不停修改头尾节点，严重影响效率

追加64字节来填满高速缓冲区的缓存行，避免头尾节点加载到同一个缓存行，使头尾节点在修改时不会相互锁定

**不应该使用追加字节优化volatile的情况**

* **缓存行非64字节宽的处理器**

* **共享变量时不会被频繁写**

  追加字节方式，本身嗲有一定的性能消耗，不被频繁写的共享变量，被锁定几率小

## synchronized实现原理

**Java中每个对象都可以作为锁**

* 对于普通同步方法，锁是**当前实例对象**
* 对于静态同步方法，锁是**当前类的Class对象**
* 对于同步方法块，锁是**Synchronized括号里配置的对象**

JVM基于进入和推出Monitor对象实现方法同步和代码快同步

代码块同步使用monitorenter和monitorexit指令实现

方法同步是另一种方式实现，JVM规范中没有详细说明

monitorenter指令是在**编译后**插入到同步代码块的开始位置，而monitorexit是插入到方法结束出和异常处

JVM保证每个monitorenter必须与对应的monitorexit与之配对

每个对象都有一个monitor，当且一个monitor被持有，他将处于锁定

线程执行monitorenter指令时，尝试获取对象的monitor所有权，即i获取对象的锁

### Java对象头

synchronized用的锁存在Java对象头中

**对象是数组类型**，则虚拟机用3字宽存储对象头，**对象是非数组类型**，则用2字宽存储

> 32位虚拟机1字宽等于4字节即32bit

| 内容                   | 说明                         |
| ---------------------- | ---------------------------- |
| Mark Word              | 存储对象的hashCode或所信息等 |
| Class Metadata Address | 存储到对象类型数据的指针     |
| Array Length           | 数组的长度（如果对象是数组） |

Java对象头中Mark Word默认存储**对象的HashCode、分代年龄和锁标记位**

![Mark Word状态变化](Picture\Java并发编程\Mark Word状态变化.png)

### 锁升级与对比

**锁升级但不能降级**，提高获取锁和释放锁的效率

* **偏向锁**

  **偏向锁加锁**

  当线程访问同步块并获取锁时，会在**对象头和栈帧中的锁记录存储偏向锁的线程ID**，线程再次进入和退出同步块不需要进行CAS来加锁解锁

  检查Mark Word是否存储指向当前线程的偏向锁，失败检测Mark Word中偏向锁的表示是否设置1，没有设置，使用CAS竞争锁，设置，则尝试使用CAS讲对象的偏向锁指向当前线程

  **偏向锁撤销**

  当其他线程尝试竞争偏向锁，持有偏向锁的线程就会释放锁

  偏向锁的撤销需要**等待全局安全点**（在这个时间点上没有正在执行的字节码）

  先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果不处于活动状态，则将对象头设置位无锁状态，如果活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中锁记录和对象头的Mark Word要么重新偏向其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程

  <img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\偏向锁获取和撤销的过程.png" alt="偏向锁的获取和撤销" style="zoom:80%;" />

  **关闭偏向锁**

  偏向锁在Java8**默认启用**，但是会在程序启动几秒后才会激活

  通过JVM参数可以关闭延迟`- XX:BiasedLockingStartupDelay= 0`

  如果确定所有锁通常情况下处于**竞争状态**，可以通过JVM参数关闭偏向锁

  `锁：- XX:- UseBiasedLocking= false`

* **轻量级锁**

  **轻量级锁加锁**

  线程执行同步块之前，JVM在当前线程的栈帧中创建用于存储锁记录的空间，并**将对象头中Mark Word复制到锁记录中**成为Displaced Mark Word，然后尝试使用CAS将**对象头中Mark Word替换为指向锁记录的指针**，失败通过CAS竞争获取锁

  **轻量级锁解锁**

  会使用原子的CAS操作将Displaced Mark Word替换回对象头，失败表示锁存在竞争，锁就会膨胀成重量级锁

  <img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\轻量级锁获取和膨胀的过程.png" alt="轻量级锁加锁和膨胀的构成" style="zoom:80%;" />

  因为自旋消耗CPU，为了避免无用自旋，一旦锁升级，不能降级

  当锁处于这个状态，其他线程试图获取锁时，都会被阻塞住

* **锁优缺点对比**

  | 锁       | 优点                                                         | 缺点                                              | 适用场景                               |
  | -------- | ------------------------------------------------------------ | ------------------------------------------------- | -------------------------------------- |
  | 偏向锁   | 加锁和解锁不需要额外的消耗，执行非同步方法相比仅存在**纳秒级**差距 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗    | 只有**一个线程**访问的同步场景         |
  | 轻量级锁 | 竞争的线程**不会阻塞**，提高了程序的响应速度                 | 如果始终得不到锁竞争的线程，使用自旋会**消耗CPU** | **追求相应速度**，同步块执行速度非常快 |
  | 重量级锁 | 线程竞争不使用自旋，不消耗CPU                                | **线程阻塞**，响应时间缓慢                        | **追求吞吐量**同步块执行速度较长       |

## 原子操作的实现原理

处理器读取一个字节时，其他处理器不能访问这个字节的内存地址

处理器还提供**总线锁定和缓存锁定**两个机制保证复杂内存操作的原子性

* **使用总线锁保证原子性**

  总线锁就是使用处理器提供一个LOCK#信号，**其他处理器的请求将会被阻塞**

* **使用缓存锁保证原子性**

  锁总线，其他处理器无法操作其他内存地址的数据，开销太大

  缓存锁定指**内存区域如果存在处理器的缓存行**中，并且在Lock操作期间被锁定，当执行操作写回到内存时，将修改内部的内存地址
  
  通过缓存一致性机制保证操作原子性，**会阻止两个以上处理器缓存的内存区域数据同时修改**
  
  当其他处理器回写锁定的缓存行数据时，会使缓存行无效
  
  **两种情况不会使用缓存锁定**
  
  * 当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行时
  * 处理器不支持缓存锁定
  
* **Java实现原子性**

  **使用循环CAS实现原子性**

  JVM中CAS操作使用处理器提供的`CMPXCHG`指令

  **CAS实现原子性的三个问题**

  * **ABA问题**

    > A——>B——>A

    CAS进行检查时会发现值并未发生变化，但实际发生变化

    **使用版本号来解决ABA问题**

    JDK中Atomic包中AtomicStampedReference解决ABA

    ```java
    java.util.concurrent.atomic.AtomicStampedReference#compareAndSet
    
    public boolean compareAndSet(V   expectedReference, // 预期引用
                                 V   newReference, // 更新后的引用
                                 int expectedStamp, // 预期标志
                                 int newStamp) { // 更新后的标志
    	...
    }
    ```

  * **循环时间长开销大**

    JVM支持处理器提供的pause指令

    pause指令作用——

    * 延迟流水线执行指令，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本
    * 可以避免在退出循环的时候因内存顺序冲突而引起CPU流水线被清空，从而提高CPU的执行效率

  * **只能保证一个共享变量的原子操作**

    JDK提供AtomicReference来保证引用对象之间的原子性

  **使用锁机制实现原子操作**

  锁机制保证只有获得锁次啊能操作锁定的内存区域

# Java内存模型

## 并发编程模型

* **线程之间通信**

  通信是指线程之间以何种机制来**交换信息**

  **共享内存**

  在共享内存的并发模型里，线程之间共享程序的公共状态，**通过写-读内存中的公共状态进行隐式通信**

  **消息传递**

  在消息传递的并发模型里，线程之间没有公共状态，线程之间必须**通过发送消息来显式进行通信**

* **线程之间同步**

  同步是指程序中用于控制不同线程间操作**发生相对顺序**的机制

  **共享内存**

  在共享内存的并发模型里，同步时**显式**进行的，必须指定代码需要在线程之间互斥执行

  **消息传递**

  消息的发送必须在消息的接受之前，因此是**隐式**进行的

Java并发采用**共享内存模型**，所以线程通信是隐式的，同步时显式的

### Java内存模型的抽象结构

线程之间的共享**变量存储在主内存**（Main Memory）中，每个线程**都有一个私有的本地内存**（Local Memory）

本地内存中存储该线程以读/写共享变量的副本（**JMM的抽象概念**）

<img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\Java内存模型抽象.png" alt="Java内存模型抽象" style="zoom:80%;" />

### 指令序列的重排序

* **编译器优化的重排序**

  编译器**在不改变单线程语义的前提下**，可以重新安排语句的执行顺序

* **指令级并行的重排序**

  处理器采用**指令级并行技术**，将多条指令重叠执行，如果**不存在数据依赖性**，处理器可以**改变**语句对应**机器指令的执行顺序**

* **内存系统的重排序**

  由于处理器使用缓存和读/写缓冲区，加载和存储操作看上去可能是乱序执行的

JMM的编译器重排序规则会**禁止特定类型的编译器重排序**

JMM的处理器重排序规则会要求Java编译器在生成指令序列时，**插入特定类型的内存屏障指令**，通过内存屏障指令来**禁止特定类型的处理器重排序**

### happens-before

happens-before规则

* **程序顺序规则**

  一个线程中每个操作，happens-before于该线程中任意后续操作

* **监视器锁规则**

  对于一个锁的解锁，happens-before于随后对这个锁的加锁

* **volatile变量规则**

  对一个volatile域的写，happens-before于任意后续对volatile域的读

* **传递性**

  如果A happens-before B，且B happens-before C，那么A happens-before C

**happens-before仅仅要求前一个操作对后一个操作可见**

## volatile的内存语义

### volatile的特性

* **可见性**

  对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写

* **原子性**

  对任意单个volatile变量的读/写具有原子性，但是**复合操作不具有原子性**

### volatile读-写的内存语义

当写入volatile变量时，JMM会把改线程对应的本地内存中的共享变量值刷新到主内存

当读**一个**volatile变量时，JMM会把该线程对应的本地内存置为无效，该线程需要从主内存读取共享变量

### volatile内存语义的实现

* 第二个操作是volatile写时，都不能重排序，**确保volatile写之前的操作不会被编译器重排序到volatile写之后**
* 第一个操作是volatile读时，都不能重排序，**确保volatile读之后的操作不会被编译器重排序到volatile读之前**
* 第一个操作是volatile写，第二个操作是volatile读时，不能重排序

volatile写前加入StoreStore，写后加入StoreLoad

volatile读后加入LoadLoad，读后加入LoadStore

## 锁的内存语义

### 锁获取和释放内存语义

当线程获取锁时，JMM会把该线程对应的本地内存置为无效

被监视器保护的临界区代码必须从主内存中读取共享变量

锁释放内存语义与volatile-写相同

### 锁内存语义的实现

* **公平锁**

  ```java
  java.util.concurrent.locks.ReentrantLock#lock
  
  public void lock() {
      sync.lock();
  }
  ```

  会调用内部类的lock方法

  ```java
  java.util.concurrent.locks.ReentrantLock.FairSync#lock
  
  final void lock() {
      acquire(1);
  }
  ```

  调用父类中的acquire中的方法

  ```java
  java.util.concurrent.locks.AbstractQueuedSynchronizer#acquire
  
  public final void acquire(int arg) {
      if (!tryAcquire(arg) &&
          acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
          selfInterrupt();
  }
  ```

  traAcquire方法不允许直接调用本类中的，因此只能调用子类中的

  ```java
  java.util.concurrent.locks.AbstractQueuedSynchronizer#tryAcquire
  
  protected boolean tryAcquire(int arg) {
      throw new UnsupportedOperationException();
  }
  ```

  ```java
  java.util.concurrent.locks.ReentrantLock.FairSync#tryAcquire
  
  protected final boolean tryAcquire(int acquires) {
      // 获取当前线程
      final Thread current = Thread.currentThread();
      // 获取volatile状态变量
      int c = getState();
      // 初次进入
      if (c == 0) {
          // 查询是否有任何线程等待获取的时间超过当前线程
          if (!hasQueuedPredecessors() &&
              // CAS设置state值
              compareAndSetState(0, acquires)) {
              // 设置当前拥有独占访问权限的线程
              setExclusiveOwnerThread(current);
              return true;
          }
      }
      // 重入
      // 判断当前线程是否为独占访问权限的线程
      else if (current == getExclusiveOwnerThread()) {
          // 修改状态值
          int nextc = c + acquires;
          if (nextc < 0)
              throw new Error("Maximum lock count exceeded");
          setState(nextc);
          return true;
      }
      return false;
  }
  ```

  释放锁流程

  ```java
  java.util.concurrent.locks.ReentrantLock#unlock
  
  public void unlock() {
      sync.release(1);
  }
  ```

  释放锁直接调用父类中的release方法

  ```java
  java.util.concurrent.locks.AbstractQueuedSynchronizer#release
  
  public final boolean release(int arg) {
      if (tryRelease(arg)) {
          Node h = head;
          if (h != null && h.waitStatus != 0)
              unparkSuccessor(h);
          return true;
      }
      return false;
  }
  ```

  同样不能直接使用父类的tryRelease，只能使用子类的

  ```java
  java.util.concurrent.locks.ReentrantLock.Sync#tryRelease
  
  protected final boolean tryRelease(int releases) {
      int c = getState() - releases;
      if (Thread.currentThread() != getExclusiveOwnerThread())
          throw new IllegalMonitorStateException();
      boolean free = false;
      if (c == 0) {
          free = true;
          setExclusiveOwnerThread(null);
      }
      setState(c);
      return free;
  }
  
  ```

* **非公平锁**

  ReentrantLock:lock()

  调用NonfairSync中的lock方法

  ```java
  java.util.concurrent.locks.NonfairSysc#lock
  
  final void lock() {
      if (compareAndSetState(0, 1))
          setExclusiveOwnerThread(Thread.currentThread());
      else
          acquire(1);
  }
  ```

  调用CAS设置state状态的值

  ```java
  java.util.concurrent.locks.AbstractQueuedSynchronizer#compareAndSetState
  
  protected final boolean compareAndSetState(int expect, int update) {
      return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
  }
  ```

公平锁和非公平锁释放时，最后都要写一个volatile变量state

公平锁获取时，首先会去读volatile变量

非公平锁获取时，首先会用CAS更新volatile变量，**同时具有volatile读和volatile写的内存语义**

## final域的内存语义

### 写final域的重排序规则

JMM禁止把编译器把final域的写重排序到构造函数外

编译器会在final域的写之后，构造函数return之前，**插入一个StoreStore屏障**

StoreStore屏障**禁止处理器把final域的写重排序到构造器之外**

防止线程安全问题，其他线程提前访问final域中的数据

### 读final域的重排序规则

在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM禁止处理器重排序这两个操作

处理器会在读final域操作的前面**插入LoadLoad屏障**

## 双重检查锁定

* 基础解决方案

```java
public class DoubleCheckedLocking {
    private static Instance instance;
    
    public static Instance getInstance() {
        if (instance == null) {
            syncronized (DoubleCheckdLocking.class) {
                if (instance == null) {
                    instance = new Instance();
                }
            }
            return instance;
        }
    }
}
```

* 基于volatile的解决方案

```java
public class SafeDoubleCheckedLocking {
    private volatile static Instance instance;
    
    public static Instance getInstance() {
        if (instance == null) {
            synchronized (SafeDoubleCheckedLocking.class) {
                if (instance == null) {
                    instance = new Instance();
                }
            }
            return instance;
        }
    }
}
```

* 基于类初始化的解决方案

  JVM在类的初始化阶段，会执行类的初始化，**在执行类的初始化期间，JVM会获取一个锁**

```java
public class InstanceFactory {
    private static class InstanceHolder {
        public static Instance instance = new Instance();
    }
    
    public static Instance getInstance() {
        return InstanceHolder.instance;
    }
}
```

# Java并发编程基础

## 线程基础

### 线程优先级

针对**频繁阻塞（休眠或者I/O操作）**的线程需要设置较高的优先级

针对**偏重计算**的线程则设置较低的优先级

线程优先级不能作为程序正确性的依赖，**OS可能完全不会例会Java线程对优先级的设定**

### 线程状态

| 状态名称     | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| NEW          | 初始状态，线程被构造，但**没有调用start方法**                |
| RUNNABLE     | 运行状态，Java线程将操作系统中的就绪和运行两种状态统称**运行中** |
| BLOCKED      | 阻塞状态，表示线程阻塞于锁                                   |
| WAITING      | 等待状态，表示线程进入等待状态，进入该状态表示需要**等待其他线程通知或终端** |
| TIME_WAITING | 超时等待状态，可以指定时间自行返回                           |
| TERMINATED   | 终止状态，表示当前线程已经执行完毕                           |

<img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\线程状态转换.png" alt="线程状态转换" style="zoom:80%;" />

### Daemon线程

当JVM中不存在非Daemon线程时，就会推出

Daemon线程作为支持性线程，其中的finally块不一定会执行

## 启动和终止线程

### 中断线程

中断线程是线程的一个**标识位属性**，表示一个运行中的线程是否被其他线程进行中断操作

其他线程通过调用该线程的interrupt方法对其进行中断操作

线程通过方法isInterrupted方法来进行判断是否被中断

静态方法Thread.interrupted方法对当前线程的中断标识位进行**复位**

抛出InterruptedException之前，JVM**先将该线程中断标识位清除**，再抛出异常

## 线程间通信

### Synchronized实现

任意线程对Object（Object由synchronized保护）的访问，首先要获得Object的监视器

如果获取失败，线程进入**同步队列**，线程状态变为**BLOCKED**

当访问Object的前驱（获得锁的线程）释放锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新尝试对监视器的获取

### 等待/通知机制

| 方法名称       | 描述                                                         |
| -------------- | ------------------------------------------------------------ |
| notify()       | 通知一个在对象上**等待的线程**，使其从**wait()方法**返回，而返回的前提是该线程获取到了对象的锁 |
| notifyAll()    | 通知所有等待在该对象上的线程                                 |
| wait()         | 调用该方法的线程进入**WATTING状态**，只有等待另外线程的通知或被中断才会返回，调用wait()后，释放对象锁 |
| wait(long)     | 超时等待一段时间，这里的参数时间是毫秒                       |
| wait(long,int) | 对于超时时间更细粒度的控制，可以到达纳秒                     |

> **wait()和sleep()区别**
>
> 1. wait和notify是Object中的方法，而sleep是Thread类中的方法
> 2. sleep方法调用后，**没有释放锁**，而wait是进入线程等待池中等待，让出资源，释放对象锁
> 3. **wait()不会自己唤醒**，需要线程调用notify/notifyAll方法唤醒，**sleep方法会自动唤醒**，如果时间不到，可以**使用interrupt方法强行打断**
> 4. sleep可以在**任何地方**使用，wait、notify、notifyAll只能**在同步控制方法和同步控制块中**使用
> 5. sleep必须捕获异常，wait、notify、notifyAll不需要捕获异常，调用wait方法**没有持有适当的锁**，执行wait后，抛出异常IllegalMonitorException
> 6. 线程sleep()睡眠到期自动返回到**就绪状态**
> 7. sleep()是静态方法，只能控制**当前正在运行的线程**

### 管道输入/输出流

管道输入/输出流主要用于线程之间的数据传输，而**传输的媒介为内存**

PipedInputStream、PipedOutputStream、PipedReader、PipedWriter

对于Piped类型的流，必须通过**调用connect()方法进行绑定**

### Thread.join()的使用

线程A执行thread.join()，A线程等待thread线程终止才从thread.join()返回

```java
java.lang.Thread#join

public final synchronized void join(long millis)
    throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) {
            wait(0);
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

当线程终止时，会调用线程自身的notifyAll()方法，释放join()阻塞的线程

# Java锁

## Lock接口

不要将获取锁的过程写在try块中，如果在获取锁时发生异常，**异常抛出的同时，也会导致锁无故释放**

| 方法名称                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| void lock()                                                  | 获取锁                                                       |
| void lockInterruptibly() throws InterruptedException         | **可中断**的获取锁，锁的获取中可以中断当前线程               |
| boolean tryLock()                                            | 尝试**非阻塞**获取锁，能够获取返回true，否则fasle            |
| boolean tryLock(long time, TimeUnit unit) throws InterruptedException | 超时获取锁，退出情况<br />1.当前线程在超时时间内获取锁<br />2.当前线程在超时时间内被中断<br />3.超时时间结束，返回false |
| void unlock()                                                | 释放锁                                                       |
| Condition newCondition()                                     | 获取等待通知组件，组件与当前锁绑定，获取锁，才能调用condition.wait()，调用后，**当前线程释放锁** |

## 队列同步器

AQS是用来构建锁或者同步组件的基础框架，使用int成员变量表示同步状态，**通过内置的FIFO队列来完成资源获取线程的排队工作**

同步器主要使用方式是**继承**，子类继承并实现抽象方法来管理同步状态

锁是面向使用者，隐藏了实现细节

同步器面向的是锁的实现类，简化锁的实现方式，屏蔽底层操作

同步器的设计是基于**模板方法模式**

| 方法名称                                           | 描述                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| protected boolean tryAcquire(int arg)              | **独占式**获取同步状态，实现该方法需要查询当前状态并判断同步状态是否符合预期，然后再进行CAS设置同步状态 |
| protected boolean tryRelease(int arg)              | **独占式**释放同步状态，等待获取同步状态的线程将有机会获取同步状态 |
| protected int try AcquireShared(int arg)           | **共享式**获取同步状态，返回大于等于0的值，表示获取成功，反之，获取失败 |
| protected boolean tryReleaseShared(int arg)        | **共享式**释放同步状态                                       |
| protected boolean isHeldExclusively()              | 当前同步器是否**在独占模式下被线程占用**，一般该方法表示是否被当前线程所独占 |
| void acquire(int arg)                              | **独占式**获取同步状态，如果当前线程获取同步状态成功，则返回，否则进入同步队列等待，该方法会嗲用重写的tryAcquire(int arg) |
| void acquireInterruptibly(int arg)                 | 与acquire方法相同，但**该方法响应中断**，当前线程未获取到同步状态而进入同步队列中，如果当前线程被中断，则该方法会抛出InterruptedException并返回 |
| boolean tryAcquireNanos(int arg, long nanos)       | 在acquireInterruptibly的基础上，添加了超时限制，如果当前线程超时时间内没有获取到同步状态，返回false |
| void acquireShared(int arg)                        | **共享式**的获取同步状态，如果当前线程未获取到同步状态，将会进入同步队列等待，与独占式获取的区别**同一时刻可以有多个线程获取到同步状态** |
| boolean tryAcquireSharedInterruptibly(int arg)     | 响应中断                                                     |
| boolean tryAcquireSharedNanos(int arg, long nanos) | 加上了超时限制                                               |
| boolean release(int arg)                           | 独占式的释放同步状态，该方法会在释放同步状态之后，**将同步队列中第一个节点包含的线程唤醒** |
| Collection<Thread\> getQueuedThreads()             | 获取等待的同步队列上的线程集合                               |

**独占锁**是同一时刻**只能有一个线程获取到锁**，其他线程只能处于同步队列中等待，只有获取锁的线程释放锁，后续的线程才能够获取锁

### 队列同步器实现分析

* **同步队列**

  同步器依赖内部的**同步队列（一个FIFO双向队列）**来完成同步状态的管理，当线程获取同步状态失败时，同步器会将当前线程以及等待状态等信息构成一个节点（Node）并将其加入同步队列，**同时会阻塞当前线程**，当同步状态释放时，会把首节点中的线程唤醒，使其再次尝试获取同步状态

  同步队列中的节点用来保存获取同步状态失败的线程引用、等待状态以及前驱和后继节点，系欸但属性类型与名称以及描述

  <img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\AQS节点.png" alt="AQS节点" style="zoom:80%;" />

  同步器拥有首节点和尾节点，没有成功获取同步状态的线程将会成为节点加入该队列的尾部

  <img src="D:\WorkSpace\Mnsx-Note\2023\Picture\Java并发编程\同步器中同步队列结构.png" alt="同步器中同步队列结构" style="zoom:80%;" />

  这个加入队列的过程必须保证线程安全，以你同步器提供基于CAS的设置尾节点的方法`compareAndSetTail(Node expect, Node update)`

* **独占式同步状态获取与释放**

  
