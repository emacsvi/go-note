---
title: go-信号及热重启
slug: go-signal-gracefull
date: "2018-11-24"
description: go-信号及热重启
categories: 
- go 
tags: 
- go 
---


go-信号及热重启。
<!--more-->

## 概念
常用代码:
```go
package main
import "fmt"
import "os"
import "os/signal"
import "syscall"
func main() {
    // Go signal notification works by sending `os.Signal`
    // values on a channel. We'll create a channel to
    // receive these notifications (we'll also make one to
    // notify us when the program can exit).
    sigs := make(chan os.Signal, 1)
    done := make(chan bool, 1)
    // `signal.Notify` registers the given channel to
    // receive notifications of the specified signals.
    signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
    // This goroutine executes a blocking receive for
    // signals. When it gets one it'll print it out
    // and then notify the program that it can finish.
    go func() {
        sig := <-sigs
        fmt.Println()
        fmt.Println(sig)
        done <- true
    }()
    // The program will wait here until it gets the
    // expected signal (as indicated by the goroutine
    // above sending a value on `done`) and then exit.
    fmt.Println("awaiting signal")
    <-done
    fmt.Println("exiting")
}
```

## 参考文献
- [看鸟窝就够了](https://colobu.com/2015/10/09/Linux-Signals/)

