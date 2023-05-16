# Java基础

## Java语言有哪些特点？

1. 面向对象（封装、继承、多态）
2. 平台无关性（Java虚拟机实现）
3. 支持多线程
4. 支持网络编程并且很方便
5. 编译与解释并存

## JVM？JDK？JRE？

JVM（Java Virtual Machine）是运行Java字节码的虚拟机，JVM有针对不同系统的特定实现，目的是使用相同的字节码，它们都会给出相同的结果，JVM是实现平台无关性的关键

JDK（Java Development kit）是功能齐全的Java SDK，它拥有JRE拥有的一切，还有编译器（javac）和工具

JRE（Java Runtime Environment）是Java运行时环境，它能够运行已编译的Java程序

## 什么是字节码？采用字节码的好处是什么？

JVM可以理解的代码叫做字节码（即扩展名为.class的文件），他不面向任何特定的处理器，只面向虚拟机

1. Java通过字节码的方式，在一定程度上解决了传统解释性语言执行效率低的问题，保留了解释性语言可移植的特点
2. 由于字节码并不针对一种特定的机器，因此，Java程序无需重新编译便可在多种不同操作系统的计算机上运行

> **.class文件->机器码**
>
> JVM类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度相对比较慢，而且有些方法和代码块是经常需要被调用的
>
> 引入JIT编译器，而JIT属于运行时编译，当JIT编译器完成第一次编译后，其回将字节码对应的机器码保存下来，下次可以直接使用
>
> > HotSpot采用了惰性评估的做法，根据二八定律，消耗大部分系统资源的只有那一小部分的代码，而这也就是JIT所需要编译的部分
> >
> > JVM回根据代码每次被执行的情况收集信息并相应地做出一些优化，因此执行的次数越多，它的速度越快
> >
> > JDK9引入了一种新的编译模式AOT，它是直接将字节码编译成机器码，这样避免JIT预热等各方面的开销

## 为什么不全部使用AOT呢？

这和Java的动态特性有关系，比如说，CGLIB动态代理使用的是ASM技术，而这种技术原理是运行时直接在内存中生成并加载修改后的字节码文件也就是.class文件，如果全部使用AOT提前编译，那么就不能使用ASM技术

## 为什么说Java语言“编译与解释并存”？

因为Java语言既具有编译型语言的特征也具有解释型语言的特征

Java程序要先经过编译，后解释两个阶段（Java编写的程序需要先经过编译步骤，生成字节码文件，然后由Java解释器来解释执行）

## 标识符和关键字的区别是什么？

标识符就是程序、类、变量、方法等的名字

有一些标识符，Java语言已经赋予了其特殊的含义，只能用在特定的地方，这些标识符就是关键字

## 移位运算符能够被那些类型使用？

由于double、float在二进制中的表现比较特殊，因此不能来进行移位操作

移位操作符实际上支持的类型只有int和long，编译器在堆short、byte、char类型进行移位钱，都会将其转换为int类型再操作

## 成员变量和局部变量的区别？

* 语法形式：
  
  * 成员变量属于类的，而局部变量是在代码块或方法中定义的变量或是方法的参数
  
  * 成员变量可以被public、private、static等修饰符修饰，而局部变量不能被访问控制修饰符及static所修饰
  * 但是成员变量和局部变量都能被final所修饰
* 存储方式：
  * 成员变量如果使用static所修饰，那么这个成员变量是属于类的，否则属于实例，存在于堆内存，局部变量则存在于栈内存
* 生存时间：
  * 成员变量是对象的一部分，它随着对象的创建而存在，而局部变量随着方法的调用而自动生成，随着方法的调用结束而消亡
* 默认值：
  * 成员变量如果没有被赋初始值，则回自动以类型的默认值而赋值（被final修饰的变量必须显式的赋值），而局部变量则不会自动赋值

## 静态变量有什么作用？

静态变量可以被类的所有实例共享，所有对象共享一份静态变量

通常情况下，静态变量会被final关键字修饰成为常量

## 字符型常量和字符串常量的区别？

* 形式：字符常量是单引号引起的一个字符，字符串常量是双引号引起的0个或若干个字符
* 含义：字符常量相当于一个整型值（ASCII值），可以参加表达式运算；字符串常量代表一个地址值
* 占内存大小：字符常量只占2个字节，字符串常量占若干个字节

## 静态方法为什么不能调用非静态成员？

1. 静态方法是属于类的，在内加载的时候就会分配内存，可以通过类名直接访问。而非静态成员属于实例对象，只有在对象实例化之后才存在，需要通过类的实例对象去访问
2. 在类的非静态成员不存在的时候静态成员就已经存在了，此时调用在内存中还不存在的非静态成员，属于非法操作

## 静态方法和实例方法有何不同？

* 调用方式：
  * 在外部调用静态方法时，可以使用类名.方法名的方式，也可以使用对象.方法名的方式，而实例方法只有后面这种方式（**调用静态方法可以无需创建对象**）
* 访问类成员是否存在限制：
  * 静态方法在访问本类的成员时，只允许访问静态成员（即静态成员变量和静态方法），不允许访问实例成员，而实例方法不存在这个限制

## 重载和重写有什么区别？

* 重载

  发生在同一个类中（或者父类和子类之间），方法名必须相同，参数类型不同、个数不同、顺序不同，方法返回和访问修饰符可以不同

* 重写

  重写发生在运行期，是子类对父类的允许访问的方法的实现过程进行重新编写

  1. 方法名、参数列表必须相同，子类方法返回值类型应比父类方法返回值类型更小或相等，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类
  2. 如果父类方法访问修饰符为private/final/static则子类不能重写该方法，但是被static修饰的方法能够被在此声明

## 基本类型和包装类型的区别？

* 成员变量包装类型不赋值就是null，而基本类型由默认值且不是null
* 包装类型可用于泛型，而基本类型不可以
* 基本数据类型的局部变量存放在Java虚拟机栈中的局部变量表中，基本数据类型的成员变量（未被static修饰）存放在Java虚拟机的堆中
* 相比于对象类型，基本数据类型占用的空间非常小

## 为什么存在某些对象实例不存在堆中？

这是因为HotSpot虚拟机引入了JIT优化之后，会对对象进行逃逸分析，如果发现某一个对象并没有逃逸到方法外部，那么就可能通过标量替换来实现栈上分配，而避免堆上分配内存

## 基本数据类型都存放在栈中吗？

不是，基本数据类型的成员变量如果没有被static修饰的话（不建议使用），就存放在堆中

## 包装类型的缓存机制了解么？

Java基本类型的包装类型的大部分都用到了缓存机制来提升性能

Byte、Short、Integer、Long这4种包装类默认创建了数值[-128 - 127]的相应类型的缓存数据

Character创建了数值在[0, 127]范围的缓存数据

Boolean直接返回True or False

## 自动装箱和拆箱了解吗？

* 装箱：将基本类型用它们对应的引用类型包装起来
* 拆箱：将包装类型转换为基本数据类型

```java
Integer i = 10 等价于 Integer i = Integer.vlaueOf(10)
int n = i 等价于 int n = i.intValue()
```

## 自动拆箱可能引发NPE问题，遇到过类似场景吗？

> 数据库的查询结果可能是null，因为自动拆箱，用基本数据类型接收有NPE风险

```java
public class AutoBoxTest {
    public static void main(String[] args) {
        long id = getNum();
    }
    
    public static Long getNum() {
        return null;
    }
}
```

因为如果包装类是NULL的话，拆箱机制的本质就是待用.longVlaue()方法，所以会出现NPE问题

三目运算符使用不当也会导致NPE异常

```java
public class Main() {
    public static void main(String[] args) {
        Integer i = null;
        Boolean flag = false;
        System.out.println(flag ? 0 : i);
    }
}
```

0是基本数据类型int，返回数据的时候会被强制拆箱成int类型，由于i的值是null因此会抛出NPE异常

**正确写法**

```java
System.out.println(flag ? new Integer(0) : i);
```

## 为什么浮点数运算的时候会有精度丢失的风险？

```java
float a = 2.0f - 1.9f;
flaot b = 1.8f - 1.7f;
a != b
```

计算机是二进制的，而且计算机表示一个数字时，宽度是有限的，无限循环的小数存储时将会被截断

## 构造器有哪些特点？

* 名字与类名相同，
* 没有返回值，但不能用void声明构造函数
* 生成类的对象时自动执行，无需调用

## 继承有什么特点？

1. 子类拥有父类所有的属性和方法（包括私有属性和私有方法），但是父类种的私有属性和方法子类无法访问
2. 子类可以拥有自己的属性和方法，即子类可以对父类进行扩展
3. 子类可以用自己的方式实现父类的方法

## 多态有什么特点？

1. 对象类型和引用类型之间具有继承/实现的关系
2. 引用类型变量发出方法调用的到底时哪个类中的方法，必须在程序运行期间才能确定
3. 多态不能调用“只在子类存在但在父类不能在”的方法
4. 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法

## Object类的常见方法有那些？

```java 
/**
 * native 方法，用于返回当前运行时对象的 Class 对象，使用了 final 关键字修饰，故不允许子类重写。
 */
public final native Class<?> getClass()
/**
 * native 方法，用于返回对象的哈希码，主要使用在哈希表中，比如 JDK 中的HashMap。
 */
public native int hashCode()
/**
 * 用于比较 2 个对象的内存地址是否相等，String 类对该方法进行了重写以用于比较字符串的值是否相等。
 */
public boolean equals(Object obj)
/**
 * naitive 方法，用于创建并返回当前对象的一份拷贝。
 */
protected native Object clone() throws CloneNotSupportedException
/**
 * 返回类的名字实例的哈希码的 16 进制的字符串。建议 Object 所有的子类都重写这个方法。
 */
public String toString()
/**
 * native 方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。
 */
public final native void notify()
/**
 * native 方法，并且不能重写。跟 notify 一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。
 */
public final native void notifyAll()
/**
 * native方法，并且不能重写。暂停线程的执行。注意：sleep 方法没有释放锁，而 wait 方法释放了锁 ，timeout 是等待时间。
 */
public final native void wait(long timeout) throws InterruptedException
/**
 * 多了 nanos 参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上 nanos 毫秒。。
 */
public final void wait(long timeout, int nanos) throws InterruptedException
/**
 * 跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念
 */
public final void wait() throws InterruptedException
/**
 * 实例被垃圾回收器回收的时候触发的操作
 */
protected void finalize() throws Throwable { }
```

## ==和equals()的区别？

* 对于基本数据类型来说，==比较的是值
* 对于引用数据类型来说，==比较的是对象的内存地址

euqals()方法存在两种使用情况：

* 类没有重写equals()方法：通过equals()比较该类的两个对象时，等价于通过"=="比较这两个对象，使用的默认是Object类equals()方法
* 类重写了equals()方法：一般我们都重写equals()方法比较两个对象中的属性是否相同

## hashCode()有什么用？

hashCode()的作用是获取哈希码，作用是确定该对象在哈希表中的索引位置

## 为什么JDK要同时提供两个方法呢？

因为在一些容器中，有了hashCode()之后，判断元素是否在对应容器中的效率会更高

如果容器中有多个hashCode相同的对象，再使用equals来判断是否真的相同，大大缩小了查找成本

## 为什么不只提供hashCode()方法？

因为两个对象的hashCode值相等并不代表这两个对象就相等

因为hashCode()所使用的哈希算法也许刚好会让多个对象传回相同的哈希值

## 为什么重写equals()时必须重写hashCode()方法？

因为两个相等的对象的hashCode值必须是相同的，也就是如果equals方法判断两个对象相等，那么两个对象的hashCode值一定相等

## String、StringBuffer、StringBuilder的区别？

* 可变性

  String是不可变的

  StringBuilder与StringStringBuffer都继承自AbstractStringBuilder类，再AbstractStringBuilder中也是使用字符数组保存字符串，不过没有使用final和private关键字修饰，最关键的是AbstractStringBuilder提供了修改字符串的方法

* 线程安全性

  String中的对象是不可变的，也就可以理解为常量，线程安全

  AbstractStringBuilder是StringBuilder和StringBuffer的公共父类，定义了一些字符串的基本操作

  StringBuffer对方法加了同步锁或者调用的方法加了同步锁，所以是线程安全的

  StringBuilder并没有对方法进行加同步锁，所以是非线程安全的

* 性能

  每次对String进行改变的时候，都会生成一个新的String对象，然后将指针指向新的String对象

  StringBuffer每次都会对StringBuffer对象进行操作，而不是生成新的对象并改变对象引用

  相同情况下使用StringBuilder相比使用StringBuffer仅能获得10%-15%左右的性能提升，但多线程不安全

## String为什么是不可变的？

* final关键字修饰的类不能被继承，修饰的方法不能被重写，修饰的变量是基本数据类型则值不能改变，修饰的变量是引用类型则不能再指向其他对象，final关键字修饰的数组保存字符串并不是String不可变的根本原因
  * 保存字符串的数组被final修饰且为私有的，并且String类没有提供/暴露修改这个字符串的方法
  * String类被final修饰导致其不能被继承，进而避免了子类破坏String不可变

## 字符串拼接用”+“还是StringBuilder？

字符串对象通过”+“的字符串拼接方式，实际上是通过StringBuilder调用append()方法是实现的，拼接完成之后调用toString()得到一个String对象

但是再循环内使用"+"进行字符串的拼接的话，存在比较明显的缺陷：编译器不会创建单个StringBuilder以复用，会导致创建过多的StringBuilder对象

## 字符串常量池的作用了解吗？

字符串常量池是JVM为了提升性能和减少内存消耗针对字符串专门开辟的一块区域，主要目的是为了避免字符串的重复创建

## intern方法有什么作用？

String.intern()是一个native方法，起作用是将指定的字符串对象的引用保存在字符串常量池中，可以简单的分为两种情况

* 如果字符串常量池中保存了对应的字符串对象的引用，就直接返回该引用
* 就再常量池中创建一个指向该字符串对象引用并返回

## 了解常量折叠吗？

对于编译器可以确定值的字符串，也就是常量字符串，JVM会将其存入字符串常量池，并且，字符串常量拼接得到的字符串常量在编译阶段已经被存放字符串常量池，这个得益于编译器的优化

* 被声明为final
* 基本类型或者字符串类型
* 声明时已经初始化
* 使用常量表达式进行初始化

## 了解String对于null的处理吗？

```java
public class Test1 {
    private static String s1;
    private static String s2;
    
    public static void main(String[] args) {
        String s = s1 + s2;
        System.out.println(s);
    }
}
```

运行结果为nullnull

PrintStream类源码中print方法对null进行了处理

```java
public void print(String s) {
    if (s == null) {
        s = "null";
    }
    write(s);
}
```

因此，一个null字符串就可以被打印在控制台上

s1和s2都是没有经过初始化的空对象，编译器会对String字符串相加进行优化，会把这一过程转化为StringBuilder的append方法

```java
public AbstractStringBuilder append(String str) {
    if (str == null) {
        return appendNull();
        ...
    }
}
```

如果append方法参数为null，那么就会调用父类的appendNull方法

```java
private AbstractStringBuilder appendNull() {
    int c = count;
    ensureCapacityInternal(c + 4);
    final char[] value = this.value;
    value[c++] = 'n';
    value[c++] = 'u';
    value[c++] = 'l';
    value[c++] = 'l';
    count = c;
    return this;
}
```

在append的过程中如果碰到的是null字符串，那么会以null的形式被添加道字符数组中

## 创建了几个对象？

首先需要了解常量池——

* class文件常量池：在class文件中保存一份常量池，主要存储编译时确定的数据，包括代码中的字面量和符号引用
* 运行时常量池：位于方法区中，全局共享，class文件常量池中的内容会在类加载后存放到方法区的运行时常量池中。除此之外，在运行期间可以将新的变量方法入运行时常量池中，相对于class文件常量池而言运行时常量池更具有动态性
* 字符串常量池：位于堆中，全局共享，这里可以先粗略的认为它存储的是String对象的直接引用，而不是直接存放对象，具体的实例对象是在堆中存放

字符串常量池StringTable本质上是一个HashTable，实际上字符串常量池HashTable采用的是数组加链表的结构，链表上的节点是一个一个的HashTableEntry，而HashTableEntry中的vlaue则存储了堆上的String对象的引用

那么字符串对象的引用是什么时候被放到字符串常量池中的？

* 使用字面量声明String对象的时候，也就是被双引号包围的字符串，在堆上创建对象，并驻留道字符串常量池中
* 调用intern()方法，当字符串常量池中没有相等的字符串时，会保存该字符串的引用

> 就HotSpot VM的实现而言，记载类时字符串字面量会进入到运行时常量池，不会进入全局的字符串常量池，即在StringTable中并没有相应的引用，在堆中也没有对应的对象产生

```java
public static void main(String[] args) {
    String s = "Hydra";
}
```

```shell
public static void main(java.lang.String[]);
	descriptor: ([Ljava/lang/String;)V
	flags: ACC_PUBLIC, ACC_STATIC
	Code:
		stack=1, locals=2, args_size=1
			0: ldc		      #2                    // String Hydra
			2: astore_1
			3: return
```

ldc，查找索引为#2对应的项，#2表示常量在常量池中的位置，这个过程中，会触发lazy resolve，在resolve过程如果发现StringTable已经有了内容匹配的String引用，则直接返回这个引用，反之则会在堆中创建一个对应的String对象然后在StringTable驻留这个对象引用，并返回这个引用，之后在压入操作数栈中

```java
public static void main(String[] args) {
    String s = new String("Hydra");
}
```

```shell
public static void main(java.lang.String[]);
	descriptor: ([Ljava/lang/String;)V
	flags: ACC_PUBLIC, ACC_STATIC
	Code:
		stack=3, locals=2, args_size=1
			0: new		      #2                    // class java/lang/String
			3: dup
			4: ldc   		  #3					// String Hydra
			6: invokespecial  #4					
			9: astore_1
			10: return
```

new，在堆中创建一个String对象，并将它的引用压入操作数栈，注意这时的对象还只是一个空壳，并没有调用类的构造方法进行初始化

dup，赋值栈顶元素，也就是赋值了上面对象的引用，并将复制后的对象压入栈顶

ldc，在堆中创建字符串对象，驻留到字符串常量池，并将字符串引用压入操作数栈

invokespecial，执行String的构造方法，这一步执行完成后得到一个完整对象

**如果字符串常量池已经驻留过某个字符串引用，在使用构造方法创建String时，只创建了一个对象**

## Exception和Error有什么区别？

* Exception：程序本身可以处理的异常，可以通过catch来进行捕获，Exception可以分为CheckedException和UncheckedException
* Eroor：Error属于程序无法处理的错误，不建议通过catch来捕获，这些错误发生后，JVM一般会选择线程终止

## CheckedException和UncheckedException有什么区别？

CheckedException受检查异常，Java代码在编译过程中，如果受检查异常没有被catch或者throws关键字处理的话，就没办法通过编译

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于受检查异常

UncheckedException即不受检查异常，Java代码在编译过程中，我们不处理也可以正常通过编译

RuntimeException及其子类都统称为非受检查异常

## 在finally语句块中使用return怎么样？

不要再finally中使用return，当try语句和finally语句中都有return语句时，try语句块中的return语句会被忽略，这是因为try语句中的return返回值会被先保存在一个本地变量中，当执行到finally语句中的return之后，这个本地变量的值就变为finally语句中的return返回值了

## finally中的代码一定会执行吗？

不一定

1. 在运行前虚拟机被终止运行
2. 程序所在的线程死亡
3. 关闭CPU

## 如何使用try-with-resources代替try-catch-finally？

1. 使用范围：任何实现了java.lang.AutoCloseable或者java.io.Closeable的对象
2. 关闭资源和finally块的执行顺序：在try-with-resource语句中，任何catch或finally块在声明的资源关闭后运行

## 何谓注解？

Annotation可以看作一种特殊的注释，主要用于修饰类、方法或者变量，提供某些信息供程序在编译或者运行时使用

## 注解的解析方法有哪几种？

* 编译期直接扫描：编译器在编译Java代码的时候对应的注解并处理
* 运行期通过反射处理：框架中自带的注解都是通过反射来进行处理的

## 什么是序列化和反序列化？

* 序列化：将数据结构或对象转换成二进制字节流的过程
* 反序列化：将再序列化过程中所生成的二进制字节流转换数据结构或对象的过程

**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中**

## 序列化协议对应TCP/IP4层协议中的哪一层？

OSI七层协议模型中，表示层的作用主要是对应用层的用户数据进行处理转换为二进制流

序列化对应OSI七层协议模型中的表示层，而OSI七层协议中的表示层对应的是TCP/IP4层协议中的应用层

## 常见序列化协议有哪些？

* JDK
* Kryo

## serialVersionUID有什么作用？

序列化号serialVersionUID属于版本控制的作用，反序列化时，会检查serialVersionUID是否和当前类的serialVersionUID一致

如果serialVersionUID不一致则抛出InvalidClassException异常

每个序列化类都应该手动指定其serialVersionUID，如果不指定，编译器会动态生成默认的serialVersionUID

## serialVersionUID被static变量修饰为什么可以被序列化？

static修饰变量是静态变量，位于方法区，本身是不会被序列化的，static变量是属于类的而不是对象

反序列化之后，static变量的值就像是默认赋给对象一样

## 如果有些字段不想进行序列化怎么办？

对于不想进行序列化的变量，可以使用transient关键字修饰

transient关键字的作用是：阻止实例中那些关键字修饰的变量序列化

* transient只能修饰变量，不能修饰类与方法
* transient修饰的变量，在反序列化后变量值将会被置成类型的默认值
* static变量不属于任何对象，所以无论有没有transient关键字修饰，均不会被序列化

## 泛型的作用是什么？

使用泛型参数，可以增强代码的可读性以及稳定性

编译器可以对泛型参数进行检测，并且通过泛型参数可以指定传入的对象类型，如果传入其他类型的对象就会报错

并且，原生List返回类型是Object，需要手动转换才能使用，使用泛型后编译器自动转换

* 使用泛型可以在编译器间进行类型检测
* 使用Object类型需要手动添加强制类型转换，降低代码可读性，提高出错概率
* 泛型可以使用自限定类型

## 什么是擦除机制？

Java的泛型是伪泛型，这是因为Java在编译期间，所有的泛型信息都会被擦除，这也是通常所说的类型擦除

编译器会在编译期间动态将泛型T擦除为Object或者T extends xxx擦除为其限定类型xxx

因此，泛型本质其实还是编译器的行为，为了保证引入泛型机制但不创建新的类型减少虚拟机的运行开销，编译器通过擦除将泛型类转换为一般类

## 什么是桥方法

桥方法用于继承泛型类时保证多态

```java
Class Node<T> {
    public T data;
    public Node(T data) {
        this.data = data;
    }
    public void setData(T data) {
        System.out.println("Node.setData");
        this.data = data;
    }
}

Class MyNode extens Node<Integer> {
    public MyNode(Integer data) {
        super(data);
    }
    
    public void setData(Object data) {
        setData((Integer) data);
    }
    
    public void setData(Integer data) {
        System.out.println("MyNode.setData");
        super.setData(data);
    }
}
```

## 什么是通配符？

通配符可以允许类型参数变化，用来解决泛型无法协变的问题

## 通配符？和通常的泛型T之间有什么区别？

* T可以用于声明变量或常量而？不行
* T一般用于声明泛型类或方法，通配符？一般用于泛型方法的调用代码和形参
* T在编译器会被擦除为限定类型或Object，通配符用于捕获具体类型

## 什么是无界通配符？

无界通配符可以接收任何类型数据，用于实现不依赖于具体类型参数的见到那方法，可以捕获参数类型并交由泛型方法进行处理

## List<?>和List有区别吗？

* List<?> list表示list是持有某种特定类型的List，但不知道具体是那种类型，因此添加元素进去会报错
* List list表示list是持有的元素类型为Object，因此可以添加任何类型的对象，只不过编译器有警告信息

## 什么是上边界通配符？什么是下边界通配符？

在使用泛型的时候，可以为传入的泛型类型实参进行上下边界的限制

上边界通配符extends可以实现泛型的向上转型即传入的类型实参必须是指定类型的子类型

下边界通配符super与上边界extends相反，它可以实现泛型的向下转型即传入的类型实参必须是指定类型的夫类型

## ? extends xxx和? super xxx有什么区别？

两者接收参数的范围不同

使用? extends xxx声明的泛型参数只能调用get()方法返回xxx类型，调用set()报错

使用? super xxx声明的泛型参数只能调用set()方法接收xxx类型，调用get()报错

## T extends xxx和? extends xxx有什么区别？

T extends xxx用于定义泛型类和方法，擦除后为xxx类型

? extends xxx用于声明方法形参，接收xxx与其子类型

## Class<?>和Class的区别

直接使用Class的话会有一个类型警告，使用Class<?>则没有，因为Class是一个泛型类，接收原生类型会产生警告

## 何为反射？

通过反射可以获取任意一个类的所有属性和方法，还可以调用这些方法和属性

## 获取Class对象的方式有哪些？

1. 知道具体类的情况下：

   ```java
   Class xxx = TargetObejct.class;
   ```

   **通过此方法后去Class对象不会进行初始化，静态代码块和静态对象不会得到执行**

2. 通过Class.forName()传入类的群路径获取：

   ```java
   Class xxx = Class.forName("top.mnsx.TargetObject");
   ```

3. 通过对象实例instance.getClass()获取：

   ```java
   TargetObject o = new TargetObejct();
   Class xxx = o.getClass();
   ```

4. 通过类加载器xxxClassLaoder.loadClass()传入类路径获取：

   ```java
   ClassLoader.getSystemClassLoader().loadClass("top.mnsx.TargetObejct");
   ```

   **通过此方法后去Class对象不会进行初始化，静态代码块和静态对象不会得到执行**

## 了解那些代理模式实现方法？

* 静态代理

  静态带实现步骤：

  1. 定义一个接口及其实现类
  2. 创建一个代理类同样实现这个接口
  3. 将目标对象注入进代理类，然后再代理类的对应方法调用目标类中的对应方法

  ```java
  public interface SmsService {
      String send(String message);
  }
  ```

  ```java
  public class SmsServiceImpl implements SmsService {
      public String send(String message) {
          System.out.println("send message: " + message);
          return message;
      }
  }
  ```

  ```java
  public class SmsProxy implements SmsService {
      private final SmsService smsService;
      
      public SmsProxy(SmsService smsService) {
          this.smsService = smsService;
      }
      
      @Override
      public String send(String message) {
      	before();
          smsService.send(message);
          after();
          return null;
      }
  }
  ```

  ```java
  public class Main() {
      public static void main(String[] args) {
          SmsService smsService = new SmsServiceImpl();
          SmsProxy smsProxy = new SmsProxy(smsService);
          smsProxy.send("java");
      }
  }
  ```

* JDK动态代理

  使用newProxyInstance()方法生成一个代理对象

  主要参数：

  * loader：类加载器，用于加载代理对象
  * interfaces：被代理类实现的一些接口
  * h：实现了InvocationHandler接口的对象

  InvocationHandler中的invoke()方法的参数：

  * proxy：动态生成的代理类
  * mehtod：与代理类对象调用的方法相对应
  * args：当前method方法的参数

  Jdk动态代理类使用步骤

  1. 定义一个接口及其实现类
  2. 自定义InvocationHandler并重写invoke方法，再invoke方法中我们会调用原生方法并自定义一些逻辑
  3. 通过Proxy.newProxyInstance(...)方法创建代理对象

  ```java
  public interface SmsService {
      String send(String message);
  }
  ```

  ```java
  public class SmsServiceImpl implements SmsService {
      public String send(String message) {
          System.out.println("send message: " + message);
          return message;
      }
  }
  ```

  ```java
  public class DebugInvocationHandler implements Invocationhandler {
      private final Object target;
      
      public DebugInvocationHandelr(Object target) {
          this.target = target;
      }
      
      public Object invoke(Object proxy, Mehtod method, Object[] args) throws InovationTargetException, IllegalAccessException {
          before();
          Object result = mehtod.invoke(target, args);
          after();
          return result;
      }
  }
  ```

  ```java
  public class JdkProxyFactory {
      public static Object getProxy(Object target) {
      	return Proxy.newProxyInstance {
              target.getClass().getClassLoader(),
              target.getClass().getInterfaces(),
              new DebugInvocationHandler(target)
          }
      }
  }
  ```

  ```java
  SmsService smsService = (SmsService) JdkProxyFactory.getProxy(new SmsServiceImpl());
  smsService.send("java");
  ```

* CGLIB动态代理

  需要自定义Method Interceptor并重写interceptor方法，intercept用于拦截增强被代理类的方法

  参数说明：

  * obj：被代理的对象
  * method：被拦截的方法
  * args：方法入参
  * proxy：用于调用原始方法

  CGLIB使用步骤：

  1. 定义一个类;
  2. 自定义MethodInterceptor并重写intercept方法，intercept用于拦截增强被代理类的方法，和Jdk动态代理中的invoke方法类似
  3. 通过Enhancer类的create()创建代理类

  ```xml
  <dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
  </dependency>
  ```

  ```java
  public class SmsServiceImpl implements SmsService {
      public String send(String message) {
          System.out.println("send message: " + message);
          return message;
      }
  }
  ```

  ```java
  public class DewbugMethodInterceptor implements MethodInterceptor {
      @Override
      public Object intercept(Object o, Method method, Obejct[] args, MethodProxy methodProxy) throws Throwalbe {
          before();
          Obejct obejct = methodProxy.invokeSuper(o, args);
          after();
          return object;
      }
  }
  ```

  ```java
  public class CglibProxyFactory {
      public static Object getProxy(Class<?> clazz) {
          Enhancer enhancer = new Enhancer();
          enhancer.setClassLoader(clazz.getClassLoader());
          enhancer.setSuperclass(clazz);
          enhancer.setCallback(new DebugMethodInterceptr());
          return enhancer.create();
      }
  }
  ```

  ```java
  SmsServiceImpl smsService = (SmsServiceImpl) CglibProxyFacotryg.getProxy(SmsServiceImpl.class);
  smsService.send("java");
  ```

# Java集合

## List，Set，Queue，Map四者的区别？

* List：存储的元素是有序的、可重复的
* Set：存储的元素是无序的、不可重复的
* Queue：按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的
* Map：使用键值对存储，key是无序的、不可重复的，value是无序的、可重复的，每一个键最多映射到一个值

## 了解集合的底层数据结构吗？

List

* ArrayList：Object数组
* Vector：Object数组
* LinkedList：双向链表

Set

* HashSet：基于HashMap实现的，底层采用HashMap来保存元素
* LinkedHashMap：LinkeHashSet是HashSet的子类，并且内部是通过LinkedHashMap来实现的
* TreeSet：红黑树

Queue

* PriorityQueue：Object数组来实现二叉堆
* ArrayQueue：Object数组 + 二叉树

Map

* **HashMap**：
  * JDK1.8之前HashMap由数组+链表组成，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的
  * JDK1.8以后，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进性数组扩容，而不是转换为红黑树）时，将链表转换为红黑树
* LinkedHashMap：LinkedHashMap继承自HashMap，所以它的底层仍然时基于拉链式散列结构组成，另外增加了一条双向链表，使得可以保持键值对插入顺序
* Hashtable：数组+链表组成
* TreeMap：红黑树

## ArrayList与LinkedList区别？

* 是否保证线程安全：ArrayList和LinkedList都是不同步的，也就是不保证线程安全
* 底层数据结构：ArrayList底层使用的是Object数组；LinkedList底层使用的是双向链表数据结构
* 插入和删除是否受元素位置的影响
  * ArrayList采用数组存储，所以插入和删除元素的时间复杂度受元素位置影响
  * LinkedList采用链表存储，所以，如果是在头尾插入或者删除元素不受元素位置的影响，时间复杂度为O(1)，如果是要在指定位置i插入和删除元素的话，时间复杂度为O(n)
* 是否支持快速随机访问：LinkedList不支持搞笑的随机元素访问，而ArrayList支持
* 内存空间占用：ArrayList的空间浪费主要体现在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在他的每一个元素都要消耗比ArrayList更多的空间

## compareble和comparator的区别？

* comparable接口实际上是出自java.lang包他有一个compareTo(Object obj)方法来排序
* comparator接口实际上是出自java.util包它有一个compare(Object obj1, Object obj2)方法来排序

## HashSet、LinkedHashSet和TreeSet三者的异同？

* HashSet、LinkedHashSet和TreeSet都是Set接口的实现类，都能保证元素唯一，并且都不是线程安全的
* HashSet、LinkedHashSet和TreeSet的主要区别在于底层数据结构不同
  * HashSet的底层数据结构是哈希表
  * LinkedHashSet的底层数据结构是链表和哈希表
  * TreeSet底层数据结构是红黑树
* 底层数据结构不同又导致这三者的应用场景不同
  * HashSet用于不需要保证元素插入和去除顺序的场景
  * LinkedHashSet用于保证元素的插入和取出顺序满足FIFO
  * TreeSet用于支持对元素自定义排序规则的场景

## ArrayDeque与LinkedList的区别？

* ArrayDeque是基于可变长的数组和双指针来实现，而LinkedList则是通过链表实现
* ArrayDeque不支持存储NULL数据，但LinkedList支持
* ArrayDeque是在JDK1.6才被引入的，而LinkedList在1.2就存在了
* ArrayDeque插入时可能存在扩容过程，不过均摊后的插入操作依然为O(1)，虽然LinkedList不需要扩容，但是每次插入数据需要申请新的堆空间，均摊性能相比更慢

## 了解PriorityQueue吗？

* PriorityQueue利用二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
* PriorityQueue通过对元素的上浮和下沉，实现了在O(logn)的时间复杂度内插入元素和删除堆顶元素
* priorityQueue是非线程安全的，且不支持存储NULL和non-comparable的对象
* priorityQueue默认是小根堆，但可以接收一个Comparator作为构造参数，从而自定义元素优先级的先后

## HashMap和Hashtable的区别？

* 线程是否安全：HashMap是非线程安全的，Hashtable是线程安全的，因为Hashtable的内部方法基本都是经过synchronized修饰
* 效率：因为线程安全的问题，HashMap要比Hashtable效率高一点
* 对Null key和Null value的支持：HashMap可以存储null的key和value，但null作为键只能有一个，null作为值可以有多个，Hashtable不允许有null键和null值，否则抛出NPE异常
* 初始容量大小和每次扩容大小的不同：
  * 创建时如果不指定容量初始值，Hashtable默认的初始大小为11，之后每次扩容，容量变为原来的2n+1。HashMap默认的初始大小为16。之后每次扩容，容量变为之前的两倍
  * 创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小看，也就是说HashMap总是使用2的幂次方作为哈希表的大小
* 底层数据结构：JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转换为红黑树（如果数组长度小于64，那么会优先选择扩充数组），Hashtable没有这种机制

HashMap中带有初始容量的构造函数

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```

保证HashMap总是使用2的幂作为哈希表的大小

```java
/**
     * Returns a power of two size for the given target capacity.
     */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

## HashMap和HashSet的区别？

HashSet底层就是基于HashMap实现的

| HashMap                   | HashSet                                                      |
| ------------------------- | ------------------------------------------------------------ |
| 实现了Map接口             | 实现Set接口                                                  |
| 存储键值对                | 仅存储对象                                                   |
| 调用put()向map中添加元素  | 调用add()方法向Set中添加元素                                 |
| HashMap使用键计算hashCode | HashSet使用成员对象来计算hashCode值，对于两个对象来说hashcode可能相同，所以equals方法用来判断对象的相等性 |

## HashMap和TreeMap的区别？

TreeMap和HashMap都继承自AbstraMap，TreeMap它还是实现了NavigableMap接口和SortedMap接口

实现NavigableMap接口让TreeMap有了对集合内元素的搜索的能力

实现SortedMap接口让TreeMap有了对集合中的元素根据键排序的能力。默认时按key的升序排序，但是可以添加排序的比较器

## HashSet如何检查重复？

在JDK1.8中，HashSet的add()方法只是简单的调用了HashMap的put方法，并且判断了一下返回值以确保是否有重复元素

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

在JDK1.8中，实际上无论HashSet中是否已经存在了某元素怒，HashSet都会直接插入，只是在add方法的返回值处返回是否存在相同元素

## 了解HashMap的底层实现吗？

* JDK1.8之前

  底层是数组和链表结合一起使用也就是链表散列。HashMap通过key的hashcode经过扰动函数处理后得到hsh值，然后通过(n-1) & hash判断当前元素存放的位置，如果当前位置存在元素的话，就判断元素的hash值以及key是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突

  所谓扰动函数值得就是HashMap的hash方法。使用hash方法也就是扰动函数是为了防止一些实现比较差的hashCode()方法换句话说使用扰动函数之后可以减少碰撞

* JDK8

  1.8的hash方法相比于1.7更加简化，但是远离不变

  ```java
  static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
  ```

  1.7hash源码

  ```java
  static int hash(int h) {
      // This function ensures that hashCodes that differ only by
      // constant multiples at each bit position have a bounded
      // number of collisions (approximately 8 at default load factor).
  
      h ^= (h >>> 20) ^ (h >>> 12);
      return h ^ (h >>> 7) ^ (h >>> 4);
  }
  ```

  相比于1.8的hash方法，1.7性能会稍差一点，因为扰动了4次

  1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（当当前数组的长度小于64，那么会先进行数组扩容）时，将链表转换成红黑树

  putVal方法中执行链表转红黑树的判断逻辑

  链表长度大于8时，执行treeifyBin的逻辑

  ```java
  // 遍历链表
  for (int binCount = 0; ; ++binCount) {
      // 遍历到链表最后一个节点
      if ((e = p.next) == null) {
          p.next = newNode(hash, key, value, null);
          // 如果链表元素个数大于等于TREEIFY_THRESHOLD（8）
          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
              // 红黑树转换（并不会直接转换成红黑树）
              treeifyBin(tab, hash);
          break;
      }
      if (e.hash == hash &&
          ((k = e.key) == key || (key != null && key.equals(k))))
          break;
      p = e;
  }
  ```

  treeifyBin方法中判断是否真的转换为红黑树

  ```java
  final void treeifyBin(Node<K,V>[] tab, int hash) {
      int n, index; Node<K,V> e;
      // 判断当前数组的长度是否小于 64
      if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
          // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
          resize();
      else if ((e = tab[index = (n - 1) & hash]) != null) {
          // 否则才将列表转换为红黑树
  
          TreeNode<K,V> hd = null, tl = null;
          do {
              TreeNode<K,V> p = replacementTreeNode(e, null);
              if (tl == null)
                  hd = p;
              else {
                  p.prev = tl;
                  tl.next = p;
              }
              tl = p;
          } while ((e = e.next) != null);
          if ((tab[index] = hd) != null)
              hd.treeify(tab);
      }
  }
  ```

## HashMap的长度为什么是2的幂次方？

  通过(n - 1) & hash来计算数组下标，取余（%）操作中如果余数是2的幂次则等价于与其除数减一的与（&）操作（也就是说hash%length==hash&(length-1)的前提是length是2的n次方）采用二进制操作&，相对于%能够提高运算效率

## 了解HashMap的死循环吗？

* JDK1.7（头插法）

​	主要原因归结于HashMap中的transfer方法，这个方法将旧数组中的元素放入新数组中

```java
do {
    Entry<K,V> next = e.next; // <--假设线程一执行到这里就被调度挂起了
    int i = indexFor(e.hash, newCapacity);
    e.next = newTable[i];
    newTable[i] = e;
    e = next;
} while (e != null);
```

<img src="D:\WorkSpace\Note\Picture\HashMap死循环.png" style="zoom:150%;" />

1. 首先线程一分配e和next指针后被调度挂起，线程二执行完transfer方法
2. 线程一被调度回来执行
   * 先执行nextTable[i] = e，e指向的是新数组中的3，所以线程一的新数组指向3
   * 然后是e = next，于是e指向了7
   * 而下一次循环的next = e.next导致了next指向了3
3. 线程一新数组指向7，继续执行，e指向3，next指向e的下一位
4. e.next = newTable[i]，相当于3的下一位指向新数组中下表为3的元素也就是7，环形链表出现

## HashMap有哪些遍历方式？

1. 迭代器遍历
   * 使用迭代器EntrySet的方式进行遍历
   * 使用迭代器KeySet的方式进行遍历
2. For Each遍历
   * 使用For Each EntrySet的方式进行遍历
   * 使用For Each KeySet的方法进行遍历
3. Lambda表达式
   * 使用Lambda表达式的方式进行遍历
4. Streams API
   * 使用Stream API单线程的方式进行遍历
   * 使用Stream API多线程的方式进行遍历

**entrySet的性能比keySet性能高出一倍**

## 快速失败和安全失败了解吗？

* 快速失败：快速失败是Java集合的一种错误检测机制
  * 再用迭代器遍历一个集合对象时，如果线程A编译过程中线程B对集合对象的内容进行了修改，则会抛出ConcuerrentModificationException
  * 原理：迭代器在遍历时直接访问集合中的内容，并且再白能力过程中使用一个modCount变量。集合在被遍历期间如果内部繁盛变化，就会改变modCount的值，每当迭代器使用hasNext()/next()遍历下一个元素之前，都会检测modCount的值是否为expectedmodCount值，不是的话抛出异常
  * **如果集合发生变化时修改了modCount值刚好又设置为expectmodeCount值，则异常也不会抛出**
* 安全失败
  * 采用安全失败机制的容器，在遍历时不是直接在容器内容上访问的，而是先复制原有集合内容，再拷贝的集合上进行遍历
  * 原理：由于迭代时是对原集合的拷贝进行遍历，所有在遍历过程中对原集合的修改并不能被迭代器检测到，所以不会触发oncurrentModificationException
  * 缺点：迭代器并不能访问到修改后的内容

  ## ConcurrentHashMap和Hashtable的区别

  * 底层数据结构：JDK1.7的ConcurrentHashMap底层采用分段的数组+链表实现，JDK1.8采用的数据结构跟HashMap1.8得结构一样，数组+链表/红黑二叉树，Hashtable和JDK1.8之前得HashMap得底层数据结构类似都是采用数组+链表得形式，数组是主题，链表则是为了解决哈希冲突才存在得

  * 实现线程安全得方式：

    * JDK1.7时，ConcurrentHashMap对整个桶进行了分割分段，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段得数据，就不会存在锁竞争，提高并发访问率
    * JDK1.8时，ConcurrentHashMap已经摒弃了Segment得概念，而是直接用Node数组+链表+红黑树得数据结构来实现，并发控制使用synchronized和CAS来操作。整个就像是优化且线程安全得HashMap，虽然JDK1.8还能看到Segment得数据结构，到那时已经简化了属性，只是为了兼容旧版本
    * Hashtable（同一把锁）：使用synchronized来保证线程安全，效率非常地下。当一个线程访问同步方法时，其他线程也访问同步方法，可能进入阻塞或者轮询状态

    1.7的ConcurrentHashMap：

    是由Segment数组结构和HashEntry数组结构组成

    Segment数组中的每个元素包含一个HashEntry数组，每个HashEntry数组属于链表结构

    1.8的ConcurrentHashMap：

    Node数组+链表/红黑树组成

    Node只能用于链表的情况，红黑树的情况需要使用TreeNode

    TreeNode时存储红黑树节点，被TreeBin包装，TreeBin通过root属性维护红黑树的根节点，因为红黑树在旋转的时候，根节点可能会被他原来的节点替换，在这个时间点，如果有其他线程要写这颗红黑树就会发生线程不安全问题，所以在ConcurrentHashMap中TreeBin通过waiter属性维护当前使用这颗红黑树的线程，来防止其他线程进入

    ```java
    static final class TreeBin<K,V> extends Node<K,V> {
            TreeNode<K,V> root;
            volatile TreeNode<K,V> first;
            volatile Thread waiter;
            volatile int lockState;
            // values for lockState
            static final int WRITER = 1; // set while holding write lock
            static final int WAITER = 2; // set when waiting for write lock
            static final int READER = 4; // increment value for setting read lock
    ...
    }
    ```

## 了解ConcurrentHashMap线程安全的具体实现方式/底层实现吗？

JDK1.8之前——

首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问启动一个段数据时，其他段的数据也能被其他线程访问

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成

Segment继承了ReetrantLock，所以Segment是一种可重入锁，扮演锁的角色，HashEntry用于存储键值对数据

一个ConcurrentHashMap里包含一个Segment数组，Segment的个数一旦初始化就不能改变，Segment数组的大小默认是16，也就是说默认可以同时支持16个线程并发写

Segment的结构和HashMap类似，是一种数组和链表结构，一个Segment包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护一个HashEtry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得对应的Segment锁

JDK1.8之后——

采用Node + CAS + synchronized来保证并发安全，数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树，Java8在链表长度超过一定阈值（8）时将链表转换为红黑树

Java8中，锁粒度更细，synchronized之锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，就不会影响其他Node的读写，效率大幅提升

## JDK1.7和JDK1.8的ConcurrentHashMap实现有什么不同？

* 线程安全实现方式：JDK1.7采用Segment分段锁来保证安全，Segment是继承自ReentrantLock，JDK1.8放弃Segment分段锁的设计，采用Node+CAS+synchronized保证线程安全，锁粒度更细，synchronized之锁定当前链表或红黑二叉树的首节点
* Hash碰撞解决方法：JDK1.7采用拉链法，JDK1.8采用拉链法结合红黑树
* 并发度：JDk1.7最大并发是Segment个数，默认是16，JDK1.8最大并发度是Node数组的大小，并发度更大

## ArrayList和Vector的区别？

* ArrayList是List的主要实现类，底层使用Object[]存储，适用于频繁的查找功能，线程不安全
* Vector是List的古老实现类，底层采用Object[]存储，线程安全

## 了解ArrayList核心源码吗？

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;


public class ArrayList<E> extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;

    /*
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;

    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};

    //用于默认大小空实例的共享空数组实例。
    //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList 所包含的元素个数
     */
    private int size;

    /**
     * 带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //如果传入的参数大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果传入的参数等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //其他情况，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *默认无参构造函数
     *DEFAULTCAPACITY_EMPTY_ELEMENTDATA 为0.初始化为10，也就是说初始其实是空数组 当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //将指定集合转换为数组
        elementData = c.toArray();
        //如果elementData数组的长度不为0
        if ((size = elementData.length) != 0) {
            // 如果elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组所以加上下面的语句用于判断）
            if (elementData.getClass() != Object[].class)
                //将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 其他情况，用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 修改这个ArrayList实例的容量是列表的当前大小。 应用程序可以使用此操作来最小化ArrayList实例的存储。
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
                ? EMPTY_ELEMENTDATA
                : Arrays.copyOf(elementData, size);
        }
    }
    //下面是ArrayList的扩容机制
    //ArrayList的扩容机制提高了性能，如果每次只扩充一个，
    //那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        //如果是true，minExpand的值为0，如果是false,minExpand的值为10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //如果最小容量大于已有的最大容量
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
    //1.得到最小扩容量
    //2.通过最小容量扩容
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            // 获取“默认的容量”和“传入参数”两者之间的最大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
    //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量，
        //若超出了，则调用hugeCapacity()来比较minCapacity和 MAX_ARRAY_SIZE，
        //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为 MAX_ARRAY_SIZE。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //比较minCapacity和 MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
    }

    /**
     *返回此列表中的元素数。
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回 true 。
     */
    public boolean isEmpty() {
        //注意=和==的区别
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true 。
     */
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此ArrayList实例的浅拷贝。 （元素本身不被复制。）
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     *以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。
     *返回的数组将是“安全的”，因为该列表不保留对它的引用。 （换句话说，这个方法必须分配一个新的数组）。
     *因此，调用者可以自由地修改返回的数组。 此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）;
     *返回的数组的运行时类型是指定数组的运行时类型。 如果列表适合指定的数组，则返回其中。
     *否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。
     *如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null 。
     *（这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。）
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }

    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此列表中的指定位置插入指定的元素。
     *先调用 rangeCheckForAdd 对index进行界限检查；然后调用 ensureCapacityInternal 方法保证capacity足够大；
     *再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除该列表中指定位置的元素。 将任何后续元素移动到左侧（从其索引中减去一个元素）。
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
        //从列表中删除的元素
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。 如果列表不包含该元素，则它不会更改。
     *返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从列表中删除所有元素。
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除所有索引为fromIndex （含）和toIndex之间的元素。
     *将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合中包含的所有元素。
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }


    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     *指定的索引表示初始调用将返回的第一个元素为next 。 初始调用previous将返回指定索引减1的元素。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     *返回列表中的列表迭代器（按适当的顺序）。
     *返回的列表迭代器是fail-fast 。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     *以正确的顺序返回该列表中的元素的迭代器。
     *返回的迭代器是fail-fast 。
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
}
```

## 了解ArrayList扩容机制吗

以无参数构造方法创建ArrayList时，实际上初始化赋值得是一个空数组，当真正对数据进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10

```java
/**
     * 将指定的元素追加到此列表的末尾。
     */
public boolean add(E e) {
    //添加元素之前，先调用ensureCapacityInternal方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //这里看到ArrayList添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```

当要add进入第一个元素时，minCapacity为1，再Math.max()方法比较厚，minCapacity为10

```java
//得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 获取默认的容量和传入参数的较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}
```

minCapacity - elementData.length > 0，所以会进入grow(minCapacity)方法

```java
/**
     * 要分配的最大数组大小
     */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
     * ArrayList扩容的核心方法。
     */
private void grow(int minCapacity) {
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
    //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果新容量大于 MAX_ARRAY_SIZE,进入(执行) `hugeCapacity()` 方法来比较 minCapacity 和 MAX_ARRAY_SIZE，
    //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为 MAX_ARRAY_SIZE 即为 `Integer.MAX_VALUE - 8`。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

所以ArrayList每次扩容之后容量都会变为原来的1.5倍左右

如果新容量大于MAX_ARRAY_SIZE进入hugeCapacity()方法来比较minCapacity和MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新内容大小则为MAX_ARRAY_SIZE即为Integer.MAX_VALUE -8

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //对minCapacity和MAX_ARRAY_SIZE进行比较
    //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
    //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
    //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

## System.arraycopy()和Arrays.copyOf()的区别？

联系：

两个源代码内部实际都是调用了System.arraycopy()方法

区别：

arraycopy()需要目标数组，将原数组拷贝到你自己定义的数组里面或者原数组，而且可以选择拷贝的起点和长度以及放入数组中的位置copyOf()是系统自动再内部新建的一个数组，并返回该数组

## 了解HashMap底层结构吗？

* JDK1.8之前

  HashMap底层是数组和链表也就是链表散列

  HashMap通过key的hashCode经过扰动函数（hash()方法）得到hash值，然后通过`(n - 1) & hash`判断当前元素存放的位置，如果存在元素的话，判断hash值和key是否相同，相同覆盖，否则拉链法解决冲突

* JDK1.8

  当链表长度大于阈值（默认为8）时，会首先调用treeifyBin()方法，这个方法会根据HashMap数组来决定是否转换为红黑树。只有当数组长度大于或者等于64的情况下，才会执行转换为红黑树操作，以减少搜索时间。否则只是执行resize()方法对数组进行扩容

## 什么是扰动函数？

扰动函数就是HashMap的hash方法，使用hash方法也就是扰动函数是为了防止一些实现比较差的hashCode()方法，减少碰撞

## 了解HashMap核心源码吗？

* Node节点类源码：

```java
// 继承自 Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
       final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
       final K key;//键
       V value;//值
       // 指向下一个节点
       Node<K,V> next;
       Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 重写hashCode()方法
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        // 重写 equals() 方法
        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
}
```

* 树节点类源码：

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 父
    TreeNode<K,V> left;    // 左
    TreeNode<K,V> right;   // 右
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;           // 判断颜色
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    // 返回根节点
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```

* 构造方法

```java
// 默认构造函数。
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
}

// 包含另一个“Map”的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);//下面会分析到这个方法
}

// 指定“容量大小”的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 指定“容量大小”和“加载因子”的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

putMapEntries方法

```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            /**
            +1.0F是因为((float)s / loadFactor)使用float计算，在转换成整数的时候会进行舍入，为了保证最终计算出来的size足够大不至于触发扩容，所以进行了+ 1.0F操作。
            **/
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

* put方法

HashMap只提供了put用于添加元素，put方法中调用了putVal()方法，但是putVal并没有提供给用户使用

putVal添加元素——

1. 如果定位到的数组位置没有元素就直接插入
2. 如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个是树节点，如果是就调用putTreeVal方法将元素添加进入，否则就遍历链表插入

![](D:\WorkSpace\Note\Picture\HashMap的put方法.png)

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素（处理hash冲突）
    else {
        Node<K,V> e; K k;
        // 判断table[i]中的元素是否与插入的key一样，若相同那就直接使用插入的值p替换掉旧的值e。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
        // 判断插入的是否是红黑树节点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 不是红黑树节点则说明为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值(默认为 8 )，执行 treeifyBin 方法
                    // 这个方法会根据 HashMap 数组来决定是否转换为红黑树。
                    // 只有当数组长度大于或者等于 64 的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是对数组扩容。
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) {
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

* get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

* resize方法

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

## 了解ConcurrentHashMap的底层数据结构吗？

* JDK7

  Java7中ConcurrentHashMap的存储结构，ConcurrentHashMap由多个Segment组合，而每一个Segment是一个类似于HashMap的结构，所以每一个HashMap的内部可以进行扩容，但是Segment的个数一旦初始化就不能改变，默认Segment的个数是16个，可以认为ConcurrentHashMap默认支持最多16个线程并发

* JDK8

  Java8的ConcurrentHashMap相当于Java7来说变化比较大，不再是之前的Segment数组+HashEntry数组+链表，而是Node数组+链表/红黑树，只是对Node加锁

## 了解JDK8的ConcurrentHashMap的核心源码吗？

* 初始化initTable

  ```java
  /**
   * Initializes table, using the size recorded in sizeCtl.
   */
  private final Node<K,V>[] initTable() {
      Node<K,V>[] tab; int sc;
      while ((tab = table) == null || tab.length == 0) {
          //　如果 sizeCtl < 0 ,说明另外的线程执行CAS 成功，正在进行初始化。
          if ((sc = sizeCtl) < 0)
              // 让出 CPU 使用权
              Thread.yield(); // lost initialization race; just spin
          else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
              try {
                  if ((tab = table) == null || tab.length == 0) {
                      int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                      @SuppressWarnings("unchecked")
                      Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                      table = tab = nt;
                      sc = n - (n >>> 2);
                  }
              } finally {
                  sizeCtl = sc;
              }
              break;
          }
      }
      return tab;
  }
  ```

  ConcurrentHashMap的初始化是通过自旋和CAS操作完成的，里面需要注意的变量sizeCtl

  * -1说明正在初始化
  * -N说明由N-1个线程正在进行扩容
  * 如果table没有初始化，表示table初始化大小
  * 如果table已经初始化，表示table容量

* put方法

  ```java
  public V put(K key, V value) {
      return putVal(key, value, false);
  }
  
  /** Implementation for put and putIfAbsent */
  final V putVal(K key, V value, boolean onlyIfAbsent) {
      // key 和 value 不能为空
      if (key == null || value == null) throw new NullPointerException();
      int hash = spread(key.hashCode());
      int binCount = 0;
      for (Node<K,V>[] tab = table;;) {
          // f = 目标位置元素
          Node<K,V> f; int n, i, fh;// fh 后面存放目标位置的元素 hash 值
          if (tab == null || (n = tab.length) == 0)
              // 数组桶为空，初始化数组桶（自旋+CAS)
              tab = initTable();
          else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
              // 桶内为空，CAS 放入，不加锁，成功了就直接 break 跳出
              if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                  break;  // no lock when adding to empty bin
          }
          else if ((fh = f.hash) == MOVED)
              tab = helpTransfer(tab, f);
          else {
              V oldVal = null;
              // 使用 synchronized 加锁加入节点
              synchronized (f) {
                  if (tabAt(tab, i) == f) {
                      // 说明是链表
                      if (fh >= 0) {
                          binCount = 1;
                          // 循环加入新的或者覆盖节点
                          for (Node<K,V> e = f;; ++binCount) {
                              K ek;
                              if (e.hash == hash &&
                                  ((ek = e.key) == key ||
                                   (ek != null && key.equals(ek)))) {
                                  oldVal = e.val;
                                  if (!onlyIfAbsent)
                                      e.val = value;
                                  break;
                              }
                              Node<K,V> pred = e;
                              if ((e = e.next) == null) {
                                  pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                  break;
                              }
                          }
                      }
                      else if (f instanceof TreeBin) {
                          // 红黑树
                          Node<K,V> p;
                          binCount = 2;
                          if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                         value)) != null) {
                              oldVal = p.val;
                              if (!onlyIfAbsent)
                                  p.val = value;
                          }
                      }
                  }
              }
              if (binCount != 0) {
                  if (binCount >= TREEIFY_THRESHOLD)
                      treeifyBin(tab, i);
                  if (oldVal != null)
                      return oldVal;
                  break;
              }
          }
      }
      addCount(1L, binCount);
      return null;
  }
  ```

  1. 根据key计算出hashCode
  2. 判断是否需要进行初始化
  3. 根据当前的key定位出的Node，如果为空表示当前位置可以写入数据，利用CAS尝试写入，失败则自旋直到成功
  4. 如果当前位置的hashCode == MOVED == -1，则需要进行扩容
  5. 如果都不满足，则利用synchronized锁写入数据
  6. 如果数量大于TREEIFY_THRESHOLD则要执行树化方法，再treeifyBin中会首先判断当前数组长度>=64时将链表转换为红黑树

* get方法

  ```java
  public V get(Object key) {
      Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
      // key 所在的 hash 位置
      int h = spread(key.hashCode());
      if ((tab = table) != null && (n = tab.length) > 0 &&
          (e = tabAt(tab, (n - 1) & h)) != null) {
          // 如果指定位置元素存在，头结点hash值相同
          if ((eh = e.hash) == h) {
              if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                  // key hash 值相等，key值相同，直接返回元素 value
                  return e.val;
          }
          else if (eh < 0)
              // 头结点hash值小于0，说明正在扩容或者是红黑树，find查找
              return (p = e.find(h, key)) != null ? p.val : null;
          while ((e = e.next) != null) {
              // 是链表，遍历查找
              if (e.hash == h &&
                  ((ek = e.key) == key || (ek != null && key.equals(ek))))
                  return e.val;
          }
      }
      return null;
  }
  ```

  1. 根据hash值计算位置
  2. 查找到指定位置，如果头节点就是要找的，直接返回它的value
  3. 如果头节点hash值小于0，说明正在扩容或者时红黑树，查找之
  4. 如果是链表，遍历查找之

# Java IO

## 什么是InputStream（字节输入流）？

InputStream用于从源头（通常是文件）读取数据（字节信息）到内存中，java.io.InputStream抽象类是所有字节输入流的父类

FileInputStream是一个比较常用的字节输入流对象，可直接指定文件路径，可以直接读取单字节数据，也可以读取至字节数组中

一般不会直接单独使用FIleInputStream，通常会配合BufferedInputStream（字节缓冲输入流）来使用

DataInputStream用于读取指定类型数据，不能单独使用，必须结合FileInputStream

ObjectInputStream用于从输入流中读取java对象（反序列化），ObjectOutputStream用于将对象写入到输出流（序列化）

## 什么是OutputStream（字节输出流）？

OutputStream用于将数据（字节数据）写到目的地（通常是文件），java.io.OutputStream抽象类是所有字节输出流的父类

FileOutputStream是常用的字节输出流对象，可直接指定文件路径，可以直接输出单字节数据，也可以输出指定的字节数组

FileOutputStream通常也配合BufferedOutputStream（字节缓冲输出流）来使用

DataOutputStream用于写入指定类型数据，不能单独使用，必须结合FileOutputStream

ObjectInputStream用于从输入流中读取Java对象，objectOutputStream将对象写入到输出流

## 为什么I/O流操作要分为字节流操作和字符流操作？

* 字符流是由Java虚拟机将字节转换的到的，这个过程还算是比较耗时的
* 如果我们不知道编码类型很容易出现乱码问题

## 常用字符编码所占字节数？

utf8：英文占1字节，中文占3字节

unicode：任何字符都站2个字节

gbk：英文占1字节，中文占2字节

## 什么是Reader（字符输入流）？

Reader用于从源头（通常是文件）读取数据（字符信息）到内存中，java.io.Reader抽象类是所有字符输入流的父类

Reader用于读取文本，InputStream用于读取原始字节

InputStreamReader是字节流转换为字符流的桥梁，其子类FileReader是基于该基础上的封装，可以直接操作字符文件

## 什么是Writer（字符输出流）？

Writer用于将数据（字符信息）写入到目的地（通常是文件），java.io.Writer抽象类是所有字节输出流的父类

OutputStreamWriter是字符流转换为字节流的桥梁，其子类FileWriter是基于该基础上的封装，可以直接将字符写入到文件

## 什么是字节缓冲流？

IO操作是很消耗性能的，缓冲区将数据加载至缓冲区，一次性读取/写入多个字节，从而避免频繁的IO操作，提高流的传输了效率

字节缓冲流这里采用了装饰器模式来增强InputStream和OutputStream子类对象的功能

字节流和字节缓冲流的性能差别主要体现在我们使用两者的时候都是调用write(int)和read()这两个一次只读取一个字节的方法的时候。

**由于字节缓冲流内部由缓冲区（字节数组），因此，字节缓冲流会先将读取到的字节存放在缓冲区，大幅减少IO次数，提高读取效率**

## 什么是BufferedInputStream（字节缓冲输入流）？

BufferedInputStream从源头（通常是文件）读取数据（字节信息）到内存的过程不会一个字节一个字节的读取，而是会先将读取到的字节存放在换红曲，并从内部缓冲区中单独读取字节。这样大幅减少IO次数，提高了读取效率

BufferedInputStream内部维护了一个缓冲区，这个缓冲区实际就是一个字节数组，通过阅读BufferedInputStream源码——

```java
public
class BufferedInputStream extends FilterInputStream {
    // 内部缓冲区数组
    protected volatile byte buf[];
    // 缓冲区的默认大小
    private static int DEFAULT_BUFFER_SIZE = 8192;
    // 使用默认的缓冲区大小
    public BufferedInputStream(InputStream in) {
        this(in, DEFAULT_BUFFER_SIZE);
    }
    // 自定义缓冲区大小
    public BufferedInputStream(InputStream in, int size) {
        super(in);
        if (size <= 0) {
            throw new IllegalArgumentException("Buffer size <= 0");
        }
        buf = new byte[size];
    }
}
```

缓冲区默认大小为8192字节，也可以通过`BufferedInputStream(InputStream in， int size)`这个构造方法来指定缓冲区大小

## 什么是BufferedOutputStream（字节缓冲输入流）？

BufferedOutputStream将数据（字节信息）写入到目的地（通常是文件）的过程中不会讲一个字节一个字节的写入，而是会先将要写入的字节存放在缓冲区，并从内部缓冲区中单独写入字节，这样减少了IO次数，提高了读取效率

BufferedOutputStream内部也维护了一个缓冲区，并且，这个缓冲区的大小也是8192字节

## 什么是字符缓冲流？

BufferedReader（字符缓冲输入流）和BufferedWriter（字符缓冲输出流）类似于BufferedInputStream（字节缓冲输入流）和BufferedOutputStream（字节缓冲输入流），内部都维护了一个字节数组作为缓冲区

## 什么是随机访问流？

随机访问流指的是支持随意跳转文件的任意位置进行读写的RandomAccessFile

RandomAccessFile的构造方法可以指定mode

```java
// openAndDelete 参数默认为 false 表示打开文件并且这个文件不会被删除
public RandomAccessFile(File file, String mode)
    throws FileNotFoundException {
    this(file, mode, false);
}
// 私有方法
private RandomAccessFile(File file, String mode, boolean openAndDelete)  throws FileNotFoundException{
  // 省略大部分代码
}
```

读写模式主要有下面四种：

* r：只读模式
* rw：读写模式
* rws：相当于rw，rws同步更新对”文件的内容“或”元数据“的修改到外部存储设备
* rwd：相当于rw，rwd同步更新对”文件的内容“的修改到外部存储设备

> 文件内容指的是文件中实际保存的数据，元数据则是用来描述文件属性比如文件的大小信息，创建和修改时间

RandomAccessFile中有一个文件指针用来表示下一个将要被写入或者读取的字节所处的位置，我们可以通过RandomAccessFile的seek(long pos)方法来设置文件指针的偏移量，如果想要获取文件指针当前的位置的话，可以使用getFilePointer()方法

RandomAccessFile比较常用的一个应用就是实现大文件的断点续传，简单来说就是上传文件中途暂停或失败之后，不需要重新上传，只需要上传那些未成功上传的文件分片即可，分片上传是断点续传的基础

## IO流中所使用的设计模式有哪些？

* 装饰器模式

  装饰器模式可以在不改变原有对象的情况下扩展其功能

  装饰器模式通过组合替代继承来扩展原始类的功能，在一些继承关系比较复杂的场景更加实用

  对于字节流来说，FIleInputStream和FileInputStream是装饰器模式的核心，分别用于加强InputStream和OutputStream子类对象的功能

  我们常用的BufferedInputStream、DataInputStream等等都是FileInputStream的子类，BufferedOutputStream、DataOutputStream等等都是FileOutputStream的子类

* 适配器模式

  适配器模式主要用于接口互不兼容的类的协调工作

  适配器模式中存在被适配的对象或者类称为适配者，作用于适配者的对象或者类成为适配器。适配器分为对象适配器和类适配器。类适配器使用继承关系来实现，对象适配器使用组合关系来实现

  IO流中的字符流和字节流的接口不同，它们之间可以协调工作就是基于适配器模式来做的，更准确点来说是对象适配器。通过适配器，我们可以及那个字节流对象适配成一个字符流对象，这样可以通过字节流对象读取或者写入字符数据

  InputStreamReader和OutputStreamWriter就是两个适配器，同时，它们两个也是字节流和字符流之间的桥梁。InputStreamReader使用streamDecoder（流解码器）对字节进行解码，实现字节流到字符流的转换，OutputStreamWriter使用streamEncoder（流编码器）对字符进行编码，实现字符流到字节流的转换

* 工厂模式

  工厂模式用于创建对象，NIO中大量用到了工厂模式

  > Files类的newInputStream方法用于创建InputStream对象（静态工厂）
  >
  > Paths类的get方法创建Path对象（静态工厂）
  >
  > ZipFileSystem类的getPath的方法创建Path对象（静态工厂）

* 观察者模式

  NIO中的文件目录监听服务使用到了观察者模式

  NIO中的文件m剥监听服务基于WatchService接口和Watchable接口。WatchService属于观察者，Wachable属于被观察者

  Watchable接口定义了一个用于将对象注册到WatchService（监控服务）并绑定监听事件的方法register

  ```java
  public interface Path
      extends Comparable<Path>, Iterable<Path>, Watchable{
  }
  
  public interface Watchable {
      WatchKey register(WatchService watcher,
                        WatchEvent.Kind<?>[] events,
                        WatchEvent.Modifier... modifiers)
          throws IOException;
  }
  ```

  WatchService用于监听文件目录的变化，同一个WatchService对象能够监听多个文件目录

  ```java
  // 创建 WatchService 对象
  WatchService watchService = FileSystems.getDefault().newWatchService();
  
  // 初始化一个被监控文件夹的 Path 类:
  Path path = Paths.get("workingDirectory");
  // 将这个 path 对象注册到 WatchService（监控服务） 中去
  WatchKey watchKey = path.register(
  watchService, StandardWatchEventKinds...);
  ```

  常用的监听事件有3种：

  * StandardWatchEventKinds.ENTRY_CREATE：文件创建
  * StandardWatchEventKinds.ENTRY_DELETE：文件删除
  * StandardWatchEventKinds.ENTRY_MODIFY：文件修改

  register方法返回watchKey对象，通过watchKey对象可以获取事件的具体信息

  ```java
  WatchKey key;
  while ((key = watchService.take()) != null) {
      for (WatchEvent<?> event : key.pollEvents()) {
        // 可以调用 WatchEvent 对象的方法做一些事情比如输出事件的具体上下文信息
      }
      key.reset();
  }
  ```

  WatchService内部是通过一个daemon thread次啊用定期轮询的方式来检测文件的变化

  ```java
  class PollingWatchService
      extends AbstractWatchService
  {
      // 定义一个 daemon thread（守护线程）轮询检测文件变化
      private final ScheduledExecutorService scheduledExecutor;
  
      PollingWatchService() {
          scheduledExecutor = Executors
              .newSingleThreadScheduledExecutor(new ThreadFactory() {
                   @Override
                   public Thread newThread(Runnable r) {
                       Thread t = new Thread(r);
                       t.setDaemon(true);
                       return t;
                   }});
      }
  
    void enable(Set<? extends WatchEvent.Kind<?>> events, long period) {
      synchronized (this) {
        // 更新监听事件
        this.events = events;
  
          // 开启定期轮询
        Runnable thunk = new Runnable() { public void run() { poll(); }};
        this.poller = scheduledExecutor
          .scheduleAtFixedRate(thunk, period, period, TimeUnit.SECONDS);
      }
    }
  }
  ```

## 适配器模式和装饰器模式有什么区别？

* 装饰器模式更侧重于动态地增强原始类的功能，装饰器类需要跟原始类继承相同的抽象类或实现相同的接口，并且装饰器模式支持对原始类嵌套使用多个装饰器
* 适配器模式更侧重于让接口不兼容而不能交互的类可以一起工作，当我们调用适配器对应的方法时，适配器内部会调用适配者类或者适配类相关的类的方法，这个过程透明的，适配器和适配者两者不需要继承相同的抽象类或者实现相同的接口

## 有哪些常见的IO模型？

UNIX系统下，IO模式一共有5种：同步阻塞I/O、同步非阻塞I/O、I/O多路复用、信号驱动I/O和一部I/O

## 了解BIO吗？

BIO属于同步阻塞IO模型

同步阻塞IO模型中，应用程序发起read调用后，会一直阻塞，直到内核把数据拷贝到用户空间

![](D:\WorkSpace\Note\Picture\BIO模型.png)

## 了解NIO吗？

Java中的NIO于java1.4引入，对应java.nio包，提供了Channel、Selector、Buffer等抽象

NIO中的N可以理解为Non-blocking，不单纯是New

它是支持面向缓冲的，基于通道的I/O操作方法，对于高负载、高并发的应用，应使用NIO

Java中的NIO可以看作是I/O多路复用模型

![](D:\WorkSpace\Note\Picture\NIO模型.png)

同步非阻塞IO模型中，应用程序会一直发起read调用，等待数据从内核空间拷贝到用户空间的这段时间内，线程依然是阻塞的，直到在内核把数据拷贝到用户空间

相对于同步则色IO模型，同步非阻塞IO模型确实有很大的改进，通过轮询操作，避免了一直阻塞

但是，这种IO模型同样存在问题：应用程序不断进行I/O系统调用轮询数据是否已经准备好的过程是十分消耗CPU资源的

![](D:\WorkSpace\Note\Picture\IO多路复用.png)

IO多路复用模型中，线程首先发起select调用，询问内核数据是否准备就绪，等内核把数据准备好了，用户线程再发起read调用。read调用的过程（数据从内核空间->用户空间）还是阻塞的

IO多路复用模型，通过减少无效的系统调用，减少了对CPU资源的消耗

## 了解AIO吗？

异步IO是居于事件和回调机制实现，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作

![](D:\WorkSpace\Note\Picture\异步IO.png)

# Java并发编程

## Java并发编程

# JVM

## 运行时数据区哪些是私有的？共享的？

![](D:\WorkSpace\Note\Picture\运行时数据区域.png)

* 线程私有的：
  * 程序计数器
  * 虚拟机栈
  * 本地方法栈
* 线程共享的
  * 堆
  * 方法区
  * 直接内存（非运行时数据区域）

## 什么是程序计数器？

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器

为了线程切换后能够恢复到正确的执行位置，每条线程都需要一个独立的程序计数器，各线程之间计数器互不影响，独立存储

 **程序计数器是唯一一个不会出先OOM的内存区域，他的生命周期随着线程的创建而创建，随着线程的结束而死亡**

## 什么是Java虚拟机栈？

Java虚拟机栈也是线程私有的，他的生命周期与线程相同，随着线程创建而创建，随着线程的死亡而死亡

除了一些Native方法调用是通过本地方法栈实现的，其他所有的Java方法都是通过栈来实现的

方法调用的数据需要通过栈进行传递，每一次方法调用都会有一个对应的栈帧被压入栈中，每一个方法调用结束后，都会有一个栈帧被弹出

栈由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法返回地址

* 局部变量表

  主要存放了编译器可知的各种数据类型、对象引用

* 操作数栈

  主要作为方法调用的中转站使用，用于存放方法执行过程中产生的中间计算结果，另外，计算过程中产生的临时变量也会存放在操作数栈中

* 动态链接

  主要服务一个方法需要调用其他方法的场景，在Java源文件被编译成字节码文件时，所有的变量和方法引用都作为符号引用保存在Class文件的常量池中，当一个方法需要调用其他方法时，需要将常量池中指向方法的符号引用转化为其在内存地址中的直接引用。**动态连接的作用就是为了将符号引用转换为调用方法的直接引用**

## 内存常见两种错误的区别？

* StackOverFlowError：若栈的内存大小i允许动态扩展，那么当线程请求栈的深度超过当前Java虚拟机栈的最大深度时，就抛出StackOverFlowError错误
* OutOfMemoryError：如果栈的内存可以动态扩展，如果虚拟机在动态扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常

## 什么是本地方法栈？

虚拟机栈为虚拟机执行Java方法服务，而本地方法栈则为虚拟机使用到的Native方法服务，在HotSpot虚拟机中和Java虚拟机合二为一

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态连接、出口信息

## 什么是堆？

Java堆是所有线程共享的一块内存区域，在虚拟机启动时创建，此区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存

Java堆是垃圾收集器管理的主要区域，因此也被称为GC堆，从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以Java堆还可以细分为新生代和老年代；再细致一些有：Eden、Survivor、Old等空间

JDK8之后永久代已经被元空间代替，元空间使用的是直接内存

> 大部分情况，对象都会首先在Eden区域，再一次新生代垃圾回收后，如果对象还存活，则会进入S0或者S1，并且对象的年龄还会加1（Eden——》Survivor后对象初始年龄为1），当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中，对象晋升到老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`来设置
>
> Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累计，累计的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringTheshold中更小的一个值，作为新的晋升年龄阈值

## 为什么是几乎所有的对象都在堆中分配？

Java中“几乎”所有的对象都在堆中分配，但是随着JIT编译器的发展和逃逸分析技术逐渐成熟，堆上分配、标量替换优化技术将会导致一些微妙的变化，所有的对象都分配到堆中也就不那么绝对了，从1.7开始默认开启逃逸分析，如果某个方法中的对象没有被返回或者未被外面使用，那么对象可以直接在栈上分配内存

## 什么是方法区？

 方法区属于是JVM运+行时数据区域的一块逻辑区域，是各个线程共享的内存区域

当虚拟机要使用一个类时，它需要读取并分析Class文件获取相关信息，再将信息存入到方法区，方法区会存储已被虚拟机加载的类信息、字段信息、方法信息、常量、静态变量、即时编译器编译后的代码存储等数据

## 为什么要将永久代替换为元空间？

1. 整个永久代有一个JVM本身设置的固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的机率更小

   可以使用`-XX: MaxMetaspaceSize`标志设置最大元空间大小，默认值为unlimited，这意味着它只受系统内存的限制

   可以使用`-XX:MetaspaceSize`来设置Metaspace的初始大小

2. 元空间里面存放的类的元数据，这样加载多少类的元数据就不由MaxPermSize控制了，而由系统的实际可用空间来控制，这样能加载的类就更多了

3. 在JDK8，合并HotSpot和JRockit的代码时，JRockit从没有一个叫做永久代的东西，合并之后就没有必要额外的设置这么一个永久代的东西

## 什么是运行时常量池？

Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有用于存放编译器生成的各种字面量和符号引用的常量池表 

字面量是源代码中的固定值的表示法，通过字面可以知道值的含义，字面量包含整数、浮点数和字符串字面量

符号引用包含类符号引用、字段符号引用、方法符号引用和接口方法符号引用

常量池表会在类加载后存放到方法区的运行时常量池中

运行时常量池的功能类似于传统编程语言的符号表，尽管它包含了比典型符号表更广泛的数据

## 什么是字符串常量池？

字符串常量池是JVM为了提升性能和减少内存消耗针对字符串专门开辟的一块区域，主要目的是为了避免字符串重复创建

HotSpot虚拟机中字符串常量池的实现是src/hotspot/share/classfile/stringTable.cpp，StringTable本质上就是一个HashSet\<String>，容量为StringTableSize（可以通过`-XX:StringTableSize`参数来设置）

StringTable中保存的是字符串对象的引用，字符串对象的引用指向堆中的字符串对象

## 为什么要将字符串常量池移动到堆中？

主要因为永久代（方法区实现）的GC回收率太低，只有在整堆收集（Full GC）的时候才会被执行GC，Java中通常会有大量的被创建的字符串等待回收，将字符串常量池放入到堆中，能够更高效及时地回收字符串内存

## 什么是直接内存？

直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OOM

本机直接内存的分配不受到Java堆的限制，但是既然是内存就会收到本机内存大小以及处理器寻址空间的限制

## 了解HotSpot虚拟机在Java堆中对象的创建吗？

1. 类加载检查

   虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过，如果没有，那必须先执行相应的类加载过程

2. 分配内存

   在类加载检查通过后，接下来虚拟机将为新生对象分配内存，对象所需的内存大小在类加载完成后便可以确定，为对象分配空间的任务等同于把一块确定大小的内存从Java堆中划分出来

   分配方式有“指针碰撞”和“空闲列表“两种，选择哪一种由Java堆是否规整决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定

   * 指针碰撞

     使用场合：堆内存规整（即没有内存碎片）的情况下

     原理：用过的内存全部整合到一边，没有用过的内存放在另一边，中间有一个分界指针，只需要向着没用过的内存方向将该指针对东对象内存大小的位置即可

     使用该分配方式的GC收集器：Serial，ParNew

   * 空闲列表

     使用场合：堆内存不规整的情况下

     原理：虚拟机会维护一个列表，该列表中记录那些内存块是可用的，在分配的时候，找一块儿足够大的内存块来划分给对象实例，最后更新列表记录

     使用该分配方式的GC收集器：CMS

   **选择以上两种方式中的哪一种，取决于Java堆内存是否规整，而Java堆内存是否规整，取决于GC收集器的算法是”标记-清除“，还是”标记-整理“，复制算法内存也是规整的**

   >  内存分配并发问题
   >
   > 作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全
   >
   > * CAS失败重试：CAS是乐观锁的一种实现方式，所谓乐观锁就是，每次不加锁而是假设没有冲突而去完后曾操作，如果因为冲突失败，知道成功为止。**虚拟机采用CAS配上失败重试的方式保证更新操作的原子性**
   >  * TLAB：为每一个线程预先在Eden区分配一块内存，JVM在给线程中的对象分配内存时，首先在TLAB分配，当对象大于TLAB中剩Z																																																																																																																																																																																																																																																																																																																																																																																																																																																																																			余内存或TLAB的内存已用尽时，在采用上述的CAS进行内存分配
   > 

3. 初始化零值

   内存分配完成后，虚拟机需要将分配到的内存空间都初始化为0值（不包括对象头），这一步操作保证过了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值

4. 设置对象头

   虚拟机要对对象进行必要的设置（这个对象是哪个类的实例，如何才能找到类的元数据信息、对象的哈希码、对象的GC分段年龄等）。这些信息存放在对象头中，另外虚拟机根据当前运行的状态不同，如是否启动偏向锁等，对象头会有不同的设置方式

5. 执行init方法

   执行new指令之后会接着执行\<init>方法，把对象按照程序员的意愿进行初始化

## 了解对象的内存布局吗？

在Hotspot虚拟机中，对象在内存中的布局可以分为3块区域：对象头、实例数据和对齐填充

Hotspot虚拟机的对象头包含两部分信息，第一部分用于存储对象自身的运行时数据，另一部分时类型指针，即指向它的类元数据的指针，通过这个指针可以确定这个对象是哪个类的实例

实例数据部分是对象真正存储的有效信息，也是在程序中所定义的各种类型的字段内容

对齐填充部分不是必然存在的，也没有意义，仅仅起占位作用，因为Hotspot虚拟机的自动内存管理系统要求对象的起始地址必须是8字节的整数倍，而对象头刚好8字节的倍数，因此当对象实例数据部分没有对齐时，就需要通过对齐填充来补全

## 了解对象的访问对位吗？

建立对象就是为了使用对象，通过栈上的reference数据来操作堆上的具体对象，对象的访问方式由虚拟机实现而定，目前主流的访问方式有：使用句柄、直接指针

* 句柄

  如果使用句柄的话，那么Java堆中将会划分出一块内存来作为句柄池，reference中存储的就是对象的句柄地址，而句柄中包含了对象实例数据和对象类型数据各自的地址信息

  ![](D:\WorkSpace\Note\Picture\句柄.png)

* 直接指针

  如果使用直接指针访问，reference中存储的直接就是对象的地址

  ![](D:\WorkSpace\Note\Picture\直接指针.png)

这两种对象访问方式各有优势，使用句柄来访问的最大好处是reference中存储的稳定的句柄地址，在对象被移动时只会改变句柄中的实例数据指针，而reference本身不需要修改，使用直接指针访问方式最大的好处就是速度快，他节省了一次指针定位的时间开销

## 了解堆空间的基本结构？

Java的自动内存管理主要是针对对象内存的回收和对象内存的分配，同时，Java自动内存管理最核心的功能是堆内存中对象的分配和回收

Java堆是垃圾收集器管理的主要区域，因此也被称为GC堆

JDK8及以后，堆内存被通常分为——

* 新生代内存
* 老年代内存
* 元空间（直接内存）

## 了解内存分配和回收原则吗？

* 对象优先在Eden区分配

  大多数情况下，对象在新生代中Eden区分配，当Eden区没有足够空间进行分配时，虚拟机将发生一次Minor GC

  GC期间对象如果无法存入Survivor空间，就会通过分配担保机制将新生代的对象提前转移到老年代中

* 大对象直接进入老年代

  大对象就是需要大量连续内存空间的对象

  大对象直接进入老年代主要是为了避免对大对象分配内存时由于分配担保机制带来的复制而降低效率

* 长期存活的对象将进入老年代

  大部分情况，对象都会首先在Eden区域分配，如果对象在Eden出生并经过第一次Minor GC后仍然能够存活，并且能被Survivor容纳的话，将被移动到Survivor空间中，并将对象年龄设为1

  对象在Survivor中每熬过一次MinorGC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15岁），就会被晋升到老年代中，对象晋升到老年代的年龄阈值，可以通过参数`-XX:MaxTenuringThreshold`来设置

  Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累计，当累计的某个年龄大小超过了survivor区的50%时，可以通过`-XX:TargetSurvivorRatio=percent`来设置，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值

## GC的分类了解吗？

针对HotSpot VM的实现——

* 部分收集（partial GC）：
  * 新生代收集（Minor GC）：只对新生代进行垃圾收集
  * 老年代收集（Major GC）：只对老年代进行垃圾收集，需要注意的是Major GC在有的语境中也用于针对整堆收集
  * 混合收集（Mixed GC）：对整个新生代和部分老年代进行垃圾收集
* 整堆收集（Full GC）：收集整个Java堆和方法区

## 了解死亡对象的判断方法吗？

* 引用计数法

  给对象中添加一个引用计数器：

  * 每当有一个地方引用它，计数器加1
  * 当引用失效，计数器就减1
  * 任何时候计数器为0的对象就是不可能再被使用的

  这个方法实现简单，效率高，但是目前驻留的虚拟机中并没有选择这个算法来管理内存，其最重要的原因是它很难解决对象之间相互循环引用的问题

* 可达性分析算法

  通过一系列的称为”GC Roots“的对象作为起点，从这些节点开始向下搜索，系欸但所走过的路径成为引用链，当一个对象到GC Roots没有任何引用链项链的话，则证明此对象是不可用的，需要被回收

## 那些对象可以作为GC Roots呢？

* 虚拟机栈（栈帧中的本地变量表）中引用的对象
* 本地方法栈（Native方法）中引用的对象
* 方法区中类静态属性引用的对象
* 方法区中常量引用的对象
* 所有被同步锁持有的对象

## 对象可以被回收，就代表一定要被回收吗？

即使在可达性分析中不可达的对象，也并非是”非死不可“的，这时候它们暂时处于”缓刑阶段“，要真正宣告一个对象死亡，至少要经历两次标记过程

可达性分析法中不可达的对象被第一次标记并且进行一次筛选，筛选的条件是此对象是否有必要执行finalize方法，当对象没有覆盖finalize方法，或finalize方法已经被虚拟机调用过时，虚拟机将这两种情况视为没有必要执行

被判定为需要执行的对象将会被放在一个队列中进行第二次标记，除非这个对象与引用链上的任何一个对象建立关联，否则就会被真的回收

## 引用类型有哪些？

* 强引用

  如果一个对象具有强引用，那么类似于必不可少，垃圾回收器不会回收它。当内存空间不足，Java虚拟机宁愿抛出OOM错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题

* 软引用

  如果内存空间足够，垃圾回收器不会回收它，如果内存空间不足了，就会回收这些对象的内存，只要垃圾回收器没有回收它，该对象就可以被程序使用，软引用可用来实现内存敏感的高速缓存

  软引用可以和一个引用队列联合使用，如果软引用所引用的对象被垃圾回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列

* 弱引用

  弱引用和软引用的区别在于：只具有弱引用的对象拥有更短暂的生命周期。在垃圾回收器线程扫描它所管线的内存区域的过程中，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。不过，由于垃圾回收器是一个优先级很低的线程，因此不一定会很快发现那些只具有弱引用的对象

  弱引用可以和一个引用队列联合使用，如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中

* 虚引用

  如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收

  虚引用主要用来跟踪对象被垃圾回收的活动

  虚引用与（弱引用、软引用）的区别在于：虚引用必须和引用队列联合使用。

  当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。程序可以通过判断引用队列中是否已经加入了虚引用，来了解被引用的对象是否将要被垃圾回收。程序如果发现某个虚引用已经被加入到引用队列，那么就可以在所引用的对象的内存被回收之前采取必要的行动

在程序设计中很少使用弱引用和虚引用，使用软引用的情况较多，因为**软引用可以加速JVM对垃圾内存的回收速度，可以维护系统的运行安全，防止内存溢出等问题的产生**

## 如何判断一个常量是废弃常量？

运行时常量池主要回收的是废弃的常量

如果在字符串常量池中存在字符串，没有任何String对象引用该字符串常量的话，就说明其为废弃常常量，如果这时发生内存回收的话而且有必要的话，那么就会被系统清理出常量池

## 如何判断一个类是无用的类？

方法区主要回收的时无用的类

类需要同时满足下面3个条件才能算是无用的类：

* 该类所有的实例都已经被回收， 也就是Java堆中不存在该类的任何实例
* 加载该类的ClassLoader已经被回收
* 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法

虚拟机可以对满足上述3个条件的无用类进行回收，仅仅是”可以“，而并不是和对象一样不适用了就会必然被回收

## 了解哪些垃圾回收算法？

* 标记-清除算法

  该算法分为”标记“和”清除“阶段：首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象，了他是最基础的收集算法，后续的算法都是对其不足进行改进得到

  该垃圾回收算法会带来两个明显的问题：

  1. 效率问题
  2. 空间问题（标记清楚后产生大量不连续的碎片）

* 标记-复制算法

  为了解决效率问题，”标记-复制“收集算法出现了，它可以将内存分为大小相同的两块，每次使用其中的一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉，这样就是每次的内存回收对内存区域的一半进行回收

* 标记-整理算法

  根据老年代的特点提出的一种标记算法，标记过程仍然与”标记-清除算法“相同，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存

* 分代回收算法

  当前虚拟机的垃圾收集都采用分代收集算法，只是根据对象存活周期的不同将内存分为几块，一般将java堆分为新生代和老年代，这样我们可以根据各个年代的特点选择合适的垃圾收集算法

  **新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法，只需要要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们选择”标记-清除“或”标记-整理“算法进行垃圾收集**

## 了解那些垃圾回收器？

* Serial收集器

  单线程收集器

  进行垃圾收集工作的时候必须要暂停所有的工作线程，直到收集结束

  新生代采用标记-复制算法，老年代采用标记-整理算法

* ParNew收集器

  ParNew收集器就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和Serial收集器完全一样

  新生代采用”标记-复制“算法，老年代采用“标记-整理算法”

  除了Serial收集器外，只有它能与CMS收集器配合工作

* Paralle Scavenge收集器

  Parallel Scavenge收集器也是使用标记-复制算法的多线程收集器，几乎与ParNew都一样

  ```shell
  -XX:+UseParallelGC
      使用 Parallel 收集器+ 老年代串行
  -XX:+UseParallelOldGC
      使用 Parallel 收集器+ 老年代并行
  ```

  Parallel Scavenge收集器关注点事吞吐量（高效率的利用CPU）

  所谓吞吐量就是CPU中用与运行用户代码的时间和CPU消耗时间的比值

  新生代采用标记-复制算法，老年代采用标记-整理算法

  这是JDK1.8默认收集器

* Serial Old收集器

  Serial收集器的老年代版本，同样是一个单线程收集器，他的主要用途：

  1. 在1.5以及以前的版本中与Parallel Scavenge收集器搭配使用
  2. 作为CMS收集器的后备方案

* Parallel Old收集器

  Paralle Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法，在注重吞吐量以及CPU资源的场合，都可以优先考虑Parallel Scavenge收集器和Parallel Old收集器

* CMS收集器

  CMS收集器是一种以获取最短回收停顿时间为目标的收集器，它非常符合在注重用户体验的应用上使用

  CMS收集器是HotSpt虚拟机上第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程和用户线程（基本上）同时工作

  CMS收集器是一种“标记-清除”算法实现的，它的运作过程相对于前面几种垃圾收集器来说更加复杂一些

  * 初始标记：暂停所有的其他线程，并记录下直接与root相连的对象，速度很快
  * 并发标记：同时开启GC和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方
  * 重新标记：重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记姐u但的时间稍长，远远比并发标记阶段时间短
  * 并发清除：开启用户线程，同时GC线程开始堆未标记的区域做清除

  主要优点：并发收集、低停顿

  主要缺点：

  * 对CPU资源敏感
  * 无法处理浮动垃圾
  * 它使用的回收算法“标记-清除”算法会导致收集结束时会有大量空间碎片产生

* G1收集器

  G1是一款面向服务器的垃圾收集器，主要针对配置多颗处理器及大容量内存的机器，以极高概率满足GC停顿时间要求的同时，还具备高吞吐量性能特征

  * 并行和并发：G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU来缩短Stop-The-World停顿时间。部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方式让Java继续执行
  * 分代收集：虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念
  * 空间整合：G1从整体来看是基于“标记-整理”算法实现的收集器，从局部上来看是基于“标记-复制”算法实现的
  * 可预测的停顿：这是G1相对于CMS的另一个优势，降低停顿时间是G1和CMS共同的关注点，但是G1还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度未M毫秒的时间片段内

  G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region

## 了解Class文件结构吗？

根据Java虚拟机规范，Class文件通过ClassFile定义，有点类似C的结构体

```java
ClassFile {
    u4             magic; //Class 文件的标志
    u2             minor_version;//Class 的小版本号
    u2             major_version;//Class 的大版本号
    u2             constant_pool_count;//常量池的数量
    cp_info        constant_pool[constant_pool_count-1];//常量池
    u2             access_flags;//Class 的访问标记
    u2             this_class;//当前类
    u2             super_class;//父类
    u2             interfaces_count;//接口
    u2             interfaces[interfaces_count];//一个类可以实现多个接口
    u2             fields_count;//Class 文件的字段属性
    field_info     fields[fields_count];//一个类可以有多个字段
    u2             methods_count;//Class 文件的方法数量
    method_info    methods[methods_count];//一个类可以有个多个方法
    u2             attributes_count;//此类的属性表中的属性数
    attribute_info attributes[attributes_count];//属性表集合
}
```

* 魔数

  每个Class文件的头4个字节称为魔数，它的唯一作用是确定这个文件是否为一个能被虚拟机接收的Class文件

* Class文件版本号

  第5和第6位是次版本号，第7和第8位是主版本号

  每当Java发布大版本，主版本号都会加1，可以使用`javap -v`来查看Class文件的版本信息

  高版本的Java虚拟机可以执行低版本编译器生成Class文件

* 常量池

  常量池的数量是constant_pool_count-1（常量池计数器是从1开始计数的，将第0项常量空出来是有特殊考虑的，索引值位0代表不引用任何一个常量池项）

  常量池主要存放两大常量：字面量和符号引用。字面量比较接近Java语言层面的常量概念，符号引用则属于编译原理方面的概念

  * 类和结构的全限定名
  * 字段的名称和描述符
  * 方法的名称和描述符

  常量池中每一项常量都是一个表，这14中表有一个共同的特点：开始的第一位是一个u1类型的标志位-tag来表示常量的类型，代表当前这个常量属于那种类型

  | 类型                             | 标志 | 描述                   |
  | -------------------------------- | ---- | ---------------------- |
  | CONSTANT_utf8_info               | 1    | UTF-8 编码的字符串     |
  | CONSTANT_Integer_info            | 3    | 整形字面量             |
  | CONSTANT_Float_info              | 4    | 浮点型字面量           |
  | CONSTANT_Long_info               | 5    | 长整型字面量           |
  | CONSTANT_Double_info             | 6    | 双精度浮点型字面量     |
  | CONSTANT_Class_info              | ７   | 类或接口的符号引用     |
  | CONSTANT_String_info             | 8    | 字符串类型字面量       |
  | CONSTANT_Fieldref_info           | ９   | 字段的符号引用         |
  | CONSTANT_Methodref_info          | 10   | 类中方法的符号引用     |
  | CONSTANT_InterfaceMethodref_info | 11   | 接口中方法的符号引用   |
  | CONSTANT_NameAndType_info        | 12   | 字段或方法的符号引用   |
  | CONSTANT_MothodType_info         | 16   | 标志方法类型           |
  | CONSTANT_MethodHandle_info       | 15   | 表示方法句柄           |
  | CONSTANT_InvokeDynamic_info      | 18   | 表示一个动态方法调用点 |

* 访问标志

  这个标志用于表示一些或者接口层次的访问信息

  ![](D:\WorkSpace\Note\Picture\类文件访问标识.png)

* 当前类、父类、接口索引集合

  类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名，由于Java语言的单继承，所以父类索引只有一个，除了java.lang.Object之外，所有的java类都有父类，因此除了java.lang.Object外，所有Java类的父类索引都不为0

  接口索引集合用来描述这个类实现了那些接口，这些被实现的接口将按implements后的接口顺序从左到右排序在接口索引集合中

* 字段表集合

  字段表用于描述接口或类中声明的变量，字段包括类级便来给你以及实例变量，但不包括在方法内部声明的局部变量

  **access_flags:** 字段的作用域（`public` ,`private`,`protected`修饰符），是实例变量还是类变量（`static`修饰符）,可否被序列化（transient 修饰符）,可变性（final）,可见性（volatile 修饰符，是否强制从主内存读写）。

  **name_index:** 对常量池的引用，表示的字段的名称；

  **descriptor_index:** 对常量池的引用，表示字段和方法的描述符；

  **attributes_count:** 一个字段还会拥有一些额外的属性，attributes_count 存放属性的个数；

  **attributes[attributes_count]:** 存放具体属性具体内容

  ![](D:\WorkSpace\Note\Picture\类文件access_flags字段.png)

* 方法表集合

  Class文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式。方法表的结构如同字段表

  ![](D:\WorkSpace\Note\Picture\类文件access_flags方法.png)

* 属性表集合

  在 Class 文件，字段表，方法表中都可以携带自己的属性表集合，以用于描述某些场景专有的信息。与 Class 文件中其它的数据项目要求的顺序、长度和内容不同，属性表集合的限制稍微宽松一些，不再要求各个属性表具有严格的顺序，并且只要不与已有的属性名重复，任何人实现的编译器都可以向属性表中写 入自己定义的属性信息，Java 虚拟机运行时会忽略掉它不认识的属性

## 了解类加载过程吗？

Class文件需要加载到虚拟机中之后才能运行和使用

系统加载Class类型的文件主要三步：**加载->连接->初始化**，连接过程又分为三步：**验证->准备->解析**

* 加载

  1. 通过全类名获取定义此类的二进制字节流
  2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
  3. 在内存中生成一个代表该类的Class对象，作为方法区这些数据的访问入口

  一个非数组类的加载过程是可控性最强的阶段，这一步可以自定义类加载器去控制字节流的获取方式（重写一个类加载器的loadClass()方法），数组类型不通过类加载器创建，它由Java虚拟机直接创建

  加载阶段和连接阶段的部分内容是交叉进行的，加载阶段尚未结束，连接阶段可能就已经开始了

* 验证

  * 文件格式验证

    验证字节流是否符合Class文件格式的规范

  * 元数据验证

    对字节码描述的信息进行语义分析（对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求

  * 字节码验证

    最复杂的一个阶段，通过数据流和控制流分析，确定程序语义是合法、符合逻辑的

  * 符号引用验证

    确保解析动作能正确执行

* 准备

  准备阶段是正式为类变量分配内存并设置类变量初始值的阶段

  1. 这时候进行内存分配的仅包括类变量（Class Variables，即静态变量，被static修饰的变量，只与类相关，因此被称为类变量），而不包括实例变量。实例变量会在对象实例化时随着对象一块分配在Java堆中
  2. 从概念上，类变量所使用的内存都应当在方法区中分配。JDK1.7之后，HotSpot已经把原本放在永久代的字符串常量池、静态变量等移动到堆中，这个时候类变量则会随着Class对象一起存放在Java堆中

* 解析

  解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析过程主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用限定符7类符号引用进行

  符号引用就是一组符号来描述目标，可以是任何字面量。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

  综上，解析阶段时虚拟机将常量池内的符号引用替换为直接引用的过程，也就是得到类或者字段、方法在内存中的指针或偏移量

* 初始化

  初始化阶段是执行初始化方法\<cinit>方法的过程，是类加载的最后一步，这一步JVM才开始执行类中定义的Java程序代码

  对于\<cinit>方法的调用，虚拟机会自己确保其在多线程环境中的安全性，因为\<cinit>方法是带锁线程安全，所以在多线程环境下进行类初始化的话可能会引起多个线程阻塞，并且这种阻塞很难被发现

  对于初始化阶段，虚拟机严格规范了有且只有6种情况下，必须对类进行初始化

  1. 当遇到new、getstatic、putstatic或invokestatic这4条直接码指令时
     * 当JVM执行new指令时会初始化类，即创建一个类的实例对象
     * 当JVM执行getstatic指令时会初始化类，即程序访问类的静态变量（不是静态常量，常量会被加载到运行时常量池）
     * 当JVM执行putstatic指令时初始化类，即程序给类的静态变量赋值
     * 当JVM执行invokestatic指令时初始化类，即程序调用类的静态方法
  2. 使用java.lang.relect包的方法对类进行反射调用时，如果类没有初始化，需要触发其初始化
  3. 初始化一个类，如果其父类还未初始化，则先触发该父类的初始化
  4. 当虚拟机启动时，用户需要定义一个要执行的主类，虚拟机会先执行这个类
  5. MethodHandler和VarHandler可以看作时轻量级的反射调用机制，而想要用这两个调用，就必须先使用findStaticVarHandler来初始化要调用的类
  6. 当一个接口定义了JDK8新加入的默认方法时，如果由这个接口的实现类发生了初始化，那该类接口要在其之前被初始化

* 卸载

  卸载类即该类的Class对象被GC

  1. 该类的所有实例对象都已被GC，也就是堆不存在该类的实例对象
  2. 该类没有在其他任何地方被引用
  3. 该类的类加载器的实例已被GC

  在JVM生命周期内，由JVM自带的类加载器加载的类是不会被卸载的，但是由自定义的类加载器的类是可以被卸载的

## 了解那些类加载器？

JVM种内置了三个重要的ClassLoader，除了BootstrapClassLoader其他类加载器均由Java实现且全部继承自java.lang.ClassLoader

1. BootstrapClassLoader（启动类加载器）：最顶层的加载类，由C++实现，负责加载%JAVA_HOME%/lib目录下的jar包和类或者被-Xbootclasspath参数指定路径种的所有类
2. ExtensionClassLoader（扩展类加载器）：主要负责加载%JRE_HOME%/lib/ext目录下的jar包和类，或被java.ext.dirs系统变量所指定路径下的jar包
3. AppClassLoader（应用程序类加载器）：面向用户的加载器，负责加载当前应用classpath下所有的jar包和类

## 了解什么是双亲委派模型吗？

每一个类都有一个对应它的类加载器。系统中的ClassLoader在协同工作的时候会默认使用双亲委派模型。即在类加载的时候，系统会首先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。加载的时候，首先会把请求委派给父类加载器的loadClass()处理，因此所有的请求最终都应该传送到顶层的启动类加载器BootstrapClassLoader中。当父类加载器无法处理时，才由自己来处理，当父类加载器为null时，会使用启动类加载器BootstrapClassLoader作为父类加载器

AppClassLoader的父类加载器为ExtClassLaoder，ExtClassLoader的父类加载器为null，null并不代表没有父类加载器，而是BootstrapClassLoader

```java
private final ClassLoader parent;
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 首先，检查请求的类是否已经被加载过
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {//父加载器不为空，调用父加载器loadClass()方法处理
                    c = parent.loadClass(name, false);
                } else {//父加载器为空，使用启动类加载器 BootstrapClassLoader 加载
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                //抛出异常说明父类加载器无法完成加载请求
            }

            if (c == null) {
                long t1 = System.nanoTime();
                //自己尝试加载
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

## 使用双亲委派模型的好处？

双亲委派模型保证了Java程序的稳定运行，可以避免类的重复加载（JVM区分不同类的方式不仅仅根据类名，相同的类文件被不同的类加载器产生的是两个不同的类），也保证了Java的核心API不被篡改，如果没有使用双亲委派模型，而是每个类加载器加载自己的话，运行程序时，就会出现多个不同的相同名称功能的类

## 如何自定义类加载器？

除了BootstrapClassLoader其他类加载器都是由Java实现的且全部继承自java.lang.ClassLoader，如果自定义自己的类加载器，继承ClassLoader即可，如果不想打破双亲委派模型，那么重写ClassLoader类中的findClass()方法即可，如果想要打破双亲委派机制，那么需要重写loadClass()方法

## 使用过那些JVM参数？

* 堆内存相关

  * 显式指定堆内存-Xms和-Xmx

    与性能有关的最常用实践之一是根据应用程序要求初始化堆内存。如果我们需要指定最小和最大堆大小

    ```shell
    -Xms<heap size>[unit]
    -Xmx<heap size>[unit]
    ```

    * heap size表示要初始化内存的具体大小
    * unit表示要初始化内存的单位

  * 显式新生代内存

    在堆总可用内存配置完成之后，第二大影响因素是为Yong Generation在堆内存所占的比例，默认情况下，YG的最小大小为1310MB，最大大小为无限制

    1. 通过-XX:NewSize和-XX:MaxNewSize指定

       ```shell
       -XX:NewSize=<young size>[unit]
       -XX:MaxNewSize=<young size>[unit]
       ```

    2. 通过-Xmn\<young size>[unit]

    适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况

    还可以通过-XX:NewRatio=\<int>来设置老年代和新生代内存的比例

  * 显式指定元空间的大小

    JDK1.8的时候，方法区被彻底移除了，取而代之是元空间，元空间使用的是本地内存

    ```java
    -XX:MetaspaceSize=N #设置 Metaspace 的初始（和最小大小）
    -XX:MaxMetaspaceSize=N #设置 Metaspace 的最大大小，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。
    ```

* 垃圾收集相关

  * 垃圾回收器

    为了提高应用程序的稳定性，选择正确的垃圾收集算法至关重要

    * 串行垃圾收集器
    * 并行垃圾收集器
    * CMS垃圾收集器
    * G1垃圾收集器

    ```java
    -XX:+UseSerialGC
    -XX:+UseParallelGC
    -XX:+UseParNewGC
    -XX:+UseG1GC
    ```

  * GC日志记录

    生产环境上，或者卡要测试GC问题的环境上，一定会配置上打印GC日志的参数，便于分析GC相关的问题

    ```java
    # 必选
    # 打印基本 GC 信息
    -XX:+PrintGCDetails
    -XX:+PrintGCDateStamps
    # 打印对象分布
    -XX:+PrintTenuringDistribution
    # 打印堆数据
    -XX:+PrintHeapAtGC
    # 打印Reference处理信息
    # 强引用/弱引用/软引用/虚引用/finalize 相关的方法
    -XX:+PrintReferenceGC
    # 打印STW时间
    -XX:+PrintGCApplicationStoppedTime
    
    # 可选
    # 打印safepoint信息，进入 STW 阶段之前，需要要找到一个合适的 safepoint
    -XX:+PrintSafepointStatistics
    -XX:PrintSafepointStatisticsCount=1
    
    # GC日志输出的文件路径
    -Xloggc:/path/to/gc-%t.log
    # 开启日志文件分割
    -XX:+UseGCLogFileRotation
    # 最多分割几个文件，超过之后从头文件开始写
    -XX:NumberOfGCLogFiles=14
    # 每个文件上限大小，超过就触发分割
    -XX:GCLogFileSize=50M
    ```

## 使用过JDK监控和故障处理工具吗？

**`jps`** (JVM Process Status）: 类似 UNIX 的 `ps` 命令。用于查看所有 Java 进程的启动类、传入参数和 Java 虚拟机参数等信息；

**`jstat`**（JVM Statistics Monitoring Tool）: 用于收集 HotSpot 虚拟机各方面的运行数据;

**`jinfo`** (Configuration Info for Java) : Configuration Info for Java,显示虚拟机配置信息;

**`jmap`** (Memory Map for Java) : 生成堆转储快照;

**`jhat`** (JVM Heap Dump Browser) : 用于分析 heapdump 文件，它会建立一个 HTTP/HTML 服务器，让用户可以在浏览器上查看分析结果;

**`jstack`** (Stack Trace for Java) : 生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合

# Java新特性

## 了解interface的变化吗？

interface的设计初衷是面向抽象，提高扩展性。但是interface修改的时候，实现它的类也必须跟着修改

为了解决接口的修改与现有的实现不兼容的问题。新interface的方法可以用default或static修饰，这样就可以有方法体，实现类也不必重写此方法

一个interface中可以有多个方法被它们修饰，这两个修饰符的区别主要也是普通方法和静态方法的区别

1. default修饰的方法，是普通实例方法，可以用this调用，额可以被子类继承重写
2. static修饰的方法，使用上和类静态方法一样，但是他不能被子类继承，只能用Interface调用

## 了解functional interface函数式接口吗？

定义：也称为SAM接口，即只有一个抽象方法，但可以有多个非常抽象方法的接口

在Java8中专门有一个包放函数式接口java.util.funtion，该包下的所有接口都有@FunctionalInterface注解，提供函数式编程

是否为函数式接口与其有无@FunctionalInterface无关，注解只是在编译时起到强制规范定义的作用

## 了解Stream流吗？



# 计算机网络

## TCP和UDP的区别？

* 是否面向连接：

  UDP在传送数据之前不需要先建立连接，而TCP提供面向连接的服务，在传送数据之前必须先建立连接，数据传送结束后要释放连接

* 是否是可靠传输：

  远地主机在收到UDP报文之后，不需要给出任何确认，并且不保证数据不丢失，不保证是否顺序送达。TCP提供可靠的传输服务，TCP在传递数据之前，会有三次握手来建立连接，而且在数据传递时，有确认、窗口、重传、拥塞控制机制。通过TCP连接传输的数据，无差错、不丢失、不重复、并且按序到达

* 是否有状态：

  这个与是否可靠传输相对应。TCP传输是有状态的，这个有状态说的是TCP会区记录自己发送消息的状态。为此，TCP需要维持复杂的连接状态表。而UDP是无状态服务，简单来说就是不管发出去之后的事情

* 传输效率：

  由于使用TCP进行传输的时候多了连接、确认、重传等机制，所以TCP的传输效率要比UDP低很多

* 传输形式：

  TCP是面向字节流的，UDP是面向报文的

* 首部开销：

  TCP首部开销（20-60字节）比UDP首部开销（8字节）大

* 是否提供广播或多播服务：

  TCP只支持点对点通信，UDP支持一对一、一对多、多对一、多对多

## 什么时候选择TCP、UDP？

* UDP一般用于即时通信：比如语音、视频、直播等。这些场景对传输数据的准确性要求不是特别高
* TCP用于对传输准确性要求特别高的场景，比如文件传输、发送和接收邮件、远程登录等等

## HTTP基于TCP还是UDP？

HTTP协议是基于TCP协议的，所以发送HTTP请求之前首先要建立TCP连接也就是经历3次握手

## 使用TCP的协议有哪些？使用UDP的协议有哪些？

* 运行于TCP协议之上的协议:
  * HTTP协议：超文本传输协议主要是为Web浏览器和Web服务器之间的通信而设计的
  * HTTPS协议：更安全的超文本传输协议，身披SSL外衣的HTTP协议
  * FTP协议：文件传输协议FTP，提供文件传输服务，基于TCP实现可靠的传输。使用FTP传输文件的好处时可以屏蔽操作系统和文件存储方式
  * SMTP协议：简单邮件传输协议，基于TCP协议，用来发送电子邮件
  * POP3/IMAP协议：POP3和IMAP两者都是负责邮件接收的协议
  * Telent协议：远程登录协议，通过一个终端登录到其他服务器
  * SSH协议：SSH是目前较为可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用SSH协议可以有效防止远程管理过程中的信息泄露问题
* 运行于UDP协议之上的协议：
  * DHCP协议：动态主机配置协议，动态配置IP地址
  * DNS：域名系统将人类可读的域名转换为机器可读的IP地址，实际上DNS同时支持UDP和TCP协议

## 从输入URL到页面展示到底发生了什么？

1. DNS解析
2. TCP连接
3. 发送HTTP请求
4. 服务器处理请求并返回HTTP报文
5. 服务器解析渲染界面
6. 连接结束

## HTTP状态码有哪些？

![](D:\WorkSpace\Note\Picture\HTTP状态码.png)

## HTTP和HTTPS有什么区别？

* 端口号：HTTP默认为80，HTTPS默认为443

* URL前缀：HTTP的URL前缀是http://，HTTPS前缀是https://

* 安全性和资源消耗：

  HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份

  HTTPS是运行在SSL/TLS之上的HTTP协议，SSL/TLS运行在TCP之上。所有传输的内容都是经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说HTTP安全性没有HTTPS高，但是HTTPS比HTTP消耗更多服务器的资源

## HTTP1.0和HTTP1.1有什么区别？

* 连接方式：HTTP1.0为短链接，HTTP1.1支持长连接

* 状态响应码：HTTP/1.1中新加入了大量的状态码，光是错误状态码新增了24种

* 缓存处理：

  在HTTP1.0中主要使用header里的If-Mofified-Since，Expires来作为缓存判断标准

  HTTP1.1则引入了更多的缓存控制策略例如Entity tag，If-Unmodified-Since，If-Match，If-None-Match等更多可供选择的缓存头来控制缓存策略

* 带宽优化及网路连接的使用：

  在Http1.0中存在一些浪费带宽的现象，例如客户端只是需要某个对象的一部分，而服务器却将整个对象送过来了，并且不支持断点续传功能

  HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206，这样就方便了开发者自由的选择以便于充分利用带宽和连接

* Host头处理：Http/1.1在请求头中加入了Host字段

## HTTP是不保存状态的协议，如何保存用户状态？

HTTP是一种不保存状态，即无状态协议。也就是说HTTP协议自身不对i请求和响应之间的通信状态进行保存

Session机制的存在就是为了解决这个问题，Session的主要作用就是通过服务端记录用户的状态

服务端保存Session的方法有很多，最常用的就是内存和数据库

## URI和URL的区别是什么？

* URI是统一资源标志符，可以唯一标识一个资源
* URL是统一字段定位符，可以提供该资源的路径，是一种具体的URI

## 什么是Mac地址？

MAC地址的全称是媒体访问控制地址。如果说，互联网中每一个资源都由IP地址唯一标识，那么一切网络设备都由MAC地址唯一标识

可以理解为，MAC地址是一个网络设备真正的身份证号，IP地址只是一种不重复的定位方式

MAC地址的长度为6字节（48比特），地址空间大小有280万亿多，MAC地址由IEEE统一管理和分配，理论上，一个网路设备中的网卡上的MAC地址是永久的，不同的网卡生产商从IEEE哪里购买自己的MAC地址空间，也就是钱24比特由IEEE统一管理，保证不会重复。而后24比特，由各家生产商自己管理，同样保证生产的两块网卡的MAC地址不会重复

MAC地址具有可携带性、永久性

MAC地址有一个特殊地址：FF-FF-FF-FF-FF-FF，该地址表示广播地址

## ARP协议解决了什么问题？

ARP协议，全称地址解析协议，他解决的是网络层地址和链路层地址之间的转换问题，因为一个IP数据报在物理上传输的过程中，总是需要知道物理上的下一个目的地在哪里，但IP地址属于逻辑地址，而MAC地址才是物理地址，ARP协议解决了IP地址转MAC地址的问题

## 了解OSI七层模型吗？

OSI七层模型是国际标准化组织提出一个网络分层模型，其大体结构以及每一层提供的功能

![](D:\WorkSpace\Note\Picture\OSI.png)

每一层都专注做一件事，并且每一层都需要使用下一层提供的功能

OSI的七层结构概念清除，理论也很完整，但是它比较复杂而且不适用，而且有些功能咋多个层中重复出现

## OSI七层模型和TCP/IP四层模型区别

* OSI的专家缺乏实际经验，它们完成OSI标准时缺乏商业驱动力
* OSI的协议实现起来过分复杂，而且运行效率很低
* OSI指定标准的周期太长，因而使得按OSI标准生产设备无法及时进入市场
* OSI的层次划分不太合理，有些功能在多个层次中重复出现

![](D:\WorkSpace\Note\Picture\OSI详解.png)

## 了解TCP/IP四层模型吗？

TCP/IP四层模型是目前被广泛次啊用的一种模型，我们可以将TCP/IP模型看作是OSI七层模型的精简版本

1. 应用层
2. 传输层
3. 网络层
4. 网络接口层

![](D:\WorkSpace\Note\Picture\OSI对应TCPIP.png)

* 应用层

  应用层位于传输层之上，主要提供两个终端设备上的应用程序之间信息交换服务，它定义了信息交换的格式，消息会交给下一层传输层来传输

  应用层交互的数据单元称为报文

  应用层协议定义了网络通信规则，对于不同的网络应用需要不同的应用层协议

* 传输层

  传输层的主要任务就是负责向两台终端设备进程之间的通信提供通用的数据传输服务

  应用进程利用该服务传送应用层报文

  * 传输控制协议TCP
  * 用户数据协议UDP

* 网络层

  网络层负责为分组交换网上的不同的主机提供通信服务

  在发送数据时，网络层把运输层的报文段或用户数据报封装成分组和包进行传送

  在TCP/IP体系结构中，由于网络层使用IP协议，因此分组也叫IP数据报

  网络层还有一个任务就是选择合适的路由，使源主机运输层传送下来的分组，能通过网络层中的路由器找到目的主机

* 网络接口层

  可以将网络接口层看作是数据链路层和物理层的合体

  1. 数据链路层通常称为链路层（两台主机之间的数据传输，总在一段一段的链路上传送的），数据链路层的作用是将网络交下来的IP数据报组装成帧，在两个相邻节点间的链路上传送帧。每一帧包括数据和必要的控制信息
  2. 物理层的作用是实现相临计算机系欸点那之间比特流的透明传送，尽可能屏蔽掉具体传输介质和物理设备的差异

## 什么是HTTP吗？

HTTP协议，超文本传输协议，用来规范超文本的传输

超文本，也就是网络上的包括文本在内的各式各样的消息，具体来说，主要是来规范浏览器和服务器端的行为

HTTP是一种无状态的协议，也就是说服务器不维护任何有关客户端过去所发请求的消息

## 了解HTTP协议通信过程吗？

HTTP是应用层协议，它以TCP作为底层协议，默认端口为80

1. 服务器在80端口等待客户的请求
2. 浏览器发起到服务器的TCP连接（创建套接字Socket）
3. 服务器接收来自浏览器的TCP连接
4. 浏览器（HTTP客户端）与Web服务器（HTTP服务器）交换HTTP消息
5. 关闭TCP连接

## 什么是HTTPS？

HTTPS协议，是HTTP的加强安全版，HTTPS是基于HTTP的，也是用TCP作为底层协议，并额外使用SSL/TLS协议用作加密和安全认证，默认端口为443

HTTPS协议中，SSL通道通常使用基于密钥的加密算法，密钥长度通常是40比特或128比特

## SSL和TLS的区别？

SSL和TLS没有太大的区别

SSL指安全套接字协议，SSL3.0进一步升级，新版本被命名为TLS1.0

因此TLS是基于SSL之上的，但由于习惯通常把HTTPS中的核心加密协议混成为SSL/TLS

## SSL/TLS的工作原理了解吗？

* 非对称加密

  SSL/TLS的核心要素是非对称加密

  非对称加密采用两个密钥——一个公钥，一个私钥

  在通信时，私钥仅由解密者保存，公钥由任何一个想与解密者通信的发送者所知

  非对称加密的公钥和私钥需要采用一种复杂的数学机制生成，公私钥对的生成算法依赖于单向陷门函数

  > 单向陷门函数——>根据x很容易得到y，但是根据y不容易得到x，就是单向函数，如果有陷门h，能够很轻易通过y得到x

* 对称加密

  使用SSL/TLS进行通信的双方需要使用非对称加密方案来通信，但是非对称加密设计较为复杂的算法，在实际通信过程中，计算的代价较高，效率太低，因此，SSL/TLS实际对消息加密使用对称加密

## 为什么SSL/TLS还需要使用非对称加密？

因为对称加密的保密性完全依赖于密钥的保密性。在双方通信之前，需要商量一个用于对称加密的密钥，网络通信的信道是不安全的，传输报文对任何人都是可见的，密钥的交换肯定不能直接在网络信道中传输。因此使用非对称加密，对对称加密的密钥进行加密，保护该密钥不再网络信道中被窃听，这样通信双方只需要一次非对称加密，交换对称加密的密钥，在之后的信息通信中，使用绝对安全的密钥，对信息进行对称加密，即可保证传输信息的保密性

## 怎么保证公钥的信赖性问题？

为了公钥传输的信赖性问题，第三方机构应运而生——证书颁发机构。CA默认是受信任的第三方，CA会给各个服务器颁发证书，整数存储在服务器上，并附有CA的电子签名

当客户端向服务器发送HTTPS请求时，一定要先获取目标服务器的证书，并且根据证书上的信息，检验证书的合法性。一旦客户端检测到证书非法，就会发生错误。客户端获取了服务器的证书后，由于证书的信任性事第三方依赖机构认证的，而证书上又包含着服务器的公钥信息，客户端就可以放心使用证书上的公钥

## 什么是数字签名？

数字签名是防止证书被伪造，第三方信赖机构CA之所能被信赖，就是靠数字签名技术

数字签名，是CA在给服务器颁发证书时，使用散列+加密的组合技术，在证书上盖章，以此来提供验伪的功能

## 建立连接-TCP三次握手了解吗？

![](D:\WorkSpace\Note\Picture\三次握手.png)

建立一个TCP连接需要三次握手

* 第一次握手：客户端发送带有SYN（SEQ=x）标志的数据包->服务端，然后客户端进入SYN_SEND状态，等待服务器的确认
* 第二次握手：服务端发送带有SYN+ACK（SEQ=y,ACK=x+1）标志的数据包->客户端，然后服务端进入SYN_RECV状态
* 第三次握手：客户端发送带有ACK（ACK=y+1）标志的数据包->服务端，然后客户端和服务端都进入ESTABLISHED状态，完成TCP三次握手

## 为什么要三次握手？

三次握手的目的是建立可靠的通信信道，三次握手的主要目的就是双方确认自己与对方的发送与接收是正常的

1. 第一次我手：Client什么都不能确认；Server确认了对方发送正常，自己接收正常
2. 第二次握手：Client确认了：自己发送正常、接收正常，对方发送接收正常；Server确认了：对方发送正常，自己接收正常
3. 第三次握手：Client确认了：自己发送接收正常，对方发送接收正常：Server确认了：自己发送接收正常，对方发送接收正常

## 断开连接-TCP四次挥手了解吗？

![](D:\WorkSpace\Note\Picture\四次挥手.png)

1. 第一次挥手：客户端发送一个FIN（SEQ=X）标志的数据包->服务端，用来关闭客户端到服务器的数据传送。然后，客户端进入FIN-WAIT-1状态
2. 第二次挥手：服务器收到这个FIN（SEQ=X）标志的数据包，它发送一个ACK（SEQ=X+1）标志的数据包->客户端。然后，此时服务端进入CLOSE-WAIT状态，客户端进入FIN-WAIT-2状态
3. 第三次挥手：服务端关闭和客户端的连接并发送FIN（SEQ=y）标志的数据包->客户端请求关闭连接，然后服务端进入LAST-ACK状态
4. 第四次挥手：客户端发送ACK（SEQ=y+1）标志的数据包->服务端并且进入TIME-WAIT状态，服务端在收到ACK（SEQ=y+1）标志的数据包后进入CLOSE状态，此时，如果客户端等待2MSL后依然没有收到回复，就证明客户端已正常关闭，随后，客户端也可以不关闭连接了

## 为什么要四次挥手？

TCP是全双工通信，可以双向传输数据，任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接

## 为什么不能将ACK和FIN合并起来?

因为服务器收到客户端断开连接的请求时，可能还有一些数据没有发完，这时先回复ACK，表示接收到了断开俩姐的请求。等到数据发完之后再发FIN，断开服务器到客户端的数据传送

## TCP如何保证传输的可靠性？

1. 基于数据块传输：应用数据被分割成TCP认为最适合的数据块，再传输给网络层，数据块被称为报文段或段
2. 对失序的数据包重新排序以及去重：TCP为了保证不发生丢包，就给每个包一个序列号，有了序列号能够将接收到的数据根据序列号排序，并且去掉重复序列号的数据就可以实现数据包去重
3. 校验和：TCP将保证它首部和数据的校验和，这是一个端到端的校验和，目的是检测数据再传输过程中的任何变化。如果收到段的检验和有差错，TCP将丢弃这段报文段和不确定接收这个报文段
4. 超时重传：当发送方发送数据之后，它启动一个定时器，等待目的端确认收到这个报文段。接收端实体对已成功收到的包发回一个相应的确认信息。如果发送端实体在合理的往返延迟内未收到确认信息，那么对应的数据包就被假设为已丢失并进行重传
5. 流量控制：TCP连接的每一方都有固定大小的缓冲空间，TCOP的接收端只允许发送端发送接收端缓冲区能接纳的数据。当接收方来不及处理发送方数据时，能提示发送方降低发送速率，防止包丢失。TCP使用的流量控制协议时可变大小的滑动窗口协议
6. 拥塞控制：当网络拥塞时，减少数据的发送

## TCP如何实现流量控制？

 TCP利用滑动窗口实现流量控制。流量控制是为了控制发送方的发送速率，保证接收方来得及接收

## 为什么需要流量控制？

双方在通信的时候，发送方的速率与接收方的速率是不一定相等，如果发送方的发送速率太快，会导致接收方处理不过来，如果接收方处理不过来，就只能把处理不过来的数据存放在接收缓冲区里。如果缓冲区满了发送方孩子啊发数据，接收方只能把收到的数据丢掉，出现丢包问题的同时浪费了网络资源

TCP为全双工通信，双方可以进行双向通信，客户端和服务端既可能是发送端有可能是服务端。因此，两端各有一个发送缓冲区和接收缓冲区，两端各自维护一个发送窗口和一个接收窗口。接收窗口大小取决于应用、系统、硬件的限制。通信双方的发送窗口和接收窗口的要求相同

TCP发送窗口可以划分为四个部分：

1. 已经发送并且确认的TCP段（已经发送并确认）
2. 已经发送但是没有确认的TCP段（已经发送未确认）
3. 未发送但是接收方准备接收的TCP段（可以发送）
4. 未发送并且接收方也未准备接收的TCP段（不可发送）

TCP接收窗口可以划分为三个部分：

1. 已经接收并且已经确认的TCP段（已经接收并且确认）
2. 等待接收且允许发送方发送TCP段（可以接收未确认）
3. 不可接收且不允许发送方发送TCP段（不可接收）

接收窗口大小是根据接收端处理数据的速度动态调整的。如果接收读取数据快，接收窗口可能会扩大。否则缩小

## TCP的拥塞控制是怎么实现的？

为了进行拥塞控制，TCP发送方要维持一个拥塞窗口的状态变量。拥塞控制窗口的大小取决于网络的拥塞程度，并且动态变化。发送方让自己的发送窗口取为拥塞窗口和接收方的接受窗口中较小的一个

TCP的拥塞控制采用了四种算法。在网络层也可以使路由器采用适当分组丢弃策略，以减少网络拥塞的发生

* 慢开始：慢开始算法的思路是当主机开始发送数据时，如果立即把大量数据字节注入到网络，那么可能引起网络阻塞，因为现在还不知道网络的符合情况。较好的方法时先探测一下，即由小到大逐渐增大发送窗口，也就是由小到大逐渐增大拥塞窗口数值，cwnd初始值为1，没经过一个传播轮次，cwnd加倍
* 拥塞避免：拥塞避免算法的思想是让拥塞窗cwnd缓慢增大，即没经过一个往返时间RTT就把发送方的cwnd加1
* 快重传与快恢复：如果接收机接收到一个不按顺序的数据段，他会立即给发送机发送一个重复确认，如果发送机接收到三个重复确认，他会假定确认件指出的数据段丢失了，并立即重传这些丢失的数据。当有单独的数据包丢失时，快速重传和恢复能最有效的工作，当有多个数据信息报在某一个短时间内丢失，他则不能很有效的工作

# MySQL

## 了解Mysql基础架构吗?

![]()

* 连接器:身份认证和权限相关(登录MySQL的时候)
* 查询缓存: 执行查询语句的时候,会先查询缓存(MySQL8.0版本后移除,因为这个功能不太实用)
* 分析器:没有命中缓存的话, SQL语句就会经过分析器,分析器会分析你的SQL语句要干什么, 有没有语法错误
* 优化器: 按照MySQL认为最优的方案去执行
* 执行器:执行语句, 然后从存储引擎返回数据. 执行语句之前会先判断是否有权限, 如果没有权限,就会报错
* 插件式存储引擎:主要负责数据的存储和读取, 采用的是插件式架构, 支持InnoDB, MyIASM, Memory等多种存储引擎

## MySQL存储引擎了解吗?

MySQL支持多种存储引擎, 你可以通过show engines命令查看MySQL支持的所有存储引擎

MySQL当前默认的存储引擎是InnoDB, 并且, 所有的存储引中只有InnoDB是事务性存储引擎, 也就是说只有InnoDB支持事务

MySQL5.5.5之前, MyISAM是MySQL的默认存储引擎

`select version()`可以查看当前MySQL版本号

`show variables like '%Storage_engine%'`命令直接查看MySQL当前默认的存储引擎

`show table status from (db_name) where name = '(table_name)'`查看数据库中某个表使用的存储引擎

## MySQL存储引擎架构了解吗?

MySQL存储引擎采用的是插件式架构,支持多种存储引擎,可以为不同的数据库表设置不同的存储引擎以适应不同场景的需要.存储引擎是基于表的,而不是数据库

并且,可以根据MySQL定义的存储引擎实现标准接口来编写一个属于自己的存储引擎

## MyISAM与InnoDB有什么区别?

* 是否支持行级锁

MyISAM只有表级锁,而InnoDB支持行级锁和表级锁默认为行级锁

* 是否支持事务

MyISAM不提供事务支持

InnoDB提供事务支持,实现了SQL标准定义了四个隔离级别,具有提交和回滚的事务能力

并且,InnoDB默认使用的REPEATEABLE-READ(可重读)隔离级别是可以解决幻读问题

* 是否支持外键

MyISAM不支持,而InnoDB支持

外键对于维护数据一致性非常有帮助,但是对性能有一定的损耗

**通常情况下,我们不建议在实际生产项目中使用外键,在业务代码中进行约束即可**

* 是否支持数据库异常崩溃后的安全回复

MyISAM不支持,而InnoDB支持

使用InnoDB的数据库在异常崩溃后,数据库重新启动的时候会保证数据库恢复到崩溃前的状态,这个恢复过程依赖于redo log

* 是否支持MVCC

MyISAM不支持,而InnoDB支持

MVCC可以看做是行级锁的升级,可以有效减少锁操作,提高性能

* 索引实现不一样

虽然MyISAM引擎和InnoDB引擎都是使用B+Tree作为索引结构,但是两者的实现方式不太一样

InnoDB引擎中,其数据文件本身就是索引文件.相比于MyISAM,索引文件和数据文件是分离的,其表数据文件本身就是按B+Tree组织的一个索引结构,树的叶子结点data域保存了完整的数据记录

* 性能有差异

InnoDB的性能比MyISAM更强大,不管是在读写混合模式下还是只读模式下随着CPU核心数的增加,InnoDB的读写能力呈线性增长.MyISAM

## 何谓数据库事务？

* 原子性：事务是最小的执行单位，不允许分割，事务的原子性确保动作要么全部完成，要么完全不起作用
* 一致性：执行事务前后，数据保持一致，无论数据是否成功，双方数据应该是不变的
* 隔离性：并发访问数据库时，一个用户的事务不被其他事务所干扰，各并发事务之间数据库是独立的
* 持久性：一个事务被提交之后，它对数据库中数据的改变是持久的，数据库发生故障也不应该对其有任何影响

## 并发事务带来了哪些问题？

* 脏读

  一个事务读取数据并且对数据进行了修改，这个修改对其他事务来说是可见的，即使当前事务没有提交。这时另外一个事务读取了这个还未提交的数据，但第一个事务突然回滚，导致数据并没有被提交到数据库，那第二个事务读取到的就是脏数据

* 丢失修改

  在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结构就被丢失，因此称为丢失修改

* 不可重复读

  值在一个事务内多次读同一个数据，在这个事务还没有结束时，另外一个事务也访问该数据，那么在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取数据可能不太一样，这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读

* 幻读

  幻读与不可重复读类似，它发生在一个事务读取了几行数据，接着另一个并发事务插入了一些数据时，在随后的查询中，第一个数据就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读

## 并发事务的控制方式有哪些？

MySQL中并发事务的控制方式无非就两种：锁和MVCC

锁可以看作是悲观控制的模式，多版本并发控制可以看作乐观控制的模式

**锁** 控制方式下会通过锁来显式控制共享资源而不是通过调度手段，MySQL中主要通过读写锁来实现并发控制

* 共享锁（S锁）：又称为读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取
* 排他锁（X锁）：又称写锁/独占锁，事务在需改记录的时候获取排他锁，不允许多个事务同时获取，如果一个记录已经被加了排他锁，那其他事务不能再对这条记录加任何类型的锁

读写锁可以做到读读并行，但是无法做到写读、写写并行。另外根据锁粒度不同，又被分为表级锁和行级锁

MVCC是多版本并发控制方法，即对一份数据会存储多个版本，通过事务的可见性来保证事务能看到自己应该看到的版本。通常会有一个全局的版本分配器来为每一行数据设置版本号，版本号是唯一的

MVCC在MySQL中实现所依赖的手段主要是：隐藏字段、read view、undo log

* undo log：undo log用于记录某行数据的多版本的数据
* read view和隐藏字段：用来判断当前版本数据的可见性

## SQL标准定义了哪z些事务隔离级别？

* READ-UNCOMMITED（读取未提交）：最低的隔离界别，允许读取尚未提交的数据更改，可能会导致脏读、幻读或不可重复读
* READ-COMMMITED（读取已提交）：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
* REPEATABLE-READ（可重复读）：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以组织脏读和不可重复读，但幻读仍有可能发生
* SERIALIZABLE（可串行化）：最高的隔离级别，完全服从ACID的隔离级别。所有的事务一次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读

## InnoDB有哪些行锁？

InnoDB行锁是通过对索引数据页上的记录枷锁实现的

* 记录锁：也被称为记录锁，属于单个行记录上的锁
* 间隙锁：锁定一个范围，不包括记录本身
* 临间锁：记录锁+间隙锁，锁定一个范围，包含记录本身，主要目的是为了解决幻读问题。记录锁只能锁已经存在的记录，为了避免插入新纪录，需要依赖间隙锁

## 共享锁和排他锁喃？

不论是行级锁还是表级锁，都存在共享锁和排他锁

* 共享锁：又称读锁，事务在读取记录的时候获取共享锁，允许多个事务同时获取
* 排他锁：又称写锁/独占锁，事务在修改记录的时候获取排他锁，不允许多个事务同时获取。如果一个记录已经被加了排他锁，那其他事务不能再对这条事务加任何类型的锁

由于MVCC的存在，对于一般的SELECT语言，Inn哦DB}不会加其他锁，不过，你可以通过以下语句显式加共享锁或排他锁

```sql
# 共享锁
SELECT ... LOCK IN SHARE MODE;
# 排他锁
SELECT ... FOR UPDATE;
```

## 意向锁有什么作用？

如果需要用到表锁的话，如果判断表中的记录没有行锁？

使用意向锁来快速判断是否可以对某个表使用表锁

* 意向共享锁：事务有意向对表中的某些记录加共享锁，加共享锁前必须先取得该表的IS锁
* 意向排他锁：事务有意向对表中的某些记录加排他锁，加排他锁，加排他锁之前比u先取得该表的IX锁

意向锁是所有数据引擎自己维护的，用户无法手动操作意向锁，再为数据行加共享/排他锁之前，InnoDB会先获取该数据行所在数据表的对应意向锁

意向锁之间是相互兼容的

## 当前读和快照读有什么区别？

**快照读**就是单纯的SELECT语句但不包括下面这两个类SELECT语句

```sql
SELECT ... FOR UPDATE
SELECT ... LOCK IN SHARE MODE
```

快照即记录的历史版本，每行记录可能存在多个历史版本

快照读的情况下，如果读取的记录正在执行UPDATE/DELETE操作，读取操作不会因此取等待记录上的X锁释放而是会读取行的一个快照

只有再事务隔离级别RC（读取已提交）和RR（可重读）下，INn哦DB才会使用一致性非锁定读

* 在RC级别下，对于快照数据，一致性非锁定读总是读取被锁定行的最新一份快照数据
* 在RR级别下，对于快照数据，一致性非锁定读总是读取本事务开始时的行数据版本

快照读比较适合对于数据一致性要求不是特别高且追求极致性能的业务场景

**当前读**就是给行记录加X锁或S锁

## MySQL如何存储IP地址？

可以将IP地址转化成整形数据结构，性能更好，占用空间更小

* INET_ATON()：把ip转为无符号整型(4-8位)
* INET_NTOA()：把整型的ip转换位地址

## 如何分析SQL的性能？

使用EXPALIN命令来分析SQL的执行计划。执行计划是指一条SQL语言在经过MySQL查询优化器的优化后，具体的执行方式

EXPLAIN适用于SELECT、DELETE、INSERT、REPLACE和UPDATE语句

| 列名          | 含义                                         |
| ------------- | -------------------------------------------- |
| id            | SELECT查询的序列标识符                       |
| select_type   | SELECT关键字对应的查询类型                   |
| table         | 用到的表名                                   |
| partitions    | 匹配的分区，对于未分区的表，值为NULL         |
| type          | 表的访问方法                                 |
| possible_keys | 可能用到的索引                               |
| key           | 实际用到的索引                               |
| key_len       | 所选索引的长度                               |
| ref           | 当使用索引等值查询时，与索引作比较的列或常量 |
| rows          | 预计要读取的行数                             |
| filtered      | 按表条件过滤后，留存的记录数的百分比         |
| extra         | 附加信息                                     |

# Redis

# Spring&SpringBoot

# MyBatis

# Netty

# 设计模式

