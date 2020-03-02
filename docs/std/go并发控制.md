---
title: "go并发控制"
slug: "go-chan-goroutine"
date: "2018-11-23 01:00:19"
categories:
    - go
tags:
    - go
---

最简单的方式,定义一个chan来做控制：

<!--more-->

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    concurrency := 3
    sem := make(chan bool, concurrency)
    urls := []string{"url1", "url2", "url3", "url4", "url5", "url6", "url7"}
    for _, url := range urls {
        sem <- true
        go func(url string) {
            defer func() { 
            <-sem }()
            fmt.Println(url)
            time.Sleep(time.Second * 3)
        }(url)
    }
    for i := 0; i < cap(sem); i++ {
        sem <- true
    }
}
```
创建一个个数为3的chan作为并发限制，也就是限制同时进行的coroutine数目为3。
用time.Sleep模拟了一个耗时三秒的操作，这样在输出结果中可以明显看到，虽然有七个任务(url1-url7)但是同时执行的只有三个，三秒后才继续执行另外三个，再三秒后才执行剩下的最后一个。
代码末尾是给chan传递值，在chan得到值之前一直会block在`sem<-true`读取这一步，这也是并发限制的原理。

## 另外一个例子的代码：
```go
package main

import (
	"flag"
	"fmt"
	"time"
)

// Fake a long and difficult work.
func DoWork() {
	time.Sleep(500 * time.Millisecond)
}

func main() {
	maxNbConcurrentGoroutines := flag.Int("maxNbConcurrentGoroutines", 5, "the number of goroutines that are allowed to run concurrently")
	nbJobs := flag.Int("nbJobs", 100, "the number of jobs that we need to do")
	flag.Parse()

	// Dummy channel to coordinate the number of concurrent goroutines.
	// This channel should be buffered otherwise we will be immediately blocked
	// when trying to fill it.
	concurrentGoroutines := make(chan struct{}, *maxNbConcurrentGoroutines)
	// Fill the dummy channel with maxNbConcurrentGoroutines empty struct.
	for i := 0; i < *maxNbConcurrentGoroutines; i++ {
		concurrentGoroutines <- struct{}{}
	}

	// The done channel indicates when a single goroutine has
	// finished its job.
	done := make(chan bool)
	// The waitForAllJobs channel allows the main program
	// to wait until we have indeed done all the jobs.
	waitForAllJobs := make(chan bool)

	// Collect all the jobs, and since the job is finished, we can
	// release another spot for a goroutine.
	go func() {
		for i := 0; i < *nbJobs; i++ {
			<-done
			// Say that another goroutine can now start.
			concurrentGoroutines <- struct{}{}
		}
		// We have collected all the jobs, the program
		// can now terminate
		waitForAllJobs <- true
	}()

	// Try to start nbJobs jobs
	for i := 1; i <= *nbJobs; i++ {
		fmt.Printf("ID: %v: waiting to launch!\n", i)
		// Try to receive from the concurrentGoroutines channel. When we have something,
		// it means we can start a new goroutine because another one finished.
		// Otherwise, it will block the execution until an execution
		// spot is available.
		<-concurrentGoroutines
		fmt.Printf("ID: %v: it's my turn!\n", i)
		go func(id int) {
			DoWork()
			fmt.Printf("ID: %v: all done!\n", id)
			done <- true
		}(i)
	}

	// Wait for all jobs to finish
	<-waitForAllJobs
}
```

## sender模块
在sender模块里面，通过配置文件来控制并发。先创建控制并发的chan
```go
var (
	SmsWorkerChan  chan int
	MailWorkerChan chan int
)

func InitWorker() {
	workerConfig := g.Config().Worker
	SmsWorkerChan = make(chan int, workerConfig.Sms)
	MailWorkerChan = make(chan int, workerConfig.Mail)
}
```
再消息的时候通过这个通道来控制并发数。是不是很简单呢：
```go
func ConsumeSms() {
	queue := g.Config().Queue.Sms
	for {
		L := redis.PopAllSms(queue)
		if len(L) == 0 {
			time.Sleep(time.Millisecond * 200)
			continue
		}
		SendSmsList(L)
	}
}

func SendSmsList(L []*model.Sms) {
	for _, sms := range L {
		SmsWorkerChan <- 1
		go SendSms(sms)
	}
}

func SendSms(sms *model.Sms) {
	defer func() {
		<-SmsWorkerChan
	}()

	url := g.Config().Api.Sms
	r := httplib.Post(url).SetTimeout(5*time.Second, 2*time.Minute)
	r.Param("tos", sms.Tos)
	r.Param("content", sms.Content)
	resp, err := r.String()
	if err != nil {
		log.Println(err)
	}

	proc.IncreSmsCount()

	if g.Config().Debug {
		log.Println("==sms==>>>>", sms)
		log.Println("<<<<==sms==", resp)
	}
}
```


## 参考文献

- [控制并发例子](https://gist.github.com/AntoineAugusti/80e99edfe205baf7a094)
- [并发](https://www.jianshu.com/p/f36ca87edb64)
- [WaitGroup方式](https://blog.csdn.net/whynottrythis/article/details/78011709)
