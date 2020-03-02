---
title: go模板template
slug: go-template
date: "2019-12-03"
description: template使用
categories: 
- go 
tags: 
- go 
---


template使用。
<!--more-->

## 概念

**text/template**足够平时开发的时候使用了。只是有一些注意的地方。记录下来。



## 不转义的实现

在新建模板之后，对template的变量注册一个自定义的函数处理，并且在最终render的时候去使用该方向即可。

关键是如下两句：

```go
// 定义函数unescaped
func unescaped (x string) interface{} { return template.HTML(x) }
// 在模板对象t中注册unescaped
t = t.Funcs(template.FuncMap{"unescaped": unescaped})
```

这样，在模板中就可以使用unescaped函数了，如：

```go
{{printf "%s" .Body | unescaped}} //[]byte
{{.Body | unescaped}} //string
```

测试实例如下，新自定义函数：

```go
func unescaped (x string) interface{} { return template.HTML(x) }
```

具体代码片断：

```go
	minerRunStr := `[program:{{.ActorId}}]
{{with .User}}user={{.}}{{end}}
autorestart=unexpected
startretries=1
exitcodes=0,1
{{with .Env}}environment={{. | unescaped}}{{end}}
command={{.InitCmd}}
stdout_logfile={{.LogFile}}
stdout_logfile_maxbytes=100MB
stdout_logfile_backups=1
stdout_capture_maxbytes=0
stdout_events_enabled=true
redirect_stderr=true`
	
// t, err := template.New("miner").Parse(minerRunStr)
	t := template.New("miner") // 创建模板
	t = t.Funcs(template.FuncMap{"unescaped": unescaped}) // 将自定义的函数注册到模板里面
	t, err := t.Parse(minerRunStr) // 解析字符串

	f, err := os.Create(filepath.Join(g.Config().XjgwPath, fmt.Sprintf("%s%s.conf", MinerFile, a.Id)))
	defer f.Close()
  // 本来可以直接写文件，发现解析出来的内容有空行，所以将其利用字符串的方式过滤完空行再写入文件内容。
	buf := new(bytes.Buffer)
	err = t.Execute(buf, m)
	if err != nil {
		log.Println(err)
	}

	result := strings.Replace(buf.String(), "\n\n", "\n", -1) // 过滤空行
```

## 参考文献
- [不转义](http://blog.xiayf.cn/2013/11/01/unescape-html-in-golang-html_template/)
- [基础用法](https://cloud.tencent.com/developer/section/1145004)

