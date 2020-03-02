---
title: go常用的一些代码备忘
slug: my-go-codes
date: "2019-03-18"
description: 自己常用的一些代码汇总
categories:
- go
tags:
- go
---

go常用的一些代码备忘汇总。
<!--more-->

#### 字符串与数字相互转换
利用**strconv**进行转换:

- 主要利用**ParseXXX(str, ...)**将字符串转化成其他类型，
- 利用**FormatXXX**转其他类型转化为字符串

```go
func strConvNumber() {
	var (
		myInt int
		str string
		myInt64 int64
		err error
	)
	// string 转 int
	str = "999"
	// atoi
	// Array to Integer
	// 字符数组(字符串)转化为整数。

	// Atoi is equivalent to ParseInt(s, 10, 0), converted to type int.
	// func Atoi(s string) (int, error)
	if myInt, err = strconv.Atoi(str); err == nil {
		fmt.Println(myInt)
	}

	// string 转 int64
	// base 进制
	// bitSize 长度 是32，64 等
	// If base == 0, the base is implied by the string's prefix:
	// base 16 for "0x", base 8 for "0", and base 10 otherwise.
	// For bases 1, below 0 or above 36 an error is returned.
	if myInt64, err = strconv.ParseInt(str, 10, 64); err == nil {
		fmt.Println(myInt64)
	}

	myInt = 12306
	str = ""
	// itoa
	// Integer to Array
	// 整数转化为字符串。golang标准库与C++标准库均有
	// int 转 string
	// Itoa is equivalent to FormatInt(int64(i), 10).
	// func Itoa(i int) string
	str = strconv.Itoa(myInt)
	if str != "" {
		fmt.Println(str)
	}

	myInt64 = 12306000000
	str = ""
	// int64 转 string
	str = strconv.FormatInt(myInt64, 10)
	if str != "" {
		fmt.Println(str)
	}

	// format xxx 转化成字符串
	// FormatBool
	// func FormatBool(b bool) string

	// FormatFloat
	// func FormatFloat(f float64, fmt byte, prec, bitSize int) string

	// FormatInt
	// func FormatInt(i int64, base int) string

	// FormatUint
	// func FormatUint(i uint64, base int) string

	// 转换成bool类型.
	b, err := strconv.ParseBool("true")
	fmt.Println(b) // true

	// 转换成Float类型
	f, err := strconv.ParseFloat("3.1415", 64)
	fmt.Println(f) // 3.1415

	// 转换成int类型
	i, err := strconv.ParseInt("-42", 10, 64)
	fmt.Println(i)

	// 转成uint类型
	u, err := strconv.ParseUint("42", 10, 64)
	fmt.Println(u)
}
```

#### 时间相互转化
```go
// 时间日期处理
func ParseFormatTime() {
	// 时间戳
	fmt.Println(time.Now().Unix()) // 1552967613

	// 格式化当时时间
	// 这是个奇葩,必须是这个时间点, 据说是go诞生之日, 记忆方法:6-1-2-3-4-5
	fmt.Println(time.Now().Format("2006-01-02 15:04:05")) // 2019-03-19 11:54:23

	// 时间戳转str格式化时间
	str_time := time.Unix(1552967613, 0).Format("2006-01-02 15:04:05")
	fmt.Println(str_time) // 2019-03-19 11:53:33

	// 格式化的时间转unix时间
	// func Parse(layout, value string) (Time, error)
	the_time, err := time.Parse("2006-01-02 15:04:05", "2019-03-18 05:50:30")
	if err == nil {
		unix_time := the_time.Unix()
		fmt.Println(unix_time) // 1552888230
	}
}
```

#### 随机数
```go
func RandNumber() {
	// 产生种子
	rand.Seed(time.Now().Unix())
	fmt.Println(rand.Intn(100)) // 产生0-100的随机整数
	fmt.Println(rand.Float64()) // 产生0.0-1.0的随机浮点点

	// 聪明的办法
	// 如果要产生负数到正数的随机值，只需要将生成的随机数减去相应数值即可
	fmt.Println(rand.Intn(100) - 50) // [-50, 50)的随机值

	// 另外一种创建种子的方式
	// 创建一个以seed为种子的源，注意该源不是协程安全的
	// func NewSource(seed int64) Source
	// 以src为源创建随机对象
	// func New(src Source) *Rand
	// 设置或重置种子，注意该函数不是协程安全的
	// func (r *Rand) Seed(seed int64)
	var (
		mySource rand.Source
		myRand *rand.Rand
	)
	mySource = rand.NewSource(time.Now().Unix() + 100)
	myRand = rand.New(mySource)
	fmt.Println(myRand.Int63n(99999999)) // 70202487
  }
```

#### 随机字符串
这样实现比较高效，具体看这篇文章[随机字符串加强版](https://colobu.com/2018/09/02/generate-random-string-in-Go/)
```go
const letterBytes = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
const (
	letterIdxBits = 6                    // 6 bits to represent a letter index
	letterIdxMask = 1<<letterIdxBits - 1 // All 1-bits, as many as letterIdxBits
	letterIdxMax  = 63 / letterIdxBits   // # of letter indices fitting in 63 bits
)
func RandStringBytesMaskImpr(n int) string {
	b := make([]byte, n)
	// A rand.Int63() generates 63 random bits, enough for letterIdxMax letters!
	for i, cache, remain := n-1, rand.Int63(), letterIdxMax; i >= 0; {
		if remain == 0 {
			cache, remain = rand.Int63(), letterIdxMax
		}
		if idx := int(cache & letterIdxMask); idx < len(letterBytes) {
			b[i] = letterBytes[idx]
			i--
		}
		cache >>= letterIdxBits
		remain--
	}
	return string(b)
  }
```

#### 遍历目录下的文件
[请参考这篇文章](https://blog.csdn.net/sylar_d/article/details/51965408)
```go
// 遍历目录
func getListPath(root string) (result []string) {
	var (
		f func(string, os.FileInfo, error) error
		ok bool // 判断过滤用的
		name string // 打印用的
	)
	f = func (p string, f os.FileInfo, err error) error {
		// p == path
		// Walk函数在遍历文件时调用。调用时将参数传递给path，这是一个绝对路径，也就是Walk函数中的root作为前缀。
		// 将root + 文件名作为path传递给WalkFunc函数。
		if f == nil {
			return err
		}
		if f.IsDir() {
			// return filepath.SkipDir 如果要路过目录则可以使用 SkipDir这个变量进行
			return nil
		}

		result = append(result, p)

		// 用strings.HasSuffix(src, suffix) 判断src中是否包含suffix结尾
		ok = strings.HasSuffix(p, ".go")
		if ok {
			// 如果要过滤一些指定的文件 可以这样匹配
			fmt.Println("this is go file")
		}
		return nil
	}

	filepath.Walk(root, f)

	fmt.Println("打印结果：")
	for _, name = range result {
		fmt.Println(name)
	}

	return
}
```

#### 读文件

#### 写文件

#### 读配置


#### 控制协程

#### 控制并发

#### channel使用

#### context使用
