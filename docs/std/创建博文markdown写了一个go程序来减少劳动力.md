---
title: 创建博文markdown写了一个go程序来减少劳动力
slug: go-create-markdown-program
date: "2019-10-21"
description: 创建博文markdown写了一个go程序来减少劳动力
categories: 
- go 
tags: 
- go 
---


创建博文markdown写了一个go程序来减少劳动力。
<!--more-->

## 概念	

最终生成一个二进制文件craeteMd 然后通过它来创建我的md模板。

命令格式如下：

```shell
./createmd -n "创建博文markdown写了一个go程序来减少劳动力" -c "go" -tags "go" -url "go-create-markdown-program"
```



## 代码

```go
package main

import (
	"flag"
	"fmt"
	"html/template"
	"os"
	"strings"
	"time"
)

// 放入markdown之中的模板变量
type MarkdownInfo struct {
	FileName    string
	Title       string
	DateString  string
	UrlPath     string
	Description string
	Categories  []string
	Tags        []string
	More        string
	MoreHTML    template.HTML // 为了不让template对<!--more--> 这样的html进行转义，所以需要用到这种类型
}

var mdInfo MarkdownInfo

// 为了flag能解析数组格式自定义的Var->Value 类型
type sliceValue []string

func newSliceValue(vals []string, p *[]string) *sliceValue {
	*p = vals
	return (*sliceValue)(p)
}

func (s *sliceValue) Set(val string) error {
	*s = sliceValue(strings.Split(val, ","))
	return nil
}
func (s *sliceValue) Get() interface{} { return []string(*s) }

func (s *sliceValue) String() string { return strings.Join([]string(*s), ",") }

func init() {
	flag.StringVar(&mdInfo.FileName, "n", "", "文件名称")
	flag.StringVar(&mdInfo.Title, "t", "", "article title")
	flag.StringVar(&mdInfo.DateString, "d", "", "2019-10-10")
	flag.StringVar(&mdInfo.UrlPath, "url", "", "url path")
	flag.StringVar(&mdInfo.Description, "desc", "", "description")
	flag.StringVar(&mdInfo.More, "m", "", "more")
	flag.Var(newSliceValue([]string{}, &mdInfo.Categories), "c", "the categories like `-c \"go,etcd\"`")
	flag.Var(newSliceValue([]string{}, &mdInfo.Tags), "tags", "the tags like `-c \"go,etcd\"`")
}

// 创建一个md的模板
func main() {
	flag.Parse()
	if mdInfo.FileName == "" {
		fmt.Println("文件名称不能为空")
		return
	}
	if mdInfo.Title == "" {
		mdInfo.Title = mdInfo.FileName
	}

	if mdInfo.DateString == "" {
		mdInfo.DateString = time.Now().Format("2006-01-02")
	}

	if mdInfo.Description == "" {
		mdInfo.Description = mdInfo.FileName
	}

	// 为了不让template对html之中的<!--more-->进行转义，所以需要定义一个template.HTML类型的变量
	if mdInfo.More == "" {
		mdInfo.MoreHTML = template.HTML(mdInfo.FileName + "。\n<!--more-->")
	} else {
		mdInfo.MoreHTML = template.HTML(mdInfo.More + "。\n<!--more-->")
	}

	if len(mdInfo.Categories) == 0 {
		mdInfo.Categories = append(mdInfo.Categories, "go")
	}

	if len(mdInfo.Tags) == 0 {
		mdInfo.Tags = append(mdInfo.Tags, mdInfo.Categories...)
	}

	if mdInfo.UrlPath == "" {
		mdInfo.UrlPath = strings.Join(mdInfo.Tags, "-") + time.Now().Format("-2006-01-02-15-04-05")
	}

	t, err := template.ParseFiles("template.txt")
	if err != nil {
		panic(err)
	}
	f, err := os.Create(mdInfo.FileName + ".md")
	if err != nil {
		panic(err)
	}
	if err = t.Execute(f, mdInfo); err != nil {
		panic(err)
	}
	fmt.Println("创建文件名：" + mdInfo.FileName + ".md")
}
```



另外template模板的代码就很简单了：

```markdown
---
title: {{ .Title }}
slug: {{ .UrlPath }}
date: "{{ .DateString }}"
description: {{ .Description }}
categories: {{ range .Categories }}
- {{ . }} {{ end }}
tags: {{ range $i, $v := .Tags }}
- {{ $v }} {{ end }}
---


{{ .MoreHTML }}

## 概念

## 参考文献
- []()
- []()
```

