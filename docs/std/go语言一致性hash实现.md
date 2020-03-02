---
title: "go语言一致性hash实现"
slug: "go-consistent"
date: "2018-11-23 01:00:22"
categories:
    - go
tags:
    - go
---

## 原理

## 实例
原理很简单，我想手动写过HashMap的人，或者做过服务器负载均衡或者集群的人都有了解。花一个小时就能搞明白。

从这个库里面来做一致性哈希算法的功能：[https://github.com/lafikl/consistent](https://github.com/lafikl/consistent)

```go
package funcs

import (
	"github.com/lafikl/consistent"
	"log"
)

func ConSis() {
	c := consistent.New()

	// adds the hosts to the ring
	c.Add("127.0.0.1:8000")
	c.Add("192.168.1.6:8000")
	c.Add("192.168.1.8:8000")
	c.Add("192.168.1.14:8000")
	c.Add("192.168.1.15:9000")



  // Returns the host that owns `key`.
	//
	// As described in https://en.wikipedia.org/wiki/Consistent_hashing
	//
	// It returns ErrNoHosts if the ring has no hosts in it.
	host, err := c.Get("/app.html") 
	if err != nil {
		log.Fatal(err)
	}

	log.Println(host) // print: 192.168.1.6:8000
	host, err = c.Get("jimny,dada  skfdkaskdfsakfdkasfkdaslfkdlasfdk k2r4o	23402ewrew0")
	if err != nil {
		log.Fatal(err)
	}

	log.Println(host) // print: 192.168.1.8:8000

  // increases the load of `host`, we have to call it before sending the request
	c.Inc(host)
	log.Println("send request to", host)
	c.Done(host)
}
```

## 参考文献

- [go consistent](https://github.com/lafikl/consistent)
