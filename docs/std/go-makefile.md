---
title: "go-makefile"
slug: "go-makefile"
date: "2018-11-23 01:00:26"
categories:
    - go
tags:
    - go
---

有时候还是喜欢用**Makefile**比较方便。这里先不论交叉编译的情况，因为交叉编译被我利用`docker container`代替了。只需要`docker attach go`进去之后**make**一下即可交叉编译。

```makefile
GONAME=$(shell basename "$(PWD)").bin

all: build

build:
	@echo "Building $(GONAME)..."
	@-go build -o $(GONAME) && ([ $$? -eq 0 ] && echo "success" && ./$(GONAME)) || echo "go build failed"

clean:
	@echo "Cleaning"
	@- rm -rf $(GONAME)

.PHONY: build clean
```


