---
title: 理解真实世界的并发bug
slug: go-bugs
date: "2019-10-20"
description: 理解真实世界的并发bug
categories:
- go
tags:
- go
---

理解真实世界的并发bug.md
<!--more-->

## 用共享内存引起的错误
- 错误分类：bug 的原因和行为。
- bug 的原因： - 用共享内存引起的错误 - 误用消息传递引起的错误
- 行为： - 阻塞错误(那些涉及(任意数量)不能继续的goroutines) - 非阻塞错误(那些不涉及任何阻塞的goroutines)

> 令人惊讶的是，我们的研究表明，消息传递并发性错误与共享内存并发性错误一样容易，有时甚至更容易。 例如，大约58%的阻塞错误是由消息传递引起的。
> 除了违反 Go 的 channel 使用规则外（在一个没有人发送数据或关闭的 channel 上等待），许多并发错误是由消息传递和Go中的其他新语义和新库的混合使用造成的，这些混合使用很容易被忽略，但是很难检测到。

为了演示消息传递中的错误，我们使用图1中 Kubernetes 的一个阻塞错误。
```go
1 func finishReq(timeout time.Duration) r ob {
2	- ch := make(chan ob)
3	+ ch := make(chan ob, 1)
4	go func() {
5		result := fn()
6		ch <- result // block
7	}
8	select {
9		case result = <- ch:
10 		return result
11 		case <- time.After(timeout):
12 		return nil
13 	}
14}
```

`finishReq` 函数在第4行使用匿名函数创建一个子 goroutine 来处理请求——这是服务器程序的常见做法。

子 goroutine 执行 `fn()` 并通过第 6 行通道 `ch` 将结果发送回父 goroutine。

子进程将会被阻塞在第 6 行，直到父进程从第 9 行获取结果。

与此同时，父进程也会阻塞，直到子进程将结果发送到 ch 时为止（第 9 行）或超时发生时（第 11 行）。

如果超时发生得更早，或者如果 go runtime（非确定性）在两种情况都有效的情况下选择第 11 行的情况，那么父进程将在第 12 行从 `finishReq()` 返回，并且没有其他人可以从 `ch` 中获取结果，从而导致子进程永远被阻塞。

> 注意：现在 kubernetes 的最新代码中函数名叫：`func finishRequest(timeout time.Duration, fn resultFunc)`

解决方案是将 ch 从一个无缓冲通道更改为一个缓冲通道，这样即使父通道已经退出，子通道 goroutine 也可以始终发送结果。

Github: https://github.com/system-pclub/go-concurrency-bugs



## 消息传递与共享内存一起的bug

消息传递和共享内存导致的阻塞 bug 几乎一样多，而且消息传递的阻塞 bug 都和 Go 的消息传递语义例如 channel 有关，消息传递和共享内存一起使用的时候会很难发现 bug。

例如 Docker 错误使用 `WaitGroup` 导致阻塞：

```go
var group sync.WaitGroup
group.Add(len(pm.plugins))
for_, p := range pm.plugins {
    go func(p *plugin) {
        defer group.Done()
    }
    group.Wait() // 阻塞
}
// 应该在这里group.Wait()
```

错误使用 channel 和 mutex 导致阻塞：

```go
func goroutine1() {
    m.Lock()
    ch <- request // 阻塞
    m.Unlock()
}

func goroutine2() {
    for{
        m.Lock()    // 阻塞
        m.Unlock()
        request <- ch
    }
}
```

共享内存导致更多的非阻塞 bug，几乎是消息传递的 8 倍。

例如在下面这段代码里，每当 `ticker` 触发时执行一次 `f()`，通过 `stopCh` 退出循环：

```go
ticker := time.NewTicker()
for {
    f()
    select {
        case <- stopCh
            return
        case <- ticker
    }
}
```

但是 select 是非确定性的，`stopCh` 和 `ticker` 同时发生时，不一定会执行 `stopChan` 的分支，正确做法是先检查一次 `stopCh`：

```go
ticker := time.NewTicker()
for {
    select{
        case <- stopCh:
            return
        default:
    }
    f()
    select {
        case <- stopCh:
            return
        case <- ticker:
    }
}
```



## 参考文献

- [理解 Go 在实际使用中的 Bugs](https://maiyang.me/post/2019-03-06-understanding-real-world-concurrency-bugs-in-go/)
- [理解真实世界的并发 Bug](https://learnku.com/articles/24850?order_by=created_at&)

