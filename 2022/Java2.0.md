# Java的基本程序设计结构

## 整数

| 类型  | 存储需求 | 取值范围                                 |
| ----- | -------- | ---------------------------------------- |
| int   | 4字节    | -2147483648~2147483647（刚好超过20亿）   |
| short | 2字节    | -32768~32767                             |
| long  | 8字节    | -9223372036854775808~9223372036854775807 |
| byte  | 1字节    | -128~127                                 |

## 浮点类型

| 类型   | 存储需求 | 取值范围                                        |
| ------ | -------- | ----------------------------------------------- |
| float  | 4字节    | 大约±3.40282347E+38F（有效位数为6-7位）         |
| double | 8字节    | 大约±1.79769313486231570E+308（有效位数为15位） |

* 正无穷大——Double.POSITIVE_INFINITY
* 负无穷大——Double.NEGATIVE_INFINITY
* NaN——Double.NaN

> **所有非数值（NaN）都是不相同的，需要使用Double.isNaN方法来判断**
>
> 浮点数值不适用于无法接受舍入误差的金融计算，**因为浮点数值采用二进制系统表示，而在二进制系统中无法精确的表示1/10**，如果不允许出现这种舍入误差，那么应该使用BigDecimal类

## char类型

char类型的值可以表示为十六进制值，其范围从\u0000到\uFFFF

**Unicode转义序列会在解析代码之前得到处理**

## Unicode和char类型

码点是指与一个编码表中的某个字符对应的代码值

UTF-16——在基本多语言平面中，每个字符用16位表示，通常称为代码单元，辅助字符编码为一堆连续的代码单元

**不要在程序中使用char类型，除非确实需要处理UTF-16代码单元，可以使用字符串作为抽象数据类型处理**

## 算术运算符

整数被0除会产生一个异常，而**浮点数被0除将会得到无穷大或NaN结果**

> double类型使用64位存储一个数值，而有些处理器则使用80位浮点寄存器
>
> 很多处理器计算x*y，并且将结果存储在80位的寄存器中，再除以z并将结果截断为64位，这样的结果可能与始终使用64位计算的结果不同
>
> 默认情况下，现在虚拟机设计者允许对中间计算结果采用扩展的精度，但是对于使用strictfp关键字标记的方法必须使用严格的浮点计算来生成可用的结果
>
> **两种方式的区别在于采用默认方式不会产生溢出，而采用严格的计算有可能产生溢出**

## 数学函数与常量

floorMod方法类似于%，如果参数为负数，结果为-1

> Math类为了达到最佳性能，所有方法都是用计算机浮点单元中的例程，这样不利于移植性
>
> 如果得到一个完全可预测的结果比运行速度更加重要的话，那么可以使用StrictMath，它实现了自由分发的数学库，确保在所有平台得到相同结果

> Math提供了一些方法使整数有更好的运算安全性，使用正常的运算符只会返回错误结果并且不做任何提示，但是使用Math提供的方法，会产生异常，可以捕获异常或者让程序终止（multiplyExact、addExact、subtractExact、incrementExact、decrementExact和negateExact）

## 括号与运算符级别

| 运算符                                                       | 结合性   |
| ------------------------------------------------------------ | -------- |
| [] . ()（方法调用）                                          | 从左向右 |
| ! ~ ++ -- +（一元运算符） -（一元运算符） ()（强制类型转换） new | 从右向左 |
| * / %                                                        | 从左向右 |
| + -                                                          | 从左向右 |
| << >> >>>                                                    | 从左向右 |
| < <= > >= instanceof                                         | 从左向右 |
| == !=                                                        | 从左向右 |
| &                                                            | 从左向右 |
| ^                                                            | 从左向右 |
| \|                                                           | 从左向右 |
| &&                                                           | 从左向右 |
| \|\|                                                         | 从左向右 |
| ?:                                                           | 从右向左 |
| = += -= *= /= %= &= \|= ^= <<= >>= >>>=                      | 从右向左 |

## 检测字符串是否相等

如果虚拟机始终将相同的字符串共享，就可以使用==运算符检测是否相等，**但实际上只有字符串字面量是共享的，而+或substring等操作得到的字符串并不是共享的**

Java字符串由char值序列组成，最常用的Unicode字符使用一个代码单元就可以表示而辅助字符需要一对代码单元表示

## 大数

使用valueOf方法可以将普通的数值转换为大数，或者可以通过一个带字符串参数的构造器

还有一些产量：BigInteger.ZERO，BigInteger.ONE和BigInteger.TEN

大数可以使用add、multiply、subtract、mod、divide

## 访问数组元素

创建一个数字数组时，所有元素都初始化为0，boolean数组的元素会初始化为false，对象数组的元素则初始化为一个特殊值null

使用for each循环，**必须是一个数组后者是一个实现了Iterable接口的类对象**

## 数组拷贝

如果希望将一个数组的所有值拷贝到一个新的数组中去，可以使用Arrays类的copyOf方法

`int[] newArray = Arrays.copyOf(oldArray, oldArray.length);`

第二个参数是新数组的长度，**这个方法通常用来增加数组的大小**

## 多维数组

可以通过`Arrays.deepToString(a));`快速打印一个二维数组的数据元素列表

# 对象与类

## 类之间的关系

![image-20221012075713066](D:\WorkSpace\Note\Picture\image-20221012075713066.png)

## 对象与对象变量

**对象变量并没有实际包含一个对象，它只是引用一个对象**

## Java类库中的LocalDate类

标准Java类库分别包含了两个类：

* 一个是用来表示时间点的Date类
* 另一个是用日历表示法表示日期的LocalDate类

## 封装的优点

**不要编写分会可变对象引用的访问器方法，这种操作破坏了封装性**

## final实例字段

将字段定义为final，**这种字段必须在构造对象时初始化**，并且以后不能修改这个字段

只是表示存储在变量中的对象引用不会再指示另一个不同的对象，但是这个对象是可以修改的

## 静态常量

**原生方法可以绕过Java语言的访问控制机制**

## 工厂方法

使用工厂方法的原因——

* 无法命令构造器，当希望有两个不同名字的构造方法，能够分别得到不同功能的对象
* 使用构造器时，无法改变所构造对象的类型

## 方法参数

Java程序设计总是采用按值调用，也就是说，**方法得到的是所有参数值的一个副本**

* 方法不能修改基本数据类型的参数
* 方法可以改变对象参数的状态
* 方法不能让一个对象参数引用一个新的对象

## 重载

**同命不同参**

## 初始化块

**只要构造这个类的对象**，这个块就会被执行

1. 如果构造器的第一行调用了另一个构造器，则基于所提供的参数执行第二个构造器
2. 否则，
   1. 所有数据字段初始化为默认值（0、false或null）
   2. 按照在类生命中出现的顺序，执行所有字段初始化方法和初始化块
3. 执行构造器主体代码

**在类第一次加载的时候**，将会进行静态字段的初始化

所有的静态字段初始化方法以及静态初始化块都将**依照类声明中出现的顺序执行**

## 类的引用

使用星号，只能导入一个包

## 类设计技巧

* 一定要保证数据私有
* 一定要对数据进行初始化
* 不要在类中使用过多的基本类型
* 不是所有的字段都需要单独的字段访问器和字段更新器
* 分解有过多职责的类
* 类名分方法名要能够体现他们的职责
* 优先使用不可变的类

# 继承

## 覆盖方法

super不同于this，并不是一个对象的引用

## 子类构造器

**一个对象变量可以指示多种实际类型的现象称为多态**

**在运行时能够自动地选择适当的方法，称为动态绑定**

## 多态

将子类当作父类，向上转型，不能使用子类的方法

**所有数组都要牢记创建时的元素类型，并负责监督仅将类型兼容的引用存储到数组中**

## 理解方法调用

1. 编译器查看对象的声明类型和方法名，**编译器会一一列举类中所有名称对应的方法和其超类中所有名称对应的方法**（超类的私有方法不可访问）

2. 编译器要确定方法调用中提供的参数类型，这个过程称为重载解析

3. **如果是private方法，static方法、final方法或者构造器，那么编译器将可以准确地知道应该调用什么方法**，这个过程称为静态绑定

   与之对应的，**如果调用的方法依赖于隐式参数的实际类型**，那么必须在运行时使用动态绑定

4. 程序运行并且采用动态绑定调用方法时，虚拟机必须调用所引用对象的实际类型对应的方法

> 每次调用方法都要完成这个搜索，时间开销相当大，**因此虚拟机预先为每个类计算了一个方法表**，其中列出了所有方法的签名和要调用的实际方法

## 防止继承：final类和方法

> 如果一个方法没有被覆盖并且很短，编译器就能对它进行优化处理，这个过程称为内联
>
> JVM中及时编译器——如果方法很简短、被频繁调用而且确实没有被覆盖，那么及时编译器就会将这个方法进行内联处理

## 抽象类

抽象方法必须要是public（默认）和protected两种限制修饰符

## 相等测试与继承

1. 自反性：对于任何非空引用，x.equals(x)应该返回true
2. 对称性：对于任何引用x和y，当且仅当y.equals(x)返回true时，x.equals(x)返回true
3. 传递性：对于任何引用x、y和z，如果x.equals(y)返回true，y.equals(z)返回true，x.equals(z)也应该返回true
4. 一致性：如果x和y引用的对象没有发生变化，反复调用x.equals(y)应该返回同样的结果
5. 对于任意非空引用x，x.equals(null)应该返回false

编写一个完美的equals的流程

* 显示参数命名为otherObject，稍后需要将它强制转换成另一个名为other的变量

* 检测this与otherObjec是否相等

  `if (this == otherObject) return true`

* 检测otherObject是否为null，如果为null，返回false

  `if (otherObject == null) return false`

* 比较this与otherObject类，如果equals的语义可以在子类中改变，基于可以使用getClass检测

  `if (getClass() != otherObject.getClass()) return false`

  如果所有的子类都有相同的相等性语义，可以使用instanceof检测

  `if (!(otherObject instanceof ClassName)) return false`

* 将otherObject强制装换为相对应类类型的变量

  `ClassName other = (ClassName)otherObject`

* 现在根据相等性概念的要求来比较字段。使用==比较基本类型字段，使用Object.equals比较对象字段。如果所有的字段都匹配，就返回true，否则返回false

  `return field = ohter.field1 && Object.equals(field2, other.field2) && ....`

## hashCode方法

Object类的默认hashCode方法会从对象的存储地址得出散列码

**自定义hashCode最好的方法是，使用Objects.hash并且提供所有需要的参数**

**如果存在数组类型的字段，那么可以使用静态的Arrays.hashCode方法计算一个数列吗，这个散列码由数组元素的散列码组成**

## 声明数组列表

如果调用add而内部数组已经满了，**数组列表就会自动地创建一个更大的数组，斌并且所有的对象从较小的数组中拷贝到较大的数组中**

一旦能够确认数组列表的大小将保持恒定，不再发生变化，就可以调用trimToSize方法，**这个方法将存储块的大小调整为保存当前元素数组所需要的存储空间，垃圾回收器将回收多余的空间**

## 对象包装器和自动装箱

**装箱和拆箱是编译器要做的工作，并不是虚拟机的工作**

## 枚举类

在比较两个枚举类型的值时，并不需要调用equals方法，直接使用==就可以

枚举的构造器总是私有的，可以像前例一样省略private修饰符

## Class类

可以通过Class.forName手动地强制加载其他类

获得Class类的三个方法——

* 通过Class.forName
* 通过T.class
* 通过obj.getClass()

**在大多数实际问题中，可以忽略类型参数，而是用原始的Class类**

虚拟机为每个类型管理一个**唯一**的Class对象，因此可以利用==运算符实现两个类对象的比较

如果有一个Class类型的对象，那么可以利用getConstuctor方法将得到一个Constructor类型的对象，人后使用newInstance方法来构造一个实例

## 资源

1. 获得拥有资源的类Class对象

2. 有些方法接受描述资源位置的URL，要调用

   `URL url = cl.getResource("about.gif");`

3. 否则，使用getResourceAsStream方法得到一个输入流来读取文件中的数据

## 使用反射在运行时分析对象

* 调用f.get(obj)将返回一个对象，其值为obj的当前字段值

* 调用f.set(obj, value)将把对象obj的f表示的字段设置为新值

**反射机制默认受限于Java访问控制，可以调用Field、Mehtod、Constructor对象的setAccessible方法覆盖Java的访问控制**

## 使用反射编写泛型数组代码

使用Array.getLength方法可以获得数组的长度

使用Class。getComponentType可以获得数组中数据的类型

## 调用任意方法和构造器

Method类有一个invoke方法，允许你调用包装在当前Method对象中的方法

第一个参数时**隐式参数**

如果方法是静态方法，那么第一个隐式参数就是null

## 继承的设计技巧

* 将公共操作和字段放在超类中
* 不要使用受保护的字段
* 使用继承实现“is-a“关系
* 除非所有继承的方法都有意义，否则不要使用继承
* 在覆盖方法时，不要改变预期的行为
* 使用多态，而不要使用类型信息
* 不要滥用反射

# 接口、lambda表达式与内部类

## 接口的概念

大数BigDecimal，使用equals比较，1.0 与 1.00是不相同的，因为两个数的精度不同，但是使用compareTo是结果为0

## 接口属性

接口中不能包含实例字段，但是可以包含常量，接口中字段总是public static final

## 静态和私有方法

在Java8中，允许在接口中增加静态方法（**违背将接口作为抽象规范的初衷**）

在Java9中，接口中的方法可以是private，private番木瓜发可以是静态方法或实例方法

## 默认方法

可以为接口提供一个默认实现，必须要default修饰符标记这样一个方法

默认方法的一个非常重要的一个用法就是**接口演化**，如果想要给一个接口添加一个非默认方法打，这样不能保证源代码兼容，如果使用默认方法，那么能够与之前的代码兼容

## 解决默认方法冲突

如果现在一个接口中将一个方法定义为默认方法，然后又在超类或者另一个接口中定义同样的方法，那么怎么解决冲突

1. 超类优先
2. 接口冲突，如果两个接口同时存在这个方法，无论另一个接口中该方法是，默认方法否，都必须通过子类覆盖来解决冲突

## 对象克隆

默认的克隆操作时*浅克隆*，并没有克隆对象中引用的其他对象

如果元对象和克隆对象共享的子对象是不可变的，那么这种共享就是安全的，或者子对象是一个不变的常量，不能通过更改器方法修改，也没有方法会生成它的引用那么也是安全的

如果子对象是可变的，那么需要重新定义clone方法来创建一个深拷贝，同时克隆所有子对象

1. 实现Cloneable接口
2. 重新定义clone方法，并指定public访问修饰符

clone方法是由Object提供的protected方法，但是要实现克隆必须实现Cloneable接口，Cloneable接口是Java提供的少数标记接口之一，**标记接口不包含任何方法，他唯一的作用就是允许在类型查询中使用instanceof**

**即使clone的默认实现能够满足要求，还是需要实现Cloneable接口，将clone重新定义为public，再调用super.clone()**

## lambda表达式的语法

**如果可以推导出一个lambda表达式的参数类型，则可以忽略琪类型**

**无需指定lambda表达式的返回类型，lambda表达式的返回类型总是由上下文推导得出**

## 函数式接口

对于只有一个抽象方法的接口，需要这种接口对象时，就可以提供一个lambda表达式，这种接口称为**函数式接口**

**可以将lambda表达式转换为函数式接口**

* BiFunction<T, U, R>
* Predicate<T\>
* Supplier<T\>

## 方法调用

表达式System.out::println是一个方法调用，**它指示编译器生成一个函数式接口的实例，覆盖这个接口的抽象方法来调用给定的方法**

1. obj::instanceMethod——等价于想方法传递参数的lambda表达式
2. Class::instanceMethod——第一个参数就会变成方法的隐式参数
3. Class::staticMethod——所有参数都传递到静态方法

**只有当lambda表达式只调用一个方法而不做其他操作时，才能把lambda表达式重写为方法引用**

如果过程中会抛出异常，lambda表达式只有在调用时才会抛出异常，而不是像方法引用，在构造时就会抛出异常

**可以在方法引用中使用this、super**

## 构造器引用

构造器引用于方法引用很类似，方法名为new

如果有多个构造器，编译器会自动才选择一个参数匹配的构造器实现

**可以用数组类型建立构造器引用**

stream.toArray方法调用这个构造器来得到一个正确类型的数组，然后填充并返回这个数组

`stream.toArray(Person[]::new)`

## 变量作用域

lambda表达式有三个部分——

1. 一个代码块
2. 参数
3. 自由变量的值，这里值非参数而且不在代码中定义的变量

**关于代码块以及自由变量值称为闭包，lambda表达式就是闭包**

lambda表达式可以捕获外围作用域中变量的值，**在lambda表达式中，只能引用值不会改变的变量**

**lambda表达式于嵌套块有相同的作用域，所以不能在lambda表达式中声明一个与局部变量同名的参数或局部变量**

## 处理lambda表达式

需要使用lambda表达式的地方

1. 在一个单独的线程中运行代码
2. 多次运行代码
3. 在算法的适当位置运行代码
4. 发生某种情况时运行代码
5. 只在必要时运行代码

| 函数式接口          | 参数类型 | 返回类型 | 抽象方法 | 描述                         | 其他方法                   |
| ------------------- | -------- | -------- | -------- | ---------------------------- | -------------------------- |
| Runnable            | 无       | void     | run      | 作为无参数或返回值的动作运行 |                            |
| Supplier<T\>        | 无       | T        | get      | 提供一个T类型的值            |                            |
| Consumer<T\>        | T        | void     | accept   | 处理一个T类型的值            | andThen                    |
| BiConsumer<T, U>    | T，U     | void     | accept   | 处理T和U类型的值             | andThen                    |
| Function<T, R>      | T        | R        | apply    | 有一个T类型参数的函数        | compose，andThen，identity |
| BiFunction<T, U, R> | T，U     | R        | apply    | 有T和U类型参数的函数         | andThen                    |
| UnaryOperator<T\>   | T        | T        | apply    | 类型T上的一元操作符          | compose，andThen，identity |
| BinaryOperator<T\>  | T，T     | T        | apply    | 类型T上的二元操作符          | andThen，maxBy，minBy      |
| Predicate<T\>       | T        | boolean  | test     | 布尔值函数                   | and，or，negate，isEqual   |
| BiPredicate<T, U>   | T，U     | boolean  | test     | 有两个参数的布尔值函数       | and，or，negate            |

**总结**

* 无参数，所以没有Bixxx）

  * Runnable——》无参无返——》执行操作

  * Supplier——》无参有返——》提供值

* 有参数，有一个Bixxx，表示两个参数的情况

  * Consumer——》有参无返——》操作值
  * Function——》 有参有返——》操作值并返回
  * UnaryOperator——》有参有返——》操作自己并返回，类似于一元操作符
  * BinaryOperator——》有两参有返——》操作两个数并返回，类似于二元操作符
  * Predicate——》有参返回布尔——》判断是否满足条件

## 内部类

* 内部类可以对同一个包中的其他类隐藏
* 内部类方法可以访问定义这个类的作用域中的数据，包括原本私有的数据

**内部类的对象总是有一个隐式引用，指向创建它的外部类对象**

## 内部类的特殊语法

通过外部类创建内部类

`outerObj.new InnerClass(params)`

**内部类的所有静态字段必须是final，并初始化一个编译时的常量值**

内部类是一个编译器现象，与虚拟机无关，**编译器将会把内部类转换为常规的类文件，用$分隔外部类名与内部类名**

## 局部内部类

声明局部类时不能有访问说明符，**局部类的作用域被限定在声明这个局部类的块中**

## 由外部方法访问变量

局部类还可以访问局部变量，**不过这些变量必须是事实最终变量**

局部类不需要声明局部变量，而是通过将方法的参数作为内部类的变量，来访问这个局部变量

```java
class TalkingClock$1TimePrinter {
    TalkingClock$1TimePrinter(TalkingClock, boolean);
    
    public void actionPerformed(java.awt.event.ActionEvent);
    
    final boolean val$beep;
    final TalkingClock this$0;
}
```

## 匿名内部类

**匿名类不能有构造器，但可以提供一个对象初始化块**

> 可以使用lambda表达式代替匿名类

## 静态内部类

使用内部类只是为了把一个类隐藏在另外一个类的内部，并不需要内部类有外围类对象的一个引用，可以将内部类声明为static

**只要内部类不需要访问外围对象，就应该使用静态内部类**

## 创建代理对象

使用Proxy类的newProxyInstance方法创建代理对象

* 一个类加载器
* 一个Class对象数组，每个元素对应需要实现的各个接口
* 一个调用处理器

```java
public class JDKProxy implements InvocationHandler {  
  
    private Object targetObject;//需要代理的目标对象   
  
    public Object newProxy(Object targetObject) {//将目标对象传入进行代理   
        this.targetObject = targetObject;   
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),  
                targetObject.getClass().getInterfaces(), this);//返回代理对象   
    }  
  
    public Object invoke(Object proxy, Method method, Object[] args)//invoke方法   
            throws Throwable {  
        doFun();//一般我们进行逻辑处理的函数  
        Object ret = null;      // 设置方法的返回值   
        ret  = method.invoke(targetObject, args);       //调用invoke方法，ret存储该方法的返回值   
        return ret;  
    }  
  
    private void doFun() {//模拟检查权限的例子   
        System.out.println("Mnsx_x");  
    }  
}  
```

**JDK自带的代理需要实现一个接口才能使用**

> CGLIB Proxy——
>
> ```java
> public class CGLibProxy implements MethodInterceptor {  
> 
>     private Object targetObject;// CGLib需要代理的目标对象   
> 
>     public Object createProxyObject(Object obj) {  
>         this.targetObject = obj;  
>         Enhancer enhancer = new Enhancer();  
>         enhancer.setSuperclass(obj.getClass());  
>         enhancer.setCallback(this);  
>         Object proxyObj = enhancer.create();  
>         return proxyObj;// 返回代理对象   
>     }  
> 
>     public Object intercept(Object proxy, Method method, Object[] args,  
>                             MethodProxy methodProxy) throws Throwable {  
>         Object obj = null;  
>         doFun();// 调用函数  
>         obj = method.invoke(targetObject, args);  
>         return obj;  
>     }  
> 
>     private void doFun() {  
>         System.out.println("Mnsx_x");  
>     }  
> }  
> ```

# 异常、断言和日志

## 声明检查型异常

* 调用一个抛出检查型异常的方法
* 检测到一个错误，并且利用throw语句抛出一个检查型异常
* 程序出现错误
* Java虚拟机或运行时库出现内部错误

**不应该声明从RuntimeException继承的那些非检查型异常**

## 捕获多个异常

当捕获的异常类型彼此之间不存在子类关系时才需要这个特性

`catch (FileNotFoundException | UnknownHostException e)`

## 再次抛出异常与异常链

希望改变异常的类型时会在catch子句中抛出一个异常

**可以用一种更好的处理方法，可以把原始异常设置为新异常的原因**

```java
try {
    access the database
} catch (SQLException original) {
    var e = new ServletException("database error");
    e.initCause(original); // 包装
    throw e;
}
```

**强烈建议使用这种包装技术，这样可以在子系统抛出高层一场，而不会丢失原始异常的细节**

## try-with-Resources语句

如果资源属于应给实现了AutoCloseable接口的类，java7为这种diamagnetic模式提供了一个很有用的快捷方式

`void close() throw Exception`

Closeable接口时AutoCloseable接口的子类，抛出一个IOException异常

如果close方法抛出异常，那么原来的异常会被重新抛出，close方法抛出的异常**将会被抑制**，这些异常将自动捕获，**并由addSuppress方法增加到原来的异常中**，可以通过调用getSuppressed方法获取抛出并被抑制的一场数组

## 分析堆栈轨迹元素

* 可以调用printStackTrace

* **更灵活的做法是使用StackWalker类**，他会生成一个StackWalker.StackFrame实例流，其中每个实例分别描述一个栈帧

  ```java
  StackWalker walker = StackWalker.getInstance();
  walker.forEach(frame -> analyze stream);
  //=================================懒加载
  walker.walk(stream -> process stream)
  ```

## 使用异常的技巧

* 异常处理不能代替简单测试
* 不要过分地细化异常
* 充分利用异常层次结构
* 不要压制异常
* 在检测错误时，“苛刻”总比放任好
* 不要羞于传递异常

## 调试技巧

* 在自定义类中应该重构toString方法来显示this对象的状态

* 可以在每一个类中放置一个单独的main方法，这样就可以提供一个单元测试桩，能够独立地测试类

* 可议使用JUnit单元测试框架

* 利用Throwable类的printStackTrace方法，可以从任意的异常对象获得堆栈轨迹

* 可以记录堆栈轨迹

  ```java
  StringWriter out = new StringWriter();
  new Throwable().printStackTrace(new PrintWriter(out));
  String description = out.toString();
  ```

* 在System.err中显示未捕获的异常的堆栈轨迹并不是一个理想的方法

# 泛型程序设计

## 泛型方法

如果想要知道泛型方法调用的最终类型是什么，**可以通过故意写错一个参数，从报错的结果中查看需要类型**

## 类型变量的限定

**限定类型用‘&’分隔，而逗号用来分隔类型变量**

在Java的继承中，可以根据需要拥有多个接口超类型，但是最多有一个限定可以是类，**如果有一个类作为限定，那么必须是限定列表中的第一个限定**

## 类型擦除

无论何时定义一个泛型类型，都会自动提供一个相应的原始类型，**这个原始类型的名字就是去掉类型参数后的泛型类型名**

**类型变量会被擦除，并替换为其限定类型，或者对于无限定的变量则替换为Object**

**原始类型用第一个限定来替换类型变量**，或者，如果没有给定限定，就替换为Object

**为了提高效率，应该将标签接口放在限定列表的最后一位（没有方法的接口）**

## 转换泛型表达式

编写一个泛型方法的调用时，如果擦除了返回类型，**编译器会插入强制类型转换**

## 转换泛型方法

> 类型擦除和多态的冲突
>
> ```java
> class Dateinterval extends Pair<LocalDate> {
>     @Override
>     public void setSecond(LocalDate second) {
>         ...
>     }
> }
> ```
>
> 希望通过继承重写父类的方法，覆盖父类的方法，限定只能通过LocalDate调用
>
> 但是类型擦除后
>
> ```java
> class DateInterval extends Pair {
> 	...
>         
>     public void setSecond(Object second) {
>         ...
>     }
> }
> ```
>
> 子类依然会从父类中继承一个Object的方法

**类型擦除于多态发生了冲突**，为了解决这个问题，**编译器会在冲突类中生成一个桥方法**

`public void setSecond(object second) {setSecond((LocalDate) second)}`

如果方法中有一个无参的方法，那么会出现

```java
LocalDate getSecond();
Object getSecond();
```

这在java语言中是不被允许的，**但是，在虚拟机中，会由参数类型和返回类型共同指定一个方法**，因此，编译器可以为这两个仅返回一类型不同的方法生成字节码，虚拟机能够正确地处理这种情况

* 虚拟机中没有泛型，只有普通的类和方法
* 所有的类型参数都会替换成它们的限定类型
* 会合成桥方法来保持多态
* 为保持类型安全性，必要时会插入强制类型转换

## 限定与局限性

* 不能使用基本类型实例化类型参数

  **Object不能存储基本类型参数**

* 运行时类型查询只适用于原始参数

  **虚拟机中的对象总有一个特定的非泛型类型，所以类型查询只产生原始类型**

* 不能创建参数化类型的数组

  ```java
  Pair<String>[] table = new Pair<>[10];
  // 擦除后
  Pair[] table = new Paire[10];
  // 问题
  table[0] = new Pair<Test>();
  // 擦除后
  table[0] = new Pair(); // 此情况下合法
  ```

  **处于这个原因，不允许创建参数化类型的数组**

  **解决方法**：使用ArrayList

* Varargs警告

  在泛型方法中使用可变参数，问题与不能创建参数化类型的数组相同，但是这里只会得到一个警告，而不是报错

  可以采用两种方法抑制这个警告

  * 添加注解@SuppressWarnings('unchecked')

  * @SafeVarargs直接注解这个方法

    > @SafeVarargs只能声明为static、final或（java9）private的构造器和方法

* 不能实例化类型变量

  **类型擦除会将T变为Object**

  解决方法：让调用者提供一个构造器表达式

  `Pair<String> p = Pair.makePair(String::new)`

* 不能构造泛型数组

  **与不能实例化参数类似**

  解决方法：提供一个数组构造器表达式

  ```java
  public static <T extends Comparable> T[] minmax(IntFunction<T[]> constr, T ...a) {
      T[] ressult = constr.apply(2);
      ...
  }
  ```

* 泛型类的静态是上下文中类型变量无效

  **无法得知泛型类型**

* 不能抛出或捕获泛型类的实例

  **泛型类扩展Throwable甚至都是不合法的**

* 可以取消对检查类型异常的检查

  ```java
  @SuppressWarnings('unchecked')
  static <T extends Throwable> void trhowAs(Throwable t) throws T {
      throw (T) t;
  }
  ```

  如果有一个检查型异常，处理时抛出这个方法，那么就会被认为是一个非检查型异常

  ```java
  try {
      do work;
  } catch (Throwable t) {
      Task.<RuntimeException>throwAs(t);
  }
  ```

# 集合

## 集合接口与实现分离

循环数组是一个有界集合，容量有限，如果程序中要收集的对象数量没有上限，最好使用链表来实现

Java迭代器位于两个元素之间，返回的是越过的元素的索引

## 集合框架中的接口

> 为了避免对链表完成随机访问操作，Java1.4提供了一个标记接口RandomAccess

## 链表

**ListIterator提供了add、previous、hasPrevious、set方法**

add方法在迭代器之前添加一个新对象

set方法用一个新元素替换调用next或previous方法返回的上一个元素

get方法属于伪随机访问，

## 散列集

散列表用链表数组表示，每个列表被称为桶，**想要查找表中对象的位置，就要先计算它的散列码，然后与桶的总数求余，所得到的结果就是保存在这个元素的桶的索引**

如果大致知道最终会有多少个元素要插入到散列表中，就可以设置桶数。通常，**将桶数设置为预期元素个数的75%-150%**，**最好将桶数设置为一个素数**，防止键的聚合

**装填因子可以确定何时对散列表进行再散列，新表的桶数是原来的两倍**

## 树集

**排序使用一个数数据结构完成的（红黑树）**

**要使用树集，必须能够比较元素，这些元素必须实现Comparable接口，或者构造集时必须提供一个Comparator**

## 优先级队列

调用remove方法，总会获得当前优先队列最小的元素，使用的数据结构为堆

**优先队列适合用在任务调度**

## 更新映射条数

`counts.merge(word, 1, Integer::sum)`

当存在时求和，不存在放1 

# 并发

## 什么是线程

**不要再调用Thread类或Runnable对象的run方法**，注解调用run方法只会再同一个线程中执行这个任务

## 线程状态

* New新建
* Runnable可运行
* Blocked阻塞
* Waiting等待
* Timed waiting计时等待
* Terminated终止

调用getState方法可以获取当前线程的状态

## 可运行线程

一旦调用start方法，线程就处于可运行状态

线程的运行时间主要由**操作系统**为线程提供具体的运行时间

正在运行的线程也是可运行状态

**抢占式调度系统**，给每一个可运行线程一个时间片来执行任务，当时间片用完时，操作系统剥夺该线程的运行权力，并给另一个线程一个时间片来执行，**当选择下一个线程时，操作系统会<u>考虑</u>线程的优先级**

>  **协作式调度系统**，一个线程只有在调用yield方法或者被阻塞或等待时才失去控制权

## 阻塞和等待线程

* 当一个线程视图获取一个**内部的对象锁**，而这个锁目前被其他线程持有，那么这个线程就会被阻塞
* 当线程等待另一个线程通知调度器出现一个条件是，就会进入等待（**lock、condition、Object.wait、Thread.join**）
* 有几个方法有超时参数，调用这些方法会让线程进入计时等待状态（**Thread.sleep，计时版的Object.wait、Thread.join、Lock.tryLock以及Condition.await**)

## 终止线程

* run方法正常退出，线程自然终止
* 因为一个**没有捕获的异常**终止了run方法，使线程意外终止

## 中断线程

interrupt方法可以用来请求终止一个线程，线程将被设置为中断状态，每个线程都会不时检查，以判断线程是否被中断

**如果线程被阻塞，就无法检查线程是否中断**，如果对一个阻塞线程使用interrupt方法，那么这个阻塞调用将被一个InterruptedException异常中断

**如果被设置了中断状态，此时调用sleep方法，他不会休眠，实际上，他会清除中断状态并抛出一个InterruptedException**

> interrupted方法是一个 静态方法，他检查当前线程是否被中断，**调用interrupted方法会清楚该线程的中断状态**
>
> isInterrupted可以用来检查是否有线程被中断，**调用这个方法不会改变中断状态**

最正确的处理InterruptException的方法应该时将该异常抛出，让调用者处理这个异常

## 守护线程

**通过调用`thread.setDaemo(true)`将一个线程转换为守护线程**

**守护线程的唯一用途是为其他线程提供服务，当只剩下守护线程时，虚拟机就会退出**

## 未捕获异常的处理器

在线程因为异常死亡之前，异常会传递到一个用于处理未捕获异常的处理器中，**这个处理器必须实现Thread.UncaughtExceptionHandler**

**可以用setUncaughtExceptionHandler方法为任何线程安装一个处理器，一可以通过Thread类的静态方法setDefaultUncaughtExceptionHandler为所有线程安装一个默认的处理器**

如果没有安装默认处理器，那么默认处理器就是null，但是如果没有为单个线程安装处理器，那么处理器就是该线程的ThreadGroup对象

ThreadGroup的uncaughtException方法执行流程——

1. **如果该线程组有父线程组，那么调用父线程组的uncaughtException方法**
2. **否则，如果Thread.getDefaultExceptionHandler方法返回一个非null的处理器，则调用该处理器**
3. **否则，如果Throwable是ThreadDeath的一个实例，那么什么都不做**
4. **否则，将线程名字以及Throwable的栈轨迹输出到System.err**

## 线程优先级

一个线程会继承构造它的线程的优先级

可以通过setPriority方法提高或降低任何一个线程的优先级

线程优先级高度依赖于系统，当虚拟机依赖于宿主机平台的线程实现时，Java线程的优先级会映射到宿主机平台的优先级，**所以现在不要使用线程优先级了**

## 锁对象

防止并发访问代码块的两种方式

* synchronized关键字
* ReentrantLock类

**要将unlock操作放在finallly字句中，防止临界区代码抛出一个异常，锁必须释放，否则其他线程将永远阻塞**

使用锁时不能使用try-with-resources语句

这种锁称为重入锁，因为线程可以反复获得已拥有的锁，**锁有一个持有计数来跟踪对lock方法的嵌套调用**

**要注意确保临界区中的代码不要因为抛出异常而跳出临界区**，抛出异常后，finally子句将自动释放锁，那么对象可以能处于被破坏的状态

公平锁比普通锁**慢很多**，不建议使用公平锁

## 条件对象

通过newCondition方法获得一个条件对象

```java
condition = bandLock.newCondition();
```

如果不满足条件那么，就会调用await方法，**当前线程现在暂停，并放弃锁**

**一旦一个线程调用await方法，他就进入这个条件的等待集**

**当锁可用时，该线程不会改变可运行状态，直到另一个线程在同一条件上调用signalAll或者signal方法**

通过signalAll方法唤醒的线程，获取锁后，将会从暂停的位置开始继续执行，所以**正常情况应该通过循环去判断是否满足条件**

**只要有一个对象的状态有变化，并且有利于等待的线程，那么就应该调用signalAll方法**

## synchronized关键字

如果一个方法声明时有synchronized关键字，那么对象的锁将保护整个代码

**内部对象锁中有且仅有一个关联条件**

**将静态方法声明为同步也是合法的**

使用内部锁的局限性——

* 不能中断一个正在尝试获得锁的线程
* 不能指定从银行是获得锁时的超时时间
* 每个锁仅有一个条件可能是不够的

使用对象锁和lock的建议——

* 最好不使用juc
* 首选synchronized
* 特别需要Lock/Condition的结构提供额外的能力，那么可以使用Lock/Condition

## 同步块

Java虚拟机对内部方法提供了内置支持，不过，同步块会编译为很长的字节码序列来管理内部锁

## volatile字段

volatile关键字为实例字段的同步访问提供了一种免锁机制，**如果声明一个字段为volatile，那么编译器和虚拟机就知道该字段可能被另一个线程并发更新**

编译器会插入适当的代码，以确保如果一个线程对声明为volatile的变量做了修改，**这个修改对读取这个变量的所有线程都是可见的**

**volatile变量不提供原子性**

**volatile变量禁止JVM指令重排序**

volatile关键字是为了防止编译器对代码过度优化，提醒编译器，这个变量是易变的

**强迫每个线程在读取volatile修饰的变量值时，需要从主内存中读取。保证数据一经改变，其它线程立即感知**

## final字段

所有线程都会在，构造器完成构造之后才能访问这个字段，并且布恩那个修改这个字段，所以这个字段是线程安全的

## 原子性

**对共享变量除了赋值之外并不做其他操作，那么可以将这些共享变量声明为volatile**

**AtomicXxx类保证原子性**

>  **AtomicInteger类提供方法incrementAndGet和decrementAndGet，原子性的自增自减**
>
> **updateAndGet提供UnaryOperator，accumulateAndGet提供BinaryOperator，来合并原子值和所提供的参数**
>
> **getAndUpdate和getAndAccumulate方法可以返回原值**

如果有大量线程要访问相同的原子值，性能会大幅下降，因为乐观更新所需太多次重试，LongAdder和LongAccumulate解决了这个问题

> **调用increment让计数器自增，或者调用add来增加一个量，最后使用sum来获取总和**

## 死锁

产生死锁的必要条件，**Java没有解决方式，只能避免**

* **互斥——无法避免**
* **请求和保持——一次性请求所有的资源**
* **不剥夺——在提出获取新的资源请求时，需要释放手中的资源，才能获取新的资源**
* **循环等待——定义资源类型的线性顺序来预防，可以将每个资源编号，当一个线程占有编号为i的资源时，下一次请求资源只能申请编号大于i的资源**

## 废弃stop和suspend的原因

* 线程会被立即终止，那么线程会立即释放它的锁，**这样会导致对象处于不一致的状态**
* suspend方法，在恢复运行之前，**这个线程拥有的锁将不能使用**，那么其他进程想要获取这个锁就会导致死锁问题

## 阻塞队列

* 如果使用队列作为线程管理工具，将要用到put和take方法（阻塞）
* add、remove和element操作会抛出异常
* offer、poll、peek，知识给出一个错误提示，不会抛出异常（null）

* offer、poll可以带有超时时间使用TimeUnit设置单位

JUC中的阻塞队列

* LinkedBlockingQueue
* LinkedBlockingDeque
* ArrayBlockingQueue（构建时需要指定容量）
* PriorityBlockingQueue优先队列
* DelayQueue实现Delayed接口，getDelay方法返回对象的剩余延迟
* LinkedTransferQueue实现了TransferQueue接口，transfer方法会将调用阻塞，允许生产者线程等待，直到消费者准备就绪可以接受元素后恢复

## 高效的映射、集和队列

JUC包含映射、有序集、队列的高效实现：ConcurrentHashMap、ConcurrentSkipListMap、ConcurrentSkipListSet和ConcurrentLinkedQueue

**这些集合的size方法不一定在常量时间内完成操作，通常需要遍历**

**mappingCount方法可以把size作为long返回**

**集合返回弱一致性的迭代器**，迭代器不一定能反映构造之后的所有更改，但是不会同一个值返回两次，并且不会抛出ConcurrentModificationException异常

> 普通集合，如果迭代器构造之后发生变化，就会抛出一个ConccurentModificationException异常

**这些集合默认情况下有至多16个同时运行的书写器线程**

> 在新版的Java中，并发散列映射将桶组织改为树，而不是链表，键类型实现COmparable保证性能O(log(n))

## 映射条目的原子更新

**compute方法可以提供一个键和一个计算新值的函数**

**ConcurrentHashMap中不允许有null**，可以使用null值指示映射中某个给定的键不存在

**merge方法可以非常方便的对一个值进行特殊操作**

`map.merge(word, 1L, Long::sum)`

## 对并发散列映射的批处理

* search为每个键或者值应用一个函数，直到函数生成一个非null的结果
* reduce组合所有键或者值，需要提供一个累加函数
* forEach为每个键或者值提供一个函数

操作的不同方式

* operationKeys：键
* operationValus：值
* operation：键或值
* operationEntries：Map.Entry

上述所有操作，**需要指定一个参数化阈值**，如果映射包含的元素多余这个阈值，就会并行完成批操作

search——》BiFunction

forEach——》BiConsumer

​			 ——》BiConsumer+类型转换器Function

reduce——》于forEach类似，也可以添加应给Funtion作为转换器

​            ——》将int、long、double输出还有相应的特殊化操作，toInt、toLong和toDouble，需要额外指定一个默认值

## 写数组的拷贝

**CopyOnWriteArrayList、CopyOnWriteArraySet是线程安全的集合**

## 并行数组算法

**Arrays.parallelSort方法可以对一个基本类型值或者对现象的数组进行排序**

**parallelSetAll是用一个函数计算得到的所有值填充一个数组**

**parallelPrefix，用一个给定组合操作的相应前缀的累加结果替换每个数组元素**

## Callable和Future

Runnable封装一个异步运行的任务

Callable和Runnab类似但是有返回值，**类型参数就是返回值类型**

Future保存异步计算的结果

```java
// Future<V>
V get(); // 调用会阻塞，直到计算完成
V get(long timeout, TimtUnit unit); // 调用会阻塞，如果超时会爆出异常TimeoutException
// get如果计算线程被中断就会抛出InterruptedException
void cancel(boolean mayInterrupt); // 取消计算，如果计算还未开始，那么会取消而且不再开始，如果计算正在进行，那么mayInterrupt参数为true，那么就会被中断
boolean isCancelled(); // 判断是否被取消
boolean isDone(); // 判断是否已经结束
```

执行Callable的方式是使用FutureTask

```java
Callable<Integer> task = ...;
FutureTask futureTask = new FutureTask<Integer>(task);
Thread threa = new Thread(futureTask);
```

## 执行器

执行器有很多静态工厂方法来构建线程池

| 方法                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| newCachedThreadPool              | 必要时创建新线程；空闲线程会保留60秒                         |
| newFixedThreadPool               | 池中包含固定数目的线程；空闲线程会一直保留                   |
| newWorkStealingPool              | 一种适合“fork-join”任务的线程池，其中复杂的任务会分解为更简单的任务，空闲线程会“密取”较简单的任务 |
| newSingleThreadExecutor          | 只有一个线程的“池”，会顺序地执行所提交的任务                 |
| newScheduledThreadPool           | 用于调度执行的固定线程池                                     |
| newSingleThreadScheduledExecutor | 用于调度执行单线程“池”                                       |

* 如果**线程生存期很短，或者大量时间都在阻塞**可以使用缓存线程池
* **为了最优的运行速度，并发线程数等于处理器内核数**应当使用固定线程池
* 单线程执行器对于**性能分析**很有帮助

```java
Future<T> submit(Callable<T> task);
Future<T> submit(Runnable tast); // future的get方法永远返回null
Future<T> submit(Runnable task, T result); // 返回指定future
```

**使用一个线程池后，调用shutdown，启动线程池的关闭序列**

## fork-join框架

将复杂面密集的任务分成子任务进行完成

## 可完成Future

CompletableFuture实现了Future接口，提供了获得结果的另一种机制，**注册一个回调，一旦结果可用，就会来利用结果调用这个回调**

```java
CompletableFuture<String> f = ...;
f.thenAccept(s -> process the result);
```

**利用这种方式，无需阻塞就可以在结果可用时，对结果进行处理**

CompletableFuture可以采用两种方式完成，得到结果或者抛出异常

```java
f.whebnComplete((s, t) -> {
    if (t == null) {process the result s}
    else {process the Trowable t}
})
```

使用方式

```java
Future f = new CompletableFuture();
execurot.execute(() -> {
    int n = work(arg);
    f.complete(n)
});
```

# Java8的流库

## 从迭代到流操作

将stream改为parallelStream就可以让流库**以并行方式**来执行操作

流与集合的区别

1. 流并不存储其元素，这些元素可能**存储在底层的集合中，或者按需生成**
2. 流操作**不会修改其数据源**，其操作只会生成一个新的流
3. 流的操作是尽可能**惰性执行**的，只有直到需要这个结果了，操作才会执行，**因此可以操作无限流**

操作流时的经典流程

1. 创建一个流
2. 指定将初始流转换为其他流的中间操作，可能包含多个步骤
3. 应用终止操作，产生结果，**这个操作会强制执行之前的惰性操作，此后流不能使用**

## 流的创建

Collections含有方法stream

Stream.of(数组)

Stream.of(a, b, c, d...) 可变参数

Stream.of(array, from, to)

Stream.empty 没有任何元素的流

Stream.generate 接收一个Supplier接口对象

Stream.iterate 接收一个UnaryOperation接口对象，和一个初始值，可以添加一个谓词来终止操作

Stream.ofNullable 会用一个对象创建一个非常短的流，如果该对象为null，那么这个流的长度就是0，如果该对象不为null，那么这个流的长度就是1

**Pattern.compile(正则表达式).splitAsStream(目标对象) 能够同构正则表达式的方式去分隔对象，并返回流**

## filter、map和flatMap方法

**flatMap将流中所有元素所产生的结果（流）来连接到一起**

## 抽取子流和组合流

limit方法可以限制流的长度

skip方法可以跳过前n个元素

**takeWhile方法需要一个Predicate接口参数，谓词为真时获得流中所有元素，然后停止**

**dropWhile方法，在谓词为真时丢弃元素，并产生一个由第一个谓词为假的字符开始的所有元素构成的流**

concat方法可以将两个流连接起来

## 其他的流转换

distinct返回一个流，将原本流中重复的元素去掉

sort返回一个排序后的流，**其中一种是操作Comparable元素的流，另一种可以接收一个Comparator**

**peek返回一个与原来流中元素相同的流，但是每次获取其中元素时会调用一个函数**

## 简单约简

**约简是一种终结操作**

count方法会返回流中元素个数

max、min返回最大值最小值

这些方法返回一个Optional<T\>的值

findFirst返回非空集合中的第一个值，**与filter组合使用效果很好**

findAny只要非空集合中有一个与之匹配则返回，**可以配合并行操作**

```java
Optional<String> startsWithQ = words.parallel().filter(s -> s.startWith("Q")).findAny();
```

anyMatch返回是否由匹配，还有allMatch、noneMatch

## 获取Optional值

* 没有任何匹配时，使用默认值

  ```java
  String result = optionalString.orElse("");
  ```

* 没有任何匹配时，通过计算返回默认值

  ```java
  String result = optionalString.orElseGet(() -> System.out.println("myapp.default"));
  ```

* 没有任何匹配时，抛出异常

  ```java
  String result = optionalString.orEleseThrow(IllegalStateException::new);
  ```

## 消费Optional值

ifPresent方法接受一个函数，如果选值存在，那么他会被传递给该函数，否则不会发生任何事

可以使用ifPresentOrElse，来处理选值不存在的情况

## 管道化Optional值

optional可以通过**map**方法来转换Optional内部的值

**filter**方法可以用来过滤不满足条件的值

**可以用or方法将空Optional替换为一个可替代的Optional，惰性操作**

## 不适合使用Optional值的方式

**get方法如果Optional值为空，那么返回NoSuchElementException**

Optional的正确用法

* Optional类型的变量永远都不应该为null
* 不要使用Optional类型的域
* 不要在集合中防止Optional对象，并且不要将他们用作imap的键，应该直接收集其中的值

## 创建Optional值

```java
Optional.of(result);
optional.empty();
Optional.ofNullable(result); // 如果result!=null等同of，反之等同于empty
```

## 用flatMap构建Optional值的函数

flatMap如果当前optional为空那么返回一个空Optional，如果不为空则调用flatMap中的函数

## 将Optional转换为流

stream方法会将一个Optional<T\>对象转换程一个具有0个或者1个元素的Stream<T\>对象

```java
Stream<User> users = ids.map(User::lookUp).flatMap(Optional::stream);
```

## 收集结果

forEach，将函数应用于每个元素，**这个方法是按照随即顺序执行的**，**forEachOrdered方法是按照流中的顺序进行执行**

toArray获得由流的元素构成的数组，因为无法在运行时创建泛型数组，**所以stream.toArray()只返回一个Object[]数组**，如果需要数组具有正确的类型，**可以将其传递到数组构造器中**

可以通过joining来进行连接操作返回一个字符串，**如果流中包含除字符串以外的元素那么需要将其转换成字符在进行操作**

使用summarizing(Int|Long|Double)可以计算总和、数量、平均值、最大值、最小值

## 收集到映射表中

Collectors.toMap方法有两个函数引元，他们用来产生映射表的键和值

**通常情况下，值应该是实际元素，因此第二个函数可以使用Function.identity()**

如果多个元素具有相同的键就会产生冲突，收集器将会抛出IllegalStateException，**可以通过第3个函数来覆盖这种行为**

## 群组和分区

groupingBy可以将具有相同特性的值群聚成组

```java
Map<String, List<Locale>> countryToLocales = Locales.collect(Collectors.groupingBy(Locale::getCountry));
```

partitioningBy将流元素分为两个列表，返回true的元素和其他的元素，**相比groupingBy更加高效**

## 下游收集器

* couting会产生收集到的元素的个数

* summing(Int|Long|Double)会接收一个函数，将这个函数应用到下游元素中

* maxBy和minBy会接收一个比较器，并分别产生下游元素中的最大值和最小值

* collectingAndThen可以将结果收集到一个集合中，然后加上一个最终操作

  ```java
  Map<Character, Integer> stringCountsByStartingLetter = strings.collect(groupingBy(s -> s.charAt(0), collectingAndThen(toSet(), Set::size)));
  ```

* mapping与collectingAndThen相反，先进行函数操作，再将其封装成集合

  ```java
  Map<Character, Set<Integer>> stringLengthsByStartingLetter = strings.collect(groupingBy(s -> s.charAt(0), mapping(String::length, toSet())));
  ```

* summarizing(Int|Double|Long)，同样可以作为下游收集器

* filtering将一个过滤器应用在每一个组上

## 约简操作

reduce接受一个二元函数，处理流中数据，**如果流为空，则返回一个Optional**

可以提供一个幺元值，如果流为空那么返回幺元值

```java
List<Integer> vlaues = ...;
Integer sum = valus.stream().reduce(0, (x, y) -> x + y);
```

## 基本类型流

**使用mapToInt、mapToLong或mapToDouble将对象流转换成基本类型流**

**使用bexed方法将基本类型流转换成对象流**

# IO流

## BIO阻塞多发多收模式

```java
public class Client {
    public static void main(String[] args) throws IOException {
        // 创建一个Socket对象，请求服务端连接
        Socket socket = new Socket("127.0.0.1", 9999);
        // 从Socket对象获取一个字节输出流
        OutputStream os = socket.getOutputStream();
        // 把字节输出流包装成一个打印流
        PrintStream ps = new PrintStream(os);
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("请说: ");
            String msg = scanner.nextLine();
            ps.println(msg);
            ps.flush();
        }
    }
}
```

```java
public class Server {
    public static void main(String[] args) {
        try {
            // 定义一个ServerSocket对象进行服务端的端口注册
            ServerSocket ss = new ServerSocket(9999);
            // 监听客户端的Socket连接请求
            Socket socket = ss.accept();
            // 从Socket管段中得到一个字节输入流对象
            InputStream is = socket.getInputStream();
            // 把字节输入流包装成一个缓冲输入流
            BufferedReader br = new BufferedReader(new InputStreamReader(is));
            String msg = null;
            while ((msg = br.readLine()) != null) {
                System.out.println("服务端收到：" + msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## BIO多线程一主多从

```java
public class Server {
    // 同时解说多个客户端Socket通信请求
    // 收到客户端Socket请求对象后交给一个独立线程处理客户端数据请求
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(6);
        try {
            // 注册端口
            ServerSocket serverSocket = new ServerSocket(9999);
            // 定义一个死循环，不断接收请求
            while (true) {
                Socket socket = serverSocket.accept();
                // 创建一个独立的线程来处理这个通信需求
                threadPool.submit(() -> {
                    try {
                        // 从Socket对象中得到一个字节输入流
                        InputStream inputStream = socket.getInputStream();
                        // 使用缓冲字字符输入流包装字节输入流
                        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                        String msg = null;
                        while ((msg = bufferedReader.readLine()) != null) {
                            System.out.println(msg);
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                });
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        try {
            // 请求与服务端的Socket连接
            Socket socket = new Socket("127.0.0.1", 9999);
            // 得到一个打印流
            PrintStream printStream = new PrintStream(socket.getOutputStream());
            // 使用循环不断发送消息
            Scanner scanner = new Scanner(System.in);
            while (true) {
                System.out.println("请说: ");
                String msg = scanner.nextLine();
                printStream.println(msg);
                printStream.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## BIO小结

* 每个Socket接收到，都会创建一个线程，线程竞争，上下文切换影响性能
* 每个线程都会占用栈空间和CPU资源
* 并不是每个Socket都进行IO操作，无意义的线程处理
* 客户端的并发访问增加，系统将发生线程栈溢出，线程创建失败，禁止宕机或者僵死

## 伪异步BIO

```java
public class Server {
    // 开发实现伪异步通信架构
    public static void main(String[] args) {
        try {
            // 注册端口
            ServerSocket serverSocket = new ServerSocket(9999);
            // 定义一个循环接收客户端的Socket连接请求
            // 初始化线程池对象
            HandlerSocketServerPool pool = new HandlerSocketServerPool(3, 10);
            while (true) {
                Socket socket = serverSocket.accept();
                // 把socket交给线程池进行处理
                // 将Socket封装成一个任务对象传给pool
                pool.execute(() -> {
                    // 从Socket管段中得到一个字节输入流对象
                    InputStream is = null;
                    try {
                        is = socket.getInputStream();
                        // 把字节输入流包装成一个缓冲输入流
                        BufferedReader br = new BufferedReader(new InputStreamReader(is));
                        String msg = null;
                        while ((msg = br.readLine()) != null) {
                            System.out.println("服务端收到：" + msg);
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                });
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class HandlerSocketServerPool {
    // 创建一个线程池成员变量存储一个线程池对象
    private ExecutorService executorService;

    /**
     * 创建这个类对象的时候就需要初始化线程池对象
     */
    public HandlerSocketServerPool(int maxTreadNum, int queueSize) {
        executorService = new ThreadPoolExecutor(3, maxTreadNum, 120,
                TimeUnit.MINUTES, new ArrayBlockingQueue<Runnable>(queueSize));
    }

    /**
     * 提供一个方法来提交任务给线程池的任务队列来缓存，等待线程池来处理
     */
    public void execute(Runnable runnable) {
        executorService.execute(runnable);
    }
}
```

```java
public class Client {
    public static void main(String[] args) throws IOException {
        // 创建一个Socket对象，请求服务端连接
        Socket socket = new Socket("127.0.0.1", 9999);
        // 从Socket对象获取一个字节输出流
        OutputStream os = socket.getOutputStream();
        // 把字节输出流包装成一个打印流
        PrintStream ps = new PrintStream(os);
        Scanner scanner = new Scanner(System.in);
        while (true) {
            System.out.println("请说: ");
            String msg = scanner.nextLine();
            ps.println(msg);
            ps.flush();
        }
    }
}
```

* 伪异步IO采用了线程池实现，因此避免了为每个请求创建一个独立线程造成线程资源耗尽的问题，**由于底层仍旧采用同步阻塞模型，因此没法根本上解决问题**
* 如果单个消息处理缓慢，或者服务器线程池中的全部线程都被阻塞，那么后续Socket的IO消息都将在队列中排队，**新的Socket请求也会被拒绝，客户端产生大量超时连接**

## BIO文件传输

```java
public class Server {
    // 接收客户端任意类型的文件，并保存到服务端
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(8888);
            while (true) {
                Socket socket = serverSocket.accept();
                // 交给一个独立线程来处理这个客户端通信的Socket
                new Thread(() -> {
                    try {
                        // 得到一个输入流读客户端发送的数据
                        DataInputStream dataInputStream = new DataInputStream(socket.getInputStream());
                        // 读取客户端发送的文件类型
                        String suffix = dataInputStream.readUTF();
                        System.out.println("接收到文件类型" + suffix);
                        // 定义一个字节输出管道负责把客户端输入的数据写出去
                        OutputStream outputStream = new FileOutputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src" + UUID.randomUUID().toString() + suffix);
                        // 从数据输入流中读入文件数据，写出到字节输出流中
                        byte[] buffer = new byte[1024];
                        int len;
                        while ((len = dataInputStream.read(buffer)) > 0) {
                            outputStream.write(buffer, 0, len);
                        }
                        outputStream.close();
                        dataInputStream.close();
                        System.out.println("接收文件保存成功");
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    // 客户端上传任意类型的文件给服务端保存起来
    public static void main(String[] args) {
        DataOutputStream dataOutputStream = null;
        InputStream inputStream = null;
        try {
            // 请求与服务端的Socket连接
            Socket socket = new Socket("127.0.0.1", 8888);
            // 将字节输出流封装成数据输出流
            dataOutputStream = new DataOutputStream(socket.getOutputStream());
            // 先发送上传文件的后缀给服务端
            dataOutputStream.writeUTF(".txt");
            // 把文件数据发送给服务端，进行接收
            inputStream = new FileInputStream("D:\\Warehouse\\Picture\\test.txt");
            byte[] buffer = new byte[1024];
            int len;
            while ((len = inputStream.read(buffer)) > 0) {
                dataOutputStream.write(buffer, 0, len);
            }
            dataOutputStream.flush();
            socket.shutdownOutput();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## BIO端口转发思想

```java
public class Server {
    // 静态集合
    public static List<Socket> allSocket = new ArrayList<>();

    // BIO端口转发思想
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(9999);
            while (true) {
                Socket socket = serverSocket.accept();
                // 把登录客户端socket存储到在线集合中
                allSocket.add(socket);
                // 为所有登录成功的集合分配一个独立线程进行通信
                new Thread(() -> {
                    try {
                        // 从Socket中获取当客户端的输入流
                        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                        String msg = null;
                        while ((msg = bufferedReader.readLine()) != null) {
                            String message = msg;
                            System.out.println(socket.toString() + "发送消息：" + msg);
                            System.out.println(allSocket);
                            // 服务端接收客户端消息后，推送给所有的在线socket
                            allSocket.stream().filter(socketItem -> socketItem != socket).forEach(socketItem -> {
                                try {
                                    System.out.println("发送消息到客户端——" + socketItem);
                                    PrintWriter ps = new PrintWriter(socketItem.getOutputStream());
                                    ps.println(message);
                                    ps.flush();
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }
                            });
                        }
                    } catch (IOException e) {
                        System.out.println("有人下线了");
                        allSocket.remove(socket);
                    }
                }).start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        try {
            // 请求与服务端的Socket连接
            Socket socket = new Socket("127.0.0.1", 9999);
            // 得到一个打印流
            PrintStream printStream = new PrintStream(socket.getOutputStream());
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            new Thread(() -> {
                while (true) {
                    try {
                        String msg = null;
                        if ((msg = bufferedReader.readLine()) != null) {
                            System.out.println("收到消息：" + msg);
                        }
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }).start();
            // 使用循环不断发送消息
            Scanner scanner = new Scanner(System.in);
            while (true) {
                System.out.println("请说: ");
                String msg = scanner.nextLine();
                printStream.println(msg);
                printStream.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## Buffer缓冲区

**数据从通道读入缓冲区，从缓冲区写入通道中**

Buffer就像一个数组，可以保存多个相同类型的数据

* 容量：作为一个内存块，Buffer具有一定的固定大小，容量不能为负，并且创建后不能更改
* 限制：表示可以操作的数据大小，limit后的数据不能读写，限制不能为负并且不能大于容量，**写入模式，限制等于buffer的容量，读取模式，limit等于写入的数据量**
* 位置：下一个要读取或者写入的数据的索引
* 标记与重置：标记是一个索引，**通过buffer中的mark方法指定一个特定的position之后，可以通过调用reset方法恢复到这个位置**

**0 <= mark <= position <= limit <= capacity**

![image-20221017131720499](D:\WorkSpace\Note\Picture\image-20221017131720499.png)

![image-20221017131812411](D:\WorkSpace\Note\Picture\image-20221017131812411.png)

**使用Buffer读写数据的四个步骤**

1. 写入数据到Buffer
2. 调用flip()方法，转换为读取模式
3. 从Buffer中读取数据
4. 调用Buffer.clear()方法或者buffer.compact()方法清除缓冲区

```java
public class BufferTest {
    // 对缓冲区常用API测试
    public static void main(String[] args) {
//        test01();
        test02();
    }

    public static void test02() {
        // 分配一个缓冲区，将容量设置为10
        ByteBuffer buffer = ByteBuffer.allocate(10);
        System.out.println(buffer.position()); // 0
        System.out.println(buffer.limit()); // 10
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // put在缓冲区中添加数据
        String name = "mnsx";
        buffer.put(name.getBytes(StandardCharsets.UTF_8));
        System.out.println(buffer.position()); // 4
        System.out.println(buffer.limit()); // 10
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // 清除缓冲区中的数据
        buffer.clear();
        System.out.println(buffer.position()); // 0
        System.out.println(buffer.limit()); // 10
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // 获取缓冲区中的数据
        char ch = (char) buffer.get();
        System.out.println(ch);
        // 定义新的缓冲区
        buffer = ByteBuffer.allocate(10);
        name = "mnsx";
        buffer.put(name.getBytes(StandardCharsets.UTF_8));
        buffer.flip();
        // 读取数据
        byte[] b = new byte[2];
        buffer.get(b);
        System.out.println(new String(b));
        System.out.println(buffer.position()); // 2
        System.out.println(buffer.limit()); // 4
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // 标记此刻的位置
        buffer.mark();
        byte[] bytes = new byte[2];
        buffer.get(bytes);
        System.out.println(new String(bytes));
        System.out.println(buffer.position()); // 4
        System.out.println(buffer.limit()); // 4
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // 回到标记位置
        buffer.reset();
        if (buffer.hasRemaining()) {
            System.out.println(buffer.remaining()); // 2
            System.out.println(buffer.position()); // 2
            System.out.println(buffer.limit()); // 4
            System.out.println(buffer.capacity()); // 10
            System.out.println("----------------------------------");
        }
    }

    public static void test01() {
        // 分配一个缓冲区，将容量设置为10
        ByteBuffer buffer = ByteBuffer.allocate(10);
        System.out.println(buffer.position()); // 0
        System.out.println(buffer.limit()); // 10
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // put在缓冲区中添加数据
        String name = "mnsx";
        buffer.put(name.getBytes(StandardCharsets.UTF_8));
        System.out.println(buffer.position()); // 4
        System.out.println(buffer.limit()); // 10
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // flip方法将缓冲区设置为当前位置，并将当前位置设置为0
        buffer.flip();
        System.out.println(buffer.position()); // 0
        System.out.println(buffer.limit()); // 4
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
        // get数据的读取
        char ch = (char) buffer.get();
        System.out.println(ch);
        System.out.println(buffer.position()); // 1
        System.out.println(buffer.limit()); // 4
        System.out.println(buffer.capacity()); // 10
        System.out.println("----------------------------------");
    }
}
```

**直接和非直接缓冲区**

buffer调用allocateDirect会产生直接缓冲区，将数据放置在非堆内存中，这样的效果是，创建缓冲区时，会有比直接缓冲区更大的消耗，但是因为直接操作内存数据，所以数据操作速率是大于非直接缓冲区的

**如果有很大的数据需要缓存，并且生命周期非常的长，那么可以使用直接缓冲区**

## Channel通道

**通道与流的优势**

* 通道可以同时进行读写，而流只读或者只写
* 通道可以实现异步读写数据
* 通道可以从缓冲区，读取或者写入数据

**常用的Channel实现类**

* FileChannel：用于读取、写入、映射和操作文件的通道
* DatagramChannel：通过UDP读写网络中的数据通道
* SocketChannel：通过TCP读写网络中的数据通道
* ServerSocketChannel：可以监听新进来的TCP连接，对每一个新进来的连接都会创建一个SocketChannel

**FileChannel常用方法**

![image-20221017144905236](D:\WorkSpace\Note\Picture\image-20221017144905236.png)

```java
public class ChannelTest {
    public static void main(String[] args) {
//       test01();
//        test02();
        test03();
    }

    public static void test03() {
        File srcFile = new File("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\io\\data01.txt");
        try {
            // 得到一个字节输入流
            FileInputStream fileInputStream = new FileInputStream(srcFile);
            // 得到一个字节输出流
            FileOutputStream fileOutputStream = new FileOutputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\data01.txt");
            // 获取文件通道
            FileChannel inChannel = fileInputStream.getChannel();
            FileChannel outChannel = fileOutputStream.getChannel();
            // 分配缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            while (true) {
                // 清空缓冲区再写入数据
                buffer.clear();
                // 开启读取一次数据
                int flag = inChannel.read(buffer);
                if (flag == -1) {
                    break;
                }
                // 切换可读模式
                buffer.flip();
                // 将数据写出
                outChannel.write(buffer);
            }
            inChannel.close();
            outChannel.close();
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public static void test02() {
        try {
            // 定义一个文件字节输入流
            FileInputStream fileInputStream = new FileInputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\io\\data01.txt");
            // 得到输入流的文件通道
            FileChannel channel = fileInputStream.getChannel();
            // 定义一个缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 读取数据到缓冲区
            channel.read(buffer);
            // 读取缓冲区中的数据并输出
            buffer.flip();
            String str = new String(buffer.array(), 0, buffer.remaining());
            System.out.println(str);
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    public static void test01() {
        // 得到一个字节输出流，通向目标文件
        FileOutputStream outputStream = null;
        try {
            outputStream = new FileOutputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\io\\data01.txt");
            // 得到字节输出流的通道
            FileChannel channel = outputStream.getChannel();
            // 分配缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put("hello nio".getBytes(StandardCharsets.UTF_8));
            // 切换写出模式
            buffer.flip();
            channel.write(buffer);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

分散：**将一个通道的数据分发到不同的缓冲区中**

聚集：**将多个缓冲区的数据集合到一个通道中传输’**

```java
public static void test04() {
    try {
        // 定义字节输入管道
        FileInputStream fileInputStream = new FileInputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\io\\data01.txt");
        FileChannel inChannel = fileInputStream.getChannel();
        // 定义字节输出管道
        FileOutputStream fileOutputStream = new FileOutputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\data01.txt");
        FileChannel outChannel = fileOutputStream.getChannel();
        // 定义多个缓冲区，数据分散
        ByteBuffer buffer1 = ByteBuffer.allocate(4);
        ByteBuffer buffer2 = ByteBuffer.allocate(1024);
        ByteBuffer[] byteBuffers = new ByteBuffer[]{buffer1, buffer2};
        // 从通道中读取数据到各个缓冲区
        inChannel.read(byteBuffers);
        // 从缓冲区中查询是否存在数据
        Arrays.stream(byteBuffers).forEach((item) -> {
            item.flip();
            System.out.println(new String(item.array(), 0, item.remaining()));
        });
        // 聚集写入
        outChannel.write(byteBuffers);
        inChannel.close();
        outChannel.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

**TransferFrom、TransferTo**

```java
public static void test05() {
    try {
        // 定义字节输入管道
        FileInputStream fileInputStream = new FileInputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\io\\data01.txt");
        FileChannel inChannel = fileInputStream.getChannel();
        // 定义字节输出管道
        FileOutputStream fileOutputStream = new FileOutputStream("D:\\WorkSpace\\JUC-Study\\juc_study\\src\\data01.txt");
        FileChannel outChannel = fileOutputStream.getChannel();
        // 复制数据
        outChannel.transferFrom(inChannel, 0, inChannel.size());
        inChannel.close();
        outChannel.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## Selector选择器

**多路复用**

* 可以用一个线程，处理多个客户端的连接，使用Selector
* Selector能够检测多个注册的通道，如果有事件发生，针对每个事件进行相应的处理
* 只有再通道真正事件发生时，才会进行读写，大大减少了系统开销，不必为每个连接都创建线程
* 避免了多线程之间的上下文切换导致的开销

`Seletor.open`创建选择器

```java
// 获取通道
ServerSocketChannel ssChannel = ServerSocketChannel.open();
// 切换非阻塞模式
ssChannel.configureBlocking(false);
// 绑定连接
ssChannel.bind(new InetSocketAddress(9898));
// 获取选择器
Selector selector = Selector.open();
// 将通道注册到选择器上，并且设置监听事件
ssChannel.register(selector, SelectonKey.OP_ACCEPT);
```

SelectionKey，**可以通过|来监听多个事件**

```java
    public static final int OP_READ = 1 << 0;

    public static final int OP_WRITE = 1 << 2;

    public static final int OP_CONNECT = 1 << 3;

    public static final int OP_ACCEPT = 1 << 4;
```

```java
public class Server {
    public static void main(String[] args) {
        try {
            // 获取通道
            ServerSocketChannel channel = ServerSocketChannel.open();
            // 切换非阻塞模式
            channel.configureBlocking(false);
            // 绑定端口
            channel.bind(new InetSocketAddress(9999));
            // 获取选择器
            Selector selector = Selector.open();
            // 注册选择器
            channel.register(selector, SelectionKey.OP_ACCEPT);
            // 使用selector轮询已经就绪的事件
            while (selector.select() > 0) {
                Iterator<SelectionKey> iterator = selector.selectedKeys().iterator();
                // 遍历
                while (iterator.hasNext()) {
                    SelectionKey key = iterator.next();
                    // 判断事件类型
                    if (key.isAcceptable()) {
                        // 获取当前接入的客户端通道
                        SocketChannel socketChannel = channel.accept();
                        // 切换成非阻塞模式
                        socketChannel.configureBlocking(false);
                        // 将通道注册到选择器
                        socketChannel.register(selector, SelectionKey.OP_READ);
                    } else if (key.isReadable()) {
                        // 获取读事件
                        SocketChannel socketChannel = (SocketChannel) key.channel();
                        // 读取数据
                        ByteBuffer buffer = ByteBuffer.allocate(1024);
                        int len = 0;
                        while ((len = socketChannel.read(buffer)) > 0) {
                            buffer.flip();
                            System.out.println(new String(buffer.array(), 0, len));
                            buffer.clear();
                        }
                    }
                    // 处理完毕移除事件
                    iterator.remove();
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Client {
    public static void main(String[] args) {
        try {
            // 获取通道
            SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("127.0.0.1", 9999));
            // 切换非阻塞模式
            socketChannel.configureBlocking(false);
            // 分配缓冲区
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            // 发送数据给服务端
            Scanner scanner = new Scanner(System.in);
            while (true) {
                System.out.println("请说: ");
                String msg = scanner.nextLine();
                buffer.put(("[" + LocalDateTime.now() + "]" + msg).getBytes(StandardCharsets.UTF_8));
                buffer.flip();
                socketChannel.write(buffer);
                buffer.clear();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

