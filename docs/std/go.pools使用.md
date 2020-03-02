---

title: go.pools使用
slug: go-ants
date: "2019-11-30"
description: go.pools使用
categories: 
- go 
tags: 
- go 
---


go.pools使用。
<!--more-->

## 概念

一直以来自己都在用自己写的协程管理library, 也用了四五年了，没发现有什么不妥。最近看一篇文章才发现原来还有可以对协程池做这么多优化的操作。于是我从新研究了一下别人的库。以便日后对自己的库进行更新。ants库的精髓就在于对协程的复用。的确是个好的想法。

## ants

### 默认的同步方式

同步**默认**使用的时候：

```go
// Init a instance pool when importing ants.
// 默认可以启动这么多协程: DefaultAntsPoolSize = 1<<31-1 = 1073741824
defaultAntsPool, _ = NewPool(DefaultAntsPoolSize)
```

```go
// 默认提交的时候利用了默认的defaultAntsPool实例。
func Submit(task func()) error {
	return defaultAntsPool.Submit(task)
}

// Submit submits a task to this pool.
func (p *Pool) Submit(task func()) error {
	if atomic.LoadInt32(&p.release) == CLOSED {
		return ErrPoolClosed
	}
	var w *goWorker
	if w = p.retrieveWorker(); w == nil {
		return ErrPoolOverload
	}
	w.task <- task
	return nil
}
```

测试同步程序的代码：

```go
func dd() {
	time.Sleep(time.Millisecond)
}

func syncGo() {
	defer ants.Release()
	// 运行1千万个
	runTimes := 10000000
	var wg sync.WaitGroup
	syncCalculateSum := func() {
		dd()
		defer wg.Done()
	}

	// 默认可以启动这么多协程: 1<<31-1 = 1073741824
	// Init a instance pool when importing ants.
	// defaultAntsPool, _ = NewPool(DefaultAntsPoolSize)
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
		err := ants.Submit(syncCalculateSum)
		if err != nil {
			panic(err)
		}
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", ants.Running()) // 实际只运行了2400个协程
	fmt.Printf("finish all tasks.\n")
}
```

运行结果(只实际启动了2460个协程去做事情)：

```bash
./gopools
running goroutines: 2460
finish all tasks.
```

### 常规的生产者与消费者方式

```go
var sum int32

// 消费者方法
func myFunc(i interface{}) {
	n := i.(int32)
	atomic.AddInt32(&sum, n)
	time.Sleep(time.Second)
	fmt.Printf("run with %d\n", n)
}

func newPool() {
	// Use the pool with a function,
	// set 10 to the capacity of goroutine pool and 1 second for expired duration.
	var runTimes = 100
	var wg sync.WaitGroup
  // 将消费方法放入池中进行回调
	p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
		myFunc(i)
		wg.Done()
	})
	defer p.Release() //不用的时候一定要记得释放
	// Submit tasks one by one.
	for i := 0; i < runTimes; i++ {
		wg.Add(1)
    _ = p.Invoke(int32(i)) // 生产者，将数据放入队列中，让work去Invoke()消费这个数，如果池中已经满了。默认情况下为block
	}
	wg.Wait()
	fmt.Printf("running goroutines: %d\n", p.Running())
	fmt.Printf("finish all tasks, result is %d\n", sum)
}
```

查看关键的几个参数：

```go
	// Max number of goroutine blocking on pool.Submit.
	// 0 (default value) means no such limit.
	MaxBlockingTasks int

	// When Nonblocking is true, Pool.Submit will never be blocked.
	// ErrPoolOverload will be returned when Pool.Submit cannot be done at once.
	// When Nonblocking is true, MaxBlockingTasks is inoperative.
	Nonblocking bool
```

也就是可以设置最大阻塞的协程数量。这个功能我觉得用处不大，比较生产者应该只有一个口子。除非是多协程的生产。

另外`Nonblocking`如果设置的话，生产的时候就不会block而是直接返回错误。



### 常用功能

```go
// Set 10000 the size of goroutine pool
p, _ := ants.NewPool(10000) // 限制池大小
p.Submit(func(){}) // 提交一个任务去执行
// 重新分配大小
p.Tune(1000) // Tune its capacity to 1000
p.Tune(100000) // Tune its capacity to 100000

//创建一个池并且初始化完成
// ants will pre-malloc the whole capacity of pool when you invoke this method
p, _ := ants.NewPool(100000, ants.WithPreAlloc(true))

defer pool.Release() //释放池子

p, _ := ants.NewPoolWithFunc(10, func(i interface{}) {
	myFunc(i)
	wg.Done()
}, ants.WithNonblocking(true), ants.WithExpiryDuration(time.Second))
```

## 参考文献
- [ants github](https://github.com/panjf2000/ants)
- [实现的博客](https://taohuawu.club/high-performance-implementation-of-goroutine-pool)

