---
title: go-map深度拷贝实现
slug: go-map-deepcopy
date: "2019-10-20"
description: go-map深度拷贝实现
categories:
- go
tags:
- go
---

go-map的深度拷贝实现原理。
<!--more-->

## 概念

最简单的方法是用json unmarshal，变成字符串，然后再用 json marshal生成新的map。这种方法对结构体也适用。

如果是map[string]interface{}和[]interface{}的组合，用代码递归也很简单：

```go
func DeepCopy(value interface{}) interface{} {
	if valueMap, ok := value.(map[string]interface{}); ok {
		newMap := make(map[string]interface{})
		for k, v := range valueMap {
			newMap[k] = DeepCopy(v)
		}

		return newMap
	} else if valueSlice, ok := value.([]interface{}); ok {
		newSlice := make([]interface{}, len(valueSlice))
		for k, v := range valueSlice {
			newSlice[k] = DeepCopy(v)
		}

		return newSlice
	}

	return value
}
```



## 参考文献
- [golang深度拷贝map](https://blog.csdn.net/xtxy/article/details/51837400)
- [Go 中 map 的 deep copy 和 compare](https://maiyang.me/post/2019-01-23-deep-copy-and-compare-in-golang/)

