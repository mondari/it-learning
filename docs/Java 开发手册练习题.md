

来源：

[【Java七天学习训练营】Day1](https://developer.aliyun.com/article/754900)

[【Java七天学习训练营】Day2](https://developer.aliyun.com/article/755082)

[【Java七天学习训练营】Day3](https://developer.aliyun.com/article/755087)

[【Java七天学习训练营】Day4](https://developer.aliyun.com/article/755086)

[【Java七天学习训练营】Day5](https://developer.aliyun.com/article/755085)

[【Java七天学习训练营】Day6](https://developer.aliyun.com/article/755084)



# 【Java七天学习训练营】Day3

**题目一： float a = 0.125f; double b = 0.125d; System.out.println((a - b) == 0.0); 代码的输出结果是什么？**
**A**. true
B. false

**题目二： double c = 0.8; double d = 0.7; double e = 0.6; 那么c-d与d-e是否相等？**
A. true
**B**. false

```
c - d = 0.10000000000000009
d - e = 0.09999999999999998
```



**题目三： System.out.println(1.0 / 0); 的结果是什么？**
A. 抛出异常
**B**. Infinity
C. NaN

**题目四： System.out.println(0.0 / 0.0); 的结果是什么？**
A. 抛出异常
B. Infinity
**C**. NaN
D. 1.0

**题目五： >>和>>>的区别是？**
A. 任何整数没有区别
B. 负整数一定没有区别
**C**. 浮点数可以>>运算，但是不可以>>>运算
D. 正整数一定没有区别

**题目六： 某个类有两个重载方法：void f(String s) 和 void f(Integer i)，那么f(null)的会调用哪个方法？**
A. 前者
B. 后者
C. 随机调用
**D**. 编译出错

**题目七： 某个类有两个重载方法：void g(double d) 和 void g(Integer i)，那么g(1)的会调用哪个方法？**
**A**. 前者
B. 后者
C. 随机调用
D. 编译出错

优先使用基本数据类型



**题目八： String a = null; switch(a)匹配case中的哪一项？**
A. null
B. "null"
C. 不与任何东西匹配，但不抛出异常
**D**. 直接抛出异常

**题目九： ` String get(String string, T t) { return string; } `此方法：**
A. 编译错误，从左往右第一个String处
B. 编译错误，T处
C. 编译错误，Alibaba处
**D**. 编译正确

**题目十： HashMap 初始容量 10000 即 new HashMap(10000)，当往里 put 10000 个元素时，需要 resize 几次（初始化的那次不算）？**
**A**. 1次
B. 2次
C. 3次
D. 0次

初始容量是比 10000 大的 2 的指数次方，当负载达到负载因子时（也就是初始容量的一半时），就会扩容