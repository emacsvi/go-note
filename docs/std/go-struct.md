---
title: "go-struct"
date: "2018-11-23 01:00:17"
categories:
    - go
tags:
    - go
---

## struct
Go语言中，也和C或者其他语言一样，我们可以声明新的类型，作为其它类型的属性或字段的容器。例如，我们可以创建一个自定义类型`person`代表一个人的实体。这个实体拥有属性：姓名和年龄。这样的类型我们称之`struct`。如下代码所示:
```go
type person struct {
	name string
	age int
}
```	
看到了吗？声明一个struct如此简单，上面的类型包含有两个字段
- 一个string类型的字段name，用来保存用户名称这个属性
- 一个int类型的字段age,用来保存用户年龄这个属性

如何使用struct呢？请看下面的代码
```go
type person struct {
	name string
	age int
}
var P person  // P现在就是person类型的变量了
P.name = "Astaxie"  // 赋值"Astaxie"给P的name属性.
P.age = 25  // 赋值"25"给变量P的age属性
fmt.Printf("The person's name is %s", P.name)  // 访问P的name属性.
```	
除了上面这种P的声明使用之外，还有另外几种声明使用方式：

- 1.按照顺序提供初始化值
```go
	P := person{"Tom", 25}
```
- 2.通过`field:value`的方式初始化，这样可以任意顺序
```go
	P := person{age:24, name:"Tom"}
```
- 3.当然也可以通过`new`函数分配一个指针，此处P的类型为*person
```go	
	P := new(person)
```

下面我们看一个完整的使用struct的例子
```go
package main
import "fmt"
// 声明一个新的类型
type person struct {
	name string
	age int
}

// 比较两个人的年龄，返回年龄大的那个人，并且返回年龄差
// struct也是传值的
func Older(p1, p2 person) (person, int) {
	if p1.age>p2.age {  // 比较p1和p2这两个人的年龄
		return p1, p1.age-p2.age
	}
	return p2, p2.age-p1.age
}

func main() {
	var tom person
	// 赋值初始化
	tom.name, tom.age = "Tom", 18
	// 两个字段都写清楚的初始化
	bob := person{age:25, name:"Bob"}
	// 按照struct定义顺序初始化值
	paul := person{"Paul", 43}

	tb_Older, tb_diff := Older(tom, bob)
	tp_Older, tp_diff := Older(tom, paul)
	bp_Older, bp_diff := Older(bob, paul)

	fmt.Printf("Of %s and %s, %s is older by %d years\n",
		tom.name, bob.name, tb_Older.name, tb_diff)

	fmt.Printf("Of %s and %s, %s is older by %d years\n",
		tom.name, paul.name, tp_Older.name, tp_diff)

	fmt.Printf("Of %s and %s, %s is older by %d years\n",
		bob.name, paul.name, bp_Older.name, bp_diff)
}
```
缩写定义一个结构体：
```go
type Employee struct {  
    firstName string
    lastName  string
    age       int
}
```
相同字段可以进行缩写：
```go
type Employee struct {  
    firstName, lastName string
    age, salary         int
}
```

## anymouse struct
It is possible to declare structures without declaring a new type and these type of structures are called **anonymous structures**.
```go
var employee struct {  
        firstName, lastName string
        age int
}
```
The above snippet creates a anonymous structure **employee**.

**Creating anonymous structures**
```go
func main() {  
    emp3 := struct {
        firstName, lastName string
        age, salary         int
    }{
        firstName: "Andreah",
        lastName:  "Nikola",
        age:       31,
        salary:    5000,
    }

    fmt.Println("Employee 3", emp3)
}
```

## struct的匿名字段
我们上面介绍了如何定义一个struct，定义的时候是字段名与其类型一一对应，实际上Go支持只提供类型，而不写字段名的方式，也就是匿名字段，也称为嵌入字段。

当匿名字段是一个struct的时候，那么这个struct所拥有的全部字段都被隐式地引入了当前定义的这个struct。

让我们来看一个例子，让上面说的这些更具体化
```go
package main

import "fmt"

type Human struct {
	name string
	age int
	weight int
}

type Student struct {
	Human  // 匿名字段，那么默认Student就包含了Human的所有字段
	speciality string
}

func main() {
	// 我们初始化一个学生
	mark := Student{Human{"Mark", 25, 120}, "Computer Science"}

	// 我们访问相应的字段
	fmt.Println("His name is ", mark.name)
	fmt.Println("His age is ", mark.age)
	fmt.Println("His weight is ", mark.weight)
	fmt.Println("His speciality is ", mark.speciality)
	// 修改对应的备注信息
	mark.speciality = "AI"
	fmt.Println("Mark changed his speciality")
	fmt.Println("His speciality is ", mark.speciality)
	// 修改他的年龄信息
	fmt.Println("Mark become old")
	mark.age = 46
	fmt.Println("His age is", mark.age)
	// 修改他的体重信息
	fmt.Println("Mark is not an athlet anymore")
	mark.weight += 60
	fmt.Println("His weight is", mark.weight)
}
```
struct组合，Student组合了Human struct和string基本类型

我们看到Student访问属性age和name的时候，就像访问自己所有用的字段一样，对，匿名字段就是这样，能够实现字段的继承。是不是很酷啊？还有比这个更酷的呢，那就是student还能访问Human这个字段作为字段名。请看下面的代码，是不是更酷了。
```go
mark.Human = Human{"Marcus", 55, 220}
mark.Human.age -= 1
```
通过匿名访问和修改字段相当的有用，但是不仅仅是struct字段哦，所有的内置类型和自定义类型都是可以作为匿名字段的。请看下面的例子
```go
package main
import "fmt"
type Skills []string

type Human struct {
	name string
	age int
	weight int
}

type Student struct {
	Human  // 匿名字段，struct
	Skills // 匿名字段，自定义的类型string slice
	int    // 内置类型作为匿名字段
	speciality string
}

func main() {
	// 初始化学生Jane
	jane := Student{Human:Human{"Jane", 35, 100}, speciality:"Biology"}
	// 现在我们来访问相应的字段
	fmt.Println("Her name is ", jane.name)
	fmt.Println("Her age is ", jane.age)
	fmt.Println("Her weight is ", jane.weight)
	fmt.Println("Her speciality is ", jane.speciality)
	// 我们来修改他的skill技能字段
	jane.Skills = []string{"anatomy"}
	fmt.Println("Her skills are ", jane.Skills)
	fmt.Println("She acquired two new ones ")
	jane.Skills = append(jane.Skills, "physics", "golang")
	fmt.Println("Her skills now are ", jane.Skills)
	// 修改匿名内置类型字段
	jane.int = 3
	fmt.Println("Her preferred number is", jane.int)
}
```
从上面例子我们看出来struct不仅仅能够将struct作为匿名字段，自定义类型、内置类型都可以作为匿名字段，而且可以在相应的字段上面进行函数操作（如例子中的append）。

这里有一个问题：如果human里面有一个字段叫做phone，而student也有一个字段叫做phone，那么该怎么办呢？

Go里面很简单的解决了这个问题，最外层的优先访问，也就是当你通过`student.phone`访问的时候，是访问student里面的字段，而不是human里面的字段。

这样就允许我们去重载通过匿名字段继承的一些字段，当然如果我们想访问重载后对应匿名类型里面的字段，可以通过匿名字段名来访问。请看下面的例子
```go
package main
import "fmt"

type Human struct {
	name string
	age int
	phone string  // Human类型拥有的字段
}
type Employee struct {
	Human  // 匿名字段Human
	speciality string
	phone string  // 雇员的phone字段
}

func main() {
	Bob := Employee{Human{"Bob", 34, "777-444-XXXX"}, "Designer", "333-222"}
	fmt.Println("Bob's work phone is:", Bob.phone)
	// 如果我们要访问Human的phone字段
	fmt.Println("Bob's personal phone is:", Bob.Human.phone)
}
```

## struct继承测试
```go
package main
import (
	"fmt"
)

func main() {
	testDerive()
}

type A struct {
	aa int
	BB string
}

type B struct {
	A
	aa int
	CC string
}

func (a *A) aFunc() {
	fmt.Println("A.aFunc")
}

func (a *A) BFunc() {
	fmt.Println("A.BFunc")
}

func (b *B) aFunc() {
	fmt.Println("B.aFunc")
}

func (b *B) CFunc() {
	fmt.Println("B.CFunc")
}

func testDerive() {
	var tb B
	//变量测试
	fmt.Println(tb.aa)
	fmt.Println(tb.A.aa)
	fmt.Println(tb.BB)
	fmt.Println(tb.A.BB)
	fmt.Println(tb.CC)
	tb.aa = 9
	tb.A.aa = 8
	tb.BB = "A.BB"
	tb.CC = "B.CC"
	fmt.Println(tb.aa)
	fmt.Println(tb.A.aa)
	fmt.Println(tb.BB)
	fmt.Println(tb.A.BB)
	fmt.Println(tb.CC)

	//函数测试
	tb.aFunc()
	tb.A.aFunc()
	tb.BFunc()
	tb.A.BFunc()
	tb.CFunc()
}
```
结果：
```shell
0
0



9
8
A.BB
A.BB
B.CC
B.aFunc
A.aFunc
A.BFunc
A.BFunc
B.CFunc
```
由以上测试可以看出来：
1. golang的继承是继承父结构体的所有属性和方法，包括大小写开头的变量和函数。
2. 如果子结构体和父结构体有同名的变量或者函数，并不会产生覆盖，可以通过“父结构名字.变量或函数名”的方式调用父结构体的同名变量或函数

## Nested Struct Initialize
内嵌结构体的初始化方式, 先看下面这个会报错的例子：`missing type in composite literal`
```go
package main

type Configuration struct {
    Val   string
    Proxy struct {
        Address string
        Port    string
    }
}

func main() {
    c := &Configuration{
        Val: "test",
        Proxy: {
            Address: "addr",
            Port:    "80",
        },
    }
}
```

有**三种方式**定义嵌套结构体并初始化它

- 1.第一种最直接的方式，需要提供每一个字段显示定义: 

```go
type Configuration struct {
    Val string
    Proxy
}

type Proxy struct {
    Address string
    Port    string
}

func main() {
    c := &Configuration{
        Val: "test",
        Proxy: Proxy{
            Address: "addr",
            Port:    "port",
        },
    }
    fmt.Println(c)
}
```
- 2.第二种很**ugly**的方式, 利用**anymouse struct**特征：

```go
c := &Configuration{
    Val: "test",
    Proxy: struct {
        Address string
        Port    string
    }{
        Address: "addr",
        Port:    "80",
    },
}
```
- 3.分开对其赋值，最方便的是第三种方式：

```go
package main
import "fmt"
type Configuration struct {
    Val   string
    Proxy struct {
        Address string
        Port    string
    }
}

func main() {
    c := &Configuration{
        Val: "test",
    }

    c.Proxy.Address = `127.0.0.1`
    c.Proxy.Port = `8080`
}
```

网上的例子：
> 问题:

golang 嵌套struct如何直接初始化？
如下
```go
 type Account struct {
    Id uint32
    Name string
     Nested struct {
          Age uint8
     }
}
```
如何直接初始化(非 对象.属性 = 初始化 形式)？ 形如：
```go
 account := &Account{ Id : 10, Name : "jim", Nested : ??????? }
 ```
问号部分如何初始化？

> 回答：
 
如果你一定要这么干……
```go
account := Account{
  Id:10, Name:"jim",
  Nesed: struct{Age unit8}{Age: 20},
}
```
**没错，匿名struct直接初始化的时候是需要给出它的结构的。**

不过不建议使用上面坑爹的方式，这样分开写不是很清楚么：
```go
acc := Account{Id:10, Name:"jim"}
acc.Nested.Age = 20
```




## 参考文献

- [struct](https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/02.4.md)
- [struct 继承](https://blog.csdn.net/nature19862001/article/details/72858243)
- [go struct very detail contain nested struct and anymouse structure](https://golangbot.com/structs/)
- [nested初始化](https://www.zhihu.com/question/22746100)
