---
title: "go-channel详解"
slug: "go-channel"
date: "2018-11-23 01:00:04"
categories:
    - go
tags:
    - go
---

## channel
goroutine允许我们并行的运行一些代码。但是要想让这些代码对我们来说更有意义，我们会有一些额外的需求--我们应该能够传递数据到正在运行的进程中；当并行的进程成功产生数据时，我们应该能从该进程中获取到数据。`channel`配合`goroutine`提供了实现这些需求的途径

`channel`可以想象为一个指定了大小和容量的管道或传送带。我们可以在其一边放置内容，然后在另一边获取内容.

我们采用一个蛋糕制作装箱工厂的例子来进行下面的说明。我们有一台用于制作蛋糕的机器，还有一台用于装箱的机器。她们通过一条传送带互相连接--蛋糕机将蛋糕放上传送带，装箱机在发现传送带上么有蛋糕时将其取走并装入箱子

在go中，`chan`关键字用于定义一个channel。`make`关键字用于创建cahnnel，创建时指定channel传递的`数据类型`

**示例代码1**
```go
ic := make(chan int) //a channel that can send and receive an int
sc := make(chan string) //a channel hat can send and receive a string
myc := make (chan my_type) //a channel for a custom defined struct type
```

在channel的变量名**前面**或**后面**，你可以使用`<-`操作符来指示channel用于发送还是接收数据(注意对应关系).假设，my_channel是一个接收int类型数据的channel，你可以像`my_channel <- 5`这样向其发送数据，并且你可以像`my_recvd_value <- my_channel`这样来从中接收收据

想象channel是一个有方向的传送带：
- 从外部指向`channel`的箭头用于向channel放置数据
- 从`channel`指向外部的箭头用于从channel获取数据

**示例代码2**
```go
my_channel := make(chan int)

//within some goroutine - to put a value on the channel
my_channel <- 5 

//within some other goroutine - to take a value off the channel
var my_recvd_value int
my_recvd_value = <- my_channel
```

当然，你也可以指定channel中的数据移动方向，只需要在创建channel时**在chan关键字旁使用`<-`指明方向**

**示例代码3**
```go
//a channel that can only send data - arrow going out is sending
ic_send_only := make (<-chan int) 

//a channel that can only receive a data - arrow going in is receiving
ic_recv_only := make (chan<- int) 
```

channel能够保有的数据个数很重要。她能指示具体有多少条数据可以同时工作。即使发送者有能力产生很多条目，如果接受者没有能力接收她们，那么她们就不能工作。这将会有很多蛋糕从传送带上掉落并浪费掉ORZ。在并行计算中，这叫做生产者-消费者同步问题(producer-consumer synchronization problem)

如果channel的容量(capacity)是1——也就是说，一旦有数据被放入channel，那么该数据必须被取走才能让另一条数据放入，这就是同步channel(synchronous channel)。channel的每一边——发送者和接受者——在同一时间只交流一条数据，然后必须等待，直到另一边完成了相应的发送或接收动作

目前为止，我们定义的所有的channel默认都是同步channel，也就是说，一条数据被放入channel后必须被取走才能再放置另一条数据。现在，我们完成上面提到的蛋糕制作装箱工厂。由于channel在不同的goroutine之间交流数据，我们有两个名为makeCakeAndSend和receiveCakeAndPack的函数。每个函数都接收一个channel的引用作为参数，这样它们可以通过该channel进行交流

```go
示例代码4
package main

import (
    "fmt"
    "time"
    "strconv"
)

var i int

func makeCakeAndSend(cs chan string) {
    i = i + 1
    cakeName := "Strawberry Cake " + strconv.Itoa(i)
    fmt.Println("Making a cake and sending ...", cakeName)
    cs <- cakeName //send a strawberry cake
}

func receiveCakeAndPack(cs chan string) {
    s := <-cs //get whatever cake is on the channel
    fmt.Println("Packing received cake: ", s)
}

func main() {
    cs := make(chan string)
    for i := 0; i<3; i++ {
        go makeCakeAndSend(cs)
        go receiveCakeAndPack(cs)

        //sleep for a while so that the program doesn’t exit immediately and output is clear for illustration
        time.Sleep(1 * 1e9)
    }
}
```
输出结果
```bash
Making a cake and sending ... Strawberry Cake 1 
Packing received cake: Strawberry Cake 1 
Making a cake and sending ... Strawberry Cake 2 
Packing received cake: Strawberry Cake 2 
Making a cake and sending ... Strawberry Cake 3 
Packing received cake: Strawberry Cake 3
```

在上述代码中，我们创建了三个制作蛋糕的函数调用，并在其之后立刻创建了三个装箱蛋糕的函数调用。我们知道，每当一个蛋糕被装箱，就会有另一个蛋糕同时被制作并准备好被装箱。当然如果你吹毛求疵，代码中确实有一个很轻微的含混之处——在打印`Making a cake and sending …`和实际发送蛋糕到channel之间有延时。代码中我们在每个循环中调用了`time.Sleep()`，用于让制作和装箱动作一个接一个的发生，这样做是正确的。由于我们的channel是同步的，而且同一时间仅支持一条数据，一个从channel中移除蛋糕的动作也就是一个装箱动作，必须在制作新蛋糕并将其放入channel之前发生

现在我们改动下上面的内容让其更像我们正常使用的代码。典型的goroutine一般是一个包含了不断循环的内容的代码块，其内部完成一些操作并且与其他的goroutine通过channel交换数据。在下面的例子中，我们将循环移至goroutine函数内部，然后我们仅调用该goroutine一次


示例代码5
```go
package main                                                                                                                                                           

import (
    "fmt"
    "time"
    "strconv"
)

func makeCakeAndSend(cs chan string) {
    for i := 1; i<=3; i++ {
        cakeName := "Strawberry Cake " + strconv.Itoa(i)
        fmt.Println("Making a cake and sending ...", cakeName)
        cs <- cakeName //send a strawberry cake
    }   
}

func receiveCakeAndPack(cs chan string) {
    for i := 1; i<=3; i++ {
        s := <-cs //get whatever cake is on the channel
        fmt.Println("Packing received cake: ", s)
    }   
}

func main() {
    cs := make(chan string)
    go makeCakeAndSend(cs)
    go receiveCakeAndPack(cs)

    //sleep for a while so that the program doesn’t exit immediately
    time.Sleep(4 * 1e9)
}
```
输出结果
```bash
Making a cake and sending ... Strawberry Cake 1 
Making a cake and sending ... Strawberry Cake 2 
Packing received cake: Strawberry Cake 1 
Packing received cake: Strawberry Cake 2 
Making a cake and sending ... Strawberry Cake 3 
Packing received cake: Strawberry Cake 3
```

输出结果在我电脑上是这样的，在你电脑上可能会不同，输出结果依赖于你机器上goroutine的执行顺序。如前所述，我们仅调用了每个goroutine一次，并且传递了一个公有的channel给她们。在每个goroutine内部有三个循环，`makeCakeAndSend`将蛋糕放入channel，`receiveCakeAndPack`将蛋糕从channel中取出。由于程序会在我们创建了两个goroutine后立即结束，因此我们必须手动增加一个时间暂停操作来让三个蛋糕都被制作和装箱好

极其重要的一点是，我们必须理解，上面的输出并没有正确的反应channel中实际的发送和接收操作。发送和接收在这里是同步的——同一时间仅有一个蛋糕。然而**由于在打印语句和实际发送与接收间的延时**，输出看起来在顺序上是错误的。而实际上发生的是：

```bash
Making a cake and sending ... Strawberry Cake 1 
Packing received cake: Strawberry Cake 1 
Making a cake and sending ... Strawberry Cake 2 
Packing received cake: Strawberry Cake 2 
Making a cake and sending ... Strawberry Cake 3 Packing received cake: Strawberry Cake 3
```

因此，一定要记住，在处理goroutine和channel时，通过打印日志分析执行顺序一定要万分小心

## range和select
> 数据接受者总是面临这样的问题：何时停止等待数据？还会有更多的数据么，还是所有内容都完成了？我应该继续等待还是该做别的了？

对于该问题，一个可选的方式是，持续的访问数据源并检查channel是否已经关闭，但是这并不是高效的解决方式。Go提供了`range`关键字，将其使用在channel上时，会自动等待channel的动作一直到channel被**关闭**

**示例代码1**
```go
package main                                                                                             
import (
    "fmt"
    "time"
    "strconv"
)

func makeCakeAndSend(cs chan string, count int) {
    for i := 1; i <= count; i++ {
        cakeName := "Strawberry Cake " + strconv.Itoa(i)
        cs <- cakeName //send a strawberry cake
    }   
}

func receiveCakeAndPack(cs chan string) {
    for s := range cs {
        fmt.Println("Packing received cake: ", s)
    }
}

func main() {
    cs := make(chan string)
    go makeCakeAndSend(cs, 5)
    go receiveCakeAndPack(cs)

    //sleep for a while so that the program doesn’t exit immediately
    time.Sleep(3 * 1e9)
}
```
我们告诉了蛋糕制作器我们需要5个蛋糕，但是蛋糕装箱器并不知道数目，而在之前版本的代码中，我们写死了具体的接收数目。上面的代码中，通过对channel使用range关键字，我们避免了给接收者写明要接收的数据个数这种不合理的需求——当channel被关闭时，接收者的for循环也被自动停止了

**Channel and select**

`select`关键字用于多个`channel`的结合，这些channel会通过类似于**are-you-ready polling**的机制来工作。select中会有`case代码块`，用于发送或接收数据——不论通过`<-`操作符指定的发送还是接收操作准备好时，channel也就准备好了。在select中也可以有一个`default代码块`，其一直是准备好的。那么，在select中，哪一个代码块被执行的算法大致如下：
- 检查每个case代码块
- 如果任意一个case代码块准备好发送或接收，执行对应内容
- 如果多余一个case代码块准备好发送或接收，随机选取一个并执行对应内容
- 如果任何一个case代码块都没有准备好，等待
- 如果有default代码块，并且没有任何case代码块准备好，执行default代码块对应内容

在下面的程序中，我们扩展蛋糕制作工厂来模拟多于一种口味的蛋糕生产的情况——现在有草莓和巧克力两种口味！但是装箱机制还是同以前一样的。由于蛋糕来自不同的channel，而装箱器不知道确切的何时会有何种蛋糕放置到某个或多个channel上，这就可以用select语句来处理所有这些情况——一旦某一个channel准备好接收蛋糕/数据，select就会完成该对应的代码块内容

注意，我们这里使用的多个返回值`case cakeName, strbry_ok := <-strbry_cs`，第二个返回值是一个`bool类型`，当其为**false**时说明channel**被关闭**了。如果是true，说明有一个值被成功传递了。我们使用这个值来**判断是否应该停止等待**

**示例代码2**
```go
package main

import (
    "fmt"
    "time"
    "strconv"
)

func makeCakeAndSend(cs chan string, flavor string, count int) {
    for i := 1; i <= count; i++ {
        cakeName := flavor + " Cake " + strconv.Itoa(i)
        cs <- cakeName //send a strawberry cake
    }   
    close(cs)
}

func receiveCakeAndPack(strbry_cs chan string, choco_cs chan string) {
    strbry_closed, choco_closed := false, false

    for {
        //if both channels are closed then we can stop
        if (strbry_closed && choco_closed) { return }
        fmt.Println("Waiting for a new cake ...")
        select {
        case cakeName, strbry_ok := <-strbry_cs:
            if (!strbry_ok) {
                strbry_closed = true
                fmt.Println(" ... Strawberry channel closed!")
            } else {
                fmt.Println("Received from Strawberry channel.  Now packing", cakeName)
            }   
        case cakeName, choco_ok := <-choco_cs:
            if (!choco_ok) {
                choco_closed = true
                fmt.Println(" ... Chocolate channel closed!")
            } else {
                fmt.Println("Received from Chocolate channel.  Now packing", cakeName)
            }   
        }   
    }   
}

func main() {
    strbry_cs := make(chan string)
    choco_cs := make(chan string)

    //two cake makers
    go makeCakeAndSend(choco_cs, "Chocolate", 3)  //make 3 chocolate cakes and send
    go makeCakeAndSend(strbry_cs, "Strawberry", 3)  //make 3 strawberry cakes and send

    //one cake receiver and packer
    go receiveCakeAndPack(strbry_cs, choco_cs)  //pack all cakes received on these cake channels

    //sleep for a while so that the program doesn’t exit immediately
    time.Sleep(2 * 1e9)
}
```
输出结果
```bash
Waiting for a new cake ... 
Received from Strawberry channel. Now packing Strawberry Cake 1 
Waiting for a new cake ... 
Received from Chocolate channel. Now packing Chocolate Cake 1 
Waiting for a new cake ... 
Received from Chocolate channel. Now packing Chocolate Cake 2 
Waiting for a new cake ... 
Received from Strawberry channel. Now packing Strawberry Cake 2 
Waiting for a new cake ... 
Received from Strawberry channel. Now packing Strawberry Cake 3 
Waiting for a new cake ... 
Received from Chocolate channel. Now packing Chocolate Cake 3 
Waiting for a new cake ... 
... Strawberry channel closed! 
Waiting for a new cake ... 
... Chocolate channel closed!
```

## 写在最后
实际上，有经验的**Gopher**一眼就能发现，示例代码1中的channel是**没有正确关闭**的，在`for range`语句的执行一直没有停止因为channel一直存在而没有被关闭，只不过随着`time.Sleep()`结束，`main`函数退出，所有的goroutine被关闭，该语句也被结束了而已

正确的解决步骤：
- a)发送器一旦停止发送数据后立即关闭channel
- b)接收器一旦停止接收内容，终止程序
- c)移除`time.Sleep`语句

修改后代码：

```go
package main

import (
    "fmt"
    "strconv"
)

func makeCakeAndSend(cs chan string, count int) {
    for i := 1; i <= count; i++ {
        cakeName := "Strawberry Cake " + strconv.Itoa(i)
        cs <- cakeName //send a strawberry cake
    }
    close(cs)
}

func receiveCakeAndPack(cs chan string) {
    for s := range cs {
        fmt.Println("Packing received cake: ", s)
    }
}

func main() {
    cs := make(chan string)
    go makeCakeAndSend(cs, 5)
    receiveCakeAndPack(cs)
}
```
这样才是对channel使用range进行处理的优雅方法

同样的，第二个例子中，time.Sleep()语句可以去除掉，我们只需要让receiveCakeAndPack函数执行完毕后退出程序即可

**修改后代码：**
```go
package main

import (
    "fmt"
    "strconv"
)

func makeCakeAndSend(cs chan string, flavor string, count int) {
    for i := 1; i <= count; i++ {
        cakeName := flavor + " Cake " + strconv.Itoa(i)
        cs <- cakeName //send a strawberry cake
    }
    close(cs)
}

func receiveCakeAndPack(strbry_cs chan string, choco_cs chan string) {
    strbry_closed, choco_closed := false, false

    for {
        //if both channels are closed then we can stop
        if strbry_closed && choco_closed {
            return
        }
        fmt.Println("Waiting for a new cake ...")
        select {
        case cakeName, strbry_ok := <-strbry_cs:
            if !strbry_ok {
                strbry_closed = true
                fmt.Println(" ... Strawberry channel closed!")
            } else {
                fmt.Println("Received from Strawberry channel.  Now packing", cakeName)
            }
        case cakeName, choco_ok := <-choco_cs:
            if !choco_ok {
                choco_closed = true
                fmt.Println(" ... Chocolate channel closed!")
            } else {
                fmt.Println("Received from Chocolate channel.  Now packing", cakeName)
            }
        }
    }
}

func main() {
    strbry_cs := make(chan string)
    choco_cs := make(chan string)

    //two cake makers
    go makeCakeAndSend(choco_cs, "Chocolate", 3)   //make 3 chocolate cakes and send
    go makeCakeAndSend(strbry_cs, "Strawberry", 4) //make 3 strawberry cakes and send

    //one cake receiver and packer
    receiveCakeAndPack(strbry_cs, choco_cs) //pack all cakes received on these cake channels
}
```

## 参考文献

- [Go中的Channel](https://www.jianshu.com/p/15c94893124c)
- [range和select](https://www.jianshu.com/p/fe5dd2efed5d)
