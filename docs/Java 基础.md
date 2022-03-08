## *什么是面向对象？

> Object-Oriented Programming，简称 OOP



面向过程 - 步骤化

面向对象 - 行为化



类用于描述客观世界中同一种类对象(Real-world objects of the same kind)的共同特征和行为。

对象是类的实例，是类的具体存在。



参考：

https://docs.oracle.com/javase/tutorial/java/concepts/index.html

## 面向对象的三大基本特征

> Fundamental Principles of OOP

继承：子类继承父类的属性和方法。

封装：隐藏对象的内部属性，对外只提供公开的方法。

多态：不同对象的同一个行为具有多个不同表现形式。

**抽象**：国内基本会遗漏这点，国外则会加上。抽象是将事物的共同特征和行为抽离出来，构造成类，而忽略不相关的其它方面

> 继承 Inheritance；封装 Encapsulation；多态 Polymorphism；**抽象 Abstraction**



Java 如何实现面向对象的三大基本特征？

- 通过 extends 关键字来实现继承
- 通过访问修饰符来实现封装
- 通过 overload 重载、override 重写、抽象类和接口来实现多态



参考：

https://docs.oracle.com/javase/tutorial/java/concepts/index.html

[聊聊 Go 语言中的面向对象编程](https://zhuanlan.zhihu.com/p/94625212)

https://www.jianshu.com/p/03ea25b3f4d9

https://blog.csdn.net/wateronly/article/details/2880869

## *面向对象的五大设计原则

> Object-Oriented Design Principles

- 单一职责原则（SRP，全称 Single Responsibility Principle）：一个类有且只有一项职责。

- 开放封闭原则（OCP，全称 Open Close Principle）：对象或实体应该对扩展开放，对修改封闭。

- 里氏替换原则（LSP，全称 Liskov Substitution Principle）：父类能用的地方，子类也要能正常使用。引申的意义就是：**子类可以扩展父类的功能，但不能修改父类的原有功能。**具体来说：
  - 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
  - 子类中可以增加自己特有的方法。
  - 当子类的方法重载父类的方法时，方法的入参要比父类更宽松。
  
- 依赖倒置原则（DIP，全称 Dependence Inversion Principle）：
  - 高层模块不应该依赖底层模块，两者都应该依赖其抽象。
  - （笔者记：也就是说高层模块和底层模块之间要有个抽象层，这样底层模块无论如何变，高层模块都不需要跟着变。Spring 中常见的有日志抽象层、网络请求抽象层、Resource 抽象层、ByteBuffer 抽象层）
  - 抽象不应该依赖细节
  - 细节应该依赖抽象 
  
- 接口分隔原则（ISP，全称 Interface Segregation Principles）：接口最小原则，不需要的接口方法不要添加接口中

  总而言之，接口分隔原则指导我们：

  - 一个类对一个类的依赖应该建立在最小的接口上
  - 建立单一接口，不要建立庞大臃肿的接口
  - 尽量细化接口，接口中的方法尽量少



- 迪米特原则（LoD，全称 Law of Demeter）：只与朋友联系，不与朋友的朋友联系，由朋友（中介）来与朋友的朋友联系，从而实现低耦合，高内聚
- 组合/聚合复用原则（CARP，全称 Composite/Aggregate Reuse Principle）



参考：

http://www.cs.utsa.edu/~cs3443/notes/designPrinciples/designPrinciples.html

https://www.oodesign.com/design-principles.html

[面向对象的三大基本特征，五大基本原则 - 风之之 - 博客园](https://www.cnblogs.com/fzz9/p/8973315.html)

[面向对象设计的七大设计原则详解 - CSDN](https://blog.csdn.net/qq_34760445/article/details/82931002)

## 基本数据类型的大小

> Primitive Data Type

- byte/8
- **char/16**
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

**注：浮点数字面量默认是 double 类型**，这个在笔试题中出现过。



参考：[Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

## *访问修饰符

> Access Modifier



## 静态代码块、普通代码块、构造方法执行顺序

静态代码块是在类加载的时候执行，普通代码块是在构造方法调用前执行，父类的构造方法先于子类执行。

示例代码：

```java
class Parent {

    static {
        System.out.println("parent static codeblock");
    }

    {
        System.out.println("parent codeblock");
    }

    Parent() {
        System.out.println("parent constructor");
    }

}

class Son extends Parent {

    static {
        System.out.println("son static codeblock");
    }

    {
        System.out.println("son codeblock");
    }

    Son() {
        System.out.println("son constructor");
    }
}

public class Test {

    public static void main(String[] args) {
        new Son();
    }

}
```

执行结果如下：

```bash
parent static codeblock
son static codeblock
parent codeblock
parent constructor
son codeblock
son constructor
```

总结上面的执行顺序：静态代码块 > 父类普通代码块 > 父类构造方法 > 子类普通代码块 > 子类构造方法



## 静态变量赋值和静态代码块哪个先执行？

答案：

1. 类加载时，~~会先初始化静态变量，然后再执行静态代码块~~，两者是按照代码先后顺序执行；
2. 在静态代码块中创建对象，对象内部的成员变量也会初始化好。

示例代码：

```java
public class Test {

    int field = 10;

    static int staticField = 20;

    static {
        // 先打印静态变量，判断是否已经初始化并赋值
        System.out.println("静态变量值为: " + staticField);
        // 再给静态变量赋值
        staticField = 30;
        System.out.println("静态变量值为: " + staticField);

        // 静态代码块中创建一个对象，判断是否会初始化其内部成员变量
        Test local = new Test();
        System.out.println("成员变量值为：" + local.field);
    }

    public static void main(String[] args) {
        new Test();
    }

}
```

执行结果如下：

```
静态变量值为: 20
静态变量值为: 30
成员变量值为：10
```



## 重写（override）和重载（overload）的区别

重写：**方法名、参数和返回值都相同**，但实现不同。它是子类去重写父类的方法，或者是实现类去实现接口的方法。

重载：**方法名相同，但参数不同**。重载的本质就是方法名相同，而方法签名不同，其中方法签名=方法名+参数。需要注意的是，**重载跟返回值没有任何关系**，不管返回值相不相同，只要方法名和参数相同，都不是有效的重载，编译时会报错。

## *接口和抽象类的区别及使用场景

1. **接口是对行为的抽象，而抽象类时对属性和行为的抽象**；

1. 接口中的所有方法都是抽象的，而抽象类不一定是；

2. 接口是实现，而抽象类是继承。接口可以实现多个，但类只能继承一个（Java 是单继承）；

4. **注意**：接口可以继承接口，并且可以继承多个接口，如下。但接口不能实现接口，因为接口中的方法都必须是抽象的。

   ```java
   public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
   		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
   ```

5. 接口中的变量不管有没有显式地声明，都是 public static final；

5. 接口中的方法不管有没有显式地声明，都是 public abstract；

6. Java 8 中接口新增默认方法和静态方法，这两种方法必须要有方法体。

**接口的使用场景**：

1. 需要将一组类视为单一的类，调用者只通过接口来与这组类发生联系。比如 Feign 接口，IService 接口(面向接口编程，MVC)；
2. 作为一种标识，没有任何属性和方法，比如 java.lang.Cloneable、java.io.Serializable；
3. 需要实现特定的多项功能，而这些功能之间可能完全没有任何联系；

**抽象类的使用场景**：

1. 定义了一组接口，但又不想强迫每个实现类都必须实现所有接口。则可以使用抽象类来实现这组接口，然后由子类选择自己感兴趣的方法来覆盖；
2. 定义了一组方法，其中一些方法是共同的，与状态无关的，可以共享的，无需子类分别实现；而另一些方法却需要各个子类根据自己的状态来实现特定的功能；



参考：[抽象类和接口的区别以及使用场景（记）](https://blog.csdn.net/lamyuqingcsdn/article/details/50501871)



**以下是我自己总结的使用场景**：

- 如果多个类有**公共的属性和方法**（不管其实现是否一样的），并且能够抽象出共同的父类，则使用抽象类将公共的属性和方法封装出来。

- 如果多个类不能抽象出共同的父类，但是却有相同的行为，则使用接口将这些行为封装起来。


比如：报纸、杂志、手机都可以用来阅读，但是三者不能抽象出共同的父类，应该使用接口将“阅读”这个行为封装起来；另外报纸、杂志两者可以抽象出共同的父类“出版物”，可以使用抽象类将两者封装起来。于是就成了酱紫：

```java
/**
 * “阅读”行为
 */
interface Readable {
    Object read();
}

/**
 * 手机
 */
class Phone implements Readable {

    @Override
    public Object read() {
        // do something...
        return null;
    }

}

/**
 * 出版物
 */
abstract class Press implements Readable {
}

/**
 * 杂志
 */
class Magazine extends Press {

    @Override
    public Object read() {
        // do something...
        return null;
    }
}

/**
 * 报纸
 */
class Newspaper extends Press {

    @Override
    public Object read() {
        // do something...
        return null;
    }
}
```



## Exception 和 Error 的区别

Exception：是异常，指的是程序在正常运行时可能会发生的意外情况，是**可以预料**的问题，应该被捕获处理。

Error：是错误，一般指与虚拟机相关的问题，是**不可以预料**的，程序是没有能力去捕获处理这些问题。常见的 Eror 有：OutOfMemoryError 内存溢出错误

## 检查性异常和运行时异常的区别

检查性异常（Checked Exception）：编译器在编译时会检查的异常。这些异常要求必须捕获处理，如果不知道怎么处理，则应该抛出去。检查性异常有：

- IOException（比如 FileNotFoundException）
- ClassNotFoundException

运行时异常（Runtime Exception）：在编译时不会检查、但运行时会出现的异常。这些异常虽然不强制要求捕获处理，但能够在编码阶段去避免。运行时异常有：

- NullPointerException 空指针异常
- IndexOutOfBoundsException 越界异常
- ClassCastException 类型转换异常
- ArithmeticException 算术异常
- IllegalArgumentException 非法传参异常

## ClassNotFoundException 和 NoClassDefFoundError 的区别
ClassNotFoundException：当程序试图根据字符串名称通过以下三个方法加载类时，如果没找到类的定义就会抛出该异常。

- Class.forName

- ClassLoader.findSystemClass

- ClassLoader.loadClass

NoClassDefFoundError：这是 JVM 在编译时能找到类的定义，但在运行时找不到类的定义而引发的错误。造成该问题的原因一般是 jar 包遭到损坏或篡改，或者打包的时候漏掉了部分类。

## throw 和 throws 的区别

throw 用于手动抛出异常

throws 用于在方法或类签名上抛出一个或多个异常

```java
public void method() throws Exception {
    throw new Exception();
}
```



## 列几个 finally 不会被执行的情况

1. try 或 catch 中异常退出

   ```java
   try {
       System.exit(1);
   } finally {
       // do something
   }
   ```

2. 无限循环

   ```java
   try {
       while (true) {
           // do something...
       }
   } finally {
       // do something...
   }
   ```
3. 线程被杀死
   当执行 try，finally 的线程被杀死时，finally 也无法执行。



总结
1，不要在 finally 中使用 return 语句。
2，finally 总是执行，除非程序或者线程被中断。  

## “==” 和 “equals” 的区别

- ”==“，如果表达式两边是基本数据类型，或者一边是包装类一边是基本数据类型，则比较两者的值是否相等。如果是其它引用数据类型，则比较两者的内存地址是否相同
- ”equals“，如果表达式两边是包装类，则比较两者的类型和值是否都相同。如果是其它引用数据类型，则看具体实现，默认实现是”==“比较两者的内存地址。

```java
// Integer 类的实现
public boolean equals(Object obj) {
    // 相比较对象的类型
    if (obj instanceof Integer) {
        // 再比较对象的值
        return value == ((Integer)obj).intValue();
    }
    return false;
}

// Object 类的实现
public boolean equals(Object obj) {
    return (this == obj);
}
```

例如：

```
Integer integer = 1;
double d = 1.0;

// 一边是包装类，一边是基本数据类型，比较值是否相等，返回结果是 true
System.out.println(integer == d);

// equals 比较包装类，先比较类型是否相同，再比较值是否相等，返回结果是 false
System.out.println(integer.equals(d));
```

## 为什么重写equals必须重写hashCode？

保证 equals 相等的对象 hashCode 也相等

## final 关键字

- final 修饰的类，不能被继承
- final 修饰的方法，不能被重写
- final 修饰的变量，不能被重复赋值（保证对象的引用不发生改变，但是对象的内部状态可以发生改变。）

**因为父类的 private 修饰的方法是不能被子类重写，所以 private 修饰的方法默认隐性被 final 修饰的。**

## *protected 关键字

protected 修饰的成员变量和方法，可以被该类自身、同一个包中的其它类、不同包下该类的子类所访问。



## *静态内部类的作用及使用场景

## 值传递与引用传递

参见 [Java 虚拟机](Java 虚拟机.md)

## Lambda 表达式的优缺点

优点：

1. 代码简洁

2. 非常容易进行并行计算


缺点：

1. 代码可读性变差（因为代码简洁了，所以可读性变差了）
2. 不容易进行调试（或者说不方便调试）
3. 若不用并行计算，很多时候计算速度没有比传统的 for 循环快（并行计算有时需要预热才显示出效率优势）

参考：[说说Lamda表达式的优缺点](https://www.nowcoder.com/questionTerminal/5d29d10e35fd4003b3c186b02ab073f0)

# 字符串篇

## String StringBuilder StringBuffer 的区别

String：不可变，每次修改都会申请新的内存空间

StringBuider：线程不安全，可变，初始容量为16，容量不够会自动扩容

StringBuffer：线程安全，其它同 StringBuilder

## 不可变类

参考： [effective-java-3rd-chinese - 17.最小化可变性](https://gitee.com/mondari/effective-java-3rd-chinese/blob/master/docs/notes/17.%20%E6%9C%80%E5%B0%8F%E5%8C%96%E5%8F%AF%E5%8F%98%E6%80%A7.md)

### 什么是不可变类和不可变对象？不可变类有哪些？

不可变类是指创建出来的对象其内部状态不能发生改变的类，任何改变都要创建一个新的对象赋予其引用变量。不可变类创建出来的对象叫做不可变对象。不可变类有 `String` 类、基本数据类型包装类以及 `BigInteger` 类和 `BigDecimal` 类。

### 如何创建不可变类？

1. 确保类不被继承。通常将类声明为 final，防止子类破坏父类的不可变行为。
2. 确保类中所有方法不改变对象内部状态。**凡是涉及到内部属性的修改，都新建一个对象返回。**
3. 将类中所有字段声明为 private。防止直接获取对象内部属性并修改。
4. 将类中所有字段声明为 final。防止对象的内部属性发生改变（值改变或引用改变）。

不可变类示例如下：

```java
// 类声明为 final
public final class ImmutableClass {
    // 内部属性声明为 private 和 final
    private final int value;

    public ImmutableClass(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    // （重点）凡是涉及到内部属性的修改，都新建一个对象返回
    public ImmutableClass setValue(int value) {
        return new ImmutableClass(value);
    }

    public static void main(String[] args) {
        ImmutableClass a = new ImmutableClass(5);
        ImmutableClass b = a;
        System.out.println(a.setValue(10).getValue());// 打印 10
        System.out.println(b.getValue());// 打印 5
    }
}
```



不可变对象并不是真正的不可变，还是能够通过反射来修改其值。不可变类和不可变对象的初衷是为了方便大家编写代码，减少编码时出错的概率。

```java
import java.lang.reflect.Field;

class Test {
    public static void main(String[] args) throws Exception {
        String s = "Hello World";
        System.out.println("s = " + s);

        Field field = String.class.getDeclaredField("value");
        field.setAccessible(true);

        char[] value = (char[]) field.get(s);
        value[5] = '_';
        System.out.println("s = " + s);
    }
}
```

控制台输出：

```
s = Hello World
s = Hello_World
```



这里简单演示一下不可变类如果不声明为final的后果：

```java
/**
 * 没有声明为final的不可变类
 */
class NonFinalImmutableClass {
    private final int value;

    public NonFinalImmutableClass(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    // 凡是涉及到内部属性的修改，都新建一个对象返回
    public NonFinalImmutableClass setValue(int value) {
        return new NonFinalImmutableClass(value);
    }

}

/**
 * 如果父类没有声明为final，则能够通过子类修改父类的不可变行为
 */
class ChildClass extends NonFinalImmutableClass {
    // 覆盖父类的不可变属性
    private int value;

    public ChildClass(int value) {
        super(value);
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    // 覆盖父类的不可变行为
    public NonFinalImmutableClass setValue(int value) {
        this.value = value;
        return this;
    }

    public static void main(String[] args) {
        NonFinalImmutableClass a = new NonFinalImmutableClass(5);
        NonFinalImmutableClass b = a;
        System.out.println(a.setValue(10).getValue());  // 10
        System.out.println(b.getValue());               // 5

        a = new ChildClass(5);
        b = a;
        // 子类修改了父类的不可变行为，导致和上面的结果不一致
        System.out.println(a.setValue(10).getValue());  // 10
        System.out.println(b.getValue());               // 10
    }
}
```



参考：https://www.cnblogs.com/dolphin0520/p/10693891.html

### String 为什么要设计成不可变？

一是为了实现字符串常量池，以提高性能。字符串是频繁使用的数据类型，如果能将字符串缓存到常量池中，当要使用的时候可以直接从中获取（字面量要相同），无需再创建一个 String 对象，则可以提高性能。如果 String 是可变的，则缓存在常量池中的 String 对象会被篡改，引起数据安全问题。

二是为了线程安全，保障多线程场景下数据不会发生篡改。

三是为了能够缓存字符串的哈希值，保证了字符串哈希值的不变性。

## Java 生成随机字符串数组

参考：[Java 生成随机字符串数组](https://www.jianshu.com/p/61db371f1635)

要求：

1. 创建一个存放指定数量的随机字符串的数组
2. 每条字符串的长度为10以内的随机整数
3. 每条字符串的字符为随机生成的字符
4. 每条随机字符串不可重复

### 生成一个随机字符

1. 先将所有的字母和0-9的数字存放于一个字符串中，以便后续使用。

```bash
String str = "aAbBcCdDeEfFgGhHiIjJkKlLmMnNoOpPqQrRsStTuUvVwWxXyYzZ0123456789";
```

2. 因为要满足随机性，所以创建一个 Random 对象，利用其中的 nextInt(str.length) 方法生成一个 0 — str.length 的随机数。

```cpp
Random random = new Random();
int index = random.nextInt(str.length());
```

3. 再将上述生成的随机数作为 str 字符串的索引取出相应的字符，及随机生成了一个字符

```cpp
char c = str.charAt(index);
```

### 生成一条长度为10以内的随机字符串

1. 因为是10以内且满足随机性，所以此处使用 Math.random() 函数，其返回值为随机 0.0 - 1.0 的 Double 类型的数

```cpp
StringBuffer stringBuffer = new StringBuffer();
int stringLength = (int) (Math.random()*10);
```

2. 现在字符串的长度可以确认，也实现了生成随机的字符，再利用 for 循环就可以生成一条长度为10以内的随机字符串

```cpp
for (int j = 0; j < stringLength; j++) {
    int index = random.nextInt(str.length());
    char c = str.charAt(index);
    stringBuffer.append(c);    
 }
//将StringBuffer转换为String类型的字符串
String string = stringBuffer.toString(); 
```

### 创建一个集合储存指定数量的随机字符串

1. 创建一个 String 类型的 List 集合

```jsx
List<String> list = new ArrayList<>();
```

2. 将每次生成的一条字符串添加到集合中，注意利用集合的 Contains() 方法判断集合中之前是否已存在相同的字符串（虽然概率很小）。

```java
String randomString;
// 过滤重复的字符串
do {
    StringBuilder stringBuilder = randomString();
    randomString = stringBuilder.toString();
} while (list.contains(randomString));
list.add(randomString);
```

### 完整代码

```java
// 测试代码
public static void main(String[] args) {
    List<String> randomString = randomStringArray(10);
    for (String s : randomString) {
        System.out.println(s);
    }
}

/**
 * 生成长度为10以内的随机字符串
 * @return
 */
private static StringBuilder randomString() {
    // 将所有字母和数字都放在一个字符串中
    String alphabet = "aAbBcCdDeEfFgGhHiIjJkKlLmMnNoOpPqQrRsStTuUvVwWxXyYzZ0123456789";
    // 控制字符串下标的随机数，从字符串中随机取出字符，以便组成一个随机字符串
    Random random = new Random();
    StringBuilder stringBuilder = new StringBuilder(10);
    // 由于字符串长度也是随机的，所以也是随机数
    double strLength = Math.random() * 10;
    for (int j = 0; j < strLength; j++) {
        // 字符串下标的随机数最大不能超过字符串的长度
        int index = random.nextInt(alphabet.length());
        char c = alphabet.charAt(index);
        stringBuilder.append(c);
    }
    return stringBuilder;
}

/**
 * 生成字符串不重复的随机字符串数组
 * @param size
 * @return
 */
private static List<String> randomStringArray(int size) {

    List<String> list = new ArrayList<>(size);
    for (int i = 0; i < size; i++) {
        String randomString;
        // 过滤重复的字符串
        do {
            StringBuilder stringBuilder = randomString();
            randomString = stringBuilder.toString();
        } while (list.contains(randomString));
        list.add(randomString);
    }
    return list;
}
```

## *删除字符串中字符出现次数最多的字符

比如 abbccd，得到结果为 ad

步骤如下：

1. 使用 String 的 subString(0, 1) 方法将字符串拆分成一个一个的字符，放到 HashMap 中，并统计其数量
2. 遍历 HashMap 中字符的个数，将字符个数最大的字符放到 ArrayList 中
3. 使用 String 的 replaceAll() 方法替换字符


# 新特性篇

## JDK8

- lambda 表达式（由此带来的函数式编程、Stream流）
- 新的日期时间类：LocalDateTime
- 对 synchronize 进行了优化
- 对 ConcurrentHashMap 进行了优化

## JDK9