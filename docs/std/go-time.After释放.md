---
title: "go-time.After释放"
slug: "go-time-after"
date: "2018-11-23 01:00:08"
categories:
    - go
tags:
    - go
---

# 结论

```go
package main
import "time"

func main()  {
    ch := make(chan int, 10)

    go func() {
        for {
            ch <- 100
        }
    }()

    idleDuration := 3 * time.Minute
    idleDelay := time.NewTimer(idleDuration)
    defer idleDelay.Stop()

    for {
        idleDelay.Reset(idleDuration)

        select {
            case x := <- ch:
                println(x)
            case <-idleDelay.C:
                return
            }
    }
}
```

## time.After释放的问题

在谢大群里看到有同学在讨论`time.After`泄漏的问题，就算时间到了也不会释放，瞬间就惊呆了，忍不住做了试验，结果发现应该没有这么的恐怖的，是有泄漏的风险不过不算是泄漏，先看API的说明：
```go
// After waits for the duration to elapse and then sends the current time
// on the returned channel.
// It is equivalent to NewTimer(d).C.
// The underlying Timer is not recovered by the garbage collector
// until the timer fires. If efficiency is a concern, use NewTimer
// instead and call Timer.Stop if the timer is no longer needed.
func After(d Duration) <-chan Time {
    return NewTimer(d).C
}
```
提到了一句`The underlying Timer is not recovered by the garbage collector`，这句挺吓人不会被`GC`回收，不过后面还有条件`until the timer fires`，说明`fire`后是会被回收的，所谓`fire`就是到时间了，写个例子证明下压压惊：
```go
package main

import "time"

func main() {
    for {
        <- time.After(10 * time.Nanosecond)
    }
}
```
显示内存稳定在`5.3MB`，CPU为`161%`，肯定被GC回收了的。当然如果放在`goroutine`也是没有问题的，一样会回收：
```go
package main

import "time"

func main() {
    for i := 0; i < 100; i++ {
        go func(){
            for {
                <- time.After(10 * time.Nanosecond)
            }
        }()
    }
    time.Sleep(1 * time.Hour)
}
```
只是资源消耗会多一点，CPU为`422%`，内存占用`6.4MB`。因此：

Remark: `time.After(d)`在`d`时间之后就会`fire`，然后被GC回收，不会造成资源泄漏的。

那么API所说的`If efficieny is a concern, user NewTimer instead and call Timer.Stop`是什么意思呢？这是因为一般`time.After`会在`select`中使用，如果另外的分支跑得更快，那么`timer`是不会立马释放的(到期后才会释放)，比如这种：

```go
select {
    case time.After(3*time.Second):
        return errTimeout
    case packet := packetChannel:
        // process packet.
}
```
如果`packet`非常多，那么总是会走到下面的分支，上面的`timer`不会立刻释放而是在`3秒后才能释放`，和下面代码一样：
```go
package main

import "time"

func main() {
    for {
        select {
        case <-time.After(3 * time.Second):
        default:
        }
    }
}
```
这个时候，就相当于会堆积了3秒的timer没有释放而已，会不断的新建和释放timer，内存会稳定在2.8GB，这个当然就不是最好的了，可以主动释放：

```go
package main

import "time"

func main() {
    for {
        t := time.NewTimer(3*time.Second)

        select {
        case <- t.C:
        default:
            t.Stop()
        }
    }
}
```
这样就不会占用`2.8GB`内存了，只有`5MB`左右。因此，总结下这个After的说明：

- `GC`肯定会回收`time.After`的，就在`d`之后就回收。一般情况下让系统自己回收就好了。
- 如果有效率问题，应该使用`Timer`在不需要时主动`Stop`。大部分时候都不用考虑这个问题的。

## 参考文献

- [time.After释放](https://gocn.io/article/403)
- [after常用](http://xiaorui.cc/2019/02/11/%E5%88%86%E6%9E%90golang-time-after%E5%BC%95%E8%B5%B7%E5%86%85%E5%AD%98%E6%9A%B4%E5%A2%9Eoom%E9%97%AE%E9%A2%98/)
