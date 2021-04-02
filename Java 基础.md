[TOC]

（带 * 号的表示有待完善）

## 面向对象的三大特征

继承：子类继承父类的特征和行为

封装：隐藏对象的内部状态和实现细节，只对外公开接口

多态：同一个行为具有多个不同表现形式。主要体现在方法重写（override）、方法重载（overload）、抽象类和接口

## *面向对象的五大基本原则

- 单一职责原则（Single Responsibility Principle，简称SRP）：一个类有且只有一项职责

- 开放封闭原则（Open Close Principle，简称OCP）：对扩展开放，对修改封闭

- 里氏替换原则（Liskov Substitution Principle，简称LSP）：顾名思义，父类能用的地方，子类也能正常使用。引申的意义就是：**子类可以扩展父类的功能，但不能修改父类的原有功能。**具体来说：
  - 子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。
  - 子类中可以增加自己特有的方法。
  - 当子类的方法重载父类的方法时，方法的前置条件（即方法的入参）要比父类方法更宽松。
  - 当子类的方法实现父类的方法时（重载/重写或实现抽象方法），方法的后置条件（即方法的返回值）要比父类更严格或相等。

- 依赖倒置原则（Dependence Inversion Principle，简称DIP）：
  - 高层模块不应该依赖底层模块，两者都应该依赖其抽象
  - 抽象不应该依赖细节
  - 细节应该依赖抽象 

- 接口隔离原则（Interface Segregation Principles，简称ISP）：接口最小原则，不需要的接口方法不要添加接口中

- 迪米特原则（Law of Demeter ，简称LoD）：只与朋友联系，不与朋友的朋友联系，由朋友（中介）来与朋友的朋友联系，从而实现低耦合，高内聚

参考：[面向对象设计的七大设计原则详解](https://blog.csdn.net/qq_34760445/article/details/82931002

## 基本数据类型的大小

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

1. 接口中的所有方法都是抽象的，而抽象类则不是；

2. 接口是实现，而抽象类是继承，一个类可以实现多个接口，但只能继承一个抽象类，因为类是单继承的；

3. **接口可以继承接口，并可以继承多个接口，比如 ApplicationContext：**

   ```java
   public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
   		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
   ```

   接口不能实现接口，因为接口中的方法都必须是抽象的。

4. 接口中的变量不管有没有显式地声明，都是 public static final；

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

检查性异常：是指编译期会检查的异常，必须捕获处理，如果不知道怎么处理则应该抛出该异常。

运行时异常：是指编译期不进行检查，但运行时会出现的异常，不强制要求捕获处理。

常见的运行时异常有：

- NullPointerException 空指针异常
- ArrayIndexOutOfBoundsException 数组越界异常
- ClassCastException 类型转换异常
- ArithmeticException 算术异常
- IllegalArgumentException 非法传参异常
- FileNotFoundException 找不到文件异常
- ClassNotFoundException 找不到类异常

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
2. 若不用并行计算，很多时候计算速度没有比传统的 for 循环快（并行计算有时需要预热才显示出效率优势）
3. 不容易进行调试（或者说不方便调试）

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
2. 确保类中所有方法不改变对象内部状态。如果要提供 setter 等修改方法，则新建一个对象返回。
3. 将类中所有字段声明为 private。从而防止直接获取对象内部属性并修改。
4. 将类中所有字段声明为 final。从而防止对象的内部状态发生改变。

不可变类示例如下：

```java
// 类声明为 final
public final class ImmutableClass {
    // 内部变量声明为 private 和 final
    private final int value;

    public ImmutableClass(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }

    // 如果要提供 setter 等修改方法，则新建一个对象返回。
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
public class Test {
    public static void main(String[] args) throws Exception {
        String s = "Hello World";
        System.out.println("s = " + s);
 
        Field valueFieldOfString = String.class.getDeclaredField("value");
        valueFieldOfString.setAccessible(true);
 
        char[] value = (char[]) valueFieldOfString.get(s);
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



参考：https://www.cnblogs.com/dolphin0520/p/10693891.html

### String 为什么是不可变

一是为了提高性能，实现字符串常量池。字符串是频繁使用的数据类型，将字符串字面量缓存到字符串常量池中，当要使用的时候从中直接获取，无需频繁创建，有利于提高性能。

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

# JSP 篇

## JSP 与 Servlet 的区别

1. JSP 经 ApplicationServer 编译后就是 Servlet
2. JSP 负责处理页面显示，Servlet 负责处理业务逻辑
3. Servlet 中没有内置对象，必须通过 HttpServletRequest 对象，HttpServletResponse 对象以及 HttpServlet 对象得到 JSP 中的内置对象。

## JSP 九大内置对象

| 内置对象    | 类型                | 作用                                                    |
| ----------- | ------------------- | ------------------------------------------------------- |
| request     | HttpServletRequest  | HTTP请求                                                |
| response    | HttpServletResponse | HTTP响应                                                |
| session     | HttpSession         | HTTP会话                                                |
| application | ServletContext      | 多个Servlet可以通过ServletContext对象来实现数据间的共享 |
| pageContext | PageContext         | 获取其他八个内置对象                                    |
| config      | ServletConfig       | 获取服务器的配置信息                                    |
| page        | Object(this)        | 代表JSP页面本身                                         |
| out         | JspWriter           | 在浏览器中打印信息                                      |
| exception   | Throwable           | 异常                                                    |

## Cookie

Expires 属性（有效期）：如果没设置该值，则默认 Cookie 在关闭浏览器时失效。

Domain 属性（作用域）：

Path 属性：

HttpOnly 属性：用于防止客户端脚本通过 document.cookie 属性访问 Cookie，有助于保护 Cookie 不被跨站脚本攻击窃取或篡改。

Secure 属性：指定是否使用[HTTPS](https://baike.baidu.com/item/HTTPS/285356)安全协议发送 Cookie。使用HTTPS安全协议，可以保护 Cookie 在浏览器和Web服务器间的传输过程中不被窃取和篡改。

SameSite 属性：

参考：https://baike.baidu.com/item/cookie/1119

## Session

## Token

Token 又叫令牌，一般在登录、认证的场景下使用。

Token 使用步骤如下：

1. 当用户登录成功时，服务端会生成一个 Token，将其保存到 Redis，并返回给客户端；

2. 客户端拿到这个 Token 后，会保存到本地，下一次网络请求时会带上这个 Token；

3. 服务端接收到请求后，会将客户端的 Token 值与数据库的 Token 值对比

   - 存在，说明当前用户登录成功过，处于已登录状态
   - 不存在，说明没有登录成功过，或者是登录失效，需要重新登录

## Cookie、Session、Token的区别

Cookie、Session 和 Token 都是保持会话的方式，三者有一定的区别。



Cookie 和 Session 的区别：

- Cookie 是保存在浏览器上，而 Session 是保存在服务器上

- Cookie 有大小和数量限制；而 Session 理论上没有限制，实际上取决于服务器的内存大小

  一个 Cookie 的大小 4KB 左右（4095~4097字节之间），不同浏览器对 Cookie 数量的限制不同，以 IE 6 为例，每个站点能设置的 Cookie 总数不能超过 20 个。



Session 和 Token 的区别：

- Session 是基于 Cookie，需要借助 Cookie 来保存 SessionId。由于基于 Cookie，所以有 CSRF 攻击的风险
- Token 和 Session 原理差不多，但是 Token 可以保存在 Cookie、Local Storage 和 SessionStorage 中。并且 Token 是在请求头中手动携带，避免了 CSRF 攻击的风险。

参考：

https://www.jianshu.com/p/b4a9569823dd

https://www.cnblogs.com/belongs-to-qinghua/articles/11353228.html



## 跨域、CORS、CSRF

跨域是一种浏览器同源安全策略，即浏览器单方面限制脚本的跨域访问。发生跨域时，请求是可以正常发起，后端也能正常处理，但在返回的时候会被浏览器拦截掉，导致响应不可用。能论证这一点的著名案例就是CSRF跨站攻击，因为即使发生跨域，仍然能够发起 CSRF 攻击，只要后端服务器能正常处理 CSRF 攻击的请求就达到了攻击的目的，响应可不可用没有关系。



CORS（Cross-Origin Resource Sharing，跨源资源共享）是处理跨域的一种方式。其它处理跨域的方式有 JSONP、Nginx转发处理等



CSRF的全称是（Cross Site Request Forgery），即为跨站请求伪造，是一种利用浏览器在请求时会自动携带登录态的 Cookie 而发起的安全攻击。



参考：

《Spring Security 实战》“第8章 跨域与CORS”、“第9章 跨域请求伪造的防护”

## 如何防止表单重复提交

**一、有很多的应用场景都会遇到重复提交问题，比如**：

1、点击提交按钮两次。
2、点击刷新按钮。
3、使用浏览器后退按钮重复之前的操作，导致重复提交表单。
4、使用浏览器历史记录重复提交表单。
5、浏览器重复的 HTTP 请求。

二、**防止表单重复提交的方法**

1. 表单提交后将提交按钮设置为不可用。但是如果客户端禁止使用 JavaScript，这个方法无效。
2. Post/Redirect/Get 模式。表单提交后进行页面重定向，转到提交成功信息页面。
3. 数据库添加唯一索引。

参考：[如何防止表单重复提交](https://www.cnblogs.com/wenlj/p/4951766.html)

# 新特性篇

## JDK8

- lambda 表达式（由此带来的函数式编程、Stream流）
- 新的日期时间类：LocalDateTime
- 对 synchronize 进行了优化
- 对 ConcurrentHashMap 进行了优化

## JDK9