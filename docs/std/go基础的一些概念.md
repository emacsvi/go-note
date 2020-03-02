---
title: go基础的一些概念
slug: heling-go-36-notes
date: "2017-04-02"
description: go基础的一些概念
categories:
- go
tags:
- go
---

**赫林**老师有关**go语言**的**36讲**，看了一些很基础的go知识发现自己居然不知道。这太说不过去了。于是记录一下。
<!--more-->

### go工作区
我们需要配置**3**个环境变量，也就是**GOROOT**、**GOPATH** 和 **GOBIN**。这里我可以简单介绍一下。

- GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。
- GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。
- GOBIN：GO 程序生成的可执行文件（executable file）的路径。

![gopath](images/gopath.png)

### 如果gopath之中设置了多个工作区，那么查找依赖包是以怎样的顺序进行的？
来自[gopath工作区解答](https://cloud.tencent.com/developer/article/1339789)

例如 a 依赖 b ，b依赖c 

那么 会先查找c包，那在工作区是如何查找这个依赖包c的呢？

首先在查找依赖包的时候，总是会先查找 GOROOT目录，也就是go语言的安装目录，如果没有找到依赖的包，才到工作区去找相应的包。

在工作区中是按照设置的先后顺序来查找的，也就是会从第一个开始，依次查找，如果找到就不再继续查找，如果没有找到，就报错了。

go get 会下载代码包到src目录，但是只会下载到第一个工作区目录。

在Go语言程序中，每个包都有一个全局唯一的导入路径。导入语句中类似"github.com/xxxx/tem"的字符串对应包的导入路径。

Go语言的规范并没有定义这些字符串的具体含义或包来自哪里，它们是由构建工具来解释的。

一个导入路径代表一个目录中的一个或多个Go源文件。

除了包的导入路径，每个包还有一个包名，包名一般是短小的名字（并不要求包名是唯一的），包名在包的声明处指定。

1.总执行顺序的角度

引入的包 -> 当前包的变量常量 -> **init()**[多个同一包则按照顺序执行] -> **main**函数

2.依赖包执行顺序

被依赖的总是优先执行初始化，一个包只会被初始化一次。
a引入b，b引入c，则执行顺序**c -> b -> a**

3.单个包执行顺序的角度

总的前提:按照包中源文件名的字典顺序来排序执行。
当前包排序后的变量常量 -> 排序后的**init()**

### 如果多个工作区中存在导入路径相同的代码包会产生冲突吗？
不冲突，因为按顺序找到所需要的包就不往后找了。

更多关于包的请看这里[包管理](https://github.com/hyper0x/go_command_tutorial/blob/master/0.3.md)

### 自定义代码包远程导入路径
请看[包管理](https://github.com/hyper0x/go_command_tutorial/blob/master/0.3.md)里面的详细介绍。

### interface{}(container).([]string)
```go
var container = []string{"go", "java", "python"}
value, ok := interface{}(container).([]string)
```
<font color=red>请记住，一对不包裹任何东西的花括号(interface{})，除了可以代表空的代码块之外，还可以用于表示不包含任何内容的数据结构（或者说数据类型）。</font>

比如你今后肯定会遇到的struct{}，它就代表了不包含任何字段和方法的、空的结构体类型。

而空接口interface{}则代表了不包含任何方法定义的、空的接口类型。

当然了，对于一些集合类的数据类型来说，{}还可以用来表示其值不包含任何元素，比如空的切片值[]string{}，以及空的字典值map[int]string{}。

### 问题：切片的底层数组什么时候会被替换？

确切地说，一个切片的底层数组永远不会被替换。为什么？虽然在扩容的时候 Go 语言一定会生成新的底层数组，但是它也同时生成了新的切片。它是把新的切片作为了新底层数组的窗口，而没有对原切片及其底层数组做任何改动。

请记住，在无需扩容时，append函数返回的是指向原底层数组的新切片，而在需要扩容时，append函数返回的是指向新底层数组的新切片。所以，严格来讲，“扩容”这个词用在这里虽然形象但并不合适。不过鉴于这种称呼已经用得很广泛了，我们也没必要另找新词了。

顺便说一下，只要新长度不会超过切片的原容量，那么使用append函数对其追加元素的时候就不会引起扩容。这只会使紧邻切片窗口右边的（底层数组中的）元素被新的元素替换掉。

切片缩容之后还是会引用底层的原数组，这有时候会造成大量缩容之后的多余内容没有被垃圾回收。可以使用新建一个数组然后copy的方式。

## 概念

## 参考文献
- [切片的本质](https://blog.go-zh.org/go-slices-usage-and-internals)
- [数组和切片](http://www.cnblogs.com/hustcat/p/4002707.html)
- [slice](http://blog.wuxu92.com/array-and-slice-in-golang/)
- [go array](https://golang.org/doc/effective_go.html#arrays)
- [slices](https://gobyexample.com/slices)
- [array and slice](https://golang.org/doc/effective_go.html#arrays)

