[TOC]



## 基本数据类型的范围

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

[Primitive Data Types](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)

## 重载和重写的区别

## 接口和抽象类的区别

## 字符流和字节流的区别

## 检查性异常和运行时异常的区别

## Exception 和 Error 的区别

## ClassNotFoundException 和 NoClassDefFoundError 的区别

## throw 和 throws 的区别

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

   需要满足以下条件：

   1. 类的所有字段必须用 final 修饰，防止对象的状态发生改变。
   2. 类必须用 final 修饰，避免子类修改父类字段的的不可变性。

   例如：

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

   

   

   

   