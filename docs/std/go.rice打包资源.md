---
title: go.rice打包资源
slug: go-rice
date: "2019-11-28"
description: go.rice打包资源
categories: 
- go 
tags: 
- go 
---


go.rice打包资源。
<!--more-->

## 概念

使用 Go 开发应用的时候，有时会遇到需要读取静态资源的情况。比如开发 Web 应用，程序需要加载模板文件生成输出的 HTML。在程序部署的时候，除了发布应用可执行文件外，还需要发布依赖的静态资源文件。这给发布过程添加了一些麻烦。既然发布单独一个可执行文件是非常简单的操作，就有人会想办法把静态资源文件打包进 Go 的程序文件中。



详细的使用方法在官方给的文档里面写的很清楚：[go.rice](https://github.com/GeertJohan/go.rice)

注意规则：

- https://github.com/GeertJohan/go.rice 官网解释得很明白如何使用了。

- Then it adds the required directories to the executable binary, There are two strategies to do this. You can 'embed' the assets by generating go source code and then compile them into the executable binary, or you can 'append' the assets to the executable binary after compiling. In both cases the rice.FindBox(..) call detects the embedded or appended resources and load those, instead of looking up files from disk.

- 上面表示有种方式将resources打包到你的可执行程序里面，利用**embed**或者**append**都可以，这两种**strategies**都会去找代码里面的`rice.FindBox()`调用的地方。

- 由于是`rice进程`序后期去**解析go代码**之中的`rice.FindBox函数`的，所以在`rice.FindBox()`参数之中只能是**字符串，不能用变量**，因为它并没有你进程的上下文，并不知道你的变量内容。

- Never call FindBox() or MustFindBox() from an init() function, as there is no guarantee the boxes are loaded at that time.

- 不能在`init()`函数里面调用`FindBox()`，因为`init()`调用规则，这可能有些包的init()在rice()的init()之前了，所以并不能保证谁先执行。

- Calling FindBox and MustFindBox
  Always call FindBox() or MustFindBox() with string literals e.g. FindBox("example"). Do not use string constants or variables. This will prevent the rice tool to fail with error Error: found call to rice.FindBox, but argument must be a string literal..

- 在FindBox()之中只能写字符串内容。

- 注意：go.rice不递归处理 import。他会分析当前目录下的 go 代码中 go.rice的使用，找到对应需要嵌入的文件夹。但是子目录下的和 import 的里面的 go.rice 使用不会分析，需要你手动`cd` 过去或者` -i`指定要处理的包执行命令。这点来说非常的不友好。

- 常用命令：`rice append --exec 可执行文件 -i 引用了打包的目录`

  ```bash
  go run github.com/GeertJohan/go.rice/rice append --exec townhall -i ./cmd/lotus-townhall -i ./build
  rice append --exec gorice
  ```

## example代码

```go
package main

import (
	rice "github.com/GeertJohan/go.rice"
	"html/template"
	"log"
	"net/http"
	"os"
)

func httpRoot() {
	// Serving a static content folder over HTTP with a rice Box:
	http.Handle("/", http.FileServer(rice.MustFindBox("site").HTTPBox()))
	http.ListenAndServe(":8080", nil)
}
func httpNoRoot() {
	// Serve a static content folder over HTTP at a non-root location:
	box := rice.MustFindBox("site")
	cssFileServer := http.StripPrefix("/css/", http.FileServer(box.HTTPBox()))
	http.Handle("/css/", cssFileServer)
	http.ListenAndServe(":8080", nil)
}

// Loading a template:
func loadTemp() {
	// find a rice.Box
	templateBox, err := rice.FindBox("site")
	if err != nil {
		log.Fatal(err)
	}
	// get file contents as string
	templateString, err := templateBox.String("message.tmpl")
	if err != nil {
		log.Fatal(err)
	}
	// parse and execute the template
	tmplMessage, err := template.New("message").Parse(templateString)
	if err != nil {
		log.Fatal(err)
	}
	err = tmplMessage.Execute(os.Stdout, map[string]string{"Message": "Hello, world!"})
}

func main() {
	httpRoot()
}
```



# lotus里面的应用

最近在看lotus里面看到这个库的应用，这些人太懒了。简单的说一下里面的应用，在生成genesis初始化的时候build目录里面的go程序有几个函数都用到了它。下面贴一些代码：

代码目录如下：

![go.rice in lotus](/images/go.rice.png)

在Makefile里面直接打包，注意目录的路径：

```makefile
lotus: $(BUILD_DEPS)
	rm -f lotus
	go build -o lotus ./cmd/lotus
	go run github.com/GeertJohan/go.rice/rice append --exec lotus -i ./build
```

查看build里面的代码片断：

```go
// 生成genesis，包括钱包地址，铸币的过程信息
package build
import rice "github.com/GeertJohan/go.rice"

func MaybeGenesis() []byte {
	builtinGen, err := rice.FindBox("genesis")
	if err != nil {
		log.Warn("loading built-in genesis: %s", err)
		return nil
	}
	genBytes, err := builtinGen.Bytes("devnet.car")
	if err != nil {
		log.Warn("loading built-in genesis: %s", err)
	}

	return genBytes
}
```

下载**v15**文件内容：

```go
const paramdir = "/var/tmp/filecoin-proof-parameters"
func GetParams(storage bool, tests bool) error {
	if err := os.Mkdir(paramdir, 0755); err != nil && !os.IsExist(err) {
		return err
	}
	var params map[string]paramFile

  /* parameters.json里面的内容如下
    "v15-proof-of-spacetime-rational-535d1050e3adca2a0dfe6c3c0c4fa12097c9a7835fb969042f82a507b13310e0.vk": {
    "cid": "QmVqSdc23to4UwduCCb25223rpSccvtcgPMfRKY1qjucDc",
    "digest": "c6d258c37243b8544238a98100e3e399",
    "sector_size": 16777216
  }
  */
	paramBytes := rice.MustFindBox("proof-params").MustBytes("parameters.json")
	if err := json.Unmarshal(paramBytes, &params); err != nil {
		return err
	}

	ft := &fetch{}

	for name, info := range params {
		if !(SupportedSectorSize(info.SectorSize) || (tests && info.SectorSize == 1<<10)) {
			continue
		}
		if !storage && strings.HasSuffix(name, ".params") {
			continue
		}

		ft.maybeFetchAsync(name, info)
	}
	return ft.wait()
}
```



生成**web**类似于nginx里面的启动静态http服务器的时候使用：

```go
		http.Handle("/", http.FileServer(rice.MustFindBox("site").HTTPBox()))
		http.HandleFunc("/send", h.send)
		http.HandleFunc("/mkminer", h.mkminer)
		http.HandleFunc("/msgwait", h.msgwait)
		http.HandleFunc("/msgwaitaddr", h.msgwaitaddr)

		fmt.Printf("Open http://%s\n", cctx.String("front"))
		go func() {
			<-ctx.Done()
			os.Exit(0)
		}()

		return http.ListenAndServe(cctx.String("front"), nil)
```



## 参考文献
- [example](https://github.com/GeertJohan/go.rice/blob/master/example/example.go)

