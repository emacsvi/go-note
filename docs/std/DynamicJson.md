---
title: Dynamic Json In Go
slug: dynamic-json-in-go
date: "2019-03-18"
description: 动态的json解析
categories:
- go
tags:
- go
---

动态的解析Json的类型，虽然go是静态的。但是还是有办法解析。而且也没有那么多**ugly code**
<!--more-->

## 概念
其实主要是利用**interface{}**类型与**json.RawMessage**进行结合使用。会觉得代码好了很多。看看下面的示例：
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

const input = `
{
	"type": "sound",
	"msg": {
		"description": "dynamite",
		"authority": "the Bruce Dickinson"
	}
}
`

type Envelope struct {
	Type string
	Msg  interface{}
}

type Sound struct {
	Description string
	Authority   string
}

func main() {
	var msg json.RawMessage
	env := Envelope{
		Msg: &msg,
	}
	if err := json.Unmarshal([]byte(input), &env); err != nil {
		log.Fatal(err)
	}
	switch env.Type {
	case "sound":
		var s Sound
		if err := json.Unmarshal(msg, &s); err != nil {
			log.Fatal(err)
		}
		var desc string = s.Description
		fmt.Println(desc)
	default:
		log.Fatalf("unknown message type: %q", env.Type)
	}
}
```
另外如果多个顶部消息的时候，你可以只解析你知道的一部分，再反复利用原始的**[]byte**字节进行其他判断完后的解析，也是一样的。
```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

const input = `
{
	"type": "sound",
	"description": "dynamite",
	"authority": "the Bruce Dickinson"
}
`

type Envelope struct {
	Type string
}

type Sound struct {
	Description string
	Authority   string
}

func main() {
	var env Envelope
	buf := []byte(input)
	if err := json.Unmarshal(buf, &env); err != nil {
		log.Fatal(err)
	}
	switch env.Type {
	case "sound":
		var s struct {
			Envelope
			Sound
		}
    // We can cope with that by unmarshaling the data twice:
		if err := json.Unmarshal(buf, &s); err != nil {
			log.Fatal(err)
		}
		var desc string = s.Description
		fmt.Println(desc)
	default:
		log.Fatalf("unknown message type: %q", env.Type)
	}
  }
```

## 参考文献
- [Dynamic Json In Go](https://eagain.net/articles/go-dynamic-json/)

