---
title: websocket实践
slug: go-websocket-practice
date: "2019-03-16"
description: go-websocket practice 实践
categories:
- go
tags:
- go
- websocket
---

websocket的实践内容，主要以`go`语言来做。
<!--more-->

## 概念

推拉模式区别
拉模式（定时轮询访问接口获取数据）
数据更新频率低，则大多数的数据请求时无效的
在线用户数量多，则服务端的查询负载很高
定时轮询拉取，无法满足时效性要求
推模式（向客户端进行数据的推送）
仅在数据更新时，才有推送
需要维护大量的在线长连接
数据更新后，可以立即推送
基于WebSocket协议做推送
浏览器支持的socket编程，轻松维持服务端的长连接
基于TCP协议之上的高层协议，无需开发者关心通讯细节
提供了高度抽象的编程接口，业务开发成本较低
WebSocket协议的交互流程
--------------------- 
客户端首先发起一个Http请求到服务端，请求的特殊之处，在于在请求里面带了一个upgrade的字段，告诉服务端，我想生成一个websocket的协议，服务端收到请求后，会给客户端一个握手的确认，返回一个switching， 意思允许客户端向websocket协议转换，完成这个协商之后，客户端与服务端之间的底层TCP协议是没有中断的，接下来，客户端可以向服务端发起一个基于websocket协议的消息，服务端也可以主动向客户端发起websocket协议的消息，websocket协议里面通讯的单位就叫message。

传输协议原理
协议升级后，继续复用Http协议的底层socket完成后续通讯
message底层会被切分成多个frame帧进行传输，从协议层面不能传输一个大包，只能切成一个个小包传输
编程时，只需操作message，无需关心frame（属于协议和类库自身去操作的）
框架底层完成TCP网络I/O，WebSocket协议的解析，开发者无需关心
--------------------- 


参考文献

- [websocket细节](https://blog.csdn.net/Wing_93/article/details/81587809)
- [数组和切片](http://www.cnblogs.com/hustcat/p/4002707.html)
- [slice](http://blog.wuxu92.com/array-and-slice-in-golang/)
- [go array](https://golang.org/doc/effective_go.html#arrays)
- [slices](https://gobyexample.com/slices)
- [array and slice](https://golang.org/doc/effective_go.html#arrays)

