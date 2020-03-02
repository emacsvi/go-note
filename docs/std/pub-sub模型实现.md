---
title: pub-sub模型实现
slug: go-pub-sub
date: "2019-11-30"
description: pub-sub模型实现
categories: 
- go 
tags: 
- go 
---


pub-sub模型实现。
<!--more-->

## 自己实现

发布订阅（publish-and-subscribe）模型通常被简写为pub/sub模型。在这个模型中，消息生产者成为发布者（publisher），而消息消费者则成为订阅者（subscriber），生产者和消费者是M:N的关系。在传统生产者和消费者模型中，是将消息发送到一个队列中，而发布订阅模型则是将消息发布给一个主题。

为此，我们构建了一个名为`pubsub`的发布订阅模型支持包：

pubsub.go

```go
package pubsub

import (
	"sync"
	"time"
)

type (
	Subscriber chan interface{}
	TopicFunc  func(v interface{}) bool
)

type Publisher struct {
	// subscribers 是程序的核心，订阅者都会注册在这里，publisher发布消息的时候也会从这里开始
	subscribers map[Subscriber]TopicFunc
	buffer      int           // 订阅者的缓冲区长度
	timeout     time.Duration // publisher 发送消息的超时时间
	// m 用来保护 subscribers
	// 当修改 subscribers 的时候(即新加订阅者或删除订阅者)使用写锁
	// 当向某个订阅者发送消息的时候(即向某个 Subscriber channel 中写入数据)，使用读锁
	m sync.RWMutex
}

// 构建一个发布者对象, 可以设置发布超时时间和缓存队列的长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
	return &Publisher{
		buffer:      buffer,
		timeout:     publishTimeout,
		subscribers: make(map[Subscriber]TopicFunc),
	}
}

// 添加一个新的订阅者，订阅全部主题
func (p *Publisher) Subscribe() Subscriber {
	return p.SubscribeTopic(nil)
}

// 添加一个新的订阅者，订阅过滤器筛选后的主题
func (p *Publisher) SubscribeTopic(topic TopicFunc) Subscriber {
	ch := make(Subscriber, p.buffer)
	p.m.Lock()
	p.subscribers[ch] = topic
	p.m.Unlock()

	return ch
}

//Evict 删除掉某个订阅者
func (p *Publisher) Evict(sub Subscriber) {
	p.m.Lock()
	defer p.m.Unlock()

	delete(p.subscribers, sub)
	close(sub)
}

// 发布一个主题
func (p *Publisher) Publish(v interface{}) {
	p.m.RLock()
	defer p.m.RUnlock()

	var wg sync.WaitGroup
	// 同时向所有订阅者写消息，订阅者利用 topic 过滤消息
	for sub, topic := range p.subscribers {
		wg.Add(1)
		go p.sendTopic(sub, topic, v, &wg)
	}

	wg.Wait()
}

//Close 关闭 Publisher，删除所有订阅者
// 关闭发布者对象，同时关闭所有的订阅者管道。
func (p *Publisher) Close() {
	p.m.Lock()
	defer p.m.Unlock()

	for sub := range p.subscribers {
		delete(p.subscribers, sub)
		close(sub)
	}
}

// 发送主题，可以容忍一定的超时
func (p *Publisher) sendTopic(sub Subscriber, topic TopicFunc, v interface{}, wg *sync.WaitGroup) {
	defer wg.Done()

	if topic != nil && !topic(v) {
		return
	}

	select {
	case sub <- v:
	case <-time.After(p.timeout):
	}
}
```

上面的测试代码,下面的例子中，有两个订阅者分别订阅了全部主题和含有"golang"的主题：：

```go
package pubsub

import (
	"fmt"
	"runtime"
	"strings"
	"sync"
	"sync/atomic"
	"testing"
	"time"

	"github.com/stretchr/testify/assert"
)

func TestPubSub(t *testing.T) {
	p := NewPublisher(100*time.Millisecond, 10)
	defer p.Close()

	// all 订阅者订阅所有消息
	all := p.Subscribe()
	// golang 订阅者仅订阅包含 golang 的消息
	golang := p.SubscribeTopic(func(v interface{}) bool {
		if s, ok := v.(string); ok {
			return strings.Contains(s, "golang")
		}
		return false
	})

	p.Publish("hello, world!")
	p.Publish("hello, golang!")

	var wg sync.WaitGroup
	wg.Add(2)

	go func() {
		for msg := range all {
			_, ok := msg.(string)
			assert.True(t, ok)
		}
		wg.Done()
	}()

	go func() {
		for msg := range golang {
			v, ok := msg.(string)
			assert.True(t, ok)
			assert.True(t, strings.Contains(v, "golang"))
		}
		wg.Done()
	}()

	p.Close()
	wg.Wait()
}
```

在发布订阅模型中，每条消息都会传送给多个订阅者。发布者通常不会知道、也不关心哪一个订阅者正在接收主题消息。订阅者和发布者可以在运行时动态添加，是一种松散的耦合关系，这使得系统的复杂性可以随时间的推移而增长。在现实生活中，像天气预报之类的应用就可以应用这个并发模式。



## 商用版本

这里使用**cskr/pubsub**库，因为lotus里面也是用的这个库。简单的说明一下这个库的实现。



## 参考文献
- [pub/sub柴大](https://chai2010.cn/advanced-go-programming-book/ch1-basic/ch1-06-goroutine.html)
- [map的key值](http://lanlingzi.cn/post/technical/2016/0904_go_map/)
- [cskr/pubsub](https://github.com/cskr/pubsub)

