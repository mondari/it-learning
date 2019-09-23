[TOC]



## 面向对象的三大特征

继承

封装

多态：重写（override）和重载（overload）

## 面向对象的基本原则



## 基本数据类型的范围

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

参考 [Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

注：浮点数字面量默认表示 double 类型

## 静态代码块、非静态代码块、构造方法执行顺序

```java
class Parent {

    static {
        System.out.println("static code block in Parent");
    }

    {
        System.out.println("code block in Parent");
    }

    Parent() {
        System.out.println("constructor in Parent");
    }

}

class Son extends Parent {

    static {
        System.out.println("static code block in Son");
    }

    {
        System.out.println("code block in Son");
    }

    Son() {
        System.out.println("constructor in Son");
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
static code block in Parent
static code block in Son
code block in Parent
constructor in Parent
code block in Son
constructor in Son
```

由此可见，静态代码块是在类加载的时候执行，非静态代码是在构造方法调用前执行，父类的构造方法先于子类执行

## 重写（override）和重载（overload）的区别

重写：父类和子类中方法的名称和参数相同，但实现不同；

重载：方法名相同，但参数不同（本质上方法签名不同）。另外，方法名和参数相同，但是返回值不同，这不是有效的重载，编译时会报错。

## 接口和抽象类的区别及使用场景

1. 接口中的所有方法都是抽象的，而抽象类只要求至少一个抽象方法；

2. 接口是实现，而抽象类是继承，一个类可以实现多个接口，但只能继承一个抽象类，因为类是单继承的；**最重要的是，接口也可以继承接口，并可以继承多个接口，比如 ApplicationContext：**

   ```java
   public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
   		MessageSource, ApplicationEventPublisher, ResourcePatternResolver
   ```

   接口不能实现接口，因为接口中的方法都必须是抽象的。

3. 接口中的变量不管有没有显式地声明，默认都是 public static final；

4. 接口中的方法不管有没有显式地声明，默认都是 public abstract；

5. Java 8 中接口新增默认方法和静态方法，这两种方法必须要有方法体。

6. 接口表示的是这个对象能做什么，而抽象类表示的是这个对象是什么。

使用场景：

1. 一般将子类中相同方法的实现提取到抽象类中，其余方法留给不同的子类去实现。
2. 接口可以作为作为一种标识，没有任何方法和属性，比如 java.lang.Cloneable、java.io.Serializable。

## Exception 和 Error 的区别

Exception：是异常，指的是程序在正常运行时可能会发生的意外情况，是可以预料的问题，应该被捕获处理。

Error：是错误，一般指与虚拟机相关的问题，是不可以预料的，程序是没有能力去捕获处理这些问题。常见的 Eror 有：OutOfMemoryError 内存溢出错误

## 检查性异常和运行时异常的区别

检查性异常：是指编译期会检查的异常，必须捕获处理，如果不知道怎么处理则应该抛出该异常。

运行时异常：是指编译期不进行检查，但运行时会出现的异常，不强制要求捕获处理。常见的运行时异常有：NullPointerException 空指针异常、ArrayIndexOutOfBoundsException 数组越界异常、ClassCastException 类型转换异常、ArithmeticException 算术异常、IllegalArgumentException 非法传参异常等

## ClassNotFoundException 和 NoClassDefFoundError 的区别
ClassNotFoundException：当程序试图根据字符串名称通过以下三个方法加载类时，如果没找到就会抛出该异常。

- Class.forName

- ClassLoader.findSystemClass

- ClassLoader.loadClass

NoClassDefFoundError：如果Java虚拟机或 ClassLoader 实例试图加载**类的定义**时无法找到该类的定义，则抛出该错误。要查找的类在编译的时候是存在的，运行的时候却找不到了。造成该问题的原因一般是 jar 包遭到损坏或篡改，或者打包的时候漏掉了部分类。

## throw 和 throws 的区别

throw 用于手动抛出异常

throws 用于在方法或类签名上抛出一个或多个异常

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

- ”==“，如果表达式两边是基本数据类型或包装类，则比较两者的值是否相等。如果是其它引用数据类型，则比较两者的内存地址是否相同
- ”equals“，如果表达式两边是包装类，则比较两者的类型和值是否都相同。如果是其它引用数据类型，则看具体实现，默认实现是”==“比较两者的内存地址。

```java
// Integer 类的实现
public boolean equals(Object obj) {
    if (obj instanceof Integer) {
        return value == ((Integer)obj).intValue();
    }
    return false;
}

// Object 类的实现
public boolean equals(Object obj) {
    return (this == obj);
}
```



## final 关键字

- final 修饰的类，不能被继承
- final 修饰的方法，不能被重写
- final 修饰的变量，不能被二次赋值。

**因为父类的 private 修饰的方法是不能被子类重写，所以 private 修饰的方法默认隐性被 final 修饰的。**

## static 关键字

静态内部类

## protected 关键字

## String 为什么是不可变类

一是为了提高性能，使其能够缓存到字符串常量池，

二是为了线程安全，多线程场景下数据不会发生篡改，

三是为了能够缓存字符串的哈希值。



1. 什么是不可变类和不可变对象？

   由不可变类创建出来的对象就是不可变对象，不可变对象一旦被创建出来，其内部状态就不能发生改变，任何修改都会创建一个新的对象赋给原来的引用变量。不可变类有 String、Integer 及其它包装类。

2. 如何创建不可变类？

   1. 类声明为 final， 避免子类修改父类字段的的不可变性。
   2. 类的所有成员变量声明为 final ，防止对象的内部状态发生改变。

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

   

## *volatile 关键字

volatile 有两个作用，一是保证变量的可见性，二是防止指令重排序优化。

什么是变量的可见性：一个线程对共享变量进行修改，另一个线程能立即看到这个修改。

什么是指令重排序优化：Object o = new Object()

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

## JSP 与 Servlet 的区别

1. JSP 编译后就是 Servlet
2. 一般页面显示放在 JSP，业务逻辑放在 Servlet
3. Servlet 中没有内置对象，JSP 中的内置对象都是必须通过 HttpServletRequest 对象，HttpServletResponse 对象以及 HttpServlet 对象得到。

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

## 如何防止表单重复提交

参考：[如何防止表单重复提交](https://www.cnblogs.com/wenlj/p/4951766.html)

**一、有很多的应用场景都会遇到重复提交问题，比如**：

1、点击提交按钮两次。
2、点击刷新按钮。
3、使用浏览器后退按钮重复之前的操作，导致重复提交表单。
4、使用浏览器历史记录重复提交表单。
5、浏览器重复的 HTTP 请求。

二、**防止表单重复提交的方法**

1. 表单提交后将提交按钮设置为不可用。但是如果客户端禁止使用 JavaScript，这个方法旧无效。
2. Post/Redirect/Get 模式。表单提交后进行页面重定向，转到提交成功信息页面。
3. 数据库添加唯一索引。