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

## 接口和抽象类的区别

1. 接口中的所有方法都是抽象的，而抽象类只要求至少一个抽象方法；

2. 一个类可以实现多个接口，但只能继承一个抽象类；**另外，接口也可以继承接口，比如**

   ```java
   public interface BlockingQueue<E> extends Queue<E>
   ```

   BlockingQueue 和 Queue 都是接口。

3. 接口中的变量只能是 public static final，且默认就是 public static final；

4. 接口中的方法只能是 public，且默认声明为 public abstract；

5. Java 8 后接口允许有默认方法和静态方法，但必须要有方法体。

使用场景

- 可以把公共的方法提取到抽象类中，然后具体的方法可以留给子类实现即可
- 接口表示的是这个对象能做什么，抽象类表示的是这个对象是什么。所以，如果关注对象的操作的话，使用接口。

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

## 列几个 finally 不会被执行的情况

1. try 或 catch 中异常退出

   ```java
   try{
       System.exit(1);
   }finally{
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



1. 首先我们看看什么是不可变对象？

   不可变对象是指对象一旦被创建，其内部状态就不能再发生改变，任何修改都会创建一个新的对象，如 String、Integer 及其它包装类。

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
   
   

## volatile 关键字

volatile 有两个作用，一是保证变量的可见性，二是防止指令重排序优化。

什么是变量的可见性：一个线程对共享变量进行修改，另一个线程能立即看到这个修改。



## cookie 和 session 的区别

- cookie 存放在客户端浏览器上，而 session 存放在服务器上。
- cookie 不安全
- cookie 有大小和数量的限制，session 的大小取决于服务器的内存
- 一般建议把登录信息可以存放在 session，其它信息存放在 cookie