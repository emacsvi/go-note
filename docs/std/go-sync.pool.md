---
title: "go-sync.pool"
slug: "go-pool"
date: "2018-11-23 01:00:01"
categories:
    - go
tags:
    - go
---

## 前言

Go 1.3 的**sync包**中加入一个新特性：`Pool`。

这个类设计的目的是用来保存和复用临时对象，以减少内存分配，降低**GC**压力。
```go
type Pool 
func (p *Pool) Get() interface{} 
func (p *Pool) Put(x interface{}) 
New func() interface{} 
```
垃圾回收一直是Go语言的一块心病，在它执行垃圾回收的时间中，你很难做什么。
在垃圾回收压力大的服务中，`GC`占据的CPU有可能超过2%，造成的Pause经常超过2ms。垃圾严重的时候，秒级的GC也出现过。
如果经常临时使用一些大型结构体，可以用Pool来减少GC。

示例代码
```go
package main
import (
 "fmt"
 "sync"
 "time"
)
type structR6 struct {
 B1 [100000]int
}
var r6Pool = sync.Pool{
 New: func() interface{} {
 return new(structR6)
 },
}
func usePool() {
 startTime := time.Now()
 for i := 0; i < 10000; i++ {
 sr6 := r6Pool.Get().(*structR6)
 sr6.B1[0] = 0
 r6Pool.Put(sr6)
 }
 fmt.Println("pool Used:", time.Since(startTime))
}
func standard() {
 startTime := time.Now()
 for i := 0; i < 10000; i++ {
 var sr6 structR6
 sr6.B1[0] = 0
 }
 fmt.Println("standard Used:", time.Since(startTime))
}
func main() {
 standard()
 usePool()
}
```
一个含有100000个int值的结构体，在标准方法中，每次均新建，重复10000次，一共需要耗费193ms；
如果用完的struct可以废物利用，放回pool中。需要新的结构体的时候，尝试去pool中取，而不是重新生成，重复10000次仅需要693us。
这样简单的操作，却节约了99.65%的时间，也节约了各方面的资源。最重要的是它可以有效减少GC CPU和GC Pause。

结果：

```shell
standard Used: 271.160179ms
pool Used: 668.08µs
```

## 参考文献

- [合理使用pool](https://www.jishux.com/p/b451ab045a20e682)
