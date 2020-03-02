---
title: go-init初始化顺序
slug: go-init-flow
date: "2019-10-20"
description: go-init顺序
categories:
- go
tags:
- go
---

go-init初始化顺序。
<!--more-->

## 概念	

如果你的项目中有一些需要初始化，而这些初始化又是通过 `init()` 来执行的，但是 `init()` 里面有一些相互依赖，你如何保证你的执行是可行的呢？

### 可能出现的问题

你在 `init()` 使用时，可能会有一些值是需要提前初始化的，否则会出现 nil 报错的问题。

### init 初始化的规则是怎样的呢？



![init顺序图](/images/init.png)

### 怎么解决呢？

可以使用 `_ "github.com/xxx/xxx"` 引入包，但是不使用来解决

## 附翻译

init相关注意如下：

- 一个包可以出线多个init函数,一个源文件也可以包含多个init函数(Multiple such functions may be defined, even within a single source file. )。
- init函数在代码中不能被显示调用、不能被引用（赋值给函数变量），否则出现编译错误（The init identifier is not declared and thus init functions cannot be referred to from anywhere in a program. ）。
- 如果当前包包含多个依赖包(import),则先初始化依赖包（If a package has imports, the imported packages are initialized before initializing the package itself. ）。
- 如果当前包有多个init函数，首先按照源文件名的字典序从前往后执行，若一个文件中出现多个init，则按照出现顺序从前往后执行（initialized by assigning initial values to all its package-level variables followed by calling all init functions in the order they appear in the source, possibly in multiple files, as presented to the compiler.）。
- 一个包被引用多次，如 A import B,C import B,A import C,B被引用多次，但B包只会初始化一次（ If multiple packages import a package, the imported package will be initialized only once. ）。
- 引入包，不可出现死循坏。即A import B,B import A,这种情况编译失败（ The importing of packages, by construction, guarantees that there can be no cyclic initialization dependencies. ）。
- 包级别的变量初始化、init函数的执行，这两个操作会在goruntine 自己调用，且顺序调用，一次一个包（Package initialization—variable initialization and the invocation of init functions—happens in a single goroutine, sequentially, one package at a time.）。
- 一个init函数里可能会启动其它goroutine,即在初始化的同时启动新的goroutine。然而，初始化依旧是顺序的，即只有上一个init执行完毕，下一个才会开始（ An init function may launch other goroutines, which can run concurrently with the initialization code. However, initialization always sequences the init functions: it will not invoke the next one until the previous one has returned.）。



总结： 在一个go文件中， 初始化顺序规则： (1)引入的包 (2) 当前包中的变量常量 (3) 当前包的init （4）main函数

注意： 

0. 当前go源文件中， 每一个被Import的包， 按其在源文件中出现顺序初始化。

1. 如果当前包有多个init在不同的源文件中， 则按源文件名以字典序从小到大排序，小的先被执行到， 同一包且同一源文件中的init,则按其出现在文件中的先后顺序依次初始化； 当前包的package level变量常量也遵循这个规则； 其实准确来说，应是按提交给编译器的源文件名顺序为准，只是在提交编译器之前， go命令行工具对源文件名按字典序排序了。

2. init只可以由go runtine自已调用， 我们在代码中不可以显示调用，也不可以被引用，如赋给a function variable。

3. 包A 引入包B , 包B又引入包C, 则包的初始化顺序为： C -> B -> A

4. 引入包，必须避免死循环，如 A 引 B , B引C, C引A.

5. 一个包被其它多个包引入，如A -> B ->C 和 H -> I -> C , C被其它包引了2次， 但是注意包C只被初始化一次。

6. 另一个大原则， 被依赖的总是先被初始化，当然呀。

7. main包总是被最后一个初始化，这很好理解，因为它总是依赖别的包。

## 参考文献
- [init顺序](https://maiyang.me/post/2019-01-03-init-order-in-golang/)
