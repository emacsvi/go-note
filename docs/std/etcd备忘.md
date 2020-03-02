---
title: etcd备忘
slug: etcd-notes
date: "2019-03-13"
description: ectd
categories:
- etcd
tags:
- etcd
- go
---

## 概念
来自[raft安全性问题以及投票过程](https://www.ph0en1x.space/2019/01/10/Raft/)
Raft一共研究的就是三个子问题：

- 如何选举领导者，当现有的领导者失效的情况下，如何选出新的领导者，选举领导者的时候如何确定新的领导者不会破坏安全性原则
- 如何进行日志的复制，什么时候可以保证日志可以安全提交
- 如何保证系统的安全性，要维护包含以下的几个性质：

  - 选举安全性：每一个任期(term)内只能有一个服务器被选举为领导者。
  - 领导者只做append操作：领导者的所有的日志entry只能append在队列尾，而不能删除和覆盖。
  - 日志匹配：如果两个日志在相同的序号上的日志entry的任期相同，那么这个日志从头到这个序号之间的日志entry时完全相同的。
  - 领导者完整性：在一个领导者上提交的日志entry，在后面的term的领导者中也必须出现。也就是说，只有同步了最新log的server才能够被选举为领导者。
  - 状态机安全性：如果一个服务器已经执行了日志上某个entry中的指令，那么其他服务器上相同序号不同的日志entry将不能够执行。

## go语言操作

#### 连接`etcd`
要访问etcd第一件事就是创建client，它需要传入一个Config配置，这里传了2个选项:

- Endpoints：etcd的多个节点服务地址，因为我是单点测试，所以只传1个。
- DialTimeout：创建client的首次连接超时，这里传了5秒，如果5秒都没有连接成功就会返回err；值得注意的是，一旦client创建成功，我们就不用再关心后续底层连接的状态了，client内部会重连。

当然，如果`err != nil`，那么一般情况下我们可以选择重试几次，或者退出程序（重启）。

查看client里面有Cluster、KV、Lease…，你会发现它们其实就代表了整个客户端的几大核心功能板块，分别用于：

- Cluster：向集群里增加etcd服务端节点之类，属于管理员操作。
- KV：我们主要使用的功能，即操作K-V。
- Lease：租约相关操作，比如申请一个TTL=10秒的租约。
- Watcher：观察订阅，从而监听最新的数据变化。
- Auth：管理etcd的用户和权限，属于管理员操作。
- Maintenance：维护etcd，比如主动迁移etcd的leader节点，属于管理员操作。

我们需要使用什么功能，就去获取对应的对象即可。
```go
package main

import (
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		config clientv3.Config
		cli *clientv3.Client
		err error
	)

  // Endpoints是一个集群数组
	config = clientv3.Config{
    Endpoints: []string{"127.0.0.1:2379", "192.168.1.5:2379"},
		DialTimeout: 5 * time.Second,
	}

	if cli, err = clientv3.New(config); err != nil {
		panic(err)
	}

	defer cli.Close()

	fmt.Println("连接成功。")
}
```

#### 常规的kv.Put操作
```go
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {
	var (
		cfg     clientv3.Config
		client  *clientv3.Client
		err     error
		kv      clientv3.KV
		putResp *clientv3.PutResponse
	)

	cfg = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: 5 * time.Second,
	}

	if client, err = clientv3.New(cfg); err != nil {
		panic(err)
	}

	defer client.Close()

	kv = clientv3.NewKV(client)

	if putResp, err = kv.Put(context.TODO(), "/cron/jobs/job1", "bye", clientv3.WithPrevKV()); err != nil {
		panic(err)
	} else {
		fmt.Println("Revision:", putResp.Header.Revision)
		if putResp.PrevKv != nil {
			fmt.Println("PreValue:", string(putResp.PrevKv.Value))
		}
	}
}
```

#### 常规的get,delete操作
```go
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"go.etcd.io/etcd/mvcc/mvccpb"
	"time"
)

func main() {
	var (
		cfg     clientv3.Config
		cli     *clientv3.Client
		err     error
		kv      clientv3.KV
		getResp *clientv3.GetResponse
		kvPair  *mvccpb.KeyValue
		delResp *clientv3.DeleteResponse
	)

	cfg = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: 5 * time.Second,
	}

	if cli, err = clientv3.New(cfg); err != nil {
		panic(err)
	}

	defer cli.Close()

	kv = clientv3.NewKV(cli)

	if getResp, err = kv.Get(context.TODO(), "/cron/jobs/job1"); err != nil {
		panic(err)
	} else {
		fmt.Println(getResp.Kvs)
	}

	// put job2
	if _, err = kv.Put(context.TODO(), "/cron/jobs/job2", "golang"); err != nil {
		panic(err)
	}

	// put job3
	if _, err = kv.Put(context.TODO(), "/cron/jobs/job3", "java"); err != nil {
		panic(err)
	}

	fmt.Println("开始查找。。。")

	// get prefix 查询以/cron/jobs前缀的，也就是这个目录
	if getResp, err = kv.Get(context.TODO(), "/cron/jobs", clientv3.WithPrefix()); err != nil {
		panic(err)
	} else {
		for _, kvPair = range getResp.Kvs {
			fmt.Println(string(kvPair.Key), "=", string(kvPair.Value))
		}
	}

	fmt.Println("开始删除。。。")

	if delResp, err = kv.Delete(context.TODO(), "/cron/jobs", clientv3.WithPrefix(), clientv3.WithPrevKV()); err != nil {
		panic(err)
	} else {
		for _, kvPair = range delResp.PrevKvs {
			fmt.Println(string(kvPair.Key), "=", string(kvPair.Value))
		}
	}
}
```
#### 利用kv操作Op对象
```go
package main

import (
	"context"
	"fmt"
	"go.etcd.io/etcd/clientv3"
	"time"
)

func main() {

	var (
		cfg    clientv3.Config
		cli    *clientv3.Client
		err    error
		kv     clientv3.KV
		commOp clientv3.Op
		opResp clientv3.OpResponse
	)

	cfg = clientv3.Config{
		Endpoints:   []string{"127.0.0.1:2379"},
		DialTimeout: 5 * time.Second,
	}

	if cli, err = clientv3.New(cfg); err != nil {
		panic(err)
	}

	defer cli.Close()

	kv = clientv3.NewKV(cli)

	// 创建op
	commOp = clientv3.OpPut("/cron/jobs/job8", "123456")

	// 执行OP
	if opResp, err = kv.Do(context.TODO(), commOp); err != nil {
		panic(err)
	}

	fmt.Println("写入Revision:", opResp.Put().Header.Revision)

	// 创建读op
	commOp = clientv3.OpGet("/cron/jobs/job8")
	// 执行读OP
	if opResp, err = kv.Do(context.TODO(), commOp); err != nil {
		panic(err)
	}

	fmt.Println(string(opResp.Get().Kvs[0].Value), " ", opResp.Get().Kvs[0].CreateRevision, opResp.Get().Kvs[0].ModRevision)
}
```

## 参考文献
- [go etcd v3 常用方法](https://yuerblog.cc/2017/12/12/etcd-v3-sdk-usage/)
