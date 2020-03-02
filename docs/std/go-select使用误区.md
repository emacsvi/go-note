---
title: "go-select使用误区"
slug: "go-select"
date: "2018-11-23 01:00:06"
categories:
    - go
tags:
    - go
---

select是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，要么是发送要么是接收。

select随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。一个默认的子句应该总是可运行的。

## select语法

**Go** 编程语言中 `select` 语句的语法如下：

```go
select {
    case communication clause  :
       statement(s);      
    case communication clause  :
       statement(s); 
    /* 你可以定义任意数量的 case */
    default : /* 可选 */
       statement(s);
}
```

以下描述了 select 语句的语法：

- 每个case都必须是一个通信
- 所有channel表达式都会被求值
- 所有被发送的表达式都会被求值
- 如果任意某个通信可以进行，它就执行；其他被忽略。
- 如果有多个case都可以运行，Select会随机`公平`地选出一个执行。其他不会执行。 
否则：
  - 如果有default子句，则执行该语句。
  - 如果没有default字句，select将阻塞，直到某个通信可以运行；Go不会重新对channel或值进行求值。

**示例：**
```go
package main

import "fmt"

func main() {
   var c1, c2, c3 chan int
   var i1, i2 int
   select {
      case i1 = <-c1:
         fmt.Printf("received ", i1, " from c1\n")
      case c2 <- i2:
         fmt.Printf("sent ", i2, " to c2\n")
      case i3, ok := (<-c3):  // same as: i3, ok := <-c3
         if ok {
            fmt.Printf("received ", i3, " from c3\n")
         } else {
            fmt.Printf("c3 is closed\n")
         }
      default:
         fmt.Printf("no communication\n")
   }    
}
```

以上代码执行结果为：
```bash
no communication
```

## 问题复现

```go
package main

import(
    "fmt"
    "time"
)

func add(ch chan int) {
    for i:=0;i<10;i++{
        ch <- i
    }
}

// timeout problem recurrent
func test2() {
    ch := make(chan int, 10)
    go add(ch)
    for {
        select {
            case <- time.After(2 * time.Second):
                fmt.Println("timeout")
                return
            case t := <- ch:
                fmt.Println(t) // if ch not empty, time.After will nerver exec
                fmt.Println("sleep one seconds ...")
                time.Sleep(1 * time.Second)
                fmt.Println("sleep one seconds end...")
        }
    }
}
```

根据条件5：如果有多个case都可以运行，Select会随机公平地选出一个执行。其他不会执行。 
但是运行上述代码，当ch通道中存在数据时，`time.After`总是得不到运行，因此到时超时未生效（就像是两个case都成立时，select 都"公平"地选择了 `case <- ch`，导致超时逻辑未生效）


## 改进1

```go
func test3() {
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()
    ch := make(chan int, 10)
    go add(ch)
    for {
        select {
        case t := <- ch:
                fmt.Println(t) // if ch not empty, time.After will nerver exec
                fmt.Println("sleep one seconds ...")
                time.Sleep(1 * time.Second)
                fmt.Println("sleep one seconds end...")
        case <- ticker.C:
                fmt.Println("timeout")
                return
        default:
        }
    }
}
```

改进1 随机性失败 
当`case <- ch` 和 `case <- ticker.C` 同时成立时，**Select**会随机公平地选出一个执行，有可能选择到前者，导致超时随机行失败


## 最终解决方式

```go
// final solution
func test4() {
    ticker := time.NewTicker(2 * time.Second)
    defer ticker.Stop()
    ch := make(chan int, 10)
    go add(ch)
    for {
        select {
        case t := <- ch:
                fmt.Println(ch) // if ch not empty, time.After will nerver exec
                fmt.Println("sleep one seconds ...")
                time.Sleep(1 * time.Second)
                fmt.Println("sleep one seconds end...")
        default: // forbid block
        }
        select {
            case <- ticker.C:
                fmt.Println("timeout")
                return
            default: // forbid block
        }
    }
}
```
一定要记得每个`select`后面加上`default`语句。 将【超时】和【收包】放在**各自单独**的select里面，【超时】一定可以执行到.


## 另网上一个场景

上周末参加Go技术聚会，京东的美女工程师讲到一个`select-case`和`time.Ticker`的使用注意事项（真实的应用场景是:在测试收包的顺序的时候，加了个tick就发现丢包了），觉得很有意思，记录一下。


```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func init() {
	runtime.GOMAXPROCS(runtime.NumCPU())
}

func main() {
	ch := make(chan int, 1024)
	go func(ch chan int) {
		for {
			val := <-ch
			fmt.Printf("val:%d\n", val)
		}
	}(ch)

	tick := time.NewTicker(1 * time.Second)
	for i := 0; i < 20; i++ {
		select {
		case ch <- i:
		case <-tick.C:
			fmt.Printf("%d: case <-tick.C\n", i)
		}	

		time.Sleep(200 * time.Millisecond)
	}
	close(ch)
	tick.Stop()
}
```

输出如下:
```bash
val:0
val:1
val:2
val:3
val:4
val:5
6: case <-tick.C
val:7
val:8
val:9
10: case <-tick.C
val:11
val:12
val:13
val:14
15: case <-tick.C
val:16
val:17
val:18
val:19
```


问题出在这个`select`里面：
```go
select {
  case ch <- i:
  case <-tick.C:
    fmt.Printf("%d: case <-tick.C\n", i)
}
```
当两个case条件都满足的时候，运行时系统会通过一个伪随机的算法决定哪个case将会被执行
所以当`tick.C`条件满足的那个循环，有某种概率造成`ch<-i`没有发送(虽然通道两端没有阻塞，满足发送条件)


解决方案1: 一旦`tick.C`随机的case被随机到，就多执行一次`ch<-i` (不体面，如果有多个case就不通用了)
```go
select {
  case ch <- i:
  case <-tick.C:
    fmt.Printf("%d: case <-tick.C\n", i)
    ch <- i
}
```


解决方案2: 将`tick.C`的case单独放到一个`select`里面，并加入一个`default`（保证不阻塞）
```go
select {
  case ch <- i:
}
select {
  case <-tick.C:
    fmt.Printf("%d: case <-tick.C\n", i)
  default:
}
```

两种解决方案的输出都是希望的结果：
```bash
val:0
val:1
val:2
val:3
val:4
5: case <-tick.C
val:5
val:6
val:7
val:8
val:9
10: case <-tick.C
val:10
val:11
val:12
val:13
val:14
15: case <-tick.C
val:15
val:16
val:17
val:18
val:19
```

## 参考文献

- [select正解姿势](https://blog.csdn.net/zongyinhu/article/details/54695434)
- [select误区](https://studygolang.com/articles/5224)
