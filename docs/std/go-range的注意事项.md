---
title: "go-range的注意事项"
slug: "go-rang"
date: "2018-11-23 01:00:24"
categories:
    - go
tags:
    - go
---

Go中的`range`关键字使用起来非常的方便，它允许你遍历某个`slice`或者`map`，并通过两个参数(**index**和**value**)，分别获取到slice或者map中某个元素所在的index以及其值。 比如像这样的用法：
```go
for index, value := range mySlice {
    fmt.Println("index: " + index)
    fmt.Println("value: " + value)
}
```

---

上面的例子足够清晰的描述了range的用法，实际上在使用range关键字的时候，还有一些需要特别注意的地方，有一些新手很容易入的**坑**。为了说明这些**坑**，我们可以从下面这个稍复杂的例子说起：

```go
type Foo struct {
    bar string
}

func main() {
    list := []Foo{
        {"A"},
        {"B"},
        {"C"},
    }

    list2 := make([]*Foo, len(list))
    for i, value := range list {
        list2[i] = &value
    }

    fmt.Println(list[0], list[1], list[2])
    fmt.Println(list2[0], list2[1], list2[2])
}
```

在这个例子中，我们干了下面的一些事情：

- 定义了一个叫做Foo的结构，里面有一个叫bar的field。随后，我们创建了一个基于Foo结构体的slice，名字叫list
- 我们还创建了一个基于Foo结构体指针类型的slice，叫做list2
- 在一个for循环中，我们试图遍历list中的每一个元素，获取其指针地址，并赋值到list2中index与之对应的位置。
- 最后，分别输出list与list2中的每个元素

从代码来看，理所当然，我们期望得到的结果应该是这样：
```go
{A} {B} {C}
&{A} &{B} &{C}
```
但是结果却出乎意料，程序的输出是这样的：
```go
{A} {B} {C}
&{C} &{C} &{C}
```

从结果来看，仿佛list2中的三个元素，都指向了list中的最后一个元素。这是为什么呢？问题就出在上面那一段`for...range`循环中。


在Go的`for...range`循环中，Go始终使用值拷贝的方式代替被遍历的元素本身，简单来说，就是`for...range`中那个value，是一个值拷贝，而不是元素本身。这样一来，当我们期望用`&`获取元素的地址时，实际上只是取到了value这个**临时变量的地址**，而非list中真正被遍历到的某个元素的地址。而在整个`for...range`循环中，value这个临时变量会被重复使用，所以，在上面的例子中，list2被填充了三个相同的地址，其实都是value的地址。而在最后一次循环中，value被赋值为{c}。因此，list2输出的时候显示出了三个&{c}。

同样的，下面的写法，跟`for...range`的例子如出一辙：
```go
var value Foo
for var i := 0; i < len(list); i++ {
    value = list[i]
    list2[i] = &value
}
```

如果我们输出list2的三个元素，结果同样是: `&{C} &{C} &{C}`

那么，怎样才是正确的写法呢？我们应该用**index**来访问`for...range`中真实的元素，并获取其指针地址：
```go
for i, _ := range list {
    list2[i] = &list[i]
}
```

这样，输出list2中的元素，就能得到我们想要的结果(&{A} &{B} &{C})了。

了解了range的正确使用姿势，那么我们下面这个例子也能迎刃而解了：

```go
package main

import "fmt"

type MyType struct {
    field string
}

func main() {
    var array [10]MyType

    for _, e := range array {
        e.field = "foo"
    }

    for _, e := range array {
        fmt.Println(e.field)
        fmt.Println("--")
    }
}
```

平常写代码最常见的场景，就是我们需要在一个循环中修改被遍历元素的值。比如上面这个例子，我们希望能使用`for...range`循环，一次性将array中每个元素的field设置为`foo`。同样，因为range值拷贝的缘故，上面的程序什么都不会输出...

而正确的做法是：

```go
for i, _ := range array {
	array[i].field = "foo"
}
```

通过index访问每个元素，并修改其field，这样，就能输出一堆”foo”了……


## 参考文献

- [聊聊Go中的Range关键字](https://xiaozhou.net/something-about-range-of-go-2016-04-10.html)
