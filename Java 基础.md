[TOC]



## *面向对象的三大特征

继承：子类继承父类的特性和行为

封装：隐藏对象的内部状态和实现细节，只对外公开接口

多态：同一个行为具有多个不同表现形式。主要体现在方法重写（override）、方法重载（overload）、抽象类和接口

## *面向对象的五大基本原则

单一职责原则（Single Responsibility Principle，简称SRP）：一个类只负责一项职责
开放封闭原则（Open Close Principle，简称OCP）：对扩展开放，对修改封闭
里氏替换原则（Liskov Substitution Principle，简称LSP）
依赖倒置原则（Dependence Inversion Principle，简称DIP）
接口隔离原则（Interface Segregation Principles，简称ISP）

## 基本数据类型的范围

- byte/8
- **char/16**
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

参考 [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

**注：浮点数字面量默认表示 double 类型**

## 静态代码块、普通代码块、构造方法执行顺序

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

由此可见，静态代码块是在类加载的时候执行，普通代码块是在构造方法调用前执行，父类的构造方法先于子类执行，即：静态代码块 > 父类普通代码块 > 父类构造方法 > 子类普通代码块 > 子类构造方法



**静态变量初始化和静态代码块哪个先执行？**

**在静态代码块中创建一个对象，并在构造方法和普通代码块中打印其成员变量，成员变量会被初始化吗？**

请带着这些问题看以下代码：

```java
public class Test {

    int field = 10;

    static int staticField = staticMethod();

    static int staticMethod() {
        System.out.println("static method execute");
        return 20;
    }

    static {
        // 静态代码块中创建一个对象
        Test test = new Test();
        // 这里打印了静态变量，从执行结果中可以判断静态变量初始化、静态代码块的执行顺序
        System.out.println("static codeblock 's field: " + test.field + ", staticField: " + staticField);

    }

    {
        System.out.println("codeblock 's field: " + field + ", staticField: " + staticField);
    }

    public Test() {
        System.out.println("constructor 's field: " + field + ", staticField: " + staticField);
    }

    public static void main(String[] args) {
        new Test();
    }

}
```

执行结果如下：

```
static method execute
codeblock 's field: 10, staticField: 20
constructor 's field: 10, staticField: 20
static codeblock 's field: 10, staticField: 20
codeblock 's field: 10, staticField: 20
constructor 's field: 10, staticField: 20
```

由此可见：

1. 类加载时，先初始化静态变量，然后再执行静态代码块；
2. 凡是调用了构造方法，都会先执行普通代码块；
3. 在静态代码块中创建对象，对象内部的成员变量也会初始化好。

## 重写（override）和重载（overload）的区别

重写：父类和子类中方法的名称和参数相同，但实现不同；

重载：方法名相同，但参数不同。方法名+参数=方法签名，重载其实就是方法名相同，但是方法签名不同。需要注意的是，重载跟返回值没有任何关系，不管返回值相不相同，只要方法名和参数相同，都不是有效的重载，编译时会报错。

## 接口和抽象类的区别及使用场景

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

**使用场景**：

- 如果多个类有相同的方法（方法的实现是一样的），并且能够抽象出共同的父类，则使用抽象类将相同的方法提取封装出来。

- 如果多个类不能抽象出共同的父类，但是却有相同的行为，则使用接口将这些行为封装起来。

- 接口可以作为作为一种标识，没有任何方法和属性，比如 java.lang.Cloneable、java.io.Serializable。

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

Exception：是异常，指的是程序在正常运行时可能会发生的意外情况，是可以预料的问题，应该被捕获处理。

Error：是错误，一般指与虚拟机相关的问题，是不可以预料的，程序是没有能力去捕获处理这些问题。常见的 Eror 有：OutOfMemoryError 内存溢出错误

## 检查性异常和运行时异常的区别

检查性异常：是指编译期会检查的异常，必须捕获处理，如果不知道怎么处理则应该抛出该异常。

运行时异常：是指编译期不进行检查，但运行时会出现的异常，不强制要求捕获处理。常见的运行时异常有：NullPointerException 空指针异常、ArrayIndexOutOfBoundsException 数组越界异常、ClassCastException 类型转换异常、ArithmeticException 算术异常、IllegalArgumentException 非法传参异常等

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

## final 关键字

- final 修饰的类，不能被继承
- final 修饰的方法，不能被重写
- final 修饰的变量，不能被重复赋值（保证对象的引用不发生改变，但是对象的内部状态可以发生改变。）

**因为父类的 private 修饰的方法是不能被子类重写，所以 private 修饰的方法默认隐性被 final 修饰的。**

## *protected 关键字

## *volatile 关键字

volatile 有两个作用，一是保证变量的可见性，二是防止指令重排序优化。

什么是变量的可见性：一个线程对共享变量进行修改，另一个线程能立即看到这个修改。

**什么是指令重排序优化**：Object o = new Object()

## *静态内部类的作用及使用场景

## 值传递与引用传递

参见 [Java 虚拟机](Java 虚拟机.md)

# 字符串篇

## *String StringBuilder StringBuffer 的区别

## 什么是不可变类，如何创建不可变类？

**什么是不可变类？**

不可变类是指创建出来的对象其内部状态不能发生改变的类，任何改变都会创建一个新的对象赋予原来的引用变量。不可变类创建出来的对象叫做不可变对象。不可变类有 String、Integer 及其它包装类。

**如何创建不可变类？**

1. 类声明为 final， 使其不能被继承。
2. 类的所有成员变量声明为 final ，防止对象的内部状态发生改变。
3. 没有 set 方法，只有 get 方法。

```java
final class ImmutableClass {

 private final String name;

    public ImmutableClass(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

}
```


## String 为什么是不可变类

一是为了提高性能，实现字符串常量池。字符串是频繁使用的数据类型，将字符串字面量缓存到字符串常量池中，当要使用的时候从中直接获取，无需频繁创建，有利于提高性能。

二是为了线程安全，保障多线程场景下数据不会发生篡改。

三是为了能够缓存字符串的哈希值，保证了字符串哈希值的不变性。

## *为什么其它包装类是不可变类？

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

1. JSP 编译后就是 Servlet
2. JSP 负责处理页面显示，Servlet 负责处理业务逻辑
3. Servlet 中没有内置对象，必须通过 HttpServletRequest 对象，HttpServletResponse 对象以及 HttpServlet 对象得到 JSP 中的内置对象。

## JSP 九大内置对象

| 内置对象    | 类型                | 作用                 |
| ----------- | ------------------- | -------------------- |
| request     | HttpServletRequest  |                      |
| response    | HttpServletResponse |                      |
| session     | HttpSession         |                      |
| application | ServletContext      |                      |
| pageContext | PageContext         | 获取其他八个内置对象 |
| config      | ServletConfig       | 获取服务器的配置信息 |
| page        | Object(this)        | 代表JSP页面本身      |
| out         | JspWriter           | 在浏览器中打印信息   |
| exception   | Throwable           |                      |

## cookie 和 session 的区别

- cookie 存放在客户端浏览器上，而 session 存放在服务器上。

- cookie 不安全

- cookie 有大小和数量的限制，而 session 的大小取决于服务器的内存

  PS：一个浏览器能创建的 Cookie 数量最多为 300 个，并且每个不能超过 4KB，每个 Web 站点能设置的 Cookie 总数不能超过 20 个

一般建议把登录信息等重要信息存放在 session，其它信息存放在 cookie

## Token 的使用场景

Token 是身份验证的一种方式，我们通常叫它：令牌。Token 一般用在授权、登录、注册场景中。

Token 使用步骤如下：

1. 当用户登录成功时，服务端会生成一个 Token，将其保存到数据库，并发送给客户端；
2. 客户端拿到这个 Token 值，会保存到本地，下一次网络请求时会带上这个 Token 值；
3. 服务端接收到请求后，会将客户端的 Token 值与数据库的 Token 值对比
   1. Token 值相同，说明当前用户登录成功过，处于已登录状态
   2. Token 值不同，说明原来的登录信息已经失效，需要重新登录
   3. Token 值不存在，说明没有登录成功过。

## 如何防止表单重复提交

参考：[如何防止表单重复提交](https://www.cnblogs.com/wenlj/p/4951766.html)

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