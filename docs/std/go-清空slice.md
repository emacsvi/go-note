---
title: "go-清空slice"
slug: "go-slice-empty"
date: "2018-11-23 01:00:02"
categories:
    - go
tags:
    - go
---

如何更加安全的清空slice
<!--more-->

## 示例

```go
data := []int{1, 2, 3, 34, 5, 5, 6, 67, 77, 87}
fmt.Println(data)
s1 := data[:0:0]
s1 = append(s1, 123)
s2 := data[:0]
s2 = append(s2, 10)
fmt.Printf("%p\n", &data[0])
fmt.Printf("%p\n", &s2[0])
fmt.Printf("%p\n", &s1[0])
```

1. 上面是test的代码片段。
2. 初始化的slice就是data，接下来是两种清空策略，当然还有其它的方式。但是这里只讨论这两种情况。
3. 一般人的用法是data[:0],我们将这个slice展开，他的样子应该是—-data[0:0:cap(data)]]。显然data[:0] 依然指向原数组的指针，我们将这个slice的首地址打印出来。也就是上面的&s1[0] == 0xc820015cc0。这个地址没有比较事说明不了什么的。我们再打印出data的首地址，&data[0]］ == 0xc820015cc0, 两个地址指向同一个内存地址。也就说明了，他们的底层数组是一样的，也就是说这样的清空食是误的。
4. 那么data[:0:0]可以清空这个slice吗？我们也是先将这个展开，data［0:0:0］,很明显这个一个全新的slice。我们将地址打印出来。0xc8201b4b18。so，我们已经成功的清空了一个slice。

总结：由于slice的写法上很有隐蔽性，建议先将其展开，再来分析。希望这篇分享帮你踩实一个坑。


## 参考文献

- [清空数据](https://aliasliyu4.wordpress.com/2016/07/16/golang%E4%B8%AD%E5%A6%82%E4%BD%95%E5%8E%BB%E6%B8%85%E7%A9%BA%E4%B8%80%E4%B8%AAslice/)
