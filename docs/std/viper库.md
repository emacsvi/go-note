---
title: viper库使用
slug: spf13-viper
date: "2019-10-13"
description: viper
categories:
- go
tags:
- go
---

viper库的使用。
<!--more-->

## 源码分析
当`import "github.com/spf13/viper"`包的时候会调用**init()**函数去**New**一个`v`的结构体实体。所以默认情况下一个进程里面只有一个配置的实例。
```go
var v *Viper

func init() {
	v = New()
}
```

## 多实例
您还可以创建多不同的viper实例以供您的应用程序使用.每实例都有自己独立的设置和配置值.每个实例可以从不同的配置文件，**K/V**存储系统等读取.viper包支持的所有函数也都有对应的viper实例方法.

```go
  x := viper.New()
  y := viper.New()

  x.SetDefault("ContentDir", "content")
  y.SetDefault("ContentDir", "foobar")

  // 或者从packr打包的文件里面读取
  box := packr.NewBox("./configs")
	configType := "yml"
	defaultConfig := box.Bytes("default.yml")
	v := viper.New()
	v.SetConfigType(configType)
	err = v.ReadConfig(bytes.NewReader(defaultConfig))
	if err != nil {
		return
	}

	configs := v.AllSettings()
	// 将default中的配置全部以默认配置写入
	for k, v := range configs {
		viper.SetDefault(k, v)
	}
	env := os.Getenv("GO_ENV")
	// 根据配置的env读取相应的配置信息
	if env != "" {
		envConfig := box.Bytes(env + ".yml")

		viper.SetConfigType(configType)
		err = viper.ReadConfig(bytes.NewReader(envConfig))
		if err != nil {
			return
		}
	}
	return
```
当使用多个viper实例时，用户需要自己管理每个实例.

## 从etcd读取
```go
//或者，您可以创建一个新的viper实例
var runtime_viper = viper.New()
 
runtime_viper.AddRemoteProvider("etcd", "http://127.0.0.1:4001", "/config/hugo.yml")
runtime_viper.SetConfigType("yaml")
 
// 第一次从远程配置中读取
err := runtime_viper.ReadRemoteConfig()
 
//解密配置
runtime_viper.Unmarshal(&runtime_conf)
 
// 打开一个goroutine来永远监听远程变化
go func(){
	for {
	    time.Sleep(time.Second * 5) // 每次请求后延迟
	    err := runtime_viper.WatchRemoteConfig()
	    if err != nil {
	        log.Errorf("unable to read remote config: %v", err)
	        continue
	    }
 
	    //将新配置解组到我们的运行时配置结构中。您还可以使用通道
        //实现信号以通知系统更改 
	    runtime_viper.Unmarshal(&runtime_conf)
	}
}()
```

## 参考文献
- [Go进阶03:怎么使用viper管理配置](https://mojotv.cn/2018/12/26/how-to-use-viper-configuration-in-golang)
- [viper](https://blog.biezhi.me/2018/10/load-config-with-viper.html)
- [viper使用](https://www.jishuwen.com/d/2vNk)
