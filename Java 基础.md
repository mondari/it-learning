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

## 重写（override）和重载（overload）的区别

重写：父类和子类中方法的名称和参数相同，但实现不同；

重载：方法名相同，但参数不同（本质上方法签名不同）。另外，方法名和参数相同，但是返回值不同，这不是有效的重载，编译时会报错。

## 接口和抽象类的区别

1. 接口中的所有方法都是抽象的，而抽象类只要求至少一个抽象方法；
2. 一个类可以实现多个接口，但只能继承一个抽象类；
3. 接口中的变量只能是 public static final，且默认就是 public static final；
4. 接口中的方法只能是 public，且默认声明为 public abstract；
5. Java 8 后接口允许有默认方法和静态方法，但必须要有方法体。

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

## final 关键字

- final 修饰的类，不能被继承
- final 修饰的方法，不能被重写
- final 修饰的变量，不能被二次赋值。

**因为父类的 private 修饰的方法是不能被子类重写，所以 private 修饰的方法默认隐性被 final 修饰的。**

## static 关键字

静态内部类

## protected 关键字

## String 为什么是不可变类

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

