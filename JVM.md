# JVM概述

## JVM简介

定义——Java Virtual Machine，Java程序的运行环境（Java二进制字节码的运行环境）

好处——

* 一次编写，到处运行
* 自动内存管理，垃圾回收功能
* 数组下标越界的检查
* 多态

比较：

![image-20221101130357279](D:\WorkSpace\Note\Picture\image-20221101130357279.png)

## 常见JVM

![image-20221101130210930](D:\WorkSpace\Note\Picture\image-20221101130210930.png)

## JVM结构

![image-20221101130251587](D:\WorkSpace\Note\Picture\image-20221101130251587.png)

# 内存结构

## 程序计数器

Program Counter Register 程序计数器（寄存器）

**记住下一条JVM指令的执行地址**

程序计数器特点——

* 线程私有的
* 唯一一个不会存在内存溢出的模块

## 虚拟机栈

Java Virtual Machine Stacks虚拟机栈

* 每个线程运行需要的内存空间，称为虚拟机栈
* 每个栈由多个栈帧组成，对应着每次方法调用时所占用的内存
* 每个线程只能由一个活动栈帧，对应当前正在执行的那个方法，栈顶部的方法

> 1. 垃圾回收不用涉及栈内存，因为栈内存会随方法结束，自动释放栈帧内存
>
> 2. 栈内存并不是设置的越大越好，因为硬盘内存是有限的，如果栈内存越大，那么线程数也就越少，不能提高程序性能
>
>    可以通过`-Xss 内存大小`来设置栈内存的大小，不设置有默认值
>
> 3. **如果方法内局部变量没有方法的作用范围，他就是线程安全的**，**如果是局部变量引用了对象，并逃离方法的作用范围，需要考虑线程安全**

### 栈内存溢出

* 栈帧过多导致栈内存溢出
* 栈帧内存过大导致栈内存溢出

`java.lang.StackOverflowError`栈内存溢出

### 线程运行诊断

temp1：cpu占用过多

* 使用top命令定位那个进程对CPU占用过高

  ![image-20221101173207826](D:\WorkSpace\Note\Picture\image-20221101173207826.png)

* `ps H -eo pid,tid,%cpu | grep 进程id`定位是哪个线程引起的CPU占用过高

  ![image-20221101173222632](D:\WorkSpace\Note\Picture\image-20221101173222632.png)

  线程17333占用CPU资源最大，换算为16进制

  ![image-20221101173433510](D:\WorkSpace\Note\Picture\image-20221101173433510.png)

* jstack 进程id

  ![image-20221101173311881](D:\WorkSpace\Note\Picture\image-20221101173311881.png)

  通过线程ID，找到有问题的行数，进一步找到有问题的行数

程序运行很长时间没有结果

![image-20221101175019983](D:\WorkSpace\Note\Picture\image-20221101175019983.png)

## 本地方法栈

Native Method Stacks本地方法栈

为本地方法运行提供存储空间

## 堆

Heap堆

通过new关键字，创建对象都会使用堆内存

堆的特点——

* 它是线程共享的，堆中对象都需要考虑线程安全问题
* 有垃圾回收机制

### 堆内存溢出

`java.lang.OutOfMemoryError: Java heap space`

使用`-Xmx 堆内存大小`来控制虚拟机中堆内存大小

### 堆内存诊断

1. jps工具

   查看当前系统有哪些java进程

2. jmap工具

   查看堆内存占用情况

3. jconsole工具

   图形工具，多功能的监控工具，可以连续检测

```java
public class Demo1_7 {
    public static void main(String[] args) {
        System.out.println("1...");
        try {
            TimeUnit.MILLISECONDS.sleep(30000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        byte[] arr = new byte[1024 * 1024 * 10];
        System.out.println("2...");
        try {
            TimeUnit.MILLISECONDS.sleep(30000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        arr = null;
        System.gc();
        System.out.println("3...");
        try {
            TimeUnit.MILLISECONDS.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

* 使用jmap工具，`jmap -heap 线程号`

  ![image-20221102112105034](D:\WorkSpace\Note\Picture\image-20221102112105034.png)

* 使用jconsole

  ![image-20221102112308741](D:\WorkSpace\Note\Picture\image-20221102112308741.png)

案例——**垃圾回收后，内存占用仍然很高**

* 源代码

```java
public class Demo1_8 {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        for (int i = 0; i < 200; ++i) {
            students.add(new Student());
        }
        try {
            TimeUnit.MILLISECONDS.sleep(10000000L);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

class Student {
    private byte[] big = new byte[1024 * 1024];
}
```

* 问题分析

![image-20221102113206387](D:\WorkSpace\Note\Picture\image-20221102113206387.png)

![image-20221102113214255](D:\WorkSpace\Note\Picture\image-20221102113214255.png)

![image-20221102113910070](D:\WorkSpace\Note\Picture\image-20221102113910070.png)

使用gc垃圾回收之后人就存在内存占用很高的问题

* 使用jvisualvm工具检测

![image-20221102113640230](D:\WorkSpace\Note\Picture\image-20221102113640230.png)

![image-20221102113650441](D:\WorkSpace\Note\Picture\image-20221102113650441.png)

![image-20221102113657928](D:\WorkSpace\Note\Picture\image-20221102113657928.png)

![image-20221102113947546](D:\WorkSpace\Note\Picture\image-20221102113947546.png)

## 方法区

Method Area方法区——概念性的区域

![image-20221102114743189](D:\WorkSpace\Note\Picture\image-20221102114743189.png)

![image-20221102114748691](D:\WorkSpace\Note\Picture\image-20221102114748691.png)

### 方法区内存溢出

```java
// 使用虚拟机参数 -XX:MaxMetaspaceSize=8m
public class Demo1_9 extends ClassLoader { // 用来加载类的二进制字节码
    public static void main(String[] args) {
        int j = 0;
        try {
            Demo1_9 test = new Demo1_9();
            for (int i = 0; i < 10000; ++i, j++) {
                // ClassWriter，用代码的方式生成类的二进制字节码
                ClassWriter cw = new ClassWriter(0);
                // 参数1，版本号  参数2，修饰符     参数3，名字      参数4，包名      参数5，父类      参数6接口
                cw.visit(Opcodes.V1_8, Opcodes.ACC_PUBLIC, "class" + i, null, "java/lang/Object", null);
                // 生成类返回字节码的byte数组
                byte[] code = cw.toByteArray();
                // 类加载器，执行类加载
                test.defineClass("Class" + i, code, 0, code.length);
            }
        } finally {
            System.out.println(j);
        }
    }
}
```

![image-20221102115834283](D:\WorkSpace\Note\Picture\image-20221102115834283.png)

### 运行时常量池

* 常量池，就是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数类型、字面量等信息
* 运行时常量池，常量池是*.class文件中的，当该类被加载，他的常量池信息将会放入运行时常量池，并把里面的符号改为真实地址

```java
public class Demo1_10 {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
```

![image-20221102120839358](D:\WorkSpace\Note\Picture\image-20221102120839358.png)

二进制字节码文件包含了（类基本信息，常量池，类方法定义，包含了虚拟机指令）

### StringTable

* 常量池中的字符串仅是符号，第一次用到时才变为对象
* 利用串池的机制，来避免重复创建字符串对象
* 字符串拼接的原理是StringBuilder（1.8）
* 字符串常量拼接的原理是编译器优化
* 可以使用intern方法，主动将串池中还没有的字符串对象放入串池

```java
public class Demo1_11 {
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
    }
}
```

![image-20221102121933978](D:\WorkSpace\Note\Picture\image-20221102121933978.png)

常量池中的信息，都会被加载到运行时常量池中，这时a、b、ab都是常量池中的符号，还没有变为java字符串对象

ldc #2 就会把a符号变为“a“字符串对象，会从StringTable中，找是否存在”a”字符串对象，如果没有那么久创建一个“a”字符串对象放入StringTable中（HashTable结构，不能扩容），懒惰创建

```java
public class Demo1_11 {
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = s1 + s2; // new StringBuilder().append("a").append("b").toString();	new String("ab");
    }
}
```

![image-20221102122400890](D:\WorkSpace\Note\Picture\image-20221102122400890.png)

```java
//que1
System.out.println(s3 == s4); // false
```

```java
public class Demo1_11 {
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";

        String s5 = "a" + "b"; // javac在编译器的优化，结果已经在编译期间确定为ab
    }
}
```

"a" + "b"会在StringTable中找有没有"ab"如果有，直接使用，如果没有就会新创建一个，并放入StringTable中

```java
// que2
System.out.println(s3 == s5); // true
```

使用intern将未进入串池的字符串常量对象手动放入，如果有则不会放入，如果没有则放入串池中，会把串池中的对象返回

```java
public class Demo1_12 {
    public static void main(String[] args) {
        String s = new String("a") + new String("b");
        String s2 = s.intern();
        System.out.println(s2 == "ab");

        String s3 = new String("a") + new String("b");
        String s4 = s3.intern();
        System.out.println(s3 == "ab");
    }
}
```

#### StringTable位置

```java
public class Demo1_13 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        int i = 0;
        try {
            for (int j = 0; j < 260000; ++j) {
                list.add(String.valueOf(j).intern());
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
```

![image-20221102125733650](D:\WorkSpace\Note\Picture\image-20221102125733650.png)

#### StringTable垃圾回收机制

![image-20221103132159620](D:\WorkSpace\Note\Picture\image-20221103132159620.png)

```java
public class Demo1_14 {
    public static void main(String[] args) {
        int i = 0;
        try {
            for (int j = 0; j < 100; ++j) {
                String.valueOf(j).intern();
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
```

![image-20221103132235157](D:\WorkSpace\Note\Picture\image-20221103132235157.png)

```java
public class Demo1_14 {
    public static void main(String[] args) {
        int i = 0;
        try {
            for (int j = 0; j < 10000; ++j) {
                String.valueOf(j).intern();
                i++;
            }
        } catch (Throwable e) {
            e.printStackTrace();
        } finally {
            System.out.println(i);
        }
    }
}
```

![image-20221103132424442](D:\WorkSpace\Note\Picture\image-20221103132424442.png)

**发生垃圾回收**

#### StringTable性能调优

* 调整-XX:StringTableSize=桶个数
* 考虑字符串对象是否入池

## 直接内存

Direct Memory直接内存

* 常见于NIO操作时，用于数据缓冲区
* 分配回收成本较高，但读写性能高
* 不受JVM内存回收管理

```java
public class Demo1_15 {
    static final String FROM = "C:\\Users\\Mnsx_x\\Videos\\Captures\\Vue App - Google Chrome 2022-10-27 13-55-07.mp4";
    static final String TO = "D:\\WorkSpace\\Temp\\test.mp4";
    static final int _1Mb = 1024 * 1024;

    public static void main(String[] args) {
        io();
        directBuffer();
    }

    private static void directBuffer() {
        long start = System.nanoTime();
        try (
                FileChannel from = new FileInputStream(FROM).getChannel();
                FileChannel to = new FileOutputStream(TO).getChannel();
        ) {
            ByteBuffer buffer = ByteBuffer.allocateDirect(_1Mb);
            while (true) {
                int len = from.read(buffer);
                if (len == -1) {
                    break;
                }
                buffer.flip();
                to.write(buffer);
                buffer.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("---nio cost" + (end - start) / 1000_000.0);
    }

    private static void io() {
        long start = System.nanoTime();
        try (
            FileInputStream from = new FileInputStream(FROM);
            FileOutputStream to = new FileOutputStream(TO);
        ) {
            byte[] buf = new byte[_1Mb];
            while (true) {
                int len = from.read(buf);
                if (len == -1) {
                    break;
                }
                to.write(buf, 0, len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        long end = System.nanoTime();
        System.out.println("---bio cost" + (end - start) / 1000_000.0);
    }
}
```

使用直接内存空间，速度比使用JVM内存更快

![image-20221103135649826](D:\WorkSpace\Note\Picture\image-20221103135649826.png)

### 直接内存溢出

```java
public class Demo1_16 {
    static int _100Mb = 1024 * 1024 * 100;

    public static void main(String[] args) {
        List<ByteBuffer> list = new ArrayList<>();
        int i = 0;
        try {
            while (true) {
                ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_100Mb);
                list.add(byteBuffer);
                i++;
            }
        } finally {
            System.out.println(i);
        }
    }
}
```

![image-20221103135604021](D:\WorkSpace\Note\Picture\image-20221103135604021.png)

### 直接内存释放原理

* 使用了Unsafe对象完成直接内存的分配回收，并且回收需要主动调用freeMemory方法
* ByteBuffer的实现类内部，使用了Cleaner（虚引用）来检测ByteBuffer对象，一旦ByteBuffer对象被垃圾回收，那么就会由ReferenceHandler线程通过Cleaner的clean方法调用freeMemory来释放直接内存

```java
public class Demo1_17 {
    static int _1GB = 1024 * 1024 * 1024;

    public static void main(String[] args) throws IOException {
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1GB);
        System.out.println("分配完毕");
        System.in.read();
        System.out.println("开始释放");
        byteBuffer = null;
        System.gc();
        System.in.read();
    }
}
```

并非使用JVM的gc进行回收

```java
public class Demo1_18 {
    static int _1GB = 1024 * 1024 * 1024;

    public static void main(String[] args) throws IOException {
        Unsafe unsafe = getUnsafe();
        long base = unsafe.allocateMemory(_1GB);
        unsafe.setMemory(base, _1GB, (byte)0);
        System.in.read();

        unsafe.freeMemory(base);
        System.in.read();
    }

    public static Unsafe getUnsafe() {
        try {
            Field f = Unsafe.class.getDeclaredField("theUnsafe");
            f.setAccessible(true);
            Unsafe unsafe = (Unsafe) f.get(null);
            return unsafe;
        } catch (NoSuchFieldException | IllegalAccessException e) {
            throw new RuntimeException();
        }
    }
}
```

而是使用unsafe类对直接内存进行控制

```java
DirectByteBuffer(int cap) {                   // package-private
    super(-1, 0, cap, cap);
    boolean pa = VM.isDirectMemoryPageAligned();
    int ps = Bits.pageSize();
    long size = Math.max(1L, (long)cap + (pa ? ps : 0));
    Bits.reserveMemory(size, cap);

    long base = 0;
    try {
        base = unsafe.allocateMemory(size);
    } catch (OutOfMemoryError x) {
        Bits.unreserveMemory(size, cap);
        throw x;
    }
    unsafe.setMemory(base, size, (byte) 0);
    if (pa && (base % ps != 0)) {
        // Round up to page boundary
        address = base + ps - (base & (ps - 1));
    } else {
        address = base;
    }
    cleaner = Cleaner.create(this, new Deallocator(base, size, cap));
    att = null;
}
```

调用`base = unsafe.allocateMemory(size);`来创建直接内存区域

创建一个`cleaner = Cleaner.create(this, new Deallocator(base, size, cap));`回调对象

```java
private static class Deallocator
    implements Runnable
{

    private static Unsafe unsafe = Unsafe.getUnsafe();

    private long address;
    private long size;
    private int capacity;

    private Deallocator(long address, long size, int capacity) {
        assert (address != 0);
        this.address = address;
        this.size = size;
        this.capacity = capacity;
    }

    public void run() {
        if (address == 0) {
            // Paranoia
            return;
        }
        unsafe.freeMemory(address);
        address = 0;
        Bits.unreserveMemory(size, capacity);
    }

}
```

调用`unsafe.freeMemory(address);`来回收直接内存

```java
public void clean() {
    if (remove(this)) {
        try {
            this.thunk.run();
        } catch (final Throwable var2) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    if (System.err != null) {
                        (new Error("Cleaner terminated abnormally", var2)).printStackTrace();
                    }

                    System.exit(1);
                    return null;
                }
            });
        }

    }
}
```

Cleaner对象的clean方法中会调用run方法，来进行空间回收

```java
public class Cleaner extends PhantomReference<Object> {
    private static final ReferenceQueue<Object> dummyQueue = new ReferenceQueue();
```

Cleaner是一个虚引用，后台有一个ReferenceHandler的线程来检测虚引用对象，虚引用对象关联的实际对象被回收时，就会调用clean方法

> 禁用显示的垃圾回收对直接内存的影响
>
> ```java
> // -XX:+DisableExplicitGC 禁用显示的垃圾回收
> public class Demo1_19 {
>     static int _1GB = 1024 * 1024 * 1024;
> 
>     public static void main(String[] args) throws IOException {
>         ByteBuffer byteBuffer = ByteBuffer.allocateDirect(_1GB);
>         System.out.println("分配完毕");
>         System.in.read();
>         System.out.println("完成释放");
>         byteBuffer = null;
>         System.gc(); // 显示垃圾回收，FULL GC，影响性能
>         System.in.read();
>     }
> }
> ```
>
> 直接内存没有被回收，只有等到内存不够用时，触发垃圾回收，才会被回收
>
> 选择直接使用`unsafe.freeMemory(base)`

# 垃圾回收

## 如何判断对象可以被回收

### 引用计数法

一个对象被引用时，计数+1，不被引用了就-1，如果为零就没有被引用了，那么就回收

弊端——**循环引用**，内存泄漏

### 可达性分析算法

* Java虚拟机中的垃圾回收器采用可达性分析来探索所有存活的对象
* 扫描堆中的对象，看是否能够沿着GC Root对象为起点的引用链找到该对象，找不到，表示可以回收
* GC Root

### 四种引用

* 强引用

  只有所以头GC Roots对象都不通过【强引用】引用该对象时，该对象才能被垃圾回收

* 软引用

  仅有软引用引用该对象时，在垃圾回收后，内存仍不足时会再次发出垃圾回收，回收软引用对象

  可以配合引用队列来释放软引用自身

* 弱引用

  仅有弱引用引用该对象时，在垃圾回收时，无论内存是否充足，都会回收弱引用对象

  可以配合引用队列来释放引用自身

* 虚引用

  必须配合引用队列使用，主要配合ByteBuffer使用，被引用对象回收时，会将虚引用入队，有ReferenceHandler线程调用虚引用相关方法释放直接内存

* 终结器引用

  无需手动编码，但其内部配合引用队列使用，在垃圾回收时，终结器引用入队（被引用的对象暂时没有被回收），再由Finalizer线程通过终结器引用对象并调用它的finalize方法，第二次GC时才能回收被引用对象

```java
// 软引用的使用，内存敏感
public class Demo2_2 {
    static final int _4MB = 1024 * 1024 * 4;

    public static void main(String[] args) {
        List<SoftReference<byte[]>> list = new ArrayList<>();
        for (int i = 0; i < 5; ++i) {
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB]);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());
        }
        System.out.println("循环结束: " + list.size());
        for (SoftReference<byte[]> ref : list) {
            System.out.println(ref.get());
        }
    }
}
```

```java
// 进行软引用的清理
public class Demo2_3 {
    static final int _4MB = 1024 * 1024 * 4;

    public static void main(String[] args) {
        List<SoftReference<byte[]>> list = new ArrayList<>();

        ReferenceQueue<byte[]> queue = new ReferenceQueue<>();

        for (int i = 0; i < 5; ++i) {
            // 将软引用关联引用队列
            SoftReference<byte[]> ref = new SoftReference<>(new byte[_4MB], queue);
            System.out.println(ref.get());
            list.add(ref);
            System.out.println(list.size());
        }

        Reference<? extends byte[]> poll = queue.poll();
        while (poll != null) {
            list.remove(poll);
            poll = queue.poll();
        }

        System.out.println();

        for (SoftReference<byte[]> ref : list) {
            System.out.println(ref.get());
        }
    }
}
```

## 垃圾回收算法

### 标记清除

定义：Mark Sweep

* 速度较快
* 会造成内存碎片

![image-20221104122752891](D:\WorkSpace\Note\Picture\image-20221104122752891.png)

### 标记整理

定义：Mark Compact

* 速度慢
* 没有内存碎片

![image-20221104122831069](D:\WorkSpace\Note\Picture\image-20221104122831069.png)

### 复制

定义：Copy

* 不会有内存碎片
* 需要占用双倍内存空间

![image-20221104122947662](D:\WorkSpace\Note\Picture\image-20221104122947662.png)

## 分代回收

![image-20221105134848681](D:\WorkSpace\Note\Picture\image-20221105134848681.png)

* 对象首先分配在伊甸园区域
* 新生代空间不够时，触发minor gc，伊甸园和幸存区from存货的对象使用copy复制到to中，存活的对象年龄加1并且交换from、to
* minor gc会引发stop the world，停止其他用户线程，让垃圾回收线程进行完垃圾回收后恢复
* 当对象寿命超过阈值时，就会晋升至老年代，最大寿命为15（4bit）
* 当老年代空间不足，会先进行一次minor gc，如果空间仍不足，那么会触发一次full gc，STW时间更长

### 相关VM参数

| 含义               | 参数                                                        |
| ------------------ | ----------------------------------------------------------- |
| 堆初始大小         | -Xms                                                        |
| 堆最大大小         | -Xmx\|-XX:MaxHeapSize=[size]                                |
| 新生代大小         | -Xmn\|-XX:NewSize=[size] + -XX:MaxNewSize=[size]            |
| 幸存区比例（动态） | -XX:InitialSurvivorRatio=[ratio] -XX:+UseAdaptiveSizePolicy |
| 幸存区比例         | -XX:ServivorRatio=[ratio]                                   |
| 晋升阈值           | -XX:MaxTenuringTheshold=[threshold]                         |
| 晋升详情           | -XX:+PrintTenuringDistribution                              |
| GC详情             | -XX:+PrintGCDetails -verbose:gc                             |
| FullGC前MinorGC    | -XX:+ScavengeBeforeFullGC                                   |

### 代码分析

```java
public class Demo2_5 {
    private static final int _512KB = 512 * 1024;
    private static final int _1MB = 1024 * 1024;
    private static final int _6MB = 1024 * 1024 * 6;
    private static final int _7MB = 1024 * 1024 * 7;
    private static final int _8MB = 1024 * 1024 * 8;

    public static void main(String[] args) {
        List<byte[]> list = new ArrayList<>();

        list.add(new byte[_7MB]);
        list.add(new byte[_512KB]);
        list.add(new byte[_512KB]);
    }
}
```

![image-20221105142756429](D:\WorkSpace\Note\Picture\image-20221105142756429.png)

> * 大对象直接晋升
>
> ```java
> public class Demo2_5 {
>     private static final int _512KB = 512 * 1024;
>     private static final int _1MB = 1024 * 1024;
>     private static final int _6MB = 1024 * 1024 * 6;
>     private static final int _7MB = 1024 * 1024 * 7;
>     private static final int _8MB = 1024 * 1024 * 8;
> 
>     public static void main(String[] args) {
>         List<byte[]> list = new ArrayList<>();
> 
>         list.add(new byte[_8MB]);
>     }
> }
> ```
>
> ![image-20221105142959281](D:\WorkSpace\Note\Picture\image-20221105142959281.png)
>
> **老年代空间足够，新生代空间不够的情况直接晋升，不会触发gc**
>
> * 一个线程的OOM不会导致整个程序异常退出
>
> ```java
> public class Demo2_5 {
>     private static final int _512KB = 512 * 1024;
>     private static final int _1MB = 1024 * 1024;
>     private static final int _7MB = 1024 * 1024 * 7;
>     private static final int _8MB = 1024 * 1024 * 8;
> 
>     public static void main(String[] args) throws InterruptedException {
>         new Thread(() -> {
>             List<byte[]> list = new ArrayList<>();
>             list.add(new byte[_8MB]);
>             list.add(new byte[_8MB]);
>         }).start();
> 
>         System.out.println("sleep...");
>         Thread.sleep(1000L);
>     }
> }
> ```

## 垃圾回收器

* 串行垃圾回收器

  * 单线程

  * 堆内存较小，适合个人电脑

* 吞吐量优先

  * 多线程

  * 堆内存较大，多核CPU

  * 让单位时间内，STW的时间最短

* 响应时间优先

  * 多线程

  * 堆内存较大，多核CPU

  * 尽可能让STW的单次时间最短

### 串行

`-XX:+UseSerialGC = Serial + SerialOld`

![image-20221105144049862](D:\WorkSpace\Note\Picture\image-20221105144049862.png)

### 吞吐量优先

```txt
-XX:+UseParallelGC ~ -XX:+UseParallelOldGC
-XX:+useAdaptiveSizePolicy
-XX:GCTimeRatio=[ratio]
-XX:MaxGCPauseMillis=[ms]
-XX:ParallelGCThreads=[n]
```

![image-20221105144810439](D:\WorkSpace\Note\Picture\image-20221105144810439.png)

### 响应时间优先

```txt
-XX:+UseConcMarkSweepGC ~ -XX:+UseParNewGc ~ SerialOld
-XX:ParallelGCThreads=n ~ -XX:ConcGCThreads=[threads]
-XX:CMSInitiatingOccupancyFraction=[percent]
-XX:+CMSScavengeBeforeRemark
```

![image-20221105145902523](D:\WorkSpace\Note\Picture\image-20221105145902523.png)

## Garbage First

适用场景——

* 同时注重吞吐量和低延迟，默认的暂停目标是200ms
* 超大顿内存，会将对划分成多个大小相同的Region
* 整体上是标记+整理算法，两个区域之间是复制算法

相关JVM参数——

* -XX:+UseG1GC
* -XX:G1HeapRegionSize=[size]
* -XX:MaxGCPauseMillis=[time]

### G1垃圾回收阶段

![image-20221105151739097](D:\WorkSpace\Note\Picture\image-20221105151739097.png)

#### Young Collection

![image-20221105152049201](D:\WorkSpace\Note\Picture\image-20221105152049201.png)

#### Young Collection + CM

* 在Young GC时会进行GC Root的初始标记

* 老年代占用对空间比例达到阈值时，进行并发标记（不会STW），由下面的JVM参数决定

  `-XX:InitialtingHeapOccupancyPercent=[percent]（默认45%）`

![image-20221105152257327](D:\WorkSpace\Note\Picture\image-20221105152257327.png)

#### Mixed Collection

会对E、S、O进行全面垃圾回收

* 最终标记（Remark）会STW
* 拷贝存活（Evacuation）会STW

`-XX:MaxGCPauseMillis=ms`

**根据时间有选择的进行拷贝存活**

![image-20221105152608297](D:\WorkSpace\Note\Picture\image-20221105152608297.png)

### Young Collection跨代引用

* 新生代回收的跨代引用问题

![image-20221106094953457](D:\WorkSpace\Note\Picture\image-20221106094953457.png)

* 卡表与Remenbered Set
* 在引用变更时通过post-write barrier + dirty card queue
* concurrent refinement threads更新Remembered Set

![image-20221106095149646](D:\WorkSpace\Note\Picture\image-20221106095149646.png)

### Remark

* pre-write barrier + satb_mark_queue

![image-20221106095306937](D:\WorkSpace\Note\Picture\image-20221106095306937.png)

### G1垃圾回收器的优化

#### JDK 8u20字符串去重

* 优点：节省大量内存
* 缺点：略微多占用了CPU时间，新生代回收时间略微增多

+XX:+UseStringDeduplication

```java
String s1 = new String("hello"); // char[]{'h', 'e', 'l', 'l', 'o'}
String s2 = new String("hello"); // char[]{'h', 'e', 'l', 'l', 'o'}
```

* 当所有新分配的字符串放到一个队列
* 当新生代回收时，G1并发检查是否有字符串重复
* 如果它们的值一样，让他们引用同一个char[]
* 与String.intern()不一样
  * String.intern()关注的是字符串对象
  * 而字符串去重关注的是char[]
  * 在JVM内部，使用了不同的字符串表

#### JDK 8u40并发标记类卸载

所有对象都经过并发标记后，就能知道那些类不再被使用，当一个类加载器的所有类都不再使用，则卸载它所加载的所有类

-XX:+ClassUnloadingWithConcurrentMark默认启用

#### JDK 8u60回收巨型对象

* 一个对象大于region的一半时，称之为巨型对象
* G1不会对巨型对象进行拷贝
* 回收时被优先考虑
* G1会跟踪老年代所有incoming引用，这样老年代incoming引用为0的巨型对象就可以再新生代垃圾回收时处理掉

## 垃圾回收调优

### 调优领域

* 内存
* 锁竞争
* cpu占用
* io

### 确定目标

* 【低延迟】还是【高吞吐量】，选择合适的回收器
* GMS，G1，ZGC
* ParallelGC

### 最快的GC是不发生GC

* 查看FullGC前后的内存占用，考虑以下问题
  * 数据是不是太多
  * 数据表示是否太臃肿
    * 对象图
    * 对象大小
  * 是否存在内存泄漏

### 新生代调优

新生代的特点

* 所有的new操作的内存分配非常廉价
  * TLAB thread-local allocation buffer
* 死亡对象的回收代价是零
* 大部分对象用过即死
* Minor GC的时间远远低于Full GC

`-Xmn`**设置新生代的空间大小**

**新生代需要为栈的25%-50%，新生代能够容纳所有[并发量*（请求-响应）]的数据**

**幸存区大到能够保留[当前活跃的对象+需要晋升对象]**

**晋升阈值配置得当，让长时间存活对象尽快晋升**

```java
-XX:MaxTenuringThreshold=[threshold]
-XX:+PrintTenuringDistribution
```

### 老年代调优

以CMS为例——

* CMS的老年代内存越大越好
* 先尝试不做调优，如果没有FullGC，那么已经满足，否则现场时调优新生代
* 观察发生Full GC时老年代内存占用，将老年代内存阈值调大1/4~1/3
  * `-XX:CMSInitiatingOccupancyFraction=percent`

# 类加载

![image-20221107080609215](D:\WorkSpace\Note\Picture\image-20221107080609215.png)

## 类文件结构

```java
public class Demo3_1 {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
```

执行`javac -parameters -d . HelloWorld.java`

```txt
[root@localhost ~]# od -t xC HelloWorld.class
0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 09
0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b 07
0000040 00 1c 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29
0000060 56 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e
0000100 75 6d 62 65 72 54 61 62 6c 65 01 00 12 4c 6f 63
0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 01
0000140 00 04 74 68 69 73 01 00 1d 4c 63 6e 2f 69 74 63
0000160 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f
0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e 01 00 16
0000220 28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72
0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 01 00 13
0000260 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69
0000300 6e 67 3b 01 00 10 4d 65 74 68 6f 64 50 61 72 61
0000320 6d 65 74 65 72 73 01 00 0a 53 6f 75 72 63 65 46
0000340 69 6c 65 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64
0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d 0c 00 1e
0000400 00 1f 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64
0000420 07 00 20 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74
0000440 63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c
0000460 6f 57 6f 72 6c 64 01 00 10 6a 61 76 61 2f 6c 61
0000500 6e 67 2f 4f 62 6a 65 63 74 01 00 10 6a 61 76 61
0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d 01 00 03 6f
0000540 75 74 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72
0000560 69 6e 74 53 74 72 65 61 6d 3b 01 00 13 6a 61 76
0000600 61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d
0000620 01 00 07 70 72 69 6e 74 6c 6e 01 00 15 28 4c 6a
0000640 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b
0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01
0000700 00 07 00 08 00 01 00 09 00 00 00 2f 00 01 00 01
0000720 00 00 00 05 2a b7 00 01 b1 00 00 00 02 00 0a 00
0000740 00 00 06 00 01 00 00 00 04 00 0b 00 00 00 0c 00
0000760 01 00 00 00 05 00 0c 00 0d 00 00 00 09 00 0e 00
0001000 0f 00 02 00 09 00 00 00 37 00 02 00 01 00 00 00
0001020 09 b2 00 02 12 03 b6 00 04 b1 00 00 00 02 00 0a
0001040 00 00 00 0a 00 02 00 00 00 06 00 08 00 07 00 0b
0001060 00 00 00 0c 00 01 00 00 00 09 00 10 00 11 00 00
0001100 00 12 00 00 00 05 01 00 10 00 00 00 01 00 13 00
0001120 00 00 02 00 14
```

**根据JVM规范，类文件结构如下**

![image-20221107081420195](D:\WorkSpace\Note\Picture\image-20221107081420195.png)

### 魔数

0-3字节，表示他是否是【class】类型的文件

0000000 **ca fe ba be** 00 00 00 34 00 23 0a 00 06 00 15 09

`cafebabe`表示是一个Java字节码文件

### 版本

4-7字节，表示类的版本00 34 （52） 表示是Java8

0000000 cafe babe **00 00 00 35** 00 23 0a 00 06 00 15 09

### 常量池

8-9字节，表示常量池长度00 23 （35） 表示常量池由#1-#34项，注意#0不计入，也没有值

0000000 ca fe ba be 00 00 00 34 **00 23** 0a 00 06 00 15 09

![image-20221107082618221](D:\WorkSpace\Note\Picture\image-20221107082618221.png)

第#1项

0a 表示一个Mehtod信息，00 06和00 15（21）表示它引用了常量池中#6和#21项来获得这个方法的【所属类】【方法名】

0000000 ca fe ba be 00 00 00 34 00 23 **0a 00 06 00 15** 09

第#2项 

09 表示一个 Field 信息，00 16（22）和 00 17（23） 表示它引用了常量池中 #22 和 # 23 项

来获得这个成员变量的【所属类】和【成员变量名】

0000000 ca fe ba be 00 00 00 34 00 23 0a 00 06 00 15 **09**

0000020 **00 16 00 17** 08 00 18 0a 00 19 00 1a 07 00 1b 07

第#3项 

08 表示一个字符串常量名称，00 18（24）表示它引用了常量池中 #24 项

0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b 07

第#4项 

0a 表示一个 Method 信息，00 19（25） 和 00 1a（26） 表示它引用了常量池中 #25 和 #26

项来获得这个方法的【所属类】和【方法名】

0000020 00 16 00 17 **08 00 18** 0a 00 19 00 1a 07 00 1b 07

第#5项 

07 表示一个 Class 信息，00 1b（27） 表示它引用了常量池中 #27 项

0000020 00 16 00 17 08 00 18 0a 00 19 00 1a **07 00 1b** 07

第#6项 

07 表示一个 Class 信息，00 1c（28） 表示它引用了常量池中 #28 项

0000020 00 16 00 17 08 00 18 0a 00 19 00 1a 07 00 1b **07**

0000040 **00 1c** 01 00 06 3c 69 6e 69 74 3e 01 00 03 28 29

第#7项 

01 表示一个 utf8 串，00 06 表示长度，3c 69 6e 69 74 3e 是【 <init> 】

0000040 00 1c **01 00 06 3c 69 6e 69 74 3e** 01 00 03 28 29

第#8项 

01 表示一个 utf8 串，00 03 表示长度，28 29 56 是【()V】其实就是表示无参、无返回值

0000040 00 1c 01 00 06 3c 69 6e 69 74 3e **01 00 03 28 29**

0000060 **56** 01 00 04 43 6f 64 65 01 00 0f 4c 69 6e 65 4e

第#9项 

01 表示一个 utf8 串，00 04 表示长度，43 6f 64 65 是【Code】

0000060 56 **01 00 04 43 6f 64 65** 01 00 0f 4c 69 6e 65 4e

第#10项 

01 表示一个 utf8 串，00 0f（15） 表示长度，4c 69 6e 65 4e 75 6d 62 65 72 54 61 62 6c 65

是【LineNumberTable】

0000060 56 01 00 04 43 6f 64 65 **01 00 0f 4c 69 6e 65 4e**

0000100 **75 6d 62 65 72 54 61 62 6c 65** 01 00 12 4c 6f 63

第#11项 

01 表示一个 utf8 串，00 12（18） 表示长度，4c 6f 63 61 6c 56 61 72 69 61 62 6c 65 54 61

62 6c 65是【LocalVariableTable】

0000100 75 6d 62 65 72 54 61 62 6c 65 **01 00 12 4c 6f 63**

0000120 **61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65** 01

第#12项 

01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【this】

0000120 61 6c 56 61 72 69 61 62 6c 65 54 61 62 6c 65 **01**

0000140 **00 04 74 68 69 73** 01 00 1d 4c 63 6e 2f 69 74 63

第#13项 

01 表示一个 utf8 串，00 1d（29） 表示长度，是【Lcn/itcast/jvm/t5/HelloWorld;】

0000140 00 04 74 68 69 73 **01 00 1d 4c 63 6e 2f 69 74 63**

0000160 **61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c 6f**

0000200 **57 6f 72 6c 64 3b** 01 00 04 6d 61 69 6e 01 00 16

第#14项 

01 表示一个 utf8 串，00 04 表示长度，74 68 69 73 是【main】

0000200 57 6f 72 6c 64 3b **01 00 04 6d 61 69 6e** 01 00 16

第#15项 

01 表示一个 utf8 串，00 16（22） 表示长度，是【([Ljava/lang/String;)V】其实就是参数为

字符串数组，无返回值

0000200 57 6f 72 6c 64 3b 01 00 04 6d 61 69 6e **01 00 16**

0000220 **28 5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72**

0000240 **69 6e 67 3b 29 56** 01 00 04 61 72 67 73 01 00 13

第#16项

01 表示一个 utf8 串，00 04 表示长度，是【args】

0000240 69 6e 67 3b 29 56 **01 00 04 61 72 67 73** 01 00 13

第#17项 

01 表示一个 utf8 串，00 13（19） 表示长度，是【[Ljava/lang/String;】

0000240 69 6e 67 3b 29 56 01 00 04 61 72 67 73 **01 00 13**

0000260 **5b 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69**

0000300 **6e 67 3b** 01 00 10 4d 65 74 68 6f 64 50 61 72 61

第#18项 

01 表示一个 utf8 串，00 10（16） 表示长度，是【MethodParameters】

0000300 6e 67 3b **01 00 10 4d 65 74 68 6f 64 50 61 72 61**

0000320 **6d 65 74 65 72 73** 01 00 0a 53 6f 75 72 63 65 46

第#19项 

01 表示一个 utf8 串，00 0a（10） 表示长度，是【SourceFile】

0000320 6d 65 74 65 72 73 **01 00 0a 53 6f 75 72 63 65 46**

0000340 **69 6c 65** 01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64

第#20项 

01 表示一个 utf8 串，00 0f（15） 表示长度，是【HelloWorld.java】

0000340 69 6c 65 **01 00 0f 48 65 6c 6c 6f 57 6f 72 6c 64**

0000360 **2e 6a 61 76 61** 0c 00 07 00 08 07 00 1d 0c 00 1e

第#21项 

0c 表示一个 【名+类型】，00 07 00 08 引用了常量池中 #7 #8 两项

0000360 2e 6a 61 76 61 **0c 00 07 00 08** 07 00 1d 0c 00 1e

第#22项 

07 表示一个 Class 信息，00 1d（29） 引用了常量池中 #29 项

0000360 2e 6a 61 76 61 0c 00 07 00 08 **07 00 1d** 0c 00 1e

第#23项 

0c 表示一个 【名+类型】，00 1e（30） 00 1f （31）引用了常量池中 #30 #31 两项

0000360 2e 6a 61 76 61 0c 00 07 00 08 07 00 1d **0c 00 1e**

0000400 **00 1f** 01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64

第#24项 

01 表示一个 utf8 串，00 0f（15） 表示长度，是【hello world】

0000400 00 1f **01 00 0b 68 65 6c 6c 6f 20 77 6f 72 6c 64**

第#25项 

07 表示一个 Class 信息，00 20（32） 引用了常量池中 #32 项

0000420 **07 00 20** 0c 00 21 00 22 01 00 1b 63 6e 2f 69 74

第#26项 

0c 表示一个 【名+类型】，00 21（33） 00 22（34）引用了常量池中 #33 #34 两项

0000420 07 00 20 **0c 00 21 00 22** 01 00 1b 63 6e 2f 69 74

第#27项 

01 表示一个 utf8 串，00 1b（27） 表示长度，是【cn/itcast/jvm/t5/HelloWorld】

0000420 07 00 20 0c 00 21 00 22 **01 00 1b 63 6e 2f 69 74**

0000440 **63 61 73 74 2f 6a 76 6d 2f 74 35 2f 48 65 6c 6c**

0000460 **6f 57 6f 72 6c 64** 01 00 10 6a 61 76 61 2f 6c 61

第#28项 

01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/Object】

0000460 6f 57 6f 72 6c 64 **01 00 10 6a 61 76 61 2f 6c 61**

0000500 **6e 67 2f 4f 62 6a 65 63 74** 01 00 10 6a 61 76 61

第#29项 

01 表示一个 utf8 串，00 10（16） 表示长度，是【java/lang/System】

0000500 6e 67 2f 4f 62 6a 65 63 74 **01 00 10 6a 61 76 61**

0000520 **2f 6c 61 6e 67 2f 53 79 73 74 65 6d** 01 00 03 6f

第#30项 

01 表示一个 utf8 串，00 03 表示长度，是【out】

0000520 2f 6c 61 6e 67 2f 53 79 73 74 65 6d **01 00 03 6f**

0000540 **75 74** 01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72

第#31项 

01 表示一个 utf8 串，00 15（21） 表示长度，是【Ljava/io/PrintStream;】

0000540 75 74 **01 00 15 4c 6a 61 76 61 2f 69 6f 2f 50 72**

0000560 **69 6e 74 53 74 72 65 61 6d 3b 01 00 13 6a 61 76**

第#32项 

01 表示一个 utf8 串，00 13（19） 表示长度，是【java/io/PrintStream】

0000560 69 6e 74 53 74 72 65 61 6d 3b **01 00 13 6a 61 76**

0000600 **61 2f 69 6f 2f 50 72 69 6e 74 53 74 72 65 61 6d**

第#33项 

01 表示一个 utf8 串，00 07 表示长度，是【println】

0000620 **01 00 07 70 72 69 6e 74 6c 6e** 01 00 15 28 4c 6a

第#34项 

01 表示一个 utf8 串，00 15（21） 表示长度，是【(Ljava/lang/String;)V】

0000620 01 00 07 70 72 69 6e 74 6c 6e **01 00 15 28 4c 6a**

0000640 **61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b**

0000660 **29 56** 00 21 00 05 00 06 00 00 00 00 00 02 00 01

### 访问标识与继承关系

![image-20221107090355424](D:\WorkSpace\Note\Picture\image-20221107090355424.png)

21 表示该 class 是一个类，公共的

0000660 29 56 **00 21** 00 05 00 06 00 00 00 00 00 02 00 01

05 表示根据常量池中 #5 找到本类全限定名

0000660 29 56 00 21 **00 05** 00 06 00 00 00 00 00 02 00 01

06 表示根据常量池中 #6 找到父类全限定名

0000660 29 56 00 21 00 05 **00 06** 00 00 00 00 00 02 00 01

表示接口的数量，本类为 0

0000660 29 56 00 21 00 05 00 06 **00 00** 00 00 00 02 00 01

### 变量信息

表示成员变量数量，本类为 0

0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01

![image-20221107090558368](D:\WorkSpace\Note\Picture\image-20221107090558368.png)

### 方法信息

表示方法数量，本类为 2

0000660 29 56 00 21 00 05 00 06 00 00 00 00 00 02 00 01

一个方法由 访问修饰符，名称，参数描述，方法属性数量，方法属性组成

* 红色代表访问修饰符（本类中是 public）

* 蓝色代表引用了常量池 #07 项作为方法名称

* 绿色代表引用了常量池 #08 项作为方法参数描述

* 黄色代表方法属性数量，本方法是 1

* 红色代表方法属性

  * 00 09 表示引用了常量池 #09 项，发现是【Code】属性

  * 00 00 00 2f 表示此属性的长度是 47

  * 00 01 表示【操作数栈】最大深度

  * 00 01 表示【局部变量表】最大槽（slot）数00 00 00 05 表示字节码长度，本例是 5

  * 2a b7 00 01 b1 是字节码指令

  * 00 00 00 02 表示方法细节属性数量，本例是 2

  * 00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性

    * 00 00 00 06 表示此属性的总长度，本例是 6

    * 00 01 表示【LineNumberTable】长度

    * 00 00 表示【字节码】行号 00 04 表示【java 源码】行号

  * 00 0b 表示引用了常量池 #11 项，发现是【LocalVariableTable】属性

    * 00 00 00 0c 表示此属性的总长度，本例是 12

    * 00 01 表示【LocalVariableTable】长度

    * 00 00 表示局部变量生命周期开始，相对于字节码的偏移量

    * 00 05 表示局部变量覆盖的范围长度

    * 00 0c 表示局部变量名称，本例引用了常量池 #12 项，是【this】

    * 00 0d 表示局部变量的类型，本例引用了常量池 #13 项，是【Lcn/itcast/jvm/t5/HelloWorld;】

    * 00 00 表示局部变量占有的槽位（slot）编号，本例是 0

![image-20221107090845695](D:\WorkSpace\Note\Picture\image-20221107090845695.png)

* 红色代表访问修饰符（本类中是 public static）

* 蓝色代表引用了常量池 #14 项作为方法名称

* 绿色代表引用了常量池 #15 项作为方法参数描述

* 黄色代表方法属性数量，本方法是 2

* 红色代表方法属性（属性1）

  * 00 09 表示引用了常量池 #09 项，发现是【Code】属性

  * 00 00 00 37 表示此属性的长度是 55

  * 00 02 表示【操作数栈】最大深度

  * 00 01 表示【局部变量表】最大槽（slot）数

  * 00 00 00 05 表示字节码长度，本例是 9

  * b2 00 02 12 03 b6 00 04 b1 是字节码指令

  * 00 00 00 02 表示方法细节属性数量，本例是 2

  * 00 0a 表示引用了常量池 #10 项，发现是【LineNumberTable】属性

    * 00 00 00 0a 表示此属性的总长度，本例是 10

    * 00 02 表示【LineNumberTable】长度

    * 00 00 表示【字节码】行号 00 06 表示【java 源码】行号

    * 00 08 表示【字节码】行号 00 07 表示【java 源码】行号

* 00 0b 表示引用了常量池 #11 项，发现是【LocalVariableTable】属性

  * 00 00 00 0c 表示此属性的总长度，本例是 12

  * 00 01 表示【LocalVariableTable】长度00 00 表示局部变量生命周期开始，相对于字节码的偏移量

  * 00 09 表示局部变量覆盖的范围长度

  * 00 10 表示局部变量名称，本例引用了常量池 #16 项，是【args】

  * 00 11 表示局部变量的类型，本例引用了常量池 #17 项，是【[Ljava/lang/String;】

  * 00 00 表示局部变量占有的槽位（slot）编号，本例是 0

![image-20221107092634972](D:\WorkSpace\Note\Picture\image-20221107092634972.png)

* 红色代表方法属性（属性2）

  * 00 12 表示引用了常量池 #18 项，发现是【MethodParameters】属性

    * 00 00 00 05 表示此属性的总长度，本例是 5

    * 01 参数数量

    * 00 10 表示引用了常量池 #16 项，是【args】

    * 00 00 访问修饰符

![image-20221107092847194](D:\WorkSpace\Note\Picture\image-20221107092847194.png)

### 附加属性

* 00 01 表示附加属性数量
* 00 13 表示引用了常量池#19项，即【SourceFile】
* 00 00 00 02 表示此属性的长度
* 00 14 表示引用了常量池#20项，即【HelloWorld.java】

![image-20221107093026354](D:\WorkSpace\Note\Picture\image-20221107093026354.png)

## 字节码指令

### init

`public top.mnsx.jvm.Helloworld();`构造方法的字节码指令

2a b7 00 01 b1

1. 2a => aload_0 加载slot 0的局部变量，即this，作为下面的invokespecial构造函数调用的参数
2. b7 => invokespecial 预备调用构造方法
3. 00 01 引用常量池中#1项，即【Method java/lang/Object."\<init>":()V】
4. b1 表示返回

### main

`public static void main(java.lang.String[]);`主方法的字节码指令

0b 00 02 12 03 b6 00 04 b1

1. b2 => getstatic用来加载静态变量
2. 00 02 引用常量池中#2项，即【Field java/lang/System.out:Ljava/io/PrintStream;】
3. 12 => ldc 加载参数
4. 03 引用常量池中的#3项，即【String hello world】
5. b6 => invokevirtual预备调用成员方法
6. 00 04 引用常量池中#4项，即【Mehtod Java/io/PrintStream.print:(Ljava/lang/String;)V】
7. b1 表示返回

## javap工具

```java
// javap -v xxx.class

Classfile /D:/WorkSpace/Temp/target/classes/top/mnsx/temp/Demo3_1.class
Last modified 2022-11-7; size 552 bytes
MD5 checksum 3abc8aba526822f1425ad614467e9ca5
Compiled from "Demo3_1.java"
public class top.mnsx.temp.Demo3_1
minor version: 0
major version: 52
flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
#1 = Methodref          #6.#20         // java/lang/Object."<init>":()V
#2 = Fieldref           #21.#22        // java/lang/System.out:Ljava/io/PrintStream;
#3 = String             #23            // hello world
#4 = Methodref          #24.#25        // java/io/PrintStream.println:(Ljava/lang/String;)V
#5 = Class              #26            // top/mnsx/temp/Demo3_1
#6 = Class              #27            // java/lang/Object
#7 = Utf8               <init>
#8 = Utf8               ()V
#9 = Utf8               Code
#10 = Utf8               LineNumberTable
#11 = Utf8               LocalVariableTable
#12 = Utf8               this
#13 = Utf8               Ltop/mnsx/temp/Demo3_1;
#14 = Utf8               main
#15 = Utf8               ([Ljava/lang/String;)V
#16 = Utf8               args
#17 = Utf8               [Ljava/lang/String;
#18 = Utf8               SourceFile
#19 = Utf8               Demo3_1.java
#20 = NameAndType        #7:#8          // "<init>":()V
#21 = Class              #28            // java/lang/System
#22 = NameAndType        #29:#30        // out:Ljava/io/PrintStream;
#23 = Utf8               hello world
#24 = Class              #31            // java/io/PrintStream
#25 = NameAndType        #32:#33        // println:(Ljava/lang/String;)V
#26 = Utf8               top/mnsx/temp/Demo3_1
#27 = Utf8               java/lang/Object
#28 = Utf8               java/lang/System
#29 = Utf8               out
#30 = Utf8               Ljava/io/PrintStream;
#31 = Utf8               java/io/PrintStream
#32 = Utf8               println
#33 = Utf8               (Ljava/lang/String;)V
{
public top.mnsx.temp.Demo3_1();
descriptor: ()V
flags: ACC_PUBLIC
Code:
stack=1, locals=1, args_size=1
0: aload_0
1: invokespecial #1                  // Method java/lang/Object."<init>":()V
4: return
LineNumberTable:
line 9: 0
LocalVariableTable:
Start  Length  Slot  Name   Signature
0       5     0  this   Ltop/mnsx/temp/Demo3_1;

public static void main(java.lang.String[]);
descriptor: ([Ljava/lang/String;)V
flags: ACC_PUBLIC, ACC_STATIC
Code:
stack=2, locals=1, args_size=1
0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
3: ldc           #3                  // String hello world
5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
8: return
LineNumberTable:
line 11: 0
line 12: 8
LocalVariableTable:
Start  Length  Slot  Name   Signature
0       9     0  args   [Ljava/lang/String;
}
SourceFile: "Demo3_1.java"
```

## 图解方法执行流程

### Java代码

```java
public class Demo3_1 {
    public static void main(String[] args) {
        int a = 10;
        int b = Short.MAX_VALUE + 1;
        int c = a + b;
        System.out.println(c);
    }
}
```

### 字节码文件

```java
Classfile /D:/WorkSpace/Temp/target/classes/top/mnsx/temp/Demo3_1.class
  Last modified 2022-11-7; size 611 bytes
  MD5 checksum fbcdc0301a58a87bfcb56a2ea51c5114
  Compiled from "Demo3_1.java"
public class top.mnsx.temp.Demo3_1
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#25         // java/lang/Object."<init>":()V
   #2 = Class              #26            // java/lang/Short
   #3 = Integer            32768
   #4 = Fieldref           #27.#28        // java/lang/System.out:Ljava/io/PrintStream;
   #5 = Methodref          #29.#30        // java/io/PrintStream.println:(I)V
   #6 = Class              #31            // top/mnsx/temp/Demo3_1
   #7 = Class              #32            // java/lang/Object
   #8 = Utf8               <init>
   #9 = Utf8               ()V
  #10 = Utf8               Code
  #11 = Utf8               LineNumberTable
  #12 = Utf8               LocalVariableTable
  #13 = Utf8               this
  #14 = Utf8               Ltop/mnsx/temp/Demo3_1;
  #15 = Utf8               main
  #16 = Utf8               ([Ljava/lang/String;)V
  #17 = Utf8               args
  #18 = Utf8               [Ljava/lang/String;
  #19 = Utf8               a
  #20 = Utf8               I
  #21 = Utf8               b
  #22 = Utf8               c
  #23 = Utf8               SourceFile
  #24 = Utf8               Demo3_1.java
  #25 = NameAndType        #8:#9          // "<init>":()V
  #26 = Utf8               java/lang/Short
  #27 = Class              #33            // java/lang/System
  #28 = NameAndType        #34:#35        // out:Ljava/io/PrintStream;
  #29 = Class              #36            // java/io/PrintStream
  #30 = NameAndType        #37:#38        // println:(I)V
  #31 = Utf8               top/mnsx/temp/Demo3_1
  #32 = Utf8               java/lang/Object
  #33 = Utf8               java/lang/System
  #34 = Utf8               out
  #35 = Utf8               Ljava/io/PrintStream;
  #36 = Utf8               java/io/PrintStream
  #37 = Utf8               println
  #38 = Utf8               (I)V
{
  public top.mnsx.temp.Demo3_1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 9: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Ltop/mnsx/temp/Demo3_1;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=4, args_size=1
         0: bipush        10
         2: istore_1
         3: ldc           #3                  // int 32768
         5: istore_2
         6: iload_1
         7: iload_2
         8: iadd
         9: istore_3
        10: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
        13: iload_3
        14: invokevirtual #5                  // Method java/io/PrintStream.println:(I)V
        17: return
      LineNumberTable:
        line 11: 0
        line 12: 3
        line 13: 6
        line 14: 10
        line 15: 17
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      18     0  args   [Ljava/lang/String;
            3      15     1     a   I
            6      12     2     b   I
           10       8     3     c   I
}
SourceFile: "Demo3_1.java"
```

### 常量池载入运行时常量池

![image-20221107094655124](D:\WorkSpace\Note\Picture\image-20221107094655124.png)

### 方法字节码载入方法区

![image-20221107095337045](D:\WorkSpace\Note\Picture\image-20221107095337045.png)

### main线程开始运行，分配栈帧内存

![image-20221107095450939](D:\WorkSpace\Note\Picture\image-20221107095450939.png)

### 执行引擎开始执行字节码

#### bipush 10

* 将一个byte压入操作数栈（其长度会补齐4个字节）
* sipush将一个short压入操作数栈（其长度会补齐4个字节）
* ldc将一个int压入操作数栈
* ldc2_w将一个long压入操作数栈（分两次压入，因为long是8个字节）
* 这里小的数字都是和字节码指令存在一起，超过short范围的数字存入了常量池

![image-20221107095707535](D:\WorkSpace\Note\Picture\image-20221107095707535.png)

#### istore_1

* 将操作数栈顶数据弹出，存入局部变量表的slot 1

![image-20221107100048029](D:\WorkSpace\Note\Picture\image-20221107100048029.png)

![image-20221107100136380](D:\WorkSpace\Note\Picture\image-20221107100136380.png)

#### ldc #3

* 从常量池加载#3数据到操作数栈
* **注意**Short.MAX_VALUE是32767，所以32767 = Shrot.MAX_VALUE + 1实际是在编译期间计算好的

![image-20221107100328295](D:\WorkSpace\Note\Picture\image-20221107100328295.png)

#### istore_2

![image-20221107100432387](D:\WorkSpace\Note\Picture\image-20221107100432387.png)

![image-20221107100450364](D:\WorkSpace\Note\Picture\image-20221107100450364.png)

#### iload_1

![image-20221107100528773](D:\WorkSpace\Note\Picture\image-20221107100528773.png)

#### iload_2

![image-20221107100559330](D:\WorkSpace\Note\Picture\image-20221107100559330.png)

#### iadd

![image-20221107100639197](D:\WorkSpace\Note\Picture\image-20221107100639197.png)

![image-20221107100659024](D:\WorkSpace\Note\Picture\image-20221107100659024.png)

#### istore_3

![image-20221107100717726](D:\WorkSpace\Note\Picture\image-20221107100717726.png)

![image-20221107100731086](D:\WorkSpace\Note\Picture\image-20221107100731086.png)

#### getstatic #4

![image-20221107100757897](D:\WorkSpace\Note\Picture\image-20221107100757897.png)

![image-20221107100838378](D:\WorkSpace\Note\Picture\image-20221107100838378.png)

#### iload_3

![image-20221107100853576](D:\WorkSpace\Note\Picture\image-20221107100853576.png)

![image-20221107100905916](D:\WorkSpace\Note\Picture\image-20221107100905916.png)

#### invokeVirtual #5

* 找到常量池#5项
* 定位到方法区java/io/PrintStream.println:(I)V方法
* 生成新的栈帧（分配locals、stack等）
* 传递参数，执行新栈帧中的字节码

![image-20221107101024264](D:\WorkSpace\Note\Picture\image-20221107101024264.png)

* 执行完毕，弹出栈帧
* 清除main操作数栈内容

![image-20221107101111757](D:\WorkSpace\Note\Picture\image-20221107101111757.png)

#### return

* 完成main方法调用，弹出main栈帧
* 程序结束

## 条件、循环判断指令

![image-20221107212252509](D:\WorkSpace\Note\Picture\image-20221107212252509.png)

* byte，short，char都会按int比较，因为操作数栈都是4字节
* goto用来进行跳转指定行号的字节码

## 构造方法

### \<cinit>()V

编译器会按从上至下的顺序，收集所有static静态代码块和静态成员赋值的代码，合并为一个特殊的方法\<cinit>()V

### \<init>()V

编译器会按从上至下的顺序，收集所有{}代码块和成员变量赋值的代码，形成新的构造方法，但原始构造方法内的代码总是再最后

## 方法调用

* new 是创建【对象】，给对象分配堆内存，执行成功会将【对象引用】压入操作数栈

* dup 是赋值操作数栈栈顶的内容，本例即为【对象引用】，为什么需要两份引用呢，一个是要配合 invokespecial 调用该对象的构造方法 "<init>":()V （会消耗掉栈顶一个引用），另一个要配合 astore_1 赋值给局部变量

* 最终方法（fifinal），私有方法（private），构造方法都是由 invokespecial 指令来调用，属于静态绑定

* 普通成员方法是由 invokevirtual 调用，属于动态绑定，即支持多态

* 成员方法与静态方法调用的另一个区别是，执行方法前是否需要【对象引用】

* 比较有意思的是 d.test4(); 是通过【对象引用】调用一个静态方法，可以看到在调用invokestatic 之前执行了 pop 指令，把【对象引用】从操作数栈弹掉了

* 还有一个执行 invokespecial 的情况是通过 super 调用父类方法

## 多态原理

当执行invokevirtual指令时

1. 先通过栈帧中的对象引用找到对象
2. 分析对象头，找到对象的实际Class
3. Class结构中由vtable，它再类加载的连接阶段就已经根据方法的重构规则生成好了
4. 查表得到方法的具体地址
5. 执行方法的字节码

