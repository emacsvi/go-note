---
title: lotus矿池备忘实践
slug: go-websocket-practice
date: "2020-01-26"
description: go-websocket practice 实践
categories:
- go
tags:
- go
- websocket
---

`lotus`矿池常用的一些备忘。
<!--more-->

## prometheus概念

ubuntu18使用root帐户：

```bash
sudo passwd root
kill -hup 4100(proemthues pid)

node_exporter
```



框架底层完成TCP网络I/O，WebSocket协议的解析，开发者无需关心
--------------------- 


参考文献

- [websocket细节](https://blog.csdn.net/Wing_93/article/details/81587809)
- [数组和切片](http://www.cnblogs.com/hustcat/p/4002707.html)
- [slice](http://blog.wuxu92.com/array-and-slice-in-golang/)
- [go array](https://golang.org/doc/effective_go.html#arrays)
- [slices](https://gobyexample.com/slices)
- [array and slice](https://golang.org/doc/effective_go.html#arrays)

