## 第一行代码

```go
package main

import "fmt"

func main() {
   fmt.Println("Hello, World!")
}
```

执行以上代码输出：

```bash
$ go run hello.go 
Hello, World!
```

## Go 代理

Go 可以通过 `go install` 来安装包，但是由于墙的原因，很多包会安装失败。所以在安装 Go 后，建议第一时间设置 Go 代理，免除后面的困扰。

Go 代理设置方法：

```
1. 右键 我的电脑 -> 属性 -> 高级系统设置 -> 环境变量
2. 在 “[你的用户名]的用户变量” 中点击 ”新建“ 按钮
3. 在 “变量名” 输入框并新增 “GOPROXY”
4. 在对应的 “变量值” 输入框中新增 “https://goproxy.io,direct”
5. 最后点击 “确定” 按钮保存设置
```



参考：https://goproxy.io/zh/docs/getting-started.html

## 语言特性

计算机软件经历了数十年的发展，形成了多种学术流派，有面向过程编程、面向对象编程、函数式编程、面向消息编程等，这些思想究竟孰优孰劣，众说纷纭。

除了 OOP 外，近年出现了一些小众的编程哲学，Go语言对这些思想亦有所吸收。例如，**Go语言接受了函数式编程的一些想法，支持匿名函数与闭包。再如，Go语言接受了以 Erlang 语言为代表的面向消息编程思想，支持 goroutine 和通道，并推荐使用消息而不是共享内存来进行并发编程。**总体来说，Go语言是一个非常现代化的语言，精小但非常强大。

- 自动垃圾回收
- 更丰富的内置类型
- **函数多返回值**
- 错误处理
- **匿名函数和闭包**
- 类型和接口
- **并发编程**
- 反射
- 语言交互性

参考：https://www.runoob.com/go/go-tutorial.html

## Go 编译 Linux 可执行文件

GoLand 编译时在“Run/Debug Configurations - Configuration - Environment”中添加以下环境变量：

```
GOARCH=amd64;GOOS=linux
```

在 Windows 下建议取消 `Run after build` 复选框。

## *环境变量

## 流程控制语句

### 条件语句

Go 没有三目运算符，只有 if 语句。

#### if 语句

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	var result int = rand.Intn(100)

	if result > 80 {
		fmt.Println("good", result)
    // 注意 else 位置不能在新一行
	} else if result > 60 {
		fmt.Println("medium", result)
	} else {
		fmt.Println("bad", result)
	}
}
```

### 循环语句

Go 的循环语句只有 for 语句，没有 while 和 do-while 语句。

#### for 语句

Go 语言的 for 循环有 3 种形式，只有其中的一种使用分号。

和 C 语言的 for 一样：

```
for init; condition; increment { }
```

和 C 的 while 一样：

```
for condition { }
```

和 C 的 `for(;;)` 一样：

```
for { }
```

- init： 给控制变量赋初值；
- condition： 循环控制条件；
- increment：每次循环后对控制变量进行增减。

参考：https://www.runoob.com/go/go-for-loop.html

### 选择语句

#### switch 语句

```go
package main

import (
	"fmt"
	"math/rand"
)

func main() {
	var marks int = rand.Intn(10)

	switch marks {
	case 90:
		fmt.Println("A")
	case 80:
		fmt.Println("B")
	case 60, 70:
		fmt.Println("C")
	default:
		fmt.Println("D")
	}
}

```

switch 默认情况下 case 最后自带 break 语句，匹配成功后就不会执行其他 case，如果我们需要执行后面的 case，可以使用 **fallthrough**

使用 fallthrough 会强制执行后面的 case 语句，fallthrough 不会判断下一条 case 的表达式结果是否为 true。

```go
package main

import "fmt"

func main() {

	switch {
	case false:
		fmt.Println("1、case 条件语句为 false")
		fallthrough
	case true:
		fmt.Println("2、case 条件语句为 true")
		fallthrough
	case false:
		fmt.Println("3、case 条件语句为 false")
		fallthrough
	case true:
		fmt.Println("4、case 条件语句为 true")
	case false:
		fmt.Println("5、case 条件语句为 false")
		fallthrough
	default:
		fmt.Println("6、默认 case")
	}
}
```

以上代码执行结果为：

```
2、case 条件语句为 true
3、case 条件语句为 false
4、case 条件语句为 true
```

参考：https://www.runoob.com/go/go-switch-statement.html

### [Exercise: Loops and Functions](https://golang.google.cn/tour/flowcontrol/8)



## defer 延迟执行语句

defer 语句将会被放入延迟调用栈中，在 defer 语句所在的函数即将返回时（类似于 Java 的 finally 代码块），按照先进后出的顺序也就是逆序执行。

```go
package main

import (
   "fmt"
)

func main() {

   fmt.Println("defer begin")

   // 将defer放入延迟调用栈
   defer fmt.Println(1)

   defer fmt.Println(2)

   // 最后一个放入, 位于栈顶, 最先调用
   defer fmt.Println(3)

   fmt.Println("defer end")
}
```

执行结果如下：

```
defer begin
defer end
3
2
1
```

参考：http://c.biancheng.net/view/61.html

## 类型

基础类型：bool、string、整型、string、byte(alias for uint8)、rune(alias for int32)、float32、float64、complex32、complex64

复合类型：array、struct、interface、func、**pointer？**

引用类型：slice、map、chan



参考：https://go.dev/tour/moretypes/1

## 变量

参考：https://www.runoob.com/go/go-variables.html

### 变量定义

```go
package main

import "fmt"

func main() {
	var a string = "Runoob"
	fmt.Println(a)

    // 一次声明和初始化多个变量
	var b, c int = 1, 2
	fmt.Println(b, c)

    // 这叫“初始化声明”，会自动推断变量类型，但只能用在局部变量上
	d := true
	fmt.Println(d)
}
```

控制台输出：

```
Runoob
1 2
true
```



只声明不初始化，默认是零值

```go
package main

import "fmt"

func main() {
   // 字符串没有初始化就为空字符串
   var a string
   // 数值类型没有初始化就为零值
   var b int
   // bool 零值为 false
   var c bool
   
   fmt.Printf("%q %v %v\n", a, b, c)
}
```

控制台输出：

```
"" 0 false
```



以下几种类型为 **nil**：

```go
var a *int
var a []int
var a map[string] int
var a chan int
var a func(string) int
var a error // error 是接口
```



局部变量不允许只声明不适用，全局变量可以。如下：

```go
package main

import "fmt"

func main() {
   // 编译时报“a declared and not used”
   var a string = "abc"
}
```



### 空白标识符 `_`

空白标识符 `_` 用于抛弃值，如值 5 在 `_, b = 5, 7` 中被抛弃。

`_` 实际上是一个只写变量，你不能得到它的值。这样做是因为 Go 语言中你必须使用所有被声明的变量，但有时你并不需要使用从一个函数得到的所有返回值，如下：

```go
func main() {
	person := Person{"小明", 18}
	// json.Marshal 将对象转换为json字符串
	result, _ := json.Marshal(&person)
	fmt.Println(result)
}
```

`json.Marshal()` 是个多返回值函数，分别返回 byte 数组和 error，上面的 `_` 意味着不想得到 error 的值，否则拿到 error 的值就要处理，不处理编译就会报错。

### 类型转换

```go
package main

import "fmt"

func main() {

   i := 10
   fmt.Println(i / 3)
   // 将整型转化为浮点型
   fmt.Println(float32(i) / 3)

}
```

控制台输出：

```
3
3.3333333
```



go 不支持隐式转换类型，比如

```go
package main

import "fmt"

func main() {

	var a int32 = 10
	var b int64

    // 编译时此行会报 “cannot use a (type int32) as type int64 in assignment”
	b = a
	fmt.Println(a, b)

}
```

参考：https://www.runoob.com/go/go-type-casting.html

## 常量

```go
//隐式类型定义
const a = "golang"

//显示类型定义
const b string = "python"
```

常量必须声明的时候就初始化，也不能重复赋值。如下：

```go
//常量的声明和初始化必须放在一块，否则编译报“missing value in const declaration”
const a
a = "golang"

//常量定义后不能再赋值，否则编译报“cannot assign to b”
const b = "python"
b = "java"
```

### iota

Go 中的一个特殊常量， 相当于 const 语句块中的行索引，索引值从0开始。如果不是在 const 语句块中，则索引值始终是0

```go
package main

import "fmt"

func main() {
    // 不在 const 语句块中
    const x = iota
    const y = iota
    fmt.Println(x,y)
    
    // 在 const 语句块中
    const (
            a = iota   //0
            b          //1
            c          //2
            d = "ha"   //独立值，iota += 1
            e          //"ha"   iota += 1
            f = 100    //iota +=1
            g          //100  iota +=1
            h = iota   //7,恢复计数
            i          //8
    )
    fmt.Println(a,b,c,d,e,f,g,h,i)
}
```

控制台输出：

```
0 0
0 1 2 ha ha 100 100 7 8
```



再看个有趣的的 iota 实例：

```go
package main

import "fmt"

const (
	i = 1 << iota // iota = 0
	j = 3 << iota // iota = 1, 3<<1 其实就是3的二进制11左移1位变为110，即6
	k             // iota = 2, 3<<2 其实就是3的二进制11左移2位变为1100，即12
	l             // iota = 3, 3<<3 其实就是3的二进制11左移3位变为11000，即24
)

func main() {
	fmt.Println("i=", i)
	fmt.Println("j=", j)
	fmt.Println("k=", k)
	fmt.Println("l=", l)
}
```

控制台输出：

```
i= 1
j= 6
k= 12
l= 24
```

参考：https://www.runoob.com/go/go-constants.html

## 函数

### 函数返回多个值

Go 函数可以返回多个值，例如：

```go
package main

import "fmt"

func swap(x, y string) (string, string) {
   return y, x
}

func main() {
   a, b := swap("Google", "Runoob")
   fmt.Println(a, b)
}
```

以上实例执行结果为：

```
Runoob Google
```

参考：https://www.runoob.com/go/go-functions.html

### 交换两个数的值

```go
a := 100
b := 200
a, b = b, a
// a == 200
// b == 100
```

参考：https://www.runoob.com/go/go-function-call-by-value.html

### println 与 fmt.Println

println 函数是在 builtin 包中，是将结果输出到标准错误中，未来可能会删除。

fmt.Println 函数是在 fmt 包中，是将结果输出到标准输出中。

### *函数作为参数

### *匿名函数和闭包

### 函数与方法

Go 语言中同时有函数和方法。**一个方法就是一个包含了接受者的函数**，接受者可以是命名类型或者结构体类型的一个值或者是一个指针。所有给定类型的方法属于该类型的方法集。

下面定义一个结构体类型和该类型的一个方法：

```go
package main

import (
   "fmt"  
)

/* 定义结构体 */
type Circle struct {
  radius float64
}

//该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
  //c.radius 即为 Circle 类型对象中的属性
  return 3.14 * c.radius * c.radius
}

func main() {
  var c1 Circle
  c1.radius = 10.00
  fmt.Println("圆的面积 = ", c1.getArea())
}
```

参考：https://www.runoob.com/go/go-method.html

## 数组

类型 `[n]T` 是一个包含 n 个类型为 T 的值的数组，其中 n 为数组的长度。

声明一个长度为 5，类型为 float32，且名叫 array 的数组：

```go
var array [5]float32
```

注意数组一旦定义下来，其长度不可变。



声明并初始化

```go
// 声明长度为5，并初始化3个值，未初始化默认为0
balance := [5]int{1, 2, 3}

// 也可以指定下标初始化，初始化结果为[0,0,1,2,3]
balance := [5]int{2:1, 3:2, 4:3}
// 或
balance := [5]int{2:1, 2, 3}
```

如果数组长度不确定，可以使用 `...` 代替数组的长度，编译器会根据元素个数自行推断数组的长度：

```go
balance := [...]int{1, 2, 3}
```



遍历数组

```go
package main

import "fmt"

func main() {

	balance := [5]int{1, 2, 3}
    
	fmt.Println("数组长度", len(balance))
	fmt.Println("数组遍历方式一：")
	for index := range balance {
		fmt.Println(balance[index])
	}
	fmt.Println("数组遍历方式二：")
	for i, elem := range balance {
		fmt.Println("index:", i, "value:", elem)
	}
}
```

控制台输出：

```
数组长度 5
数组遍历方式一：
1
2
3
0
0
数组遍历方式二：
index: 0 value: 1
index: 1 value: 2
index: 2 value: 3
index: 3 value: 0
index: 4 value: 0
```



参考：https://www.runoob.com/go/go-arrays.html

## 切片(Slice)

切片是数组的引用，所以改变切片的值也会改变数组的值。

切片可以看做是数组的一个片段，其长度可变，但最大不能超过数组的长度。

通过 append 函数可以动态调整切片引用的数组。append 方法往切片添加元素时，如果数组的长度不足以支撑切片元素的数量时，就会分配一个更大的数组，新数组长度**一般？**为之前的 2 倍，并且切片将指向新数组。

### 切片定义

**直接初始化切片**

```
s :=[] int {1,2,3 } 
```

**[]** 表示是切片类型，**{1,2,3}** 初始化值依次是 **1,2,3**，其 **cap=len=3**。



**切片可以通过数组来初始化**

```
s := arr[:] 
```

初始化切片 **s**，是数组 arr 的引用。

```
s := arr[startIndex:endIndex] 
```

将 arr 中从下标 startIndex 到 endIndex-1 下的元素创建为一个新的切片。

```
s := arr[startIndex:] 
```

默认 endIndex 时将表示一直到arr的最后一个元素。

```
s := arr[:endIndex] 
```

默认 startIndex 时将表示从 arr 的第一个元素开始。



**切片可以通过切片或内置函数 make() 初始化**

```
s1 := s[startIndex:endIndex] 
```

通过切片 s 初始化切片 s1。

```
s :=make([]int,len,cap) 
```

通过内置函数 **make()** 初始化切片**s**，**[]int** 标识为其元素类型为 int 的切片。



### 数组与切片区别

- 切片有长度 len 和容量 cap 两个属性，而数组只有长度 len 一个属性，但数组仍然可以调用 cap() 函数，其返回结果与 len() 函数一致。

- 数组的长度是数组中元素个数。因为数组的长度是数组类型的一部分，所以一旦数组定义下来，其长度不可变。

- 切片的长度是切片中元素个数，容量是从切片第一个元素算起的数组中元素个数。**切片容量的计算是最容易出错的地方**，在计算切片的容量时，必须先知道切片第一个元素在数组中的位置，然后统计该元素到数组最后一个元素的个数，才是切片的容量。

- 切片长度和容量快速计算公式：

  slice.len = slice.highBound - slice.lowBound
  slice.cap = array.len - slice.lowBound

```go
package main

import "fmt"

func main() {
	array := [5]int{1, 2, 3}// [1 2 3 0 0]
	fmt.Println("数组长度:", len(array)) //5
 	fmt.Println("数组cap=len:", cap(array)) //5

	slice := array[1:3]// [2 3]
	fmt.Println("切片长度:", len(slice)) // 2
	fmt.Println("切片容量:", cap(slice)) // 4
}
```



**注意在 Go 中，数组是值传递，而切片才是引用传递，且其引用的是数组**。

```go
package main

import "fmt"

func main() {
	// 数组
	array := [...]int{2, 3, 5, 7, 11}
	reverseArray(array)
	fmt.Println(array)

	// 切片
	slice := []int{2, 3, 5, 7, 11}
	reverseSlice(slice)
	fmt.Println(slice)
}

/**
倒序排序数组，值传递
*/
func reverseArray(array [5]int) {
	array[0], array[len(array)-1] = array[len(array)-1], array[0]
}

/**
倒序排序切片，引用传递
*/
func reverseSlice(slice []int) {
	slice[0], slice[len(slice)-1] = slice[len(slice)-1], slice[0]
}

```

控制台输出：

```
[2 3 5 7 11]
[11 3 5 7 2]
```



把上面的代码稍微改一下：

```go
package main

import "fmt"

func main() {
	// 数组
	array := [...]int{2, 3, 5, 7, 11}
	reverseArray(array)
	fmt.Println("array before: ", array)

	// 切片
	slice := array[:]
	// 对切片倒序，其实倒的是数组本身
	reverseSlice(slice)
	fmt.Println("array after: ", array)
}

func reverseArray(array [5]int) {
	array[0], array[len(array)-1] = array[len(array)-1], array[0]
}

func reverseSlice(slice []int) {
	slice[0], slice[len(slice)-1] = slice[len(slice)-1], slice[0]
}

```

控制台输出：

```
array before:  [2 3 5 7 11]
array after:  [11 3 5 7 2]
```



参考：

https://www.runoob.com/go/go-slice.html

### [Exercise: Slices](https://golang.google.cn/tour/moretypes/18)

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
	image := make([][]uint8, dy)
	for x := range image {
		image[x] = make([]uint8, dx)
		for y := range image[x] {
			image[x][y] = uint8((x + y) / 2)
		}
	}
	return image
}

func main() {
	pic.Show(Pic)
}

```

参考：https://nylira.com/a-tour-of-go-solutions/

## 集合(map)

map 是无序的键值对集合，但和 Java 不一样的是，**其元素返回顺序不确定**。

```go
package main

import "fmt"

func main() {
	//创建key为string、value为int类型的map
	m := make(map[string]int)
	m["A"] = 90
	m["B"] = 80
	m["C"] = 70
	m["D"] = 60
	for key := range m {
		fmt.Println(key, m[key])
	}
}

```

控制台输出（**不保证一定按这个顺序**）：

```
A 90
B 80
C 70
D 60
```



查询和删除map中元素

```go
package main

import "fmt"

func main() {
	m := make(map[string]int)
	m["A"] = 90
	m["B"] = 80
	m["C"] = 70
	m["D"] = 60

	//删除元素
	delete(m, "D")

	//查询元素是否存在
	value, ok := m["D"]
	if ok {
		fmt.Println(value)
	} else {
		fmt.Println("key not exist")
	}
}
```

参考：https://www.runoob.com/go/go-map.html

## 指针

Go 有指针，但 Go 没有指针运算，即不像 C 语言有“指针地址增加”或“指针地址减少”这些操作。

当一个指针被定义后没有分配到任何变量时，它的值为 nil。nil 指针也称为空指针。

```go
package main

import "fmt"

func main() {
	var ptr *int

	fmt.Printf("ptr 的值为 : %v 或 %x\n", ptr, ptr)
	if ptr == nil {
		fmt.Printf("ptr 是空指针")
	}

}
```

以上实例输出结果为：

```
ptr 的值为 : <nil> 或 0
ptr 是空指针
```

参考：

https://www.runoob.com/go/go-pointers.html

https://golang.google.cn/tour/moretypes/1

## 结构体

Go 结构体是属性的集合。

参考：https://www.runoob.com/go/go-structures.html

### 结构体定义与创建

Go 结构体相当于 Java 中的类

```go
package main

import "fmt"

type Books struct {
	title   string
	author  string
	subject string
	sn      int
}

func main() {

	// 创建一个新的结构体
	fmt.Println(Books{"Go 语言", "www.runoob.com", "Go 语言教程", 6495407})

	// 也可以使用 key => value 格式
	fmt.Println(Books{title: "Go 语言", author: "www.runoob.com", subject: "Go 语言教程", sn: 6495407})

	// 忽略的字段为 0 或 空
	fmt.Println(Books{title: "Go 语言", author: "www.runoob.com"})
}

```

控制台输出：

```go
{Go 语言 www.runoob.com Go 语言教程 6495407}
{Go 语言 www.runoob.com Go 语言教程 6495407}
{Go 语言 www.runoob.com  0}
```

### 结构体作为函数参数

需要注意的是，结构体作为函数参数是值传递。要想引用传递，就要用结构体指针。

```go
package main

import "fmt"

type Book struct {
	title   string
	author  string
	subject string
	sn      int
}

/**
结构体作为函数参数是值传递
*/
func printBook(book Book) {
	fmt.Printf("Book title : %s\n", book.title)
	fmt.Printf("Book author : %s\n", book.author)
	fmt.Printf("Book subject : %s\n", book.subject)
	fmt.Printf("Book sn : %d\n", book.sn)
}

/**
要想引用传递，就要用结构体指针
*/
func updateBook(book *Book) {
	book.title = "Go 语言"
	book.author = "www.runoob.com"
	book.subject = "Go 语言教程"
	book.sn = 6495407
}

func main() {
	var book Book

	book.title = "Python 教程"
	book.author = "www.runoob.com"
	book.subject = "Python 语言教程"
	book.sn = 6495700
	printBook(book)

	updateBook(&book)
	printBook(book)
}
```

以上实例执行运行结果为：

```bash
Book title : Python 教程
Book author : www.runoob.com
Book subject : Python 语言教程
Book sn : 6495700
Book title : Go 语言
Book author : www.runoob.com
Book subject : Go 语言教程
Book sn : 6495407
```

### 结构体中属性的首字母大小写问题

-  首字母大写相当于 public。
-  首字母小写相当于 private。

**注意:** 这个 public 和 private 是相对于包（go 文件首行的 package 后面跟的包名）来说的。

**敲黑板，划重点**

当要将结构体对象转换为 JSON 时，对象中的属性首字母必须是大写，才能正常转换为 JSON。

示例一：

```go
type Person struct {
　　　Name string　　　　　　//Name字段首字母大写
　　　age int               //age字段首字母小写
}

func main() {
　　person:=Person{"小明",18}
　　if result,err:=json.Marshal(&person);err==nil{  //json.Marshal 将对象转换为json字符串
　　　　fmt.Println(string(result))
　　}
}
```

控制台输出：

```go
{"Name":"小明"}    //只有Name，没有age
```

示例二：

```go
type Person  struct{
   　　Name  string      //都是大写
   　　Age    int               
}
```

控制台输出：

```go
{"Name":"小明","Age":18}   //两个字段都有
```

那这样 JSON 字符串以后就只能是大写了么？ 当然不是，可以使用 tag 标记要返回的字段名。

示例三：

```go
type Person  struct{
   　　Name  string   `json:"name"`　  //标记json名字为name　　　
   　　Age    int     `json:"age"`
   　　Time int64    `json:"-"`        // 标记忽略该字段

}

func main(){
　　person:=Person{"小明",18, time.Now().Unix()}
　　if result,err:=json.Marshal(&person);err==nil{
　　　fmt.Println(string(result))
　　}
}
```

控制台输出：

```go
{"name":"小明","age":18}
```

### 结构体方法

参见上面的[函数与方法](###函数与方法)

## 接口(interface)

Go 接口是方法签名的集合（而 Go 结构体是属性的集合），它把所有的具有共性的方法定义在一起，任何其他类型只要实现了这些方法就是实现了这个接口。

Go 中的接口需要结构体来实现。

```go
package main

import (
   "fmt"
)

/**
定义接口
*/
type Phone interface {
   call()
}

/**
定义结构体
*/
type NokiaPhone struct {
}

/*
实现接口方法
*/
func (nokiaPhone NokiaPhone) call() {
   fmt.Println("I am Nokia, I can call you!")
}

type ApplePhone struct {
}

func (iPhone ApplePhone) call() {
   fmt.Println("I am iPhone, I can call you!")
}

func main() {
   var phone Phone

   phone = new(NokiaPhone)
   phone.call()

   phone = new(ApplePhone)
   phone.call()

}
```

参考：https://www.runoob.com/go/go-interfaces.html

## *错误处理

## *并发

## select 语句

> The `select` statement lets a goroutine wait on multiple communication operations.
>
> A `select` blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.

```go
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}

```

参考：https://go.dev/tour/concurrency/5

## *反射

## 刷题

### [Go支持给任意类型添加方法。这一说法是否正确？](https://www.nowcoder.com/questionTerminal/c894f2df2ef349c6bb92ea6efafbf3a3)

这个说法是错误的。只能给自定义类型(包括内置类型，但不包括指针类型)添加方法。

**内置类型**：

```
func(i int) test() {}
```

报错：cannot define new methods on non-local type int

**指针类型**： 

```
type integer *int func(i integer) test() {}
```

 报错：invalid receiver type integer (integer is a pointer type)

### [`var x = nil` 和 `var x string = nil`](https://www.nowcoder.com/profile/9951268/test/50560060/138285)

```go
var x = nil // Cannot assign nil without the explicit type
var x string = nil // cannot use nil as type string in assignment
```

总结：nil 不能赋值给没有声明类型的变量，且不能赋值给默认值不是 nil 的变量

### [cap 与 len 函数](https://www.nowcoder.com/profile/9951268/test/50560060/138308)

cap 适用于以下类型：

- array：同 len 函数，返回数组中元素的个数
- slice：返回切片的容量
- **数组指针**：返回所指数组的长度，即使所指数组是 nil
- channel：返回 buffer 容量

len 适用于以下类型：

- array：返回数组中元素的个数
- slice：返回切片中元素的个数
- **数组指针**：返回所指数组的长度，即使所指数组是 nil
- channel：返回 buffer 中元素的个数
- map：返回 map 中元素的个数
- string：字符串长度

示例：

```go
package main

import "fmt"

func main() {
	array := [5]int{1, 2, 3}
	fmt.Printf("array's len: %v, cap: %v\n", len(array), cap(array))
	slices := array[1:3]
	fmt.Printf("slice's len: %v, cap: %v\n", len(slices), cap(slices))
	var nilArray [3]int // 定义一个空数组
	p := &nilArray
	fmt.Printf("pointer to array's len: %v, cap: %v\n", len(p), cap(p))
	channel := make(chan int, 4)
	fmt.Printf("channel's len: %v, cap: %v\n", len(channel), cap(channel))
	mapping := make(map[string]string, 2)
	fmt.Printf("map's len: %v, cap() don't support map!\n", len(mapping))
	str := "string"
	fmt.Printf("string's len: %v, cap() don't support string!\n", len(str))
}
```

执行结果：

```
array's len: 5, cap: 5
slice's len: 2, cap: 4                      
pointer to array's len: 3, cap: 3           
channel's len: 0, cap: 4                    
map's len: 0, cap() don't support map!      
string's len: 6, cap() don't support string!
```

